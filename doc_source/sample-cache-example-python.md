# Data key caching example in Python<a name="sample-cache-example-python"></a>

This code example creates a basic implementation of data key caching with a [local cache](data-caching-details.md#simplecache) in Python\. For details about the Python implementation of the AWS Encryption SDK, see [AWS Encryption SDK for Python](python.md)\.

The code creates two instances of a local cache; one for data producers that are encrypting data and another for data consumers \(Lambda functions\) that are decrypting data\. For implementation details, see the [Python documentation](https://aws-encryption-sdk-python.readthedocs.io/en/latest/) for the AWS Encryption SDK\.

For a very simple example that focuses on the basic elements of data key caching, see [Using data key caching to encrypt messages](python-example-code.md#python-example-caching)\.

## Producer<a name="producer-python"></a>

The producer gets a map, converts it to JSON, uses the AWS Encryption SDK to encrypt it, and pushes the ciphertext record to an [Kinesis stream](https://aws.amazon.com/kinesis/streams/) in each AWS Region\. 

The code defines a [caching cryptographic materials manager](data-caching-details.md#caching-cmm) \(caching CMM\) and associates it with a [local cache](data-caching-details.md#simplecache) and an underlying [AWS KMS master key provider](concepts.md#master-key-provider)\. The caching CMM caches the data keys \(and [related cryptographic materials](data-caching-details.md#cache-entries)\) from the master key provider\. It also interacts with the cache on behalf of the SDK and enforces security thresholds that you set\. 

Because the call to the `encrypt` method specifies a caching CMM, instead of a regular [cryptographic materials manager \(CMM\)](concepts.md#crypt-materials-manager) or master key provider, the method will use data key caching\.

```
"""
Copyright 2017 Amazon.com, Inc. or its affiliates. All Rights Reserved.

Licensed under the Apache License, Version 2.0 (the "License"). You may not use this file except
in compliance with the License. A copy of the License is located at

https://aws.amazon.com/apache-2-0/

or in the "license" file accompanying this file. This file is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the
specific language governing permissions and limitations under the License.
"""
import json
import uuid

from aws_encryption_sdk import encrypt, KMSMasterKeyProvider, CachingCryptoMaterialsManager, LocalCryptoMaterialsCache
from aws_encryption_sdk.key_providers.kms import KMSMasterKey
import boto3


class MultiRegionRecordPusher(object):
    """Pushes data to Kinesis Streams in multiple Regions."""
    CACHE_CAPACITY = 100
    MAX_ENTRY_AGE_SECONDS = 300.0
    MAX_ENTRY_MESSAGES_ENCRYPTED = 100

    def __init__(self, regions, kms_alias_name, stream_name):
        self._kinesis_clients = []
        self._stream_name = stream_name

        # Set up KMSMasterKeyProvider with cache
        _key_provider = KMSMasterKeyProvider()

        # Add MasterKey and Kinesis client for each Region
        for region in regions:
            self._kinesis_clients.append(boto3.client('kinesis', region_name=region))
            regional_master_key = KMSMasterKey(
                client=boto3.client('kms', region_name=region),
                key_id=kms_alias_name
            )
            _key_provider.add_master_key_provider(regional_master_key)

        cache = LocalCryptoMaterialsCache(capacity=self.CACHE_CAPACITY)
        self._materials_manager = CachingCryptoMaterialsManager(
            master_key_provider=_key_provider,
            cache=cache,
            max_age=self.MAX_ENTRY_AGE_SECONDS,
            max_messages_encrypted=self.MAX_ENTRY_MESSAGES_ENCRYPTED
        )

    def put_record(self, record_data):
        """JSON serializes and encrypts the received record data and pushes it to all target streams.

        :param dict record_data: Data to write to stream
        """
        # Kinesis partition key to randomize write load across stream shards
        partition_key = uuid.uuid4().hex

        encryption_context = {'stream': self._stream_name}

        # JSON serialize data
        json_data = json.dumps(record_data)

        # Encrypt data
        encrypted_data, _header = encrypt(
            source=json_data,
            materials_manager=self._materials_manager,
            encryption_context=encryption_context
        )

        # Put records to Kinesis stream in all Regions
        for client in self._kinesis_clients:
            client.put_record(
                StreamName=self._stream_name,
                Data=encrypted_data,
                PartitionKey=partition_key
            )
```

## Consumer<a name="consumer-python"></a>

The data consumer is an [AWS Lambda](https://aws.amazon.com/lambda/) function that is triggered by [Kinesis](https://aws.amazon.com/kinesis/) events\. It decrypts and deserializes each record, and writes the plaintext record to a [DynamoDB](https://aws.amazon.com/dynamodb/) table in the same Region\.

Like the producer code, the consumer code enables data key caching by using a caching cryptographic materials manager \(caching CMM\) in calls to the `decrypt` method\. 

```
"""
Copyright 2017 Amazon.com, Inc. or its affiliates. All Rights Reserved.

Licensed under the Apache License, Version 2.0 (the "License"). You may not use this file except
in compliance with the License. A copy of the License is located at

https://aws.amazon.com/apache-2-0/

or in the "license" file accompanying this file. This file is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the
specific language governing permissions and limitations under the License.
"""
import base64
import json
import logging
import os

from aws_encryption_sdk import decrypt, KMSMasterKeyProvider, CachingCryptoMaterialsManager, LocalCryptoMaterialsCache
import boto3

_LOGGER = logging.getLogger(__name__)
_is_setup = False
CACHE_CAPACITY = 100
MAX_ENTRY_AGE_SECONDS = 600.0

def setup():
    """Sets up clients that should persist across Lambda invocations."""
    global materials_manager
    key_provider = KMSMasterKeyProvider()
    cache = LocalCryptoMaterialsCache(capacity=CACHE_CAPACITY)
           
    #  Because the cache is used only for decryption, the code doesn't set
    #   the max bytes or max message security thresholds that are enforced
    #   only on on data keys used for encryption.
    materials_manager = CachingCryptoMaterialsManager(
        master_key_provider=key_provider,
        cache=cache,
        max_age=MAX_ENTRY_AGE_SECONDS
    )
    global table
    table_name = os.environ.get('TABLE_NAME')
    table = boto3.resource('dynamodb').Table(table_name)
    global _is_setup
    _is_setup = True


def lambda_handler(event, context):
    """Decrypts all incoming Kinesis records and writes records to DynamoDB."""
    _LOGGER.debug('New event:')
    _LOGGER.debug(event)
    if not _is_setup:
        setup()
    with table.batch_writer() as batch:
        for record in event.get('Records', []):
            # Record data base64-encoded by Kinesis
            ciphertext = base64.b64decode(record['kinesis']['data'])

            # Decrypt and unpack record
            plaintext, header = decrypt(
                source=ciphertext,
                materials_manager=materials_manager
            )
            item = json.loads(plaintext)

            # Verify the encryption context value
            stream_name = record['eventSourceARN'].split('/', 1)[1]
            if stream_name != header.encryption_context['stream']:
                raise ValueError('Wrong Encryption Context!')

            # Write record to DynamoDB
            batch.put_item(Item=item)
```