# Data key caching example in Java<a name="sample-cache-example-java"></a>

This code sample creates a basic implementation of data key caching with a [local cache](data-caching-details.md#simplecache) in Java\. For details about the Java implementation of the AWS Encryption SDK, see [AWS Encryption SDK for Java](java.md)\.

The code creates two instances of a local cache: one for data producers that are encrypting data and another for data consumers \(AWS Lambda functions\) that are decrypting data\. For implementation details, see the [Javadoc](https://aws.github.io/aws-encryption-sdk-java/javadoc/) for the AWS Encryption SDK\.

## Producer<a name="producer-java"></a>

The producer gets a map, converts it to JSON, uses the AWS Encryption SDK to encrypt it, and pushes the ciphertext record to a [Kinesis stream](https://aws.amazon.com/kinesis/streams/) in each AWS Region\. 

The code defines a [caching cryptographic materials manager](data-caching-details.md#caching-cmm) \(caching CMM\) and associates it with a [local cache](data-caching-details.md#simplecache) and an underlying [AWS KMS master key provider](concepts.md#master-key-provider)\. The caching CMM caches the data keys \(and [related cryptographic materials](data-caching-details.md#cache-entries)\) from the master key provider\. It also interacts with the cache on behalf of the SDK and enforces security thresholds that you set\. 

Because the call to the `encryptData` method specifies a caching CMM, instead of a regular [cryptographic materials manager \(CMM\)](concepts.md#crypt-materials-manager) or master key provider, the method will use data key caching\.

```
/*
 * Copyright 2017 Amazon.com, Inc. or its affiliates. All Rights Reserved.
 *
 * Licensed under the Apache License, Version 2.0 (the "License"). You may not use this file except
 * in compliance with the License. A copy of the License is located at
 *
 * http://aws.amazon.com/apache2.0
 *
 * or in the "license" file accompanying this file. This file is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the
 * specific language governing permissions and limitations under the License.
 */
package com.amazonaws.crypto.examples.kinesisdatakeycaching;

import java.nio.ByteBuffer;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.UUID;
import java.util.concurrent.TimeUnit;

import com.amazonaws.ClientConfiguration;
import com.amazonaws.auth.DefaultAWSCredentialsProviderChain;
import com.amazonaws.encryptionsdk.AwsCrypto;
import com.amazonaws.encryptionsdk.CryptoResult;
import com.amazonaws.encryptionsdk.MasterKeyProvider;
import com.amazonaws.encryptionsdk.caching.CachingCryptoMaterialsManager;
import com.amazonaws.encryptionsdk.caching.LocalCryptoMaterialsCache;
import com.amazonaws.encryptionsdk.kms.KmsMasterKey;
import com.amazonaws.encryptionsdk.kms.KmsMasterKeyProvider;
import com.amazonaws.encryptionsdk.multi.MultipleProviderFactory;
import com.amazonaws.regions.Region;
import com.amazonaws.services.kinesis.AmazonKinesis;
import com.amazonaws.services.kinesis.AmazonKinesisClientBuilder;
import com.amazonaws.util.json.Jackson;

/**
 * Pushes data to Kinesis Streams in multiple Regions.
 */
public class MultiRegionRecordPusher {
    private static long MAX_ENTRY_AGE_MILLISECONDS = 300000;
    private static long MAX_ENTRY_USES = 100;
    private static int MAX_CACHE_ENTRIES = 100;
    private final String streamName_;
    private ArrayList<AmazonKinesis> kinesisClients_;
    private CachingCryptoMaterialsManager cachingMaterialsManager_;
    private AwsCrypto crypto_;

    /**
     * Creates an instance of this object with Kinesis clients for all target Regions
     * and a cached key provider containing KMS master keys in all target Regions.
     */
    public MultiRegionRecordPusher(final Region[] regions, final String kmsAliasName, final String streamName){
        streamName_ = streamName;
        crypto_ = new AwsCrypto();
        kinesisClients_ = new ArrayList<AmazonKinesis>();

        DefaultAWSCredentialsProviderChain credentialsProvider = new DefaultAWSCredentialsProviderChain();
        ClientConfiguration clientConfig = new ClientConfiguration();

        // Build KmsMasterKey and AmazonKinesisClient objects for each target region
        List<KmsMasterKey> masterKeys = new ArrayList<KmsMasterKey>();
        for (Region region : regions) {
            kinesisClients_.add(AmazonKinesisClientBuilder.standard()
                    .withCredentials(credentialsProvider)
                    .withRegion(region.getName())
                    .build());

            KmsMasterKey regionMasterKey = new KmsMasterKeyProvider(
                credentialsProvider,
                region,
                clientConfig,
                kmsAliasName
            ).getMasterKey(kmsAliasName);

            masterKeys.add(regionMasterKey);
        }

        // Collect KmsMasterKey objects into single provider and add cache
        MasterKeyProvider<?> masterKeyProvider = MultipleProviderFactory.buildMultiProvider(
                KmsMasterKey.class,
                masterKeys
        );

        cachingMaterialsManager_ = CachingCryptoMaterialsManager.newBuilder()
            .withMasterKeyProvider(masterKeyProvider)
            .withCache(new LocalCryptoMaterialsCache(MAX_CACHE_ENTRIES))
            .withMaxAge(MAX_ENTRY_AGE_MILLISECONDS, TimeUnit.MILLISECONDS)
            .withMessageUseLimit(MAX_ENTRY_USES)
            .build();
    }

    /**
     * JSON serializes and encrypts the received record data and pushes it to all target streams.
     */
    public void putRecord(final Map<Object, Object> data){
        String partitionKey = UUID.randomUUID().toString();
        Map<String, String> encryptionContext = new HashMap<String, String>();
        encryptionContext.put("stream", streamName_);

        // JSON serialize data
        String jsonData = Jackson.toJsonString(data);

        // Encrypt data
        CryptoResult<byte[], ?> result = crypto_.encryptData(
            cachingMaterialsManager_,
            jsonData.getBytes(),
            encryptionContext
        );
        byte[] encryptedData = result.getResult();

        // Put records to Kinesis stream in all Regions
        for (AmazonKinesis regionalKinesisClient : kinesisClients_) {
            regionalKinesisClient.putRecord(
                streamName_,
                ByteBuffer.wrap(encryptedData),
                partitionKey
            );
        }
    }
}
```

## Consumer<a name="consumer-java"></a>

The data consumer is an [AWS Lambda](https://aws.amazon.com/lambda/) function that is triggered by [Kinesis](https://aws.amazon.com/kinesis/) events\. It decrypts and deserializes each record, and writes the plaintext record to an [Amazon DynamoDB](https://aws.amazon.com/dynamodb/) table in the same Region\.

Like the producer code, the consumer code enables data key caching by using a caching cryptographic materials manager \(caching CMM\) in calls to the `decryptData` method\. 

```
/*
 * Copyright 2017 Amazon.com, Inc. or its affiliates. All Rights Reserved.
 *
 * Licensed under the Apache License, Version 2.0 (the "License"). You may not use this file except
 * in compliance with the License. A copy of the License is located at
 *
 * http://aws.amazon.com/apache2.0
 *
 * or in the "license" file accompanying this file. This file is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the
 * specific language governing permissions and limitations under the License.
 */
package com.amazonaws.crypto.examples.kinesisdatakeycaching;

import java.io.UnsupportedEncodingException;
import java.nio.ByteBuffer;
import java.util.concurrent.TimeUnit;

import com.amazonaws.encryptionsdk.AwsCrypto;
import com.amazonaws.encryptionsdk.CryptoResult;
import com.amazonaws.encryptionsdk.caching.CachingCryptoMaterialsManager;
import com.amazonaws.encryptionsdk.caching.LocalCryptoMaterialsCache;
import com.amazonaws.encryptionsdk.kms.KmsMasterKey;
import com.amazonaws.encryptionsdk.kms.KmsMasterKeyProvider;
import com.amazonaws.services.dynamodbv2.AmazonDynamoDBClientBuilder;
import com.amazonaws.services.dynamodbv2.document.DynamoDB;
import com.amazonaws.services.dynamodbv2.document.Item;
import com.amazonaws.services.dynamodbv2.document.Table;
import com.amazonaws.services.lambda.runtime.Context;
import com.amazonaws.services.lambda.runtime.events.KinesisEvent;
import com.amazonaws.services.lambda.runtime.events.KinesisEvent.KinesisEventRecord;
import com.amazonaws.util.BinaryUtils;

/**
 * Decrypts all incoming Kinesis records and writes records to DynamoDB.
 */
public class LambdaDecryptAndWrite {
    private static final long MAX_ENTRY_AGE_MILLISECONDS = 600000;
    private static final int MAX_CACHE_ENTRIES = 100;
    private CachingCryptoMaterialsManager cachingMaterialsManager_;
    private AwsCrypto crypto_;
    private Table table_;

    /**
     * Because the cache is used only for decryption, the code doesn't set
     * the max bytes or max message security thresholds that are enforced
     * only on on data keys used for encryption.  
     */
    public LambdaDecryptAndWrite() {
    	String cmkArn = System.getenv("CMK_ARN");
        cachingMaterialsManager_ = CachingCryptoMaterialsManager.newBuilder()
            .withMasterKeyProvider(new KmsMasterKeyProvider(cmkArn))
            .withCache(new LocalCryptoMaterialsCache(MAX_CACHE_ENTRIES))
            .withMaxAge(MAX_ENTRY_AGE_MILLISECONDS, TimeUnit.MILLISECONDS)
            .build();

        crypto_ = new AwsCrypto();
        String tableName = System.getenv("TABLE_NAME");
        DynamoDB dynamodb = new DynamoDB(AmazonDynamoDBClientBuilder.defaultClient());
        table_ = dynamodb.getTable(tableName);
    }

    /**
     * 
     * @param event
     * @param context
     */
    public void handleRequest(KinesisEvent event, Context context) throws UnsupportedEncodingException{
        for (KinesisEventRecord record : event.getRecords()) {
            ByteBuffer ciphertextBuffer = record.getKinesis().getData();
            byte[] ciphertext = BinaryUtils.copyAllBytesFrom(ciphertextBuffer);

            // Decrypt and unpack record
            CryptoResult<byte[], ?> plaintextResult = crypto_.decryptData(cachingMaterialsManager_, ciphertext);

            // Verify the encryption context value
            String streamArn = record.getEventSourceARN();
            String streamName = streamArn.substring(streamArn.indexOf("/") + 1);
            if (!streamName.equals(plaintextResult.getEncryptionContext().get("stream"))) {
                throw new IllegalStateException("Wrong Encryption Context!");
            }

            // Write record to DynamoDB
            String jsonItem = new String(plaintextResult.getResult(), "UTF-8");
            System.out.println(jsonItem);
            table_.putItem(Item.fromJSON(jsonItem));
        }
    }
}
```