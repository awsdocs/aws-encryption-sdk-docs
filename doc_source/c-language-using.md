# Using the AWS Encryption SDK for C<a name="c-language-using"></a>

This topic explains some of the features of the AWS Encryption SDK for C that are not supported in other programming language implementations\. 

For details about programming with the AWS Encryption SDK, see the [C examples](c-examples.md), the [examples](https://github.com/aws/aws-encryption-sdk-c/tree/master/examples) in the [aws\-encryption\-sdk\-c repository](https://github.com/aws/aws-encryption-sdk-c/) on GitHub, and the [C API documentation](https://aws.github.io/aws-encryption-sdk-c/html/)\.

See also: [Choosing a Keyring](choose-keyring.md)

**Topics**
+ [Using Keyrings](#using-c-sdk)
+ [Patterns for Encrypting and Decrypting Data](#c-language-using-pattern)
+ [Reference Counting](#c-language-using-release)

## Using Keyrings<a name="using-c-sdk"></a>

One of the most important tasks you perform when using the AWS Encryption SDK for C is selecting and configuring a keyring\. A [keyring](concepts.md#keyring) generates, encrypts, and decrypts data keys\. Each keyring is typically associated with a wrapping key or a service that provides and protects wrapping keys\. You can use the keyrings that the AWS Encryption SDK provides, or write your own compatible custom keyrings\. For help in choosing a keyring, see [Choosing a Keyring](choose-keyring.md)\. 

The keyrings in the AWS Encryption SDK for C make it easier for you to determine which wrapping keys are used to encrypt and decrypt your data\. Keyrings take the place of master key providers in the Java and Python implementations of the AWS Encryption SDK\. Despite this architectural difference, all of the language implementations are fully interoperable\. However, the keyring and master key provider must be configured with the same or corresponding wrapping keys\. For details, see [Keyring Compatibility](choose-keyring.md#keyring-compatibility)\.

You instantiate and configure your keyring, but you don't interact with it directly\. The [cryptographic materials manager \(CMM\)](concepts.md#crypt-materials-manager) in your session interacts with the keyring for you\. 

When you encrypt data, the CMM asks the keyring for encryption materials\. The keyring returns a plaintext key and a copy of that key encrypted by each of its wrapping keys\. The AWS Encryption SDK uses the plaintext key to encrypt the data and stores the encrypted data keys with the data in the [encrypted message](concepts.md#message) that it returns\.

![\[Encrypting with a keyring\]](http://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/images/keyring-encrypt.png)

When you decrypt data, the CMM passes in the encryption keys from the encrypted message and asks the keyring to decrypt any one of them\. The keyring uses its wrapping keys to decrypt one of the encrypted data keys and returns a plaintext key\. The AWS Encryption SDK uses the plaintext key to decrypt the data\.

![\[Decrypting with a keyring\]](http://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/images/keyring-decrypt.png)

You can use a single keyring or also combine keyrings of the same type or a different type into a [multi\-keyring](choose-keyring.md#use-multi-keyring)\. When you encrypt data, the multi\-keyring returns a copy of the data key encrypted by all of the wrapping keys in all of the keyrings that comprise the multi\-keyring\. You can decrypt the data using a keyring configured with any one of the wrapping keys in the multi\-keyring\.

For help selecting and configuring your keyrings, see [Choosing a Keyring](choose-keyring.md)\.

## Patterns for Encrypting and Decrypting Data<a name="c-language-using-pattern"></a>

When you use the AWS Encryption SDK for C, you follow a pattern similar to this: create a [keyring](concepts.md#keyring), create a [CMM](concepts.md#crypt-materials-manager) that uses the keyring, create a session that uses the CMM \(and keyring\), and then process the session\.

1\. Create a keyring\.  
Configure your [keyring](concepts.md#keyring) with the wrapping keys that you want to use to encrypt your data keys\. This example uses a [KMS keyring](choose-keyring.md#use-kms-keyring) with one AWS KMS customer master key \(CMK\), but you can use any type of keyring in its place\.  
When you specify an AWS KMS CMK for a keyring, you must use the Amazon Resource Name \(ARN\) of the CMK\. In an encryption keyring, you can specify the key ARN or alias ARN\. In a decryption keyring, you must use the key ARN\. For help finding the key ARN, see [Finding the Key ID and ARN](https://docs.aws.amazon.com/kms/latest/developerguide/viewing-keys.html#find-cmk-id-arn) in the *AWS Key Management Service Developer Guide*\. To find the alias ARN, use the [ListAliases](https://docs.aws.amazon.com/kms/latest/APIReference/API_ListAliases.html) API\.  

```
const char * KEY_ARN = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab"    

struct aws_cryptosdk_keyring *kms_keyring = 
       Aws::Cryptosdk::KmsKeyring::Builder().Build(KEY_ARN);
```

2\. Create a session\.  
In the AWS Encryption SDK for C, you use a *session* to encrypt a single plaintext message or decrypt a single ciphertext message, regardless of its size\. The session maintains the state of the message throughout its processing\.   
Configure your session with an allocator, a keyring, and a mode: `AWS_CRYPTOSDK_ENCRYPT` or `AWS_CRYPTOSDK_DECRYPT`\. If you need to change the mode of the session, use the `aws_cryptosdk_session_reset` method\.  
When you create a session with a keyring, the AWS Encryption SDK for C automatically creates a default cryptographic materials manager \(CMM\) for you\. You don't need to create, maintain, or destroy this object\.   
For example, the following session uses the allocator and the keyring that was defined in step 1\. When you encrypt data, the mode is `AWS_CRYPTOSDK_ENCRYPT`\.  

```
struct aws_cryptosdk_session * session = aws_cryptosdk_session_new_from_keyring(allocator, AWS_CRYPTOSDK_ENCRYPT, kms_keyring);
```

3\. Set the message size\.  
When encrypting data, you need to call the `aws_cryptosdk_session_set_message_size` method, which tells the AWS Encryption SDK the length of the plaintext data\. You can call this method before or during the session processing, but you must call it once\. Otherwise, the session processing method doesn't know where the input data ends\.  
Don't call this method when decrypting\. The AWS Encryption SDK uses the footer in the [encrypted message](message-format.md) to determine where the ciphertext ends\.  

```
const char *plaintext = "Hello world!";
const size_t plaintext_len = strlen(plaintext);

aws_cryptosdk_session_set_message_size(session, plaintext_length)
```

4\. Encrypt or decrypt the data\.  
To process the data in the session, use the `aws_cryptosdk_session_process` method\. If the input buffer is large enough to hold the entire plaintext, and the output buffer is large enough to hold the entire ciphertext, you need to call the method only once\. However, if you need to handle streaming data, you can call the method in a loop\. For an example, see the [file\_streaming\.cpp](https://github.com/aws/aws-encryption-sdk-c/blob/master/examples/file_streaming.cpp) example\.  
When the session is configured to encrypt data, the plaintext fields describe the input and the ciphertext fields describe the output\. The `plaintext` field holds the message that you want to encrypt and the `ciphertext` field gets the [encrypted message](message-format.md) that the encrypt method returns\.   

```
// Encrypting data
aws_cryptosdk_session_process(session,
                              ciphertext,
                              ciphertext_buffer_size,
                              ciphertext_length,
                              plaintext,
                              plaintext_length,
                              &plaintext_consumed)
```
When the session is configured to decrypt data, the ciphertext fields describe the input and the plaintext fields describe the output\. The `ciphertext` field holds the [encrypted message](message-format.md) that the encrypt method returned, and the `plaintext` field gets the plaintext message that the decrypt method returns\.  
To decrypt the data, call the `aws_cryptosdk_session_process` method\.  

```
// Decrypting data
aws_cryptosdk_session_process(session,
                              plaintext,
                              plaintext_buffer_size,
                              plaintext_length,
                              ciphertext,
                              ciphertext_length,
                              &ciphertext_consumed)
```

## Reference Counting<a name="c-language-using-release"></a>

To prevent memory leaks, be sure to release your references to all objects that you create when you are finished with them\. Otherwise, you end up with memory leaks\. The SDK provides methods to make this task easier\.

Whenever you create a parent object with one of the following child objects, the parent object gets and maintains a reference to the child object, as follows:
+ A [keyring](concepts.md#keyring), such as creating a session with a keyring
+ A default [cryptographic materials manager](concepts.md#crypt-materials-manager) \(CMM\), such as creating a session or custom CMM with a default CMM
+ A [data key cache](data-key-caching.md), such as creating a caching CMM with a keyring and cache

Unless you need an independent reference to the child object, you can release your reference to the child object as soon as you create the parent object\. The remaining reference to the child object is released when the parent object is destroyed\. This pattern ensures that you maintain the reference to each object only for as long as you need it, and you don't leak memory due to unreleased references\. 

You are only responsible for releasing references to the child objects that you create explicitly\. You are not responsible for managing references to any objects that the SDK creates for you\. If the SDK creates an object, such as the default CMM that the `aws_cryptosdk_caching_cmm_new_from_keyring` method adds to a session, the SDK manages the creation and destruction of the object and its references\.

In the following example, when you create a session with a [keyring](concepts.md#keyring), the session gets a reference to the keyring, and maintains that reference until the session is destroyed\. If you do not need to maintain an additional reference to the keyring, you can use the `aws_cryptosdk_keyring_release` method to release the keyring object as soon as the session is created\. This method decrements the reference count for the keyring\. The session's reference to the keyring is released when you call `aws_cryptosdk_session_destroy` to destroy the session\. 

```
// The session gets a reference to the keyring.
struct aws_cryptosdk_session *session =
	aws_cryptosdk_session_new_from_keyring(alloc, AWS_CRYPTOSDK_ENCRYPT, keyring);

// After you create a session with a keyring, release the reference to the keyring object.
aws_cryptosdk_keyring_release(keyring);
```

For more complex tasks, such as reusing a keyring for multiple sessions or specifying an algorithm suite in a CMM, you might need to maintain an independent reference to the object\. If so, don't call the release methods immediately\. Instead, release your references when you are no longer using the objects, in addition to destroying the session\.

This reference counting technique also works when you are using alternate CMMs, such as the caching CMM for [data key caching](data-key-caching.md)\. When you create a caching CMM from a cache and a keyring, the caching CMM gets a reference to both objects\. Unless you need them for another task, you can release your independent references to the cache and keyring as soon as the caching CMM is created\. Then, when you create a session with the caching CMM, you can release your reference to the caching CMM\. 

Notice that you are only responsible for releasing references to objects that you create explicitly\. Objects that the methods create for you, such as the default CMM that underlies the caching CMM, are managed by the method\.

```
/ Create the caching CMM from a cache and a keyring.
struct aws_cryptosdk_cmm *caching_cmm = aws_cryptosdk_caching_cmm_new_from_keyring(allocator, cache, kms_keyring, NULL, 60, AWS_TIMESTAMP_SECS);

// Release your references to the cache and the keyring.
aws_cryptosdk_materials_cache_release(cache);
aws_cryptosdk_keyring_release(kms_keyring);

// Create a session with the caching CMM.
struct aws_cryptosdk_session *session = aws_cryptosdk_session_new_from_cmm(allocator, AWS_CRYPTOSDK_ENCRYPT, caching_cmm);

// Release your references to the caching CMM.
aws_cryptosdk_cmm_release(caching_cmm);

// ...

aws_cryptosdk_session_destroy(session);
```