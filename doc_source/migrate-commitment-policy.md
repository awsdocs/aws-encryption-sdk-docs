# Setting your commitment policy<a name="migrate-commitment-policy"></a>

[Key commitment](concepts.md#key-commitment) assures that your encrypted data always decrypts to the same plaintext\. To provide this security property, beginning in version 1\.7\.*x*, AWS Encryption SDK uses new [algorithm suites](supported-algorithms.md) with key commitment\. To determine whether your data is encrypted and decrypted with key commitment, use the [commitment policy](concepts.md#commitment-policy) configuration setting\. Encrypting and decrypting data with key commitment is an [AWS Encryption SDK best practice](best-practices.md)\.

Setting a commitment policy is an important step in migrating from version 1\.7\.*x* of the AWS Encryption SDK to 2\.0\.*x* and later\. After setting and changing your commitment policy, be sure to test your application thoroughly before deploying it in production\. For migration guidance, see [How to migrate and deploy the AWS Encryption SDK](migration-guide.md)\.

The commitment policy setting has three valid values in version 2\.0\.*x*\. In version 1\.7\.*x*, only `ForbidEncryptAllowDecrypt` is valid\.
+ `ForbidEncryptAllowDecrypt` — The AWS Encryption SDK cannot encrypt with key commitment\. It can decrypt ciphertexts encrypted with or without key commitment\. 

  In version 1\.7\.*x*, this is the only valid value\. It ensures that you don't encrypt with key commitment until you are fully prepared to decrypt with key commitment\. Setting the value explicitly prevents your commitment policy from changing automatically to `require-encrypt-require-decrypt` when you upgrade to version 2\.1\.*x*\. Instead, you can [migrate your commitment policy](#migrate-commitment-policy) in stages\.
+ `RequireEncryptAllowDecrypt` — The AWS Encryption SDK must encrypt with key commitment\. It can decrypt ciphertexts encrypted with or without key commitment\. This value is added in version 2\.0\.*x*\.
+ `RequireEncryptRequireDecrypt` — The AWS Encryption SDK must encrypt with key commitment\. It only decrypts ciphertexts with key commitment\. This value is added in version 2\.0\.*x*\. It is the default value in version 2\.0\.*x*\.

In version 1\.7\.*x*, the only valid commitment policy value is `ForbidEncryptAllowDecrypt`\. After you migrate to version 2\.0\.*x*, you can [change your commitment policy in stages](migration-guide.md) as you are ready\. Don't update your commitment policy to `RequireEncryptRequireDecrypt` until you are certain that you don't have any messages encrypted without key commitment\. 

These examples show you how to set your commitment policy in versions 1\.7\.*x* and 2\.0\.*x*\. The technique depends on your programming language\. 

**Learn more about migration**

For AWS Encryption SDK for Java, AWS Encryption SDK for Python, and the AWS Encryption CLI, learn about required changes to master key providers in [Updating AWS KMS master key providers](migrate-mkps-v2.md)\.

For AWS Encryption SDK for C and AWS Encryption SDK for JavaScript, learn about an optional update to keyrings in [Updating AWS KMS keyrings](migrate-keyrings-v2.md)\.

## How to set your commitment policy<a name="migrate-commitment-step1"></a>

The technique that you use to set your commitment policy differs slightly with each language implementation\. These examples show you how to do it\. Before changing your commitment policy, review the multi\-stage approach in [How to migrate and deploy](migration-guide.md)\. 

------
#### [ C ]

Beginning in version 1\.7\.*x* of the AWS Encryption SDK for C, you use the `aws_cryptosdk_session_set_commitment_policy` function to set the commitment policy on your encrypt and decrypt sessions\. The commitment policy that you set applies to all encrypt and decrypt operations called on that session\.

The `aws_cryptosdk_session_new_from_keyring` and `aws_cryptosdk_session_new_from_cmm` functions are deprecated in version 1\.7\.*x* and removed in version 2\.0\.*x*\. These functions are replaced by `aws_cryptosdk_session_new_from_keyring_2` and `aws_cryptosdk_session_new_from_cmm_2` functions that return a session\.

When you use the `aws_cryptosdk_session_new_from_keyring_2` and `aws_cryptosdk_session_new_from_cmm_2` in version 1\.7\.*x*, you are required to call the `aws_cryptosdk_session_set_commitment_policy` function with the `COMMITMENT_POLICY_FORBID_ENCRYPT_ALLOW_DECRYPT` commitment policy value\. In version 2\.0\.*x*, calling this function is optional and it takes all valid values\. The default commitment policy for version 2\.0\.*x* and later is `COMMITMENT_POLICY_REQUIRE_ENCRYPT_REQUIRE_DECRYPT`\.

For a complete example, see [string\.cpp](https://github.com/aws/aws-encryption-sdk-c/blob/master/examples/string.cpp)\.

```
// Create an AWS KMS keyring
const char * key_arn = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";
struct aws_cryptosdk_keyring *kms_keyring = Aws::Cryptosdk::KmsKeyring::Builder().Build(key_arn);

// Create an encrypt session with a CommitmentPolicy setting
struct aws_cryptosdk_session *encrypt_session = aws_cryptosdk_session_new_from_keyring_2(
    alloc, AWS_CRYPTOSDK_ENCRYPT, kms_keyring);

aws_cryptosdk_keyring_release(kms_keyring);
aws_cryptosdk_session_set_commitment_policy(encrypt_session,
    COMMITMENT_POLICY_FORBID_ENCRYPT_ALLOW_DECRYPT);

...
// Encrypt your data 

size_t plaintext_consumed_output;
aws_cryptosdk_session_process(encrypt_session,
                              ciphertext_output,
                              ciphertext_buf_sz_output,
                              ciphertext_len_output,
                              plaintext_input,
                              plaintext_len_input,
                              &plaintext_consumed_output)
...

// Create a decrypt session with a CommitmentPolicy setting

struct aws_cryptosdk_keyring *kms_keyring = Aws::Cryptosdk::KmsKeyring::Builder().Build(key_arn);
struct aws_cryptosdk_session *decrypt_session = *aws_cryptosdk_session_new_from_keyring_2(
        alloc, AWS_CRYPTOSDK_DECRYPT, kms_keyring);
aws_cryptosdk_keyring_release(kms_keyring);
aws_cryptosdk_session_set_commitment_policy(decrypt_session,
        COMMITMENT_POLICY_FORBID_ENCRYPT_ALLOW_DECRYPT);

// Decrypt your ciphertext
size_t ciphertext_consumed_output;
aws_cryptosdk_session_process(decrypt_session,
                              plaintext_output,
                              plaintext_buf_sz_output,
                              plaintext_len_output,
                              ciphertext_input,
                              ciphertext_len_input,
                              &ciphertext_consumed_output)
```

------
#### [ AWS Encryption CLI ]

To set a commitment policy in the AWS Encryption CLI, use the `--commitment-policy` parameter\. This parameter is introduced in version 1\.8\.*x*\. 

In version 1\.8\.*x*, when you use the `--wrapping-keys` parameter in an `--encrypt` or `--decrypt` command, a `--commitment-policy` parameter with the `forbid-encrypt-allow-decrypt` value is required\. Otherwise, the `--commitment-policy` parameter is invalid\.

In version 2\.1\.*x*, the `--commitment-policy` parameter is optional and defaults to the `require-encrypt-require-decrypt` value, which won't encrypt or decrypt any ciphertext encrypted without key commitment\. However, we recommend that you set the commitment policy explicitly in all encrypt and decrypt calls to help with maintenance and troubleshooting\.

This example sets the commitment policy\. It also uses the `--wrapping-keys` parameter that replaces the `--master-keys` parameter beginning in version 1\.8\.*x*\. For details, see [Updating AWS KMS master key providers](migrate-mkps-v2.md)\. For complete examples, see [Examples of the AWS Encryption CLI](crypto-cli-examples.md)\.

```
\\ To run this example, replace the fictitious key ARN with a valid value. 
$ cmkArn=arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab

\\ Encrypt your plaintext data - no change to algorithm suite used
$ aws-encryption-cli --encrypt \
                     --input hello.txt \
                     --wrapping-keys key=$cmkArn \
                     --commitment-policy forbid-encrypt-allow-decrypt \
                     --metadata-output ~/metadata \
                     --encryption-context purpose=test \
                     --output .

\\ Decrypt your ciphertext - supports key commitment on 1.7 and later
$ aws-encryption-cli --decrypt \
                     --input hello.txt.encrypted \
                     --wrapping-keys key=$cmkArn \
                     --commitment-policy forbid-encrypt-allow-decrypt \
                     --encryption-context purpose=test \
                     --metadata-output ~/metadata \
                     --output .
```

------
#### [ Java ]

Beginning in version 1\.7\.*x* of the AWS Encryption SDK for Java, you set the commitment policy on your instance of `AwsCrypto`, the object that represents the AWS Encryption SDK client\. This commitment policy setting applies to all encrypt and decrypt operations called on that client\.

The `AwsCrypto()` constructor is deprecated in 1\.7\.*x* and removed in 2\.0\.*x*\. It's replaced by a new `Builder` class, a `Builder.withCommitmentPolicy()` method, and the `CommitmentPolicy` enumerated type\. 

In version 1\.7\.*x*, the `Builder` class requires the `Builder.withCommitmentPolicy()` method and the `CommitmentPolicy.ForbidEncryptAllowDecrypt` argument\. Beginning in version 2\.0\.*x*, the `Builder.withCommitmentPolicy()` method is optional; the default value is `CommitmentPolicy.RequireEncryptRequireDecrypt`\.

For a complete example, see [SetCommitmentPolicyExample\.java](https://github.com/aws/aws-encryption-sdk-java/blob/master/src/examples/java/com/amazonaws/crypto/examples/SetCommitmentPolicyExample.java)\.

```
// Instantiate the client
final AwsCrypto crypto = AwsCrypto.builder()
    .withCommitmentPolicy(CommitmentPolicy.ForbidEncryptAllowDecrypt)
    .build();

// Create a master key provider in strict mode
String awsKmsCmk = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";

KmsMasterKeyProvider masterKeyProvider = KmsMasterKeyProvider.builder()
    .buildStrict(awsKmsCmk);

// Encrypt your plaintext data
CryptoResult<byte[], KmsMasterKey> encryptResult = crypto.encryptData(
    masterKeyProvider,
    sourcePlaintext,
    encryptionContext);
byte[] ciphertext = encryptResult.getResult();

// Decrypt your ciphertext
CryptoResult<byte[], KmsMasterKey> decryptResult = crypto.decryptData(
        masterKeyProvider,
        ciphertext);
byte[] decrypted = decryptResult.getResult();
```

------
#### [ JavaScript ]

Beginning in version 1\.7\.*x* of the AWS Encryption SDK for JavaScript, you can set the commitment policy when you call the new `buildClient` function that instantiates an AWS Encryption SDK client\. The `buildClient` function takes an enumerated value that represents your commitment policy\. It returns updated `encrypt` and `decrypt` functions that enforce your commitment policy when you encrypt and decrypt\.

In version 1\.7\.*x*, the `buildClient` function requires the `CommitmentPolicy.FORBID_ENCRYPT_ALLOW_DECRYPT` argument\. Beginning in version 2\.0\.*x*, the commitment policy argument is optional and the default value is `CommitmentPolicy.REQUIRE_ENCRYPT_REQUIRE_DECRYPT`\.

The code for Node\.js and the browser are identical for this purpose, except that browser needs a statement to set credentials\. 

The following example encrypts data with an AWS KMS keyring\. The new `buildClient` function sets the commitment policy to the 1\.7\.*x* default value\. The upgraded `encrypt` and `decrypt` functions that `buildClient` returns enforce the commitment policy you set\. 

```
import { buildClient } from '@aws-crypto/client-node'
const { encrypt, decrypt } = buildClient(CommitmentPolicy.FORBID_ENCRYPT_ALLOW_DECRYPT)

// Create an AWS KMS keyring
const generatorKeyId = 'arn:aws:kms:us-west-2:111122223333:alias/ExampleAlias'
const keyIds = ['arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab']
const keyring = new KmsKeyringNode({ generatorKeyId, keyIds })

// Encrypt your plaintext data
const { ciphertext } = await encrypt(keyring, plaintext, { encryptionContext: context })

// Decrypt your ciphertext
const { decrypted, messageHeader } = await decrypt(keyring, ciphertext)
```

------
#### [ Python ]

Beginning in version 1\.7\.*x* of the AWS Encryption SDK for Python, you set the commitment policy on your instance of `EncryptionSDKClient`, a new object that represents the AWS Encryption SDK client\. The commitment policy that you set applies to all `encrypt` and `decrypt` calls that use that instance of the client\.

In version 1\.7\.*x*, the `EncryptionSDKClient` constructor requires the `CommitmentPolicy.ForbidEncryptAllowDecrypt` enumerated value\. Beginning in version 2\.0\.*x*, the commitment policy argument is optional and the default value is `CommitmentPolicy.RequireEncryptRequireDecrypt`\.

This example uses the new `EncryptionSDKClient` constructor and sets the commitment policy to the 1\.7\.*x* default value\. The constructor instantiates a client that represents the AWS Encryption SDK\. When you call the `encrypt`, `decrypt`, or `stream` methods on this client, they enforce the commitment policy that you set\. This example also uses the new constructor for the `StrictAwsKmsMasterKeyProvider` class, which specifies AWS KMS CMKs when encrypting and decrypting\. 

For a complete example, see [set\_commitment\.py](https://github.com/aws/aws-encryption-sdk-python/blob/master/examples/src/set_commitment.py)\.

```
# Instantiate the client
client = EncryptionSDKClient(CommitmentPolicy.ForbidEncryptAllowDecrypt)

// Create a master key provider in strict mode
aws_kms_cmk = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab"
aws_kms_strict_master_key_provider = StrictAwsKmsMasterKeyProvider(
        key_ids=[aws_kms_cmk]
)

# Encrypt your plaintext data
ciphertext, encrypt_header = client.encrypt(
        source=source_plaintext,
        encryption_context=encryption_context,
        master_key_provider=aws_kms_strict_master_key_provider
)

# Decrypt your ciphertext
decrypted, decrypt_header = client.decrypt(
        source=ciphertext,
        master_key_provider=aws_kms_strict_master_key_provider
)
```

------