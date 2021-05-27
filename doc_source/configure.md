# Configuring the AWS Encryption SDK<a name="configure"></a>

The AWS Encryption SDK is designed to be easy to use\. Although the AWS Encryption SDK has several configuration options, the default values are carefully chosen to be practical and secure for most applications\. However, you might need to adjust your configuration to improve performance or include a custom feature in your design\. 

When configuring your implementation, review the AWS Encryption SDK [best practices](best-practices.md) and implement as many as you can\.

**Topics**
+ [Select a programming language](#config-language)
+ [Select wrapping keys](#config-keys)
+ [Examine the security features](#config-security)
+ [Review other options](#config-other)

## Select a programming language<a name="config-language"></a>

The AWS Encryption SDK is available in multiple [programming languages](programming-languages.md)\. The language implementations are designed to be fully interoperable and to offer the same features, although they might be implemented in different ways\. Typically, you use the library that is compatible with your application\. However, you might select a programming language for a particular implementation\. For example, if you prefer to work with [keyrings](choose-keyring.md), you might choose the AWS Encryption SDK for C or the AWS Encryption SDK for JavaScript\. 

## Select wrapping keys<a name="config-keys"></a>

The AWS Encryption SDK generates a unique symmetric data key to encrypt each message\. Unless you are using [data key caching](data-key-caching.md), you don't need to configure, manage, or use the data keys\. The AWS Encryption SDK does it for you\.

However, you must select one or more wrapping keys to encrypt each data key\. The AWS Encryption SDK supports AES symmetric keys and RSA asymmetric keys in different sizes\. It also supports [AWS Key Management Service](https://docs.aws.amazon.com/kms/latest/developerguide/) \(AWS KMS\) symmetric customer master keys\. You are responsible for the safety and durability of your wrapping keys, so we recommend that you use an encryption key in a hardware security module or a key infrastructure service, such as AWS KMS\. 

To specify your wrapping keys for encryption and decryption, you use a keyring \(C and JavaScript\) or a master key provider \(Java and Python\)\. You can specify one wrapping key or multiple wrapping keys of the same or different types\. If you use multiple wrapping keys to wrap a data key, each wrapping key will encrypt a copy of the same data key\. The encrypted data keys \(one per wrapping key\) are stored with the encrypted data in the encrypted message that the AWS Encryption SDK returns\. To decrypt the data, the AWS Encryption SDK must first use one of your wrapping keys to decrypt an encrypted data key\. 

## Examine the security features<a name="config-security"></a>

The AWS Encryption SDK supports the following security features\. Although they aren't required, we recommend that you use them whenever it's practical\.

### Choosing an algorithm suite<a name="config-algorithm"></a>

The AWS Encryption SDK supports several [symmetric and asymmetric encryption algorithms](concepts.md#symmetric-key-encryption) for encrypting your data keys under the wrapping keys you specify\. However, when it uses those data keys to encrypt your data, the AWS Encryption SDK defaults to a [recommended algorithm suite](supported-algorithms.md#recommended-algorithms) that uses the AES\-GCM algorithm with key derivation, digital signatures, and key commitment\. Although this algorithm suite is likely to be suitable for most applications, you can choose an alternate algorithm suite\. For example, some trust models would be satisfied by an algorithm suite without [digital signatures](concepts.md#digital-sigs)\. 

 For information about the algorithm suites that the AWS Encryption SDK supports, see [Supported algorithm suites in the AWS Encryption SDK](supported-algorithms.md)\.

### Limiting encrypted data keys<a name="config-limit-keys"></a>

You can limit the number of encrypted data keys in an encrypted message\. This best practice feature can help you detect a misconfigured keyring when encrypting or a malicious ciphertext when decrypting\. It also prevents unnecessary, expensive, and potentially exhaustive calls to your key infrastructure\. Limiting encrypted data keys is most valuable when you are decrypting messages from an untrusted source\. 

Although most encrypted messages have one encrypted data key for each wrapping key used in the encryption, an encrypted message can contain up to 65,535 encrypted data keys\. A malicious actor might construct an encrypted message with thousands of encrypted data keys, none of which can be decrypted\. As a result, the AWS Encryption SDK would attempt to decrypt each encrypted data key until it exhausted the encrypted data keys in the message\.

To limit encrypted data keys, use the `MaxEncryptedDataKeys` parameter\. This parameter is available for all supported programming languages beginning in versions 1\.9\.*x* and 2\.2\.*x* of the AWS Encryption SDK\. It is optional and valid when encrypting and decrypting\. The following examples decrypt data that was encrypted under three different wrapping keys\. The `MaxEncryptedDataKeys` value is set to 3\.

------
#### [ C ]

```
// Construct an AWS KMS keyring
struct aws_cryptosdk_keyring *kms_keyring = 
      Aws::Cryptosdk::KmsKeyring::Builder().Build(key_arn1, { key_arn2, key_arn3 });

// Create a session
struct aws_cryptosdk_session *session = 
    aws_cryptosdk_session_new_from_keyring_2(alloc, AWS_CRYPTOSDK_DECRYPT, kms_keyring);
aws_cryptosdk_keyring_release(kms_keyring);  

// Limit encrypted data keys
aws_cryptosdk_session_set_max_encrypted_data_keys(session, 3);
  
// Decrypt
size_t ciphertext_consumed_output;
aws_cryptosdk_session_process(session,
    plaintext_output,
    plaintext_buf_sz_output,
    plaintext_len_output,
    ciphertext_input,
    ciphertext_len_input,
    &ciphertext_consumed_output);
assert(aws_cryptosdk_session_is_done(session));
assert(ciphertext_consumed == ciphertext_len);
```

------
#### [ CLI ]

```
# Decrypt with limited encrypted data keys

$ aws-encryption-cli --decrypt \
    --input hello.txt.encrypted \
    --wrapping-keys key=$key_arn1 key=$key_arn2 key=$key_arn3 \
    --buffer \
    --max-encrypted-data-keys 3 \
    --encryption-context purpose=test \
    --metadata-output ~/metadata \
    --output .
```

------
#### [ Java ]

```
// Construct a client with limited encrypted data keys
final AwsCrypto crypto = AwsCrypto.builder()
  .withMaxEncryptedDataKeys(3)
  .build();

// Create an AWS KMS master key provider
final KmsMasterKeyProvider keyProvider = KmsMasterKeyProvider.builder()
      .buildStrict(keyArn1, keyArn2, keyArn3);
   
// Decrypt
final CryptoResult<byte[], KmsMasterKey> decryptResult = crypto.decryptData(keyProvider, ciphertext)
```

------
#### [ JavaScript Browser ]

```
// Construct a client with limited encrypted data keys
const { encrypt, decrypt } = buildClient({ maxEncryptedDataKeys: 3 })

declare const credentials: {
  accessKeyId: string
  secretAccessKey: string
  sessionToken: string
}
const clientProvider = getClient(KMS, {
  credentials: { accessKeyId, secretAccessKey, sessionToken }
})

// Create an AWS KMS keyring
const keyring = new KmsKeyringBrowser({
  clientProvider,
  keyIds: [keyArn1, keyArn2, keyArn3],
})

// Decrypt
const { plaintext, messageHeader } = await decrypt(keyring, ciphertext)
```

------
#### [ JavaScript Node\.js ]

```
// Construct a client with limited encrypted data keys
const { encrypt, decrypt } = buildClient({ maxEncryptedDataKeys: 3 })

// Create an AWS KMS keyring
const keyring = new KmsKeyringBrowser({
  keyIds: [keyArn1, keyArn2, keyArn3],
})
  
// Decrypt
const { plaintext, messageHeader } = await decrypt(keyring, ciphertext)
```

------
#### [ Python ]

```
# Instantiate a client with limited encrypted data keys
client = aws_encryption_sdk.EncryptionSDKClient(max_encrypted_data_keys=3)

# Create an AWS KMS master key provider
master_key_provider = aws_encryption_sdk.StrictAwsKmsMasterKeyProvider(
    key_ids=[key_arn1, key_arn2, key_arn3])

# Decrypt
plaintext, header = client.decrypt(source=ciphertext, key_provider=master_key_provider)
```

------

### Working with streaming data<a name="config-stream"></a>

When you stream data for decryption, be aware that the AWS Encryption SDK returns decrypted plaintext after the integrity checks are complete, but before the digital signature is verified\. To ensure that you don't return or use plaintext until the signature is verified, we recommend that you buffer the streamed plaintext until the entire decryption process is complete\. 

This issue arises only when you are streaming ciphertext for decryption, and only when you are using an algorithm suite, such as the [default algorithm suite](supported-algorithms.md), that includes [digital signatures](concepts.md#digital-sigs)\. 

To make the buffering easier, some AWS Encryption SDK language implementations, such as AWS Encryption SDK for JavaScript in Node\.js, include a buffering feature as part of the decrypt method\. The AWS Encryption CLI, which always streams input and output introduced a `--buffer` parameter in versions 1\.9\.*x* and 2\.2\.*x*\. In other language implementations, you can use existing buffering features\.

If you are using an algorithm suite without digital signatures, be sure to use the `decrypt-unsigned` feature in each language implementation\. This feature decrypts ciphertext but fails if it encounters signed ciphertext\. For details, see [Choosing an algorithm suite](#config-algorithm)\. 

## Review other options<a name="config-other"></a>

The AWS Encryption SDK supports following configuration options\. Before using these options, consider the risks and benefits and determine whether the option is right for your application\.

### Data key caching<a name="config-caching"></a>

In general, reusing data keys is discouraged, but the AWS Encryption SDK offers a [data key caching](data-key-caching.md) option that provides limited reuse of data keys\. Data key caching can improve the performance of some applications and reduce calls to your key infrastructure\. Before using data key caching in production, adjust the [security thresholds](thresholds.md), and test to make sure that the benefits outweigh the disadvantages of reusing data keys\. 