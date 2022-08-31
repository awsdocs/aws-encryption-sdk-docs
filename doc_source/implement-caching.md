# How to use data key caching<a name="implement-caching"></a>

This topic shows you how to use data key caching in your application\. It takes you through the process step by step\. Then, it combines the steps in a simple example that uses data key caching in an operation to encrypt a string\.

The examples in this section show how to use [version 2\.0\.*x*](about-versions.md) and later of the AWS Encryption SDK\. For examples that use earlier versions, find your release in the [Releases](https://github.com/aws/aws-encryption-sdk-c/releases) list of the GitHub repository for your [programming language](programming-languages.md)\.

For complete and tested examples of using data key caching in the AWS Encryption SDK, see:
+ C/C\+\+: [caching\_cmm\.cpp](https://github.com/aws/aws-encryption-sdk-c/blob/master/examples/caching_cmm.cpp)
+ Java: [SimpleDataKeyCachingExample\.java](https://github.com/aws/aws-encryption-sdk-java/blob/master/src/examples/java/com/amazonaws/crypto/examples/SimpleDataKeyCachingExample.java)
+ JavaScript Browser: [caching\_cmm\.ts](https://github.com/aws/aws-encryption-sdk-javascript/blob/master/modules/example-browser/src/caching_cmm.ts)
+ JavaScript Node\.js: [caching\_cmm\.ts](https://github.com/aws/aws-encryption-sdk-javascript/blob/master/modules/example-node/src/caching_cmm.ts)
+ Python: [data\_key\_caching\_basic\.py](https://github.com/aws/aws-encryption-sdk-python/blob/master/examples/src/data_key_caching_basic.py)

The [AWS Encryption SDK for \.NET](dot-net.md) does not support data key caching\.

**Topics**
+ [Using data key caching: Step\-by\-step](#implement-caching-steps)
+ [Data key caching example: Encrypt a string](#caching-example-encrypt-string)

## Using data key caching: Step\-by\-step<a name="implement-caching-steps"></a>

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
  
  cache = aws_encryption_sdk.LocalCryptoMaterialsCache(MAX_CACHE_SIZE)
  ```

------

   
+ Create a [master key provider](concepts.md#master-key-provider) \(Java and Python\) or a [keyring](concepts.md#keyring) \(C and JavaScript\)\. These examples use an AWS Key Management Service \(AWS KMS\) master key provider or a compatible [AWS KMS keyring](use-kms-keyring.md)\.

   

------
#### [ C ]

  ```
  // Create an AWS KMS keyring
  //   The input is the Amazon Resource Name (ARN) 
  //   of an AWS KMS key
  
  struct aws_cryptosdk_keyring *kms_keyring = Aws::Cryptosdk::KmsKeyring::Builder().Build(kms_key_arn);
  ```

------
#### [ Java ]

  ```
  // Create an AWS KMS master key provider
  //   The input is the Amazon Resource Name (ARN) 
  //   of an AWS KMS key
  
  MasterKeyProvider<KmsMasterKey> keyProvider = KmsMasterKeyProvider.builder().buildStrict(kmsKeyArn);
  ```

------
#### [ JavaScript Browser ]

  In the browser, you must inject your credentials securely\. This example defines credentials in a webpack \(kms\.webpack\.config\) that resolves credentials at runtime\. It creates an AWS KMS client provider instance from an AWS KMS client and the credentials\. Then, when it creates the keyring, it passes the client provider to the constructor along with the AWS KMS key \(`generatorKeyId)`\.

  ```
  const { accessKeyId, secretAccessKey, sessionToken } = credentials
  
  const clientProvider = getClient(KMS, {
      credentials: {
        accessKeyId,
        secretAccessKey,
        sessionToken
      }
    })
  
  /*  Create an AWS KMS keyring
   *  You must configure the AWS KMS keyring with at least one AWS KMS key
   *  The input is the Amazon Resource Name (ARN) 
   */ of an AWS KMS key
  
  const keyring = new KmsKeyringBrowser({
      clientProvider,
      generatorKeyId,
      keyIds,
    })
  ```

------
#### [ JavaScript Node\.js ]

  ```
  /* Create an AWS KMS keyring
   *   The input is the Amazon Resource Name (ARN) 
  */   of an AWS KMS key
  
  const keyring = new KmsKeyringNode({ generatorKeyId })
  ```

------
#### [ Python ]

  ```
  # Create an AWS KMS master key provider
  #  The input is the Amazon Resource Name (ARN) 
  #  of an AWS KMS key
  
  key_provider = aws_encryption_sdk.StrictAwsKmsMasterKeyProvider(key_ids=[kms_key_arn])
  ```

------

   
+ [Create a caching cryptographic materials manager](data-caching-details.md#caching-cmm) \(caching CMM\)\. 

   

  Associate your caching CMM with your cache and your master key provider or keyring\. Then, [set cache security thresholds](thresholds.md) on the caching CMM\. 

   

------
#### [ C ]

  In the AWS Encryption SDK for C, you can create a caching CMM from an underlying CMM, such as the default CMM, or from a keyring\. This example creates the caching CMM from a keyring\.

  After you create the caching CMM, you can release your references to the keyring and the cache\. For details, see [Reference counting](c-language-using.md#c-language-using-release)\.

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
/* Create a session with the caching CMM. Set the session mode to encrypt. */
struct aws_cryptosdk_session *session = aws_cryptosdk_session_new_from_cmm_2(allocator, AWS_CRYPTOSDK_ENCRYPT, caching_cmm);

/* Set a message bound of 1000 bytes */
aws_status = aws_cryptosdk_session_set_message_bound(session, 1000);

/* Encrypt the message using the session with the caching CMM */
aws_status = aws_cryptosdk_session_process(
             session, output_buffer, output_capacity, &output_produced, input_buffer, input_len, &input_consumed);

/* Release your references to the caching CMM and the session. */
aws_cryptosdk_cmm_release(caching_cmm);
aws_cryptosdk_session_destroy(session);
```

------
#### [ Java ]

```
// When the call to encryptData specifies a caching CMM,
// the encryption operation uses the data key cache
final AwsCrypto encryptionSdk = AwsCrypto.standard();
return encryptionSdk.encryptData(cachingCmm, plaintext_source).getResult();
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
# Set up an encryption client
client = aws_encryption_sdk.EncryptionSDKClient()

# When the call to encrypt specifies a caching CMM,
# the encryption operation uses the data key cache
#
encrypted_message, header = client.encrypt(
    source=plaintext_source,
    materials_manager=caching_cmm
)
```

------

## Data key caching example: Encrypt a string<a name="caching-example-encrypt-string"></a>

This simple code example uses data key caching when encrypting a string\. It combines the code from the [step\-by\-step procedure](#implement-caching-steps) into test code that you can run\.

The example creates a [local cache](data-caching-details.md#simplecache) and a [master key provider](concepts.md#master-key-provider) or [keyring](concepts.md#keyring) for an AWS KMS key\. Then, it uses the local cache and master key provider or keyring to create a caching CMM with appropriate [security thresholds](thresholds.md)\. In Java and Python, the encryption request specifies the caching CMM, the plaintext data to encrypt, and an [encryption context](data-caching-details.md#caching-encryption-context)\. In C, the caching CMM is specified in the session, and the session is provided to the encryption request\.

To run these examples, you need to supply the [Amazon Resource Name \(ARN\) of an AWS KMS key](https://docs.aws.amazon.com/kms/latest/developerguide/viewing-keys.html)\. Be sure that you have [permission to use the AWS KMS key](https://docs.aws.amazon.com/kms/latest/developerguide/key-policies.html#key-policy-default-allow-users) to generate a data key\.

For more detailed, real\-world examples of creating and using a data key cache, see [Data key caching example code](sample-cache-example-code.md)\.

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
    const char *kms_key_arn,
    int max_entry_age,
    int cache_capacity) {
    const uint64_t MAX_ENTRY_MSGS = 100;

    struct aws_allocator *allocator = aws_default_allocator();
    
    // Load error strings for debugging
    aws_cryptosdk_load_error_strings();

    // Create a keyring
    struct aws_cryptosdk_keyring *kms_keyring = Aws::Cryptosdk::KmsKeyring::Builder().Build(kms_key_arn);

    // Create a cache
    struct aws_cryptosdk_materials_cache *cache = aws_cryptosdk_materials_cache_local_new(allocator, cache_capacity);

    // Create a caching CMM
    struct aws_cryptosdk_cmm *caching_cmm = aws_cryptosdk_caching_cmm_new_from_keyring(
        allocator, cache, kms_keyring, NULL, max_entry_age, AWS_TIMESTAMP_SECS);
    if (!caching_cmm) abort();

    if (aws_cryptosdk_caching_cmm_set_limit_messages(caching_cmm, MAX_ENTRY_MSGS)) abort();

    // Create a session
    struct aws_cryptosdk_session *session =        
        aws_cryptosdk_session_new_from_cmm_2(allocator, AWS_CRYPTOSDK_ENCRYPT, caching_cmm);
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
// Copyright Amazon.com Inc. or its affiliates. All Rights Reserved.
// SPDX-License-Identifier: Apache-2.0

package com.amazonaws.crypto.examples;

import com.amazonaws.encryptionsdk.AwsCrypto;
import com.amazonaws.encryptionsdk.CryptoMaterialsManager;
import com.amazonaws.encryptionsdk.MasterKeyProvider;
import com.amazonaws.encryptionsdk.caching.CachingCryptoMaterialsManager;
import com.amazonaws.encryptionsdk.caching.CryptoMaterialsCache;
import com.amazonaws.encryptionsdk.caching.LocalCryptoMaterialsCache;
import com.amazonaws.encryptionsdk.kmssdkv2.KmsMasterKey;
import com.amazonaws.encryptionsdk.kmssdkv2.KmsMasterKeyProvider;
import java.nio.charset.StandardCharsets;
import java.util.Collections;
import java.util.Map;
import java.util.concurrent.TimeUnit;

/**
 * <p>
 * Encrypts a string using an &KMS; key and data key caching
 *
 * <p>
 * Arguments:
 * <ol>
 * <li>KMS Key ARN: To find the Amazon Resource Name of your &KMS; key,
 *     see 'Find the key ID and ARN' at https://docs.aws.amazon.com/kms/latest/developerguide/find-cmk-id-arn.html
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

    public static byte[] encryptWithCaching(String kmsKeyArn, int maxEntryAge, int cacheCapacity) {
        // Plaintext data to be encrypted
        byte[] myData = "My plaintext data".getBytes(StandardCharsets.UTF_8);

        // Encryption context
        // Most encrypted data should have an associated encryption context
        // to protect integrity. This sample uses placeholder values.
        // For more information see:
        // blogs.aws.amazon.com/security/post/Tx2LZ6WBJJANTNW/How-to-Protect-the-Integrity-of-Your-Encrypted-Data-by-Using-AWS-Key-Management
        final Map<String, String> encryptionContext = Collections.singletonMap("purpose", "test");

        // Create a master key provider
        MasterKeyProvider<KmsMasterKey> keyProvider = KmsMasterKeyProvider.builder()
            .buildStrict(kmsKeyArn);

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
        final AwsCrypto encryptionSdk = AwsCrypto.standard();
        return encryptionSdk.encryptData(cachingCmm, myData, encryptionContext).getResult();
    }
}
```

------
#### [ JavaScript Browser ]

```
// Copyright Amazon.com Inc. or its affiliates. All Rights Reserved.
// SPDX-License-Identifier: Apache-2.0

/* This is a simple example of using a caching CMM with a KMS keyring
 * to encrypt and decrypt using the AWS Encryption SDK for Javascript in a browser.
 */

import {
  KmsKeyringBrowser,
  KMS,
  getClient,
  buildClient,
  CommitmentPolicy,
  WebCryptoCachingMaterialsManager,
  getLocalCryptographicMaterialsCache,
} from '@aws-crypto/client-browser'
import { toBase64 } from '@aws-sdk/util-base64-browser'

/* This builds the client with the REQUIRE_ENCRYPT_REQUIRE_DECRYPT commitment policy,
 * which enforces that this client only encrypts using committing algorithm suites
 * and enforces that this client
 * will only decrypt encrypted messages
 * that were created with a committing algorithm suite.
 * This is the default commitment policy
 * if you build the client with `buildClient()`.
 */
const { encrypt, decrypt } = buildClient(
  CommitmentPolicy.REQUIRE_ENCRYPT_REQUIRE_DECRYPT
)

/* This is injected by webpack.
 * The webpack.DefinePlugin or @aws-sdk/karma-credential-loader will replace the values when bundling.
 * The credential values are pulled from @aws-sdk/credential-provider-node
 * Use any method you like to get credentials into the browser.
 * See kms.webpack.config
 */
declare const credentials: {
  accessKeyId: string
  secretAccessKey: string
  sessionToken: string
}

/* This is done to facilitate testing. */
export async function testCachingCMMExample() {
  /* This example uses an &KMS; keyring. The generator key in a &KMS; keyring generates and encrypts the data key.
   * The caller needs kms:GenerateDataKey permission on the &KMS; key in generatorKeyId.
   */
  const generatorKeyId =
    'arn:aws:kms:us-west-2:658956600833:alias/EncryptDecrypt'

  /* Adding additional KMS keys that can decrypt.
   * The caller must have kms:Encrypt permission for every &KMS; key in keyIds.
   * You might list several keys in different AWS Regions.
   * This allows you to decrypt the data in any of the represented Regions.
   * In this example, the generator key
   * and the additional key are actually the same &KMS; key.
   * In `generatorId`, this &KMS; key is identified by its alias ARN.
   * In `keyIds`, this &KMS; key is identified by its key ARN.
   * In practice, you would specify different &KMS; keys,
   * or omit the `keyIds` parameter.
   * This is *only* to demonstrate how the &KMS; key ARNs are configured.
   */
  const keyIds = [
    'arn:aws:kms:us-west-2:658956600833:key/b3537ef1-d8dc-4780-9f5a-55776cbb2f7f',
  ]

  /* Need a client provider that will inject correct credentials.
   * The credentials here are injected by webpack from your environment bundle is created
   * The credential values are pulled using @aws-sdk/credential-provider-node.
   * See kms.webpack.config
   * You should inject your credential into the browser in a secure manner
   * that works with your application.
   */
  const { accessKeyId, secretAccessKey, sessionToken } = credentials

  /* getClient takes a KMS client constructor
   * and optional configuration values.
   * The credentials can be injected here,
   * because browsers do not have a standard credential discovery process the way Node.js does.
   */
  const clientProvider = getClient(KMS, {
    credentials: {
      accessKeyId,
      secretAccessKey,
      sessionToken,
    },
  })

  /* You must configure the KMS keyring with your &KMS; keys */
  const keyring = new KmsKeyringBrowser({
    clientProvider,
    generatorKeyId,
    keyIds,
  })

  /* Create a cache to hold the data keys (and related cryptographic material).
   * This example uses the local cache provided by the Encryption SDK.
   * The `capacity` value represents the maximum number of entries
   * that the cache can hold.
   * To make room for an additional entry,
   * the cache evicts the oldest cached entry.
   * Both encrypt and decrypt requests count independently towards this threshold.
   * Entries that exceed any cache threshold are actively removed from the cache.
   * By default, the SDK checks one item in the cache every 60 seconds (60,000 milliseconds).
   * To change this frequency, pass in a `proactiveFrequency` value
   * as the second parameter. This value is in milliseconds.
   */
  const capacity = 100
  const cache = getLocalCryptographicMaterialsCache(capacity)

  /* The partition name lets multiple caching CMMs share the same local cryptographic cache.
   * By default, the entries for each CMM are cached separately. However, if you want these CMMs to share the cache,
   * use the same partition name for both caching CMMs.
   * If you don't supply a partition name, the Encryption SDK generates a random name for each caching CMM.
   * As a result, sharing elements in the cache MUST be an intentional operation.
   */
  const partition = 'local partition name'

  /* maxAge is the time in milliseconds that an entry will be cached.
   * Elements are actively removed from the cache.
   */
  const maxAge = 1000 * 60

  /* The maximum number of bytes that will be encrypted under a single data key.
   * This value is optional,
   * but you should configure the lowest practical value.
   */
  const maxBytesEncrypted = 100

  /* The maximum number of messages that will be encrypted under a single data key.
   * This value is optional,
   * but you should configure the lowest practical value.
   */
  const maxMessagesEncrypted = 10

  const cachingCMM = new WebCryptoCachingMaterialsManager({
    backingMaterials: keyring,
    cache,
    partition,
    maxAge,
    maxBytesEncrypted,
    maxMessagesEncrypted,
  })

  /* Encryption context is a *very* powerful tool for controlling
   * and managing access.
   * When you pass an encryption context to the encrypt function,
   * the encryption context is cryptographically bound to the ciphertext.
   * If you don't pass in the same encryption context when decrypting,
   * the decrypt function fails.
   * The encryption context is ***not*** secret!
   * Encrypted data is opaque.
   * You can use an encryption context to assert things about the encrypted data.
   * The encryption context helps you to determine
   * whether the ciphertext you retrieved is the ciphertext you expect to decrypt.
   * For example, if you are are only expecting data from 'us-west-2',
   * the appearance of a different AWS Region in the encryption context can indicate malicious interference.
   * See: https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/concepts.html#encryption-context
   *
   * Also, cached data keys are reused ***only*** when the encryption contexts passed into the functions are an exact case-sensitive match.
   * See: https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/data-caching-details.html#caching-encryption-context
   */
  const encryptionContext = {
    stage: 'demo',
    purpose: 'simple demonstration app',
    origin: 'us-west-2',
  }

  /* Find data to encrypt. */
  const plainText = new Uint8Array([1, 2, 3, 4, 5])

  /* Encrypt the data.
   * The caching CMM only reuses data keys
   * when it know the length (or an estimate) of the plaintext.
   * However, in the browser,
   * you must provide all of the plaintext to the encrypt function.
   * Therefore, the encrypt function in the browser knows the length of the plaintext
   * and does not accept a plaintextLength option.
   */
  const { result } = await encrypt(cachingCMM, plainText, { encryptionContext })

  /* Log the plain text
   * only for testing and to show that it works.
   */
  console.log('plainText:', plainText)
  document.write('</br>plainText:' + plainText + '</br>')

  /* Log the base64-encoded result
   * so that you can try decrypting it with another AWS Encryption SDK implementation.
   */
  const resultBase64 = toBase64(result)
  console.log(resultBase64)
  document.write(resultBase64)

  /* Decrypt the data.
   * NOTE: This decrypt request will not use the data key
   * that was cached during the encrypt operation.
   * Data keys for encrypt and decrypt operations are cached separately.
   */
  const { plaintext, messageHeader } = await decrypt(cachingCMM, result)

  /* Grab the encryption context so you can verify it. */
  const { encryptionContext: decryptedContext } = messageHeader

  /* Verify the encryption context.
   * If you use an algorithm suite with signing,
   * the Encryption SDK adds a name-value pair to the encryption context that contains the public key.
   * Because the encryption context might contain additional key-value pairs,
   * do not include a test that requires that all key-value pairs match.
   * Instead, verify that the key-value pairs that you supplied to the `encrypt` function are included in the encryption context that the `decrypt` function returns.
   */
  Object.entries(encryptionContext).forEach(([key, value]) => {
    if (decryptedContext[key] !== value)
      throw new Error('Encryption Context does not match expected values')
  })

  /* Log the clear message
   * only for testing and to show that it works.
   */
  document.write('</br>Decrypted:' + plaintext)
  console.log(plaintext)

  /* Return the values to make testing easy. */
  return { plainText, plaintext }
}
```

------
#### [ JavaScript Node\.js ]

```
// Copyright Amazon.com Inc. or its affiliates. All Rights Reserved.
// SPDX-License-Identifier: Apache-2.0

import {
  KmsKeyringNode,
  buildClient,
  CommitmentPolicy,
  NodeCachingMaterialsManager,
  getLocalCryptographicMaterialsCache,
} from '@aws-crypto/client-node'

/* This builds the client with the REQUIRE_ENCRYPT_REQUIRE_DECRYPT commitment policy,
 * which enforces that this client only encrypts using committing algorithm suites
 * and enforces that this client
 * will only decrypt encrypted messages
 * that were created with a committing algorithm suite.
 * This is the default commitment policy
 * if you build the client with `buildClient()`.
 */
const { encrypt, decrypt } = buildClient(
  CommitmentPolicy.REQUIRE_ENCRYPT_REQUIRE_DECRYPT
)

export async function cachingCMMNodeSimpleTest() {
  /* An &KMS; key is required to generate the data key.
   * You need kms:GenerateDataKey permission on the &KMS; key in generatorKeyId.
   */
  const generatorKeyId =
    'arn:aws:kms:us-west-2:658956600833:alias/EncryptDecrypt'

  /* Adding alternate &KMS; keys that can decrypt.
   * Access to kms:Encrypt is required for every &KMS; key in keyIds.
   * You might list several keys in different AWS Regions.
   * This allows you to decrypt the data in any of the represented Regions.
   * In this example, the generator key
   * and the additional key are actually the same &KMS; key.
   * In `generatorId`, this &KMS; key is identified by its alias ARN.
   * In `keyIds`, this &KMS; key is identified by its key ARN.
   * In practice, you would specify different &KMS; keys,
   * or omit the `keyIds` parameter.
   * This is *only* to demonstrate how the &KMS; key ARNs are configured.
   */
  const keyIds = [
    'arn:aws:kms:us-west-2:658956600833:key/b3537ef1-d8dc-4780-9f5a-55776cbb2f7f',
  ]

  /* The &KMS; keyring must be configured with the desired &KMS; keys
   * This example passes the keyring to the caching CMM
   * instead of using it directly.
   */
  const keyring = new KmsKeyringNode({ generatorKeyId, keyIds })

  /* Create a cache to hold the data keys (and related cryptographic material).
   * This example uses the local cache provided by the Encryption SDK.
   * The `capacity` value represents the maximum number of entries
   * that the cache can hold.
   * To make room for an additional entry,
   * the cache evicts the oldest cached entry.
   * Both encrypt and decrypt requests count independently towards this threshold.
   * Entries that exceed any cache threshold are actively removed from the cache.
   * By default, the SDK checks one item in the cache every 60 seconds (60,000 milliseconds).
   * To change this frequency, pass in a `proactiveFrequency` value
   * as the second parameter. This value is in milliseconds.
   */
  const capacity = 100
  const cache = getLocalCryptographicMaterialsCache(capacity)

  /* The partition name lets multiple caching CMMs share the same local cryptographic cache.
   * By default, the entries for each CMM are cached separately. However, if you want these CMMs to share the cache,
   * use the same partition name for both caching CMMs.
   * If you don't supply a partition name, the Encryption SDK generates a random name for each caching CMM.
   * As a result, sharing elements in the cache MUST be an intentional operation.
   */
  const partition = 'local partition name'

  /* maxAge is the time in milliseconds that an entry will be cached.
   * Elements are actively removed from the cache.
   */
  const maxAge = 1000 * 60

  /* The maximum amount of bytes that will be encrypted under a single data key.
   * This value is optional,
   * but you should configure the lowest value possible.
   */
  const maxBytesEncrypted = 100

  /* The maximum number of messages that will be encrypted under a single data key.
   * This value is optional,
   * but you should configure the lowest value possible.
   */
  const maxMessagesEncrypted = 10

  const cachingCMM = new NodeCachingMaterialsManager({
    backingMaterials: keyring,
    cache,
    partition,
    maxAge,
    maxBytesEncrypted,
    maxMessagesEncrypted,
  })

  /* Encryption context is a *very* powerful tool for controlling
   * and managing access.
   * When you pass an encryption context to the encrypt function,
   * the encryption context is cryptographically bound to the ciphertext.
   * If you don't pass in the same encryption context when decrypting,
   * the decrypt function fails.
   * The encryption context is ***not*** secret!
   * Encrypted data is opaque.
   * You can use an encryption context to assert things about the encrypted data.
   * The encryption context helps you to determine
   * whether the ciphertext you retrieved is the ciphertext you expect to decrypt.
   * For example, if you are are only expecting data from 'us-west-2',
   * the appearance of a different AWS Region in the encryption context can indicate malicious interference.
   * See: https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/concepts.html#encryption-context
   *
   * Also, cached data keys are reused ***only*** when the encryption contexts passed into the functions are an exact case-sensitive match.
   * See: https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/data-caching-details.html#caching-encryption-context
   */
  const encryptionContext = {
    stage: 'demo',
    purpose: 'simple demonstration app',
    origin: 'us-west-2',
  }

  /* Find data to encrypt.  A simple string. */
  const cleartext = 'asdf'

  /* Encrypt the data.
   * The caching CMM only reuses data keys
   * when it know the length (or an estimate) of the plaintext.
   * If you do not know the length,
   * because the data is a stream
   * provide an estimate of the largest expected value.
   *
   * If your estimate is smaller than the actual plaintext length
   * the AWS Encryption SDK will throw an exception.
   *
   * If the plaintext is not a stream,
   * the AWS Encryption SDK uses the actual plaintext length
   * instead of any length you provide.
   */
  const { result } = await encrypt(cachingCMM, cleartext, {
    encryptionContext,
    plaintextLength: 4,
  })

  /* Decrypt the data.
   * NOTE: This decrypt request will not use the data key
   * that was cached during the encrypt operation.
   * Data keys for encrypt and decrypt operations are cached separately.
   */
  const { plaintext, messageHeader } = await decrypt(cachingCMM, result)

  /* Grab the encryption context so you can verify it. */
  const { encryptionContext: decryptedContext } = messageHeader

  /* Verify the encryption context.
   * If you use an algorithm suite with signing,
   * the Encryption SDK adds a name-value pair to the encryption context that contains the public key.
   * Because the encryption context might contain additional key-value pairs,
   * do not include a test that requires that all key-value pairs match.
   * Instead, verify that the key-value pairs that you supplied to the `encrypt` function are included in the encryption context that the `decrypt` function returns.
   */
  Object.entries(encryptionContext).forEach(([key, value]) => {
    if (decryptedContext[key] !== value)
      throw new Error('Encryption Context does not match expected values')
  })

  /* Return the values so the code can be tested. */
  return { plaintext, result, cleartext, messageHeader }
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
from aws_encryption_sdk import CommitmentPolicy


def encrypt_with_caching(kms_key_arn, max_age_in_cache, cache_capacity):
    """Encrypts a string using an &KMS; key and data key caching.

    :param str kms_key_arn: Amazon Resource Name (ARN) of the &KMS; key
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

    # Set up an encryption client with an explicit commitment policy. Note that if you do not explicitly choose a
    # commitment policy, REQUIRE_ENCRYPT_REQUIRE_DECRYPT is used by default.
    client = aws_encryption_sdk.EncryptionSDKClient(commitment_policy=CommitmentPolicy.REQUIRE_ENCRYPT_REQUIRE_DECRYPT)

    # Create a master key provider for the &KMS; key
    key_provider = aws_encryption_sdk.StrictAwsKmsMasterKeyProvider(key_ids=[kms_key_arn])

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
    encrypted_message, _header = client.encrypt(
        source=my_data, materials_manager=caching_cmm, encryption_context=encryption_context
    )

    return encrypted_message
```

------