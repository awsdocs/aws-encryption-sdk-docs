# How to Implement Data Key Caching<a name="implement-caching"></a>

This topic shows you how to implement data key caching in your application\. It takes you through the process step by step\. Then, it combines the steps in a simple example that uses data key caching in an operation to encrypt a string\.

**Topics**
+ [Implement Data Key Caching: Step\-by\-Step](#implement-caching-steps)
+ [Data Key Caching Example: Encrypt a String](#caching-example-encrypt-string)

## Implement Data Key Caching: Step\-by\-Step<a name="implement-caching-steps"></a>

These step\-by\-step instructions show you how to create the components that you need to implement data key caching\.
+ [Create a data key cache](data-caching-details.md#simplecache), such as a LocalCryptoMaterialsCache\.

   

------
#### [ Java ]

  ```
  //Cache capacity (maximum number of entries) is required
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

   
+ Create a [master key provider](concepts.md#master-key-provider)\. This example uses an AWS Key Management Service \(AWS KMS\) master key provider\.

   

------
#### [ Java ]

  ```
  //Create a KMS master key provider
  //  The input is the Amazon Resource Name (ARN) 
  //  of a KMS customer master key (CMK)
  
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

   
+ [Create a caching cryptographic materials manager](data-caching-details.md#caching-cmm) \(caching CMM\)\. 

   

  Associate your caching CMM with your cache and master key provider\. Then, [set cache security thresholds](thresholds.md) on the caching CMM\. 

   

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

## Data Key Caching Example: Encrypt a String<a name="caching-example-encrypt-string"></a>

This simple code example uses data key caching when encrypting a string\. It combines the code from the [step\-by\-step procedure](#implement-caching-steps) into test code that you can run\.

The example creates a [LocalCryptoMaterialsCache](data-caching-details.md#simplecache) and a [master key provider](concepts.md#master-key-provider) for an AWS KMS [customer master key](http://docs.aws.amazon.com/kms/latest/developerguide//concepts.html#master_keys) \(CMK\)\. Then, it uses the cache and master key provider to create a caching CMM with appropriate [security thresholds](thresholds.md)\. The encryption request specifies the caching CMM, the plaintext data to encrypt, and an [encryption context](data-caching-details.md#caching-encryption-context)\. 

To run the example, you need to supply the [Amazon Resource Name \(ARN\) of a KMS CMK](http://docs.aws.amazon.com/kms/latest/developerguide/viewing-keys.html)\. Be sure that you have [permission to use the CMK](http://docs.aws.amazon.com/kms/latest/developerguide/key-policies.html#key-policy-default-allow-users) to generate a data key\.

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
"""Example of basic configuration and use of data key caching."""
import aws_encryption_sdk


def encrypt_with_caching(kms_cmk_arn, max_age_in_cache, cache_capacity):
    """Encrypts a string using an AWS KMS customer master key (CMK) and data key caching.

    :param str kms_cmk_arn: Amazon Resource Name (ARN) of the KMS customer master key
    :param float max_age_in_cache: Maximum time in seconds that a cached entry can be used
    :param int cache_capacity: Maximum number of entries in the cache
    """
    # Data to be encrypted
    my_data = 'My plaintext data'

    # Security thresholds
    #   Max messages (and max bytes) per data key are optional
    MAX_ENTRY_MESSAGES = 100

    # Create an encryption context.
    encryption_context = {'purpose': 'test'}

    # Create a master key provider for the KMS master key
    key_provider = aws_encryption_sdk.KMSMasterKeyProvider(key_ids=[kms_cmk_arn])

    # Create a cache
    cache = aws_encryption_sdk.LocalCryptoMaterialsCache(cache_capacity)

    # Create a caching CMM
    caching_cmm = aws_encryption_sdk.CachingCryptoMaterialsManager(
        master_key_provider=key_provider,
        cache=cache,
        max_age=max_age_in_cache,
        max_messages_encrypted=MAX_ENTRY_MESSAGES
    )

    # When the encrypt request specifies a caching CMM,
    # the encryption operation uses the data key cache
    encrypted_message, _header = aws_encryption_sdk.encrypt(
        source=my_data,
        materials_manager=caching_cmm,
        encryption_context=encryption_context
    )

    return encrypted_message
```

------