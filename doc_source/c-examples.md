# AWS Encryption SDK for C examples<a name="c-examples"></a>

The following examples show you how to use the AWS Encryption SDK for C to encrypt and decrypt data\. 

The examples in this section show how to use versions 2\.0\.*x* and later of the AWS Encryption SDK for C\. For examples that use earlier versions, find your release in the [Releases](https://github.com/aws/aws-encryption-sdk-c/releases) list of the [aws\-encryption\-sdk\-c repository](https://github.com/aws/aws-encryption-sdk-c/) repository on GitHub\.

When you install and build the AWS Encryption SDK for C, the source code for these and other examples are included in the `examples` subdirectory, and they are compiled and built into the `build` directory\. You can also find them in the [examples](https://github.com/aws/aws-encryption-sdk-c/tree/master/examples) subdirectory of the [aws\-encryption\-sdk\-c](https://github.com/aws/aws-encryption-sdk-c/) repository on GitHub\.

**Topics**
+ [Encrypting and decrypting strings](#c-example-strings)

## Encrypting and decrypting strings<a name="c-example-strings"></a>

The following example shows you how to use the AWS Encryption SDK for C to encrypt and decrypt a string\.

This example features the [AWS KMS keyring](use-kms-keyring.md), a type of keyring that uses an AWS KMS key in [AWS Key Management Service \(AWS KMS\)](https://docs.aws.amazon.com/kms/latest/developerguide/) to generate and encrypt data keys\. The example includes code written in C\+\+\. The AWS Encryption SDK for C requires the AWS SDK for C\+\+ to call AWS KMS when using AWS KMS keyrings\. If you're using a keyring that doesn't interact with AWS KMS, such as a raw AES keyring, a raw RSA keyring, or a multi\-keyring that doesn't include an AWS KMS keyring, the AWS SDK for C\+\+ is not required\.

For help creating an AWS KMS key, see [Creating Keys](https://docs.aws.amazon.com/kms/latest/developerguide/create-keys.html) in the *AWS Key Management Service Developer Guide*\. For help identifying the AWS KMS keys in an AWS KMS keyring, see [Identifying AWS KMS keys in an AWS KMS keyring](use-kms-keyring.md#kms-keyring-id)\.

**See the complete code sample**: [string\.cpp](https://github.com/aws/aws-encryption-sdk-c/blob/master/examples/string.cpp)

**Topics**
+ [Encrypt a string](#c-example-string-encrypt)
+ [Decrypt a string](#c-example-string-decrypt)

### Encrypt a string<a name="c-example-string-encrypt"></a>

The first part of this example uses an AWS KMS keyring with one AWS KMS key to encrypt a plaintext string\. 

Step 1\. Load error strings\.  
Call the `aws_cryptosdk_load_error_strings()` method in your C or C\+\+ code\. It loads error information that is very useful for debugging\.  
You only need to call it once, such as in your `main` method\.  

```
/* Load error strings for debugging */
aws_cryptosdk_load_error_strings();
```

Step 2: Construct the keyring\.  
Create an AWS KMS keyring for encryption\. The keyring in this example is configured with one AWS KMS key, but you can configure an AWS KMS keyring with multiple AWS KMS keys, including AWS KMS keys in different AWS Regions and different accounts\.   
To identify an AWS KMS key in an encryption keyring in the AWS Encryption SDK for C, specify a [key ARN](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#key-id-key-ARN) or [alias ARN](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#key-id-alias-arn)\. In a decryption keyring, you must use a key ARN\. For details, see [Identifying AWS KMS keys in an AWS KMS keyring](use-kms-keyring.md#kms-keyring-id)\.  
[Identifying AWS KMS keys in an AWS KMS keyring](use-kms-keyring.md#kms-keyring-id)  
When you create a keyring with multiple AWS KMS keys, you specify the AWS KMS key used to generate and encrypt the plaintext data key, and an optional array of additional AWS KMS keys that encrypt the same plaintext data key\. In this case, you specify only the generator AWS KMS key\.   
Before running this code, replace the example key ARN with a valid one\.  

```
const char * key_arn = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";    

struct aws_cryptosdk_keyring *kms_keyring = 
       Aws::Cryptosdk::KmsKeyring::Builder().Build(key_arn);
```

Step 3: Create a session\.  
Create a session using the allocator, a mode enumerator, and the keyring\.  
Every session requires a mode: either `AWS_CRYPTOSDK_ENCRYPT` to encrypt or `AWS_CRYPTOSDK_DECRYPT` to decrypt\. To change the mode of an existing session, use the `aws_cryptosdk_session_reset` method\.  
After you create a session with the keyring, you can release your reference to the keyring using the method that the SDK provides\. The session retains a reference to the keyring object during its lifetime\. References to the keyring and session objects are released when you destroy the session\. This [reference counting](c-language-using.md#c-language-using-release) technique helps to prevent memory leaks and to prevent the objects from being released while they are in use\.  

```
struct aws_cryptosdk_session *session = 
       aws_cryptosdk_session_new_from_keyring_2(alloc, AWS_CRYPTOSDK_ENCRYPT, kms_keyring);

/* When you add the keyring to the session, release the keyring object */
aws_cryptosdk_keyring_release(kms_keyring);
```

Step 4: Set the encryption context\.  
An [encryption context](concepts.md#encryption-context) is arbitrary, non\-secret additional authenticated data\. When you provide an encryption context on encrypt, the AWS Encryption SDK cryptographically binds the encryption context to the ciphertext so that the same encryption context is required to decrypt the data\. Using an encryption context is optional, but we recommend it as a best practice\.  
First, create a hash table that includes the encryption context strings\.  

```
/* Allocate a hash table for the encryption context */
int set_up_enc_ctx(struct aws_allocator *alloc, struct aws_hash_table *my_enc_ctx) 

// Create encryption context strings
AWS_STATIC_STRING_FROM_LITERAL(enc_ctx_key1, "Example");
AWS_STATIC_STRING_FROM_LITERAL(enc_ctx_value1, "String");
AWS_STATIC_STRING_FROM_LITERAL(enc_ctx_key2, "Company");
AWS_STATIC_STRING_FROM_LITERAL(enc_ctx_value2, "MyCryptoCorp");

// Put the key-value pairs in the hash table
aws_hash_table_put(my_enc_ctx, enc_ctx_key1, (void *)enc_ctx_value1, &was_created)
aws_hash_table_put(my_enc_ctx, enc_ctx_key2, (void *)enc_ctx_value2, &was_created)
```
Get a mutable pointer to the encryption context in the session\. Then, use the `aws_cryptosdk_enc_ctx_clone` function to copy the encryption context into the session\. Keep the copy in `my_enc_ctx` so you can validate the value after decrypting the data\.  
The encryption context is part of the session, not a parameter passed to the session process function\. This guarantees that the same encryption context is used for every segment of a message, even if the session process function is called multiple times to encrypt the entire message\.  

```
struct aws_hash_table *session_enc_ctx = aws_cryptosdk_session_get_enc_ctx_ptr_mut(session);

aws_cryptosdk_enc_ctx_clone(alloc, session_enc_ctx, my_enc_ctx)
```

Step 5: Encrypt the string\.  
To encrypt the plaintext string, use the `aws_cryptosdk_session_process_full` method with the session in encryption mode\. This method, introduced in AWS Encryption SDK versions 1\.9\.*x* and 2\.2\.*x*, is designed for non\-streaming encryption and decryption\. To handle streaming data, call the `aws_cryptosdk_session_process` in a loop\.  
When encrypting, the plaintext fields are input fields; the ciphertext fields are output fields\. When the processing is complete, the `ciphertext_output` field contains the [encrypted message](concepts.md#message), including the actual ciphertext, encrypted data keys, and the encryption context\. You can decrypt this encrypted message by using the AWS Encryption SDK for any supported programming language\.  

```
/* Gets the length of the plaintext that the session processed */
size_t ciphertext_len_output;
if (AWS_OP_SUCCESS != aws_cryptosdk_session_process_full(session,
                                  ciphertext_output,
                                  ciphertext_buf_sz_output,
                                  &ciphertext_len_output,
                                  plaintext_input,
                                  plaintext_len_input)) {
    aws_cryptosdk_session_destroy(session);
    return 8;
}
```

Step 6: Clean up the session\.  
The final step destroys the session including references to the CMM and the keyring\.  
If you prefer, instead of destroying the session, you can reuse the session with the same keyring and CMM to decrypt the string, or to encrypt or decrypt other messages\. To use the session for decrypting, use the `aws_cryptosdk_session_reset` method to change the mode to `AWS_CRYPTOSDK_DECRYPT`\.

### Decrypt a string<a name="c-example-string-decrypt"></a>

The second part of this example decrypts an encrypted message that contains the ciphertext of the original string\. 

Step 1: Load error strings\.  
Call the `aws_cryptosdk_load_error_strings()` method in your C or C\+\+ code\. It loads error information that is very useful for debugging\.  
You only need to call it once, such as in your `main` method\.  

```
/* Load error strings for debugging */
aws_cryptosdk_load_error_strings();
```

Step 2: Construct the keyring\.  
When you decrypt data in AWS KMS, you pass in the [encrypted message](concepts.md#message) that the encrypt API returned\. The [Decrypt API](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) doesn't take an AWS KMS key as input\. Instead, AWS KMS uses the same AWS KMS key to decrypt the ciphertext that it used to encrypt it\. However, the AWS Encryption SDK lets you specify an AWS KMS keyring with AWS KMS keys on encrypt and decrypt\.  
On decrypt, you can configure a keyring with only the AWS KMS keys that you want to use to decrypt the encrypted message\. For example, you might want to create a keyring with only the AWS KMS key that is used by a particular role in your organization\. The AWS Encryption SDK will never use an AWS KMS key unless it appears in the decryption keyring\. If the SDK can't decrypt the encrypted data keys by using the AWS KMS keys in the keyring that you provide, either because none of AWS KMS keys in the keyring were used to encrypt any of the data keys, or because the caller doesn't have permission to use the AWS KMS keys in the keyring to decrypt, the decrypt call fails\.  
When you specify an AWS KMS key for a decryption keyring, you must use its [key ARN](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#key-id-key-ARN)\. [Alias ARNs](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#key-id-alias-arn) are permitted only in encryption keyrings\. For help identifying the AWS KMS keys in an AWS KMS keyring, see [Identifying AWS KMS keys in an AWS KMS keyring](use-kms-keyring.md#kms-keyring-id)\.  
In this example, we specify a keyring configured with the same AWS KMS key used to encrypt the string\. Before running this code, replace the example key ARN with a valid one\.  

```
const char * key_arn = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab"    

struct aws_cryptosdk_keyring *kms_keyring =
        Aws::Cryptosdk::KmsKeyring::Builder().Build(key_arn);
```

Step 3: Create a session\.  
Create a session using the allocator and the keyring\. To configure the session for decryption, configure the session with the `AWS_CRYPTOSDK_DECRYPT` mode\.   
After you create a session with a keyring, you can release your reference to the keyring using the method that the SDK provides\. The session retains a reference to the keyring object during its lifetime, and both the session and keyring are released when you destroy the session\. This reference counting technique helps to prevent memory leaks and to prevent the objects from being released while they are in use\.  

```
struct aws_cryptosdk_session *session =	
	aws_cryptosdk_session_new_from_keyring_2(alloc, AWS_CRYPTOSDK_DECRYPT, kms_keyring);

/* When you add the keyring to the session, release the keyring object */
aws_cryptosdk_keyring_release(kms_keyring);
```

Step 4: Decrypt the string\.  
To decrypt the string, use the `aws_cryptosdk_session_process_full` method with the session that is configured for decryption\. This method, introduced in AWS Encryption SDK versions 1\.9\.*x* and 2\.2\.*x*, is designed for non\-streaming encryption and decryption\. To handle streaming data, call the `aws_cryptosdk_session_process` in a loop\.  
When decrypting, the ciphertext fields are input fields and the plaintext fields are output fields\. The `ciphertext_input` field holds the [encrypted message](message-format.md) that the encrypt method returned\. When the processing is complete, the `plaintext_output` field contains the plaintext \(decrypted\) string\.  

```
size_t plaintext_len_output;

if (AWS_OP_SUCCESS != aws_cryptosdk_session_process_full(session,
                                  plaintext_output,
                                  plaintext_buf_sz_output,
                                  &plaintext_len_output,
                                  ciphertext_input,
                                  ciphertext_len_input)) {
    aws_cryptosdk_session_destroy(session);
    return 13;
}
```

Step 5: Verify the encryption context\.  
Be sure that the actual encryption context — the one that was used to decrypt the message — contains the encryption context that you provided when encrypting the message\. The actual encryption context might include extra pairs, because the [cryptographic materials manager](concepts.md#crypt-materials-manager) \(CMM\) can add pairs to the provided encryption context before encrypting the message\.  
In the AWS Encryption SDK for C, you are not required to provide an encryption context when decrypting because the encryption context is included in the encrypted message that the SDK returns\. But, before it returns the plaintext message, your decrypt function should verify that all pairs in the provided encryption context appear in the encryption context that was used to decrypt the message\.  
First, get a read\-only pointer to the hash table in the session\. This hash table contains the encryption context that was used to decrypt the message\.   

```
const struct aws_hash_table *session_enc_ctx = aws_cryptosdk_session_get_enc_ctx_ptr(session);
```
Then, loop through the encryption context in the `my_enc_ctx` hash table that you copied when encrypting\. Verify that each pair in the `my_enc_ctx` hash table that was used to encrypt appears in the `session_enc_ctx` hash table that was used to decrypt\. If any key is missing, or that key has a different value, stop processing and write an error message\.  

```
for (struct aws_hash_iter iter = aws_hash_iter_begin(my_enc_ctx); !aws_hash_iter_done(&iter);
      aws_hash_iter_next(&iter)) {
     struct aws_hash_element *session_enc_ctx_kv_pair;
     aws_hash_table_find(session_enc_ctx, iter.element.key, &session_enc_ctx_kv_pair)

    if (!session_enc_ctx_kv_pair ||
        !aws_string_eq(
            (struct aws_string *)iter.element.value, (struct aws_string *)session_enc_ctx_kv_pair->value)) {
        fprintf(stderr, "Wrong encryption context!\n");
        abort();
    }
}
```

Step 6: Clean up the session\.  
After you verify the encryption context, you can destroy the session, or reuse it\. If you need to reconfigure the session, use the `aws_cryptosdk_session_reset` method\.  

```
aws_cryptosdk_session_destroy(session);
```