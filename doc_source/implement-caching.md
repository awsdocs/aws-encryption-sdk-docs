# How to Use Data Key Caching<a name="implement-caching"></a>

This topic shows you how to use data key caching in your application\. It takes you through the process step by step\. Then, it combines the steps in a simple example that uses data key caching in an operation to encrypt a string\.


|  | 
| --- |
|   The AWS Encryption SDK for C is a preview release\. The code and the documentation are subject to change\.  | 

**Topics**
+ [Using Data Key Caching: Step\-by\-Step](#implement-caching-steps)
+ [Data Key Caching Example: Encrypt a String](#caching-example-encrypt-string)

## Using Data Key Caching: Step\-by\-Step<a name="implement-caching-steps"></a>

These step\-by\-step instructions show you how to create the components that you need to implement data key caching\.
+ [Create a data key cache](data-caching-details.md#simplecache)\. In these examples, we use the local cache that the AWS Encryption SDK provides\. We limit the cache to 10 data keys\.

   

------
#### [ Java ]

  ```
  // Cache capacity (maximum number of entries) is required
  int MAX_CACHE_SIZE = 10; 
  
  CryptoMaterialsCache cache = new LocalCryptoMaterialsCache(MAX_CACHE_SIZE);
  ```

------
#### [ Python ]

  ```
  # Cache capacity (maximum number of entries) is required
  MAX_CACHE_SIZE = 10
  
  cache = LocalCryptoMaterialsCache(MAX_CACHE_SIZE)
  ```

------
#### [ C ]

  ```
  // Cache capacity (maximum number of entries) is required
  size_t capacity = 10; 
  struct aws_allocator *alloc = aws_default_allocator();
  
  struct aws_cryptosdk_materials_cache *cache = aws_cryptosdk_materials_cache_local_new(alloc, capacity);
  ```

------

   
+ Create a [master key provider](concepts.md#master-key-provider) \(Java and Python\) or a [keyring](concepts.md#keyring)\. These examples use an AWS Key Management Service \(AWS KMS\) master key provider or a compatible [KMS keyring](choose-keyring.md#use-kms-keyring)\.

   

------
#### [ Java ]

  ```
  // Create a KMS master key provider
  //   The input is the Amazon Resource Name (ARN) 
  //   of a KMS customer master key (CMK)
  
  MasterKeyProvider<KmsMasterKey> keyProvider = new KmsMasterKeyProvider(kmsCmkArn);
  ```

------
#### [ Python ]

  ```
  # Create a KMS master key provider
  #  The input is the Amazon Resource Name (ARN) 
  #  of a KMS customer master key (CMK)
  
  key_provider = aws_encryption_sdk.KMSMasterKeyProvider(key_ids=[kms_cmk_arn])
  ```

------
#### [ C ]

  ```
  // Create a KMS keyring
  //   The input is the Amazon Resource Name (ARN) 
  //   of a KMS customer master key (CMK)
  
  struct aws_cryptosdk_keyring *kms_keyring = Aws::Cryptosdk::KmsKeyring::Builder().Build(key_arn);
  ```

------

   
+ Create the underlying [cryptographic materials manager](concepts.md#crypt-materials-manager) \(CMM\)\. This step is required in C\. It is optional in Java and Python\. After you create the CMM, you can [release your reference](c-language-using.md#c-language-using-release) to its keyring\.

   

------
#### [ C ]

  ```
  // Create a CMM with the allocator and KMS keyring
  struct aws_cryptosdk_cmm *default_cmm = aws_cryptosdk_default_cmm_new(alloc, kms_keyring);
  
  // Release your reference to the KMS keyring.
  aws_cryptosdk_keyring_release(kms_keyring);
  ```

------
+ [Create a caching cryptographic materials manager](data-caching-details.md#caching-cmm) \(caching CMM\)\. 

   

  Associate your caching CMM with your cache and an underlying CMM \(all languages\) or master key provider \(Java and Python only\)\. Then, [set cache security thresholds](thresholds.md) on the caching CMM\. 

   

------
#### [ Java ]

  ```
  /*
   * Security thresholds
   *   Max entry age is required. 
   *   Max messages (and max bytes) per entry are optional
   */
  int MAX_ENTRY_AGE_SECONDS = 60;
  int MAX_ENTRY_MSGS = 10;
         
  //Create a caching CMM
  CryptoMaterialsManager cachingCmm =
      CachingCryptoMaterialsManager.newBuilder().withMasterKeyProvider(keyProvider)
                                   .withCache(cache)
                                   .withMaxAge(MAX_ENTRY_AGE_SECONDS, TimeUnit.SECONDS)
                                   .withMessageUseLimit(MAX_ENTRY_MSGS)
                                   .build();
  ```

------
#### [ Python ]

  ```
  # Security thresholds
  #   Max entry age is required. 
  #   Max messages (and max bytes) per entry are optional
  #
  MAX_ENTRY_AGE_SECONDS = 60.0
  MAX_ENTRY_MESSAGES = 10
         
  # Create a caching CMM
  caching_cmm = CachingCryptoMaterialsManager(
      master_key_provider=key_provider,
      cache=cache,
      max_age=MAX_ENTRY_AGE_SECONDS,
      max_messages_encrypted=MAX_ENTRY_MESSAGES
  )
  ```

------
#### [ C ]

  ```
  // Create the caching CMM
  //   Set the partition ID to NULL.
  //   Set the required maximum age value to 60 seconds.
  struct aws_cryptosdk_cmm *caching_cmm = aws_cryptosdk_caching_cmm_new(alloc, cache, default_cmm, NULL, 60, AWS_TIMESTAMP_SECS);
  
  // Add an optional message threshold
  //   The cached data key will not be used for more than 10 messages.
  aws_status = aws_cryptosdk_caching_cmm_set_limit_messages(caching_cmm, 10);
  
  // Release your references to the cache and underlying CMM.
  aws_cryptosdk_materials_cache_release(cache);
  aws_cryptosdk_cmm_release(default_cmm);
  ```

------

That's all you need to do\. Then, let the AWS Encryption SDK manage the cache for you, or add your own cache management logic\.

When you want to use data key caching in a call to encrypt or decrypt data, specify your caching CMM instead of a master key provider or other CMM\.

**Note**  
If you are encrypting data streams, or any data of unknown size, be sure to specify the data size in the request\. The AWS Encryption SDK does not use data key caching when encrypting data of unknown size\.

------
#### [ Java ]

```
// When the call to encryptData specifies a caching CMM,
// the encryption operation uses the data key cache
//
final AwsCrypto encryptionSdk = new AwsCrypto();
byte[] message = encryptionSdk.encryptData(cachingCmm, plaintext_source).getResult();
```

------
#### [ Python ]

```
# When the call to encrypt specifies a caching CMM,
# the encryption operation uses the data key cache
#
encrypted_message, header = aws_encryption_sdk.encrypt(
    source=plaintext_source,
    materials_manager=caching_cmm
)
```

------
#### [ C ]

In the AWS Encryption SDK for C, you create a session with the caching CMM and then process the session\. 

By default, when the message size is unknown and unbounded, the AWS Encryption SDK does not cache data keys\. To allow caching when you don't know the exact data size, use the `aws_cryptosdk_session_set_message_bound` method to set a maximum size for the message\. Set the bound larger than the estimated message size\. If the actual message size exceeds the bound, the encryption operation fails\.

```
// Create a session with the caching CMM. Set the session mode to encrypt.
struct aws_cryptosdk_session *session = aws_cryptosdk_session_new_from_cmm(alloc, AWS_CRYPTOSDK_ENCRYPT, caching_cmm);

// Set a message bound of 1000 bytes
aws_status = aws_cryptosdk_session_set_message_bound(session, 1000);

// Encrypt the message using the session with the caching CMM
aws_status = aws_cryptosdk_session_process(
             session, output_buffer, output_capacity, &output_produced, input_buffer, input_len, &input_consumed);
```

------

## Data Key Caching Example: Encrypt a String<a name="caching-example-encrypt-string"></a>

This simple code example uses data key caching when encrypting a string\. It combines the code from the [step\-by\-step procedure](#implement-caching-steps) into test code that you can run\.

The example creates a [local cache](data-caching-details.md#simplecache) and a [master key provider](concepts.md#master-key-provider) for an AWS KMS [customer master key](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#master_keys) \(CMK\)\. Then, it uses the cache and master key provider to create a caching CMM with appropriate [security thresholds](thresholds.md)\. The encryption request specifies the caching CMM, the plaintext data to encrypt, and an [encryption context](data-caching-details.md#caching-encryption-context)\. 

To run the example, you need to supply the [Amazon Resource Name \(ARN\) of a KMS CMK](https://docs.aws.amazon.com/kms/latest/developerguide/viewing-keys.html)\. Be sure that you have [permission to use the CMK](https://docs.aws.amazon.com/kms/latest/developerguide/key-policies.html#key-policy-default-allow-users) to generate a data key\.

For more detailed, real\-world examples of creating and using a data key cache, see [Data Key Caching Example in Java](sample-cache-example-java.md) and [Data Key Caching Example in Python](sample-cache-example-python.md)\. 

------
#### [ Java ]

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


import java.nio.charset.StandardCharsets;
import java.util.Collections;
import java.util.Map;
import java.util.concurrent.TimeUnit;

import javax.xml.bind.DatatypeConverter;
 
import com.amazonaws.encryptionsdk.AwsCrypto;
import com.amazonaws.encryptionsdk.CryptoMaterialsManager;
import com.amazonaws.encryptionsdk.MasterKeyProvider;
import com.amazonaws.encryptionsdk.caching.CachingCryptoMaterialsManager;
import com.amazonaws.encryptionsdk.caching.CryptoMaterialsCache;
import com.amazonaws.encryptionsdk.caching.LocalCryptoMaterialsCache;
import com.amazonaws.encryptionsdk.kms.KmsMasterKey;
import com.amazonaws.encryptionsdk.kms.KmsMasterKeyProvider;

/**
 * <p>
 * Encrypts a string using an AWS KMS customer master key (CMK) and data key caching
 * 
 * <p>
 * Arguments:
 * <ol>
 * <li>KMS CMK ARN: To find the Amazon Resource Name of your AWS KMS customer master key (CMK), 
 *     see 'Viewing Keys' at http://docs.aws.amazon.com/kms/latest/developerguide/viewing-keys.html
 * <li>Max entry age: Maximum time (in seconds) that a cached entry can be used
 * <li>Cache capacity: Maximum number of entries in the cache
 * </ol>
 */
public class SimpleDataKeyCachingExample {
    /*
     * Security thresholds
     *   Max entry age is required. 
     *   Max messages (and max bytes) per data key are optional
     */
    private static final int MAX_ENTRY_MSGS = 100;
    
    public static byte[] encryptWithCaching(String kmsCmkArn, int maxEntryAge, int cacheCapacity) {
        // Plaintext data to be encrypted
        byte[] myData = "My plaintext data".getBytes(StandardCharsets.UTF_8);

        // Encryption context
        final Map<String, String> encryptionContext = Collections.singletonMap("purpose", "test");

        // Create a master key provider
        MasterKeyProvider<KmsMasterKey> keyProvider = new KmsMasterKeyProvider(kmsCmkArn);

        // Create a cache
        CryptoMaterialsCache cache = new LocalCryptoMaterialsCache(cacheCapacity);

        // Create a caching CMM
        CryptoMaterialsManager cachingCmm =
            CachingCryptoMaterialsManager.newBuilder().withMasterKeyProvider(keyProvider)
                                         .withCache(cache)
                                         .withMaxAge(maxEntryAge, TimeUnit.SECONDS)
                                         .withMessageUseLimit(MAX_ENTRY_MSGS)
            .build();

        // When the call to encryptData specifies a caching CMM,
        // the encryption operation uses the data key cache
        //
        final AwsCrypto encryptionSdk = new AwsCrypto();
        return encryptionSdk.encryptData(cachingCmm, myData, encryptionContext).getResult();
}
```

------
#### [ Python ]

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

------