# How to Use Data Key Caching<a name="implement-caching"></a>

This topic shows you how to use data key caching in your application\. It takes you through the process step by step\. Then, it combines the steps in a simple example that uses data key caching in an operation to encrypt a string\.

**Note**  
The AWS Encryption SDK for JavaScript is a beta release\. The code and the documentation are subject to change\.

For complete and tested examples of using data key caching in the AWS Encryption SDK, see:
+ C/C\+\+: [caching\_cmm\.cpp](https://github.com/aws/aws-encryption-sdk-c/blob/master/examples/caching_cmm.cpp)
+ JavaScript Browser: [caching\_materials\_manager\.ts](https://github.com/awslabs/aws-encryption-sdk-javascript/blob/master/modules/example-browser/src/caching_materials_manager.ts)
+ JavaScript Node\.js [caching\_materials\_manager\.ts](https://github.com/awslabs/aws-encryption-sdk-javascript/blob/master/modules/example-node/src/caching_materials_manager.ts)
+ Python: [data\_key\_caching\_basic\.py](https://github.com/aws/aws-encryption-sdk-python/blob/master/examples/src/data_key_caching_basic.py)

**Topics**
+ [Using Data Key Caching: Step\-by\-Step](#implement-caching-steps)
+ [Data Key Caching Example: Encrypt a String](#caching-example-encrypt-string)

## Using Data Key Caching: Step\-by\-Step<a name="implement-caching-steps"></a>

These step\-by\-step instructions show you how to create the components that you need to implement data key caching\.
+ [Create a data key cache](data-caching-details.md#simplecache)\. In these examples, we use the local cache that the AWS Encryption SDK provides\. We limit the cache to 10 data keys\.

   

------
#### [ C ]

  ```
  // Cache capacity (maximum number of entries) is required
  size_t cache_capacity = 10; 
  struct aws_allocator *allocator = aws_default_allocator();
  
  struct aws_cryptosdk_materials_cache *cache = aws_cryptosdk_materials_cache_local_new(allocator, cache_capacity);
  ```

------
#### [ Java ]

  ```
  // Cache capacity (maximum number of entries) is required
  int MAX_CACHE_SIZE = 10; 
  
  CryptoMaterialsCache cache = new LocalCryptoMaterialsCache(MAX_CACHE_SIZE);
  ```

------
#### [ JavaScript Browser ]

  ```
  const capacity = 10
  
  const cache = getLocalCryptographicMaterialsCache(capacity)
  ```

------
#### [ JavaScript Node\.js ]

  ```
  const capacity = 10
  
  const cache = getLocalCryptographicMaterialsCache(capacity)
  ```

------
#### [ Python ]

  ```
  # Cache capacity (maximum number of entries) is required
  MAX_CACHE_SIZE = 10
  
  cache = LocalCryptoMaterialsCache(MAX_CACHE_SIZE)
  ```

------

   
+ Create a [master key provider](concepts.md#master-key-provider) \(Java and Python\) or a [keyring](concepts.md#keyring) \(C and JavaScript\)\. These examples use an AWS Key Management Service \(AWS KMS\) master key provider or a compatible [KMS keyring](choose-keyring.md#use-kms-keyring)\.

   

------
#### [ C ]

  ```
  // Create a KMS keyring
  //   The input is the Amazon Resource Name (ARN) 
  //   of a KMS customer master key (CMK)
  
  struct aws_cryptosdk_keyring *kms_keyring = Aws::Cryptosdk::KmsKeyring::Builder().Build(kms_cmk_arn);
  ```

------
#### [ Java ]

  ```
  // Create a KMS master key provider
  //   The input is the Amazon Resource Name (ARN) 
  //   of a KMS customer master key (CMK)
  
  MasterKeyProvider<KmsMasterKey> keyProvider = new KmsMasterKeyProvider(kmsCmkArn);
  ```

------
#### [ JavaScript Browser ]

  In the browser, you must inject your credentials securely\. This example defines credentials in a webpack \(kms\.webpack\.config\) that resolves credentials at runtime\. It creates an AWS KMS client provider instance from a AWS KMS client and the credentials\. Then, when it creates the keyring, it passes the client provider to the constructor along with the AWS KMS customer master key \(`generatorKeyId)`\.

  ```
  const { accessKeyId, secretAccessKey, sessionToken } = credentials
  
  const clientProvider = getClient(KMS, {
      credentials: {
        accessKeyId,
        secretAccessKey,
        sessionToken
      }
    })
  
  /*  Create a KMS keyring
   *  The input is the Amazon Resource Name (ARN) 
   */ of a KMS customer master key (CMK)
  
  const keyring = new KmsKeyringBrowser({ clientProvider, generatorKeyId })
  ```

------
#### [ JavaScript Node\.js ]

  ```
  /* Create a KMS keyring
   *   The input is the Amazon Resource Name (ARN) 
  */   of a KMS customer master key (CMK)
  
  const keyring = new KmsKeyringNode({ generatorKeyId })
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

   

  Associate your caching CMM with your cache and your master key provider or keyring\. Then, [set cache security thresholds](thresholds.md) on the caching CMM\. 

   

------
#### [ C ]

  In the AWS Encryption SDK for C, you can create a caching CMM from an underlying CMM, such as the default CMM, or from a keyring\. This example creates the caching CMM from a keyring\.

  After you create the caching CMM, you can release your references to the keyring and the cache\. For details, see [Reference Counting](c-language-using.md#c-language-using-release)\.

  ```
  // Create the caching CMM
  //   Set the partition ID to NULL.
  //   Set the required maximum age value to 60 seconds.
  struct aws_cryptosdk_cmm *caching_cmm = aws_cryptosdk_caching_cmm_new_from_keyring(allocator, cache, kms_keyring, NULL, 60, AWS_TIMESTAMP_SECS);
  
  // Add an optional message threshold
  //   The cached data key will not be used for more than 10 messages.
  aws_status = aws_cryptosdk_caching_cmm_set_limit_messages(caching_cmm, 10);
  
  // Release your references to the cache and the keyring.
  aws_cryptosdk_materials_cache_release(cache);
  aws_cryptosdk_keyring_release(kms_keyring);
  ```

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
#### [ JavaScript Browser ]

  ```
  /*
   * Security thresholds
   *   Max age (in milliseconds) is required.
   *   Max messages (and max bytes) per entry are optional.
   */
  const maxAge = 1000 * 60
  const maxMessagesEncrypted = 10
  
  /* Create a caching CMM from a keyring  */
  const cachingCmm = new WebCryptoCachingMaterialsManager({
    backingMaterials: keyring,
    cache,
    maxAge,
    maxMessagesEncrypted
  })
  ```

------
#### [ JavaScript Node\.js ]

  ```
  /*
   * Security thresholds
   *   Max age (in milliseconds) is required.
   *   Max messages (and max bytes) per entry are optional.
   */
  const maxAge = 1000 * 60
  const maxMessagesEncrypted = 10
  
  /* Create a caching CMM from a keyring  */
  const cachingCmm = new NodeCachingMaterialsManager({
    backingMaterials: keyring,
    cache,
    maxAge,
    maxMessagesEncrypted
  })
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
#### [ C ]

In the AWS Encryption SDK for C, you create a session with the caching CMM and then process the session\. 

By default, when the message size is unknown and unbounded, the AWS Encryption SDK does not cache data keys\. To allow caching when you don't know the exact data size, use the `aws_cryptosdk_session_set_message_bound` method to set a maximum size for the message\. Set the bound larger than the estimated message size\. If the actual message size exceeds the bound, the encryption operation fails\.

```
// Create a session with the caching CMM. Set the session mode to encrypt.
struct aws_cryptosdk_session *session = aws_cryptosdk_session_new_from_cmm(allocator, AWS_CRYPTOSDK_ENCRYPT, caching_cmm);

// Set a message bound of 1000 bytes
aws_status = aws_cryptosdk_session_set_message_bound(session, 1000);

// Encrypt the message using the session with the caching CMM
aws_status = aws_cryptosdk_session_process(
             session, output_buffer, output_capacity, &output_produced, input_buffer, input_len, &input_consumed);

// Release your references to the caching CMM and the session.
aws_cryptosdk_cmm_release(caching_cmm);
aws_cryptosdk_session_destroy(session);
```

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
#### [ JavaScript Browser ]

```
const { result } = await encrypt(cachingCmm, plaintext)
```

------
#### [ JavaScript Node\.js ]

When you use the caching CMM in the AWS Encryption SDK for JavaScript for Node\.js, the `encrypt` method requires the length of the plaintext\. If you don't provide it, the data key is not cached\. If you provide a length, but the plaintext data that you supply exceeds that length, the encrypt operation fails\. If you don't know the exact length of the plaintext, such as when you're streaming data, provide the largest expected value\.

```
const { result } = await encrypt(cachingCmm, plaintext, { plaintextLength: plaintext.length })
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

The example creates a [local cache](data-caching-details.md#simplecache) and a [master key provider](concepts.md#master-key-provider) or [keyring](concepts.md#keyring) for an AWS KMS [customer master key](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#master_keys) \(CMK\)\. Then, it uses the local cache and master key provider or keyring to create a caching CMM with appropriate [security thresholds](thresholds.md)\. In Java and Python, the encryption request specifies the caching CMM, the plaintext data to encrypt, and an [encryption context](data-caching-details.md#caching-encryption-context)\. In C, the caching CMM is specified in the session, and the session is provided to the encryption request\.

To run these examples, you need to supply the [Amazon Resource Name \(ARN\) of a KMS CMK](https://docs.aws.amazon.com/kms/latest/developerguide/viewing-keys.html)\. Be sure that you have [permission to use the CMK](https://docs.aws.amazon.com/kms/latest/developerguide/key-policies.html#key-policy-default-allow-users) to generate a data key\.

For more detailed, real\-world examples of creating and using a data key cache, see [Data Key Caching Example in Java](sample-cache-example-java.md) for Java, [Data Key Caching Example in Python](sample-cache-example-python.md) for Python, and [caching\_cmm\.cpp](https://github.com/aws/aws-encryption-sdk-c/blob/master/examples/caching_cmm.cpp) for C/C\+\+\.

------
#### [ C ]

```
/*
 * Copyright 2019 Amazon.com, Inc. or its affiliates. All Rights Reserved.
 *
 * Licensed under the Apache License, Version 2.0 (the "License"). You may not use
 * this file except in compliance with the License. A copy of the License is
 * located at
 *
 *     http://aws.amazon.com/apache2.0/
 *
 * or in the "license" file accompanying this file. This file is distributed on an
 * "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or
 * implied. See the License for the specific language governing permissions and
 * limitations under the License.
 */

#include <aws/cryptosdk/cache.h>
#include <aws/cryptosdk/cpp/kms_keyring.h>
#include <aws/cryptosdk/session.h>

void encrypt_with_caching(
    uint8_t *ciphertext,     // output will go here (assumes ciphertext_capacity bytes already allocated)
    size_t *ciphertext_len,  // length of output will go here
    size_t ciphertext_capacity,
    const char *kms_cmk_arn,
    int max_entry_age,
    int cache_capacity) {
    const uint64_t MAX_ENTRY_MSGS = 100;

    struct aws_allocator *allocator = aws_default_allocator();

    // Create a keyring
    struct aws_cryptosdk_keyring *kms_keyring = Aws::Cryptosdk::KmsKeyring::Builder().Build(kms_cmk_arn);

    // Create a cache
    struct aws_cryptosdk_materials_cache *cache = aws_cryptosdk_materials_cache_local_new(allocator, cache_capacity);

    // Create a caching CMM
    struct aws_cryptosdk_cmm *caching_cmm = aws_cryptosdk_caching_cmm_new_from_keyring(
        allocator, cache, kms_keyring, NULL, max_entry_age, AWS_TIMESTAMP_SECS);
    if (!caching_cmm) abort();

    if (aws_cryptosdk_caching_cmm_set_limit_messages(caching_cmm, MAX_ENTRY_MSGS)) abort();

    // Create a session
    struct aws_cryptosdk_session *session =
        aws_cryptosdk_session_new_from_cmm(allocator, AWS_CRYPTOSDK_ENCRYPT, caching_cmm);
    if (!session) abort();

    // Encryption context
    struct aws_hash_table *enc_ctx = aws_cryptosdk_session_get_enc_ctx_ptr_mut(session);
    if (!enc_ctx) abort();
    AWS_STATIC_STRING_FROM_LITERAL(enc_ctx_key, "purpose");
    AWS_STATIC_STRING_FROM_LITERAL(enc_ctx_value, "test");
    if (aws_hash_table_put(enc_ctx, enc_ctx_key, (void *)enc_ctx_value, NULL)) abort();

    // Plaintext data to be encrypted
    const char *my_data = "My plaintext data";
    size_t my_data_len  = strlen(my_data);
    if (aws_cryptosdk_session_set_message_size(session, my_data_len)) abort();

    // When the session uses a caching CMM, the encryption operation uses the data key cache
    // specified in the caching CMM.
    size_t bytes_read;
    if (aws_cryptosdk_session_process(
            session,
            ciphertext,
            ciphertext_capacity,
            ciphertext_len,
            (const uint8_t *)my_data,
            my_data_len,
            &bytes_read))
        abort();
    if (!aws_cryptosdk_session_is_done(session) || bytes_read != my_data_len) abort();

    aws_cryptosdk_session_destroy(session);
    aws_cryptosdk_cmm_release(caching_cmm);
    aws_cryptosdk_materials_cache_release(cache);
    aws_cryptosdk_keyring_release(kms_keyring);
}
```

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
#### [ JavaScript Browser ]

```
/*
* Copyright 2019 Amazon.com, Inc. or its affiliates. All Rights Reserved.
*
* Licensed under the Apache License, Version 2.0 (the "License"). You may not use
* this file except in compliance with the License. A copy of the License is
* located at
*
*     http://aws.amazon.com/apache2.0/
*
* or in the "license" file accompanying this file. This file is distributed on an
* "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or
* implied. See the License for the specific language governing permissions and
* limitations under the License.
*/

/* This example shows how to use a data key cache when encrypting data 
*  with JavaScript in the browser. It uses a KMS keyring, but you can use
*  any valid keyring.
*/

import {
KmsKeyringBrowser,
KMS,
getClient,
encrypt,
decrypt,
WebCryptoCachingMaterialsManager,
getLocalCryptographicMaterialsCache
} from '@aws-crypto/client-browser'
import { toBase64 } from '@aws-sdk/util-base64-browser'

/* Begin by providing your credentials to the browser in a secure manner. The AWS Encryption
*  SDK for JavaScript examples use the webpack.DefinePlugin, which replaces the credential
*  constants with your actual credentials.
*/
declare const credentials: {accessKeyId: string, secretAccessKey:string, sessionToken:string }

/* Create a client provider that will inject the correct credentials.
* The credentials here are injected by webpack from your environment bundle.
* The credential values are pulled using @aws-sdk/credential-provider-node.
* See kms.webpack.config
*/
const { accessKeyId, secretAccessKey, sessionToken } = credentials

/* getClient takes a KMS client constructor and optional configuration values.
*/
const clientProvider = getClient(KMS, {
  credentials: {
  accessKeyId,
  secretAccessKey,
  sessionToken
  }
})

/* This example uses a KMS keyring. First, it defines an AWS KMS customer master key (CMK)
*  as the generator key. To use this CMK as a generator in an encrypt function, you
*  need kms:GenerateDataKey permission on the CMK.
*
*  Before running this code, replace the example CMK ID with a valid one from your
*  AWS account.
*/
const generatorKeyId = 'arn:aws:kms:us-west-2:111122223333:alias/EncryptDecrypt'

/* This example also specifies an additional key, which is optional. You must have
*  kms:Encrypt permission on these CMKs. 

* The data key that encrypts your data is encrypted by the generator key and by each 
* of the additional keys. The encrypted message that the encrypt function returns 
* contains all of the encrypted data keys. To decrypt the message, you must decrypt
* any one of the encrypted data keys.
*
*  Before running this code, replace the example CMK ID with a valid one from your
*  AWS account.
*/
const keyIds = ['arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab']


/* Create the KMS keyring. Pass in the client provider, the required generator key,
*  and any (optional) additional keys.
*/
const keyring = new KmsKeyringBrowser({ clientProvider, generatorKeyId, keyIds })


/* Create a cache to hold the data keys and related cryptographic material.
*  In this example, we use the local cache provided by the Encryption SDK.
*
* `capacity` represents the maximum number of data keys in the cache.
*  When the cache is full, the oldest entry is evicted to make room for a
*  newer one. This value is required. 
*/
const capacity = 10

* There is also a second optional parameter, "proactiveFrequency". 
* By default, every 60 seconds (60,000 milliseconds) all data keys in the
* cache are checked to verify that they conform to all data key caching 
* thresholds. Data keys that exceed any threshold are evicted.
*
* To change how often this check runs, pass in a second value for the  
* "proactiveFrequency" parameter. Its value is in milliseconds.
* This example changes the frequency to 30000 (milleseconds).
*/
const proactiveFrequency = 30000

const cache = getLocalCryptographicMaterialsCache(capacity, proactiveFrequency)

/* Set the security thresholds on the caching CMM:
*  maxAge, maxBytesEncrypted, maxMessagesEncrypted.
*  Only maxAge is required.
*/

/* maxAge is the time in milliseconds that an entry is cached.
*  The cache actively removes entries that have exceeded the thresholds.
*/
const maxAge = 1000 * 60

/* The maximum amount of bytes encrypted under a single data key.
* This value is optional, but you should configure the lowest practical value.
*/
const maxBytesEncrypted = 100

/* The maximum number of messages encrypted under a single data key.
* This value is optional, but you should configure the lowest practical value.
*/
const maxMessagesEncrypted = 100

/* Use the keyring, local cache, and security thresholds to create
* a caching CMM.
*/  
const cachingCmm = new WebCryptoCachingMaterialsManager({
  backingMaterials: keyring,
  cache,
  maxAge,
  maxBytesEncrypted,
  maxMessagesEncrypted
})

/* The encryption context is non-secret data that is cryptographically bound 
* to the ciphertext when it is encrypted. To decrypt the data, the Encryption SDK
* uses the same encryption context. (It gets the encryption context from the header
* of the encrypted message; you don't have to supply it.)
*
* In addition, cached data keys are reused only when their encryption contexts match. 
* Therefore, you can use the encryption context to create subgroups of data keys in 
* your cache.
*  
* See: https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/data-caching-details.html#caching-encryption-context   
*/
const encryptionContext = {
  stage: 'demo',
  purpose: 'simple demonstration app',
  origin: 'us-west-2'
}

/* Define the data to encrypt. */
const plainText = new Uint8Array([1, 2, 3, 4, 5])

/* Encrypt the data.
*
* When the call to the encrypt function specifies a caching CMM,
* the encryption operation uses the data key cache.
*
*/
const { result } = await encrypt(cachingCmm, plainText, { encryptionContext })
```

------
#### [ JavaScript Node\.js ]

```
/*
 * Copyright 2019 Amazon.com, Inc. or its affiliates. All Rights Reserved.
 *
 * Licensed under the Apache License, Version 2.0 (the "License"). You may not use
 * this file except in compliance with the License. A copy of the License is
 * located at
 *
 *     http://aws.amazon.com/apache2.0/
 *
 * or in the "license" file accompanying this file. This file is distributed on an
 * "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or
 * implied. See the License for the specific language governing permissions and
 * limitations under the License.
 */
 
import { KmsKeyringNode, encrypt, decrypt, NodeCachingMaterialsManager, getLocalCryptographicMaterialsCache } from '@aws-crypto/client-node'

export async function cachingMaterialsManagerNodeSimpleTest ( plaintext ) {

  /* A KMS CMK is required to generate the data key.
   * You need kms:GenerateDataKey permission on the CMK in generatorKeyId.
   */
  const generatorKeyId = 'arn:aws:kms:us-west-2:111122223333:alias/EncryptDecrypt'

  /* Configure the KMS keyring with the desired CMKs. 
   * A generator key is required when encrypting data. You can also add an array of additional keys.
   */
  const keyring = new KmsKeyringNode({ generatorKeyId })


  /* Create a cache to hold the data keys and related cryptographic material.
   * In this example, we use the local cache provided by the Encryption SDK.
   *
   * `capacity` represents the maximum number of data keys in the cache.
   * When the cache is full, the oldest entry is evicted to make room for a
   * newer one. This value is required. 
   */
  const capacity = 10
    
  /* There is also a second optional parameter, "proactiveFrequency". 
   * By default, every 60 seconds (60,000 milliseconds) all data keys in the
   * cache are checked to verify that they conform to all data key caching 
   * thresholds. Data keys that exceed any threshold are evicted.
   *
   * To change how often this check runs, pass in a second value for the  
   * "proactiveFrequency" parameter. Its value is in milliseconds.
   * This example changes the frequency to 30000 (milleseconds).
   */
  const proactiveFrequency = 30000
    
  const cache = getLocalCryptographicMaterialsCache(capacity, proactiveFrequency)
  

  /* Set the security thresholds on the caching CMM:
   *  maxAge, maxBytesEncrypted, maxMessagesEncrypted.
   *  Only maxAge is required.
   */

  /* maxAge is the time in milliseconds that an entry is cached.
   * The cache actively removes elements that exceed its thresholds.
   */
  const maxAge = 1000 * 60

  /* The maximum amount of bytes that will be encrypted under a single data key.
   * The default value is 2^53 - 1 bytes. This value is optional, but you should
   * set it to the lowest practical value.   
   */
  const maxBytesEncrypted = 100

  /* The maximum number of messages encrypted under a single data key.
   * The default value is 2^32 bytes. This value is optional, but you should set
   * it to the lowest practical value.
   */
  const maxMessagesEncrypted = 100

  /* Use the keyring, local cache, and security thresholds to create
   * a caching CMM.
   */  
  const cachingCmm = new NodeCachingMaterialsManager({
    backingMaterials: keyring,
    cache,
    maxAge,
    maxBytesEncrypted,
    maxMessagesEncrypted
  })

  /* The encryption context is non-secret data that is cryptographically bound 
   * to the ciphertext when it is encrypted. To decrypt the data, the Encryption SDK
   * uses the same encryption context. (It gets the encryption context from the header
   * of the encrypted message; you don't have to supply it.)
   *
   * In addition, cached data keys are reused only when their encryption contexts match. 
   * Therefore, you can use the encryption context to create subgroups of data keys in 
   * your cache.
   *  
   * See: https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/data-caching-details.html#caching-encryption-context   
   */
  const encryptionContext = {
    stage: 'demo',
    purpose: 'simple demonstration app',
    origin: 'us-west-2'
  }

  /* Encrypt the data.
   *
   * When the call to the encrypt function specifies a caching CMM,
   * the encryption operation uses the data key cache.
   *
   * When you use the caching CMM, the encrypt method requires the length
   * of the plaintext. If you don't provide it, the encrypt operation fails. 
   * If you don't know the exact length of the plaintext, such as when you're
   * streaming data, provide the largest expected value.
   */
  const { result } = await encrypt(cachingCmm, plaintext, { encryptionContext, plaintextLength: plaintext.length })
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