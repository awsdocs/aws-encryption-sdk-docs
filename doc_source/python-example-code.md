# AWS Encryption SDK for Python example code<a name="python-example-code"></a>

The following examples show you how to use the AWS Encryption SDK for Python to encrypt and decrypt data\.

**Topics**
+ [Strings](#python-example-strings)
+ [Byte streams](#python-example-streams)
+ [Byte streams with multiple master key providers](#python-example-multiple-providers)
+ [Using data key caching to encrypt messages](#python-example-caching)

## Encrypting and decrypting strings<a name="python-example-strings"></a>

The following example shows you how to use the AWS Encryption SDK to encrypt and decrypt strings\. This example uses a customer master key \(CMK\) in [AWS Key Management Service \(AWS KMS\)](https://aws.amazon.com/kms/) as the master key\.

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

from __future__ import print_function

import aws_encryption_sdk


def cycle_string(key_arn, source_plaintext, botocore_session=None):
    """Encrypts and then decrypts a string using a KMS customer master key (CMK)

    :param str key_arn: Amazon Resource Name (ARN) of the KMS CMK
    (http://docs.aws.amazon.com/kms/latest/developerguide/viewing-keys.html)
    :param bytes source_plaintext: Data to encrypt
    :param botocore_session: Existing Botocore session instance
    :type botocore_session: botocore.session.Session
    """

    # Create a KMS master key provider.
    kms_kwargs = dict(key_ids=[key_arn])
    if botocore_session is not None:
        kms_kwargs['botocore_session'] = botocore_session
    master_key_provider = aws_encryption_sdk.KMSMasterKeyProvider(**kms_kwargs)

    # Encrypt the plaintext source data.
    ciphertext, encryptor_header = aws_encryption_sdk.encrypt(
        source=source_plaintext,
        key_provider=master_key_provider
    )
    print('Ciphertext: ', ciphertext)

    # Decrypt the ciphertext.
    cycled_plaintext, decrypted_header = aws_encryption_sdk.decrypt(
        source=ciphertext,
        key_provider=master_key_provider
    )

    # Verify that the "cycled" (encrypted, then decrypted) plaintext is identical to the source
    # plaintext.
    assert cycled_plaintext == source_plaintext

    # Verify that the encryption context used in the decrypt operation includes all key pairs from
    # the encrypt operation. (The SDK can add pairs, so don't require an exact match.)
    #
    # In production, always use a meaningful encryption context. In this sample, we omit the
    # encryption context (no key pairs).
    assert all(
        pair in decrypted_header.encryption_context.items()
        for pair in encryptor_header.encryption_context.items()
    )

    print('Decrypted: ', cycled_plaintext)
```

## Encrypting and decrypting byte streams<a name="python-example-streams"></a>

The following example shows you how to use the AWS Encryption SDK to encrypt and decrypt byte streams\. This example doesn't use AWS\. It uses a static, ephemeral master key provider\.

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

import filecmp
import os

import aws_encryption_sdk
from aws_encryption_sdk.internal.crypto import WrappingKey
from aws_encryption_sdk.key_providers.raw import RawMasterKeyProvider
from aws_encryption_sdk.identifiers import WrappingAlgorithm, EncryptionKeyType


class StaticRandomMasterKeyProvider(RawMasterKeyProvider):
    """Randomly and consistently generates 256-bit keys for each unique key ID."""
    provider_id = 'static-random'

    def __init__(self, **kwargs):
        self._static_keys = {}

    def _get_raw_key(self, key_id):
        """Returns a static, randomly-generated symmetric key for the specified key ID.

        :param str key_id: Key ID
        :returns: Wrapping key that contains the specified static key
        :rtype: :class:`aws_encryption_sdk.internal.crypto.WrappingKey`
        """
        try:
            static_key = self._static_keys[key_id]
        except KeyError:
            static_key = os.urandom(32)
            self._static_keys[key_id] = static_key
        return WrappingKey(
            wrapping_algorithm=WrappingAlgorithm.AES_256_GCM_IV12_TAG16_NO_PADDING,
            wrapping_key=static_key,
            wrapping_key_type=EncryptionKeyType.SYMMETRIC
        )


def cycle_file(source_plaintext_filename):
    """Encrypts and then decrypts a file under a custom static master key provider.

    :param str source_plaintext_filename: Filename of file to encrypt
    """

    # Create a static random master key provider.
    key_id = os.urandom(8)
    master_key_provider = StaticRandomMasterKeyProvider()
    master_key_provider.add_master_key(key_id)

    ciphertext_filename = source_plaintext_filename + '.encrypted'
    cycled_plaintext_filename = source_plaintext_filename + '.decrypted'

    # Encrypt the plaintext source data.
    with open(source_plaintext_filename, 'rb') as plaintext, open(ciphertext_filename, 'wb') as ciphertext:
        with aws_encryption_sdk.stream(
            mode='e',
            source=plaintext,
            key_provider=master_key_provider
        ) as encryptor:
            for chunk in encryptor:
                ciphertext.write(chunk)

    # Decrypt the ciphertext.
    with open(ciphertext_filename, 'rb') as ciphertext, open(cycled_plaintext_filename, 'wb') as plaintext:
        with aws_encryption_sdk.stream(
            mode='d',
            source=ciphertext,
            key_provider=master_key_provider
        ) as decryptor:
            for chunk in decryptor:
                plaintext.write(chunk)

    # Verify that the "cycled" (encrypted, then decrypted) plaintext is identical to the source 
    # plaintext.
    assert filecmp.cmp(source_plaintext_filename, cycled_plaintext_filename)

    # Verify that the encryption context used in the decrypt operation includes all key pairs from
    # the encrypt operation.
        #
    # In production, always use a meaningful encryption context. In this sample, we omit the
    # encryption context (no key pairs).
    assert all(
        pair in decryptor.header.encryption_context.items()
        for pair in encryptor.header.encryption_context.items()
    )
    return ciphertext_filename, cycled_plaintext_filename
```

## Encrypting and decrypting byte streams with multiple master key providers<a name="python-example-multiple-providers"></a>

The following example shows you how to use the AWS Encryption SDK with more than one master key provider\. Using more than one master key provider creates redundancy if one master key provider is unavailable for decryption\. This example uses an AWS KMS customer master key \(CMK\) and an RSA key pair as the master keys\.

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

import filecmp
import os

import aws_encryption_sdk
from aws_encryption_sdk.internal.crypto import WrappingKey
from aws_encryption_sdk.key_providers.raw import RawMasterKeyProvider
from aws_encryption_sdk.identifiers import WrappingAlgorithm, EncryptionKeyType
from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives import serialization
from cryptography.hazmat.primitives.asymmetric import rsa


class StaticRandomMasterKeyProvider(RawMasterKeyProvider):
    provider_id = 'static-random'

    def __init__(self, **kwargs):
        self._static_keys = {}

    def _get_raw_key(self, key_id):
        """Returns a static, randomly generated, RSA key for the specified key ID.

        :param str key_id: User-defined ID for the static key
        :returns: Wrapping key that contains the specified static key
        :rtype: :class:`aws_encryption_sdk.internal.crypto.WrappingKey`
        """
        try:
            static_key = self._static_keys[key_id]
        except KeyError:
            private_key = rsa.generate_private_key(
                public_exponent=65537,
                key_size=4096,
                backend=default_backend()
            )
            static_key = private_key.private_bytes(
                encoding=serialization.Encoding.PEM,
                format=serialization.PrivateFormat.PKCS8,
                encryption_algorithm=serialization.NoEncryption()
            )
            self._static_keys[key_id] = static_key
        return WrappingKey(
            wrapping_algorithm=WrappingAlgorithm.RSA_OAEP_SHA1_MGF1,
            wrapping_key=static_key,
            wrapping_key_type=EncryptionKeyType.PRIVATE
        )


def cycle_file(key_arn, source_plaintext_filename, botocore_session=None):
    """Encrypts and then decrypts a file using a KMS master key provider and a custom static master
    key provider. Both master key providers are used to encrypt the plaintext file, so either one alone 
    can decrypt it.

    :param str key_arn: Amazon Resource Name (ARN) of the KMS Customer Master Key (CMK) (http://docs.aws.amazon.com/kms/latest/developerguide/viewing-keys.html)    
    :param str source_plaintext_filename: Filename of file to encrypt
    :param botocore_session: existing botocore session instance
    :type botocore_session: botocore.session.Session
    """

    # "Cycled" means encrypted and then decrypted 
    ciphertext_filename = source_plaintext_filename + '.encrypted'    
    cycled_kms_plaintext_filename = source_plaintext_filename + '.kms.decrypted'
    cycled_static_plaintext_filename = source_plaintext_filename + '.static.decrypted'

    # Create a KMS master key provider
    kms_kwargs = dict(key_ids=[key_arn])
    if botocore_session is not None:
        kms_kwargs['botocore_session'] = botocore_session
    kms_master_key_provider = aws_encryption_sdk.KMSMasterKeyProvider(**kms_kwargs)

    # Create a static master key provider and add a master key to it
    static_key_id = os.urandom(8)
    static_master_key_provider = StaticRandomMasterKeyProvider()
    static_master_key_provider.add_master_key(static_key_id)

    # Create a master key provider that includes the KMS and static master key providers
    kms_master_key_provider.add_master_key_provider(static_master_key_provider)

    # Encrypt plaintext with both KMS and static master keys
    with open(source_plaintext_filename, 'rb') as plaintext, open(ciphertext_filename, 'wb') as ciphertext:
        with aws_encryption_sdk.stream(
            source=plaintext,
            mode='e',
            key_provider=kms_master_key_provider
        ) as encryptor:
            for chunk in encryptor:
                ciphertext.write(chunk)

    # Decrypt the ciphertext with only the KMS master key
    with open(ciphertext_filename, 'rb') as ciphertext, open(cycled_kms_plaintext_filename, 'wb') as plaintext:
        with aws_encryption_sdk.stream(
            source=ciphertext,
            mode='d',
            key_provider=aws_encryption_sdk.KMSMasterKeyProvider(**kms_kwargs)
        ) as kms_decryptor:
            for chunk in kms_decryptor:
                plaintext.write(chunk)

    # Decrypt the ciphertext with only the static master key
    with open(ciphertext_filename, 'rb') as ciphertext, open(cycled_static_plaintext_filename, 'wb') as plaintext:
        with aws_encryption_sdk.stream(
            source=ciphertext,
            mode='d',
            key_provider=static_master_key_provider
        ) as static_decryptor:
            for chunk in static_decryptor:
                plaintext.write(chunk)

    # Verify that the "cycled" (encrypted, then decrypted) plaintext is identical to the source 
    # plaintext.
    assert filecmp.cmp(source_plaintext_filename, cycled_kms_plaintext_filename)
    assert filecmp.cmp(source_plaintext_filename, cycled_static_plaintext_filename)

    # Verify that the encryption context in the decrypt operation includes all key pairs from the
    # encrypt operation.
    #
    # In production, always use a meaningful encryption context. In this sample, we omit the
    # encryption context (no key pairs).
    assert all(
        pair in kms_decryptor.header.encryption_context.items()
        for pair in encryptor.header.encryption_context.items()
    )
    assert all(
        pair in static_decryptor.header.encryption_context.items()
        for pair in encryptor.header.encryption_context.items()
    )
    return ciphertext_filename, cycled_kms_plaintext_filename, cycled_static_plaintext_filename
```

## Using data key caching to encrypt messages<a name="python-example-caching"></a>

The following example shows how to use [data key caching](data-key-caching.md) in the AWS Encryption SDK for Python\. It is designed to show you how to configure an instance of the [local cache](data-caching-details.md#simplecache) \(LocalCryptoMaterialsCache\) with the required capacity value and an instance of the [caching cryptographic materials manager](data-caching-details.md#caching-cmm) \(caching CMM\) with [cache security thresholds](thresholds.md)\. 

This very basic example creates a function that encrypts a fixed string\. It lets you specify an AWS KMS customer master key, the required cache size \(capacity\), and a maximum age value\. For a more complex, real\-world example of data key caching, see [Data key caching example in Python](sample-cache-example-python.md)\.

Although it is optional, this example also uses an [encryption context](concepts.md#encryption-context) as additional authenticated data\. When you decrypt data that was encrypted with an encryption context, be sure that your application verifies that the encryption context is the one that you expect before returning the plaintext data to your caller\. An encryption context is a best practice element of any encryption or decryption operation, but it plays a special role in data key caching\. For details, see [Encryption Context: How to Select Cache Entries](data-caching-details.md#caching-encryption-context)\.

```
# Copyright 2017 Amazon.com, Inc. or its affiliates. All Rights Reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License"). You
# may not use this file except in compliance with the License. A copy of
# the License is located at
#
# http://aws.amazon.com/apache2.0/
#
# or in the "license" file accompanying this file. This file is
# distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF
# ANY KIND, either express or implied. See the License for the specific
# language governing permissions and limitations under the License.
"""Example of encryption with data key caching."""
import aws_encryption_sdk


def encrypt_with_caching(kms_cmk_arn, max_age_in_cache, cache_capacity):
    """Encrypts a string using an AWS KMS customer master key (CMK) and data key caching.

    :param str kms_cmk_arn: Amazon Resource Name (ARN) of the KMS customer master key
    :param float max_age_in_cache: Maximum time in seconds that a cached entry can be used
    :param int cache_capacity: Maximum number of entries to retain in cache at once
    """
    # Data to be encrypted
    my_data = "My plaintext data"

    # Security thresholds
    #   Max messages (or max bytes per) data key are optional
    MAX_ENTRY_MESSAGES = 100

    # Create an encryption context
    encryption_context = {"purpose": "test"}

    # Create a master key provider for the KMS customer master key (CMK)
    key_provider = aws_encryption_sdk.KMSMasterKeyProvider(key_ids=[kms_cmk_arn])

    # Create a local cache
    cache = aws_encryption_sdk.LocalCryptoMaterialsCache(cache_capacity)

    # Create a caching CMM
    caching_cmm = aws_encryption_sdk.CachingCryptoMaterialsManager(
        master_key_provider=key_provider,
        cache=cache,
        max_age=max_age_in_cache,
        max_messages_encrypted=MAX_ENTRY_MESSAGES,
    )

    # When the call to encrypt data specifies a caching CMM,
    # the encryption operation uses the data key cache specified
    # in the caching CMM
    encrypted_message, _header = aws_encryption_sdk.encrypt(
        source=my_data, materials_manager=caching_cmm, encryption_context=encryption_context
    )

    return encrypted_message
```