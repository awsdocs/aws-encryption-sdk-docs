# Multi\-keyrings<a name="use-multi-keyring"></a>

You can combine keyrings into a multi\-keyring\. A *multi\-keyring* is a keyring that consists of one or more individual keyrings of the same or a different type\. The effect is like using several keyrings in a series\. When you use a multi\-keyring to encrypt data, any of the wrapping keys in any of its keyrings can decrypt that data\.

When you create a multi\-keyring to encrypt data, you designate one of the keyrings as the *generator keyring*\. All other keyrings are known as *child keyrings*\. The generator keyring generates and encrypts the plaintext data key\. Then, all of the wrapping keys in all of the child keyrings encrypt the same plaintext data key\. The multi\-keyring returns the plaintext key and one encrypted data key for each wrapping key in the multi\-keyring\. If you create a multi\-keyring with no generator keyring, you can use it to decrypt data, but not to encrypt\. If the generator keyring is a [KMS keyring](use-kms-keyring.md), the generator key in the AWS KMS keyring generates and encrypts the plaintext key\. Then, all additional AWS KMS keys in the AWS KMS keyring, and all wrapping keys in all child keyrings in the multi\-keyring, encrypt the same plaintext key\. 

When decrypting, the AWS Encryption SDK uses the keyrings to try to decrypt one of the encrypted data keys\. The keyrings are called in the order that they are specified in the multi\-keyring\. Processing stops as soon as any key in any keyring can decrypt an encrypted data key\. 

Beginning in [version 1\.7\.*x*](about-versions.md#version-1.7), when an encrypted data key is encrypted under an AWS Key Management Service \(AWS KMS\) keyring \(or master key provider\), the AWS Encryption SDK always passes the key ARN of the AWS KMS key to the `KeyId` parameter of the AWS KMS [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operation\. This is an AWS KMS best practice that assures that you decrypt the encrypted data key with the wrapping key you intend to use\.

To see a working example of a multi\-keyring, see:
+ C: [multi\_keyring\.cpp](https://github.com/aws/aws-encryption-sdk-c/blob/master/examples/multi_keyring.cpp)[]()
+ JavaScript Node\.js: [multi\_keyring\.ts](https://github.com/aws/aws-encryption-sdk-javascript/blob/master/modules/example-node/src/multi_keyring.ts)
+ JavaScript Browser: [multi\_keyring\.ts](https://github.com/aws/aws-encryption-sdk-javascript/blob/master/modules/example-browser/src/multi_keyring.ts)

To create a multi\-keyring, first instantiate the child keyrings\. In this example, we use an AWS KMS keyring and a Raw AES keyring, but you can combine any supported keyrings in a multi\-keyring\.

------
#### [ C ]

```
/* Define an AWS KMS keyring. For details, see [string\.cpp](https://github.com/aws/aws-encryption-sdk-c/blob/master/examples/string.cpp) */
struct aws_cryptosdk_keyring *kms_keyring = Aws::Cryptosdk::KmsKeyring::Builder().Build(example_key);

// Define a Raw AES keyring. For details, see [raw\_aes\_keyring\.c](https://github.com/aws/aws-encryption-sdk-c/blob/master/examples/raw_aes_keyring.c) */
struct aws_cryptosdk_keyring *aes_keyring = aws_cryptosdk_raw_aes_keyring_new(
        alloc, wrapping_key_namespace, wrapping_key_name, wrapping_key, AWS_CRYPTOSDK_AES256);
```

------
#### [ JavaScript Browser ]

```
const clientProvider = getClient(KMS, { credentials })

// Define an AWS KMS keyring. For details, see [kms\_simple\.ts](https://github.com/aws/aws-encryption-sdk-javascript/blob/master/modules/example-browser/src/kms_simple.ts). 
const kmsKeyring = new KmsKeyringBrowser({ generatorKeyId: exampleKey })

// Define a Raw AES keyring. For details, see [aes\_simple\.ts](https://github.com/aws/aws-encryption-sdk-javascript/blob/master/modules/example-browser/src/aes_simple.ts).
const aesKeyring = new RawAesKeyringWebCrypto({ keyName, keyNamespace, wrappingSuite, masterKey })
```

------
#### [ JavaScript Node\.js ]

```
// Define an AWS KMS keyring. For details, see [kms\_simple\.ts](https://github.com/aws/aws-encryption-sdk-javascript/blob/master/modules/example-node/src/kms_simple.ts). 
const kmsKeyring = new KmsKeyringNode({ generatorKeyId: exampleKey })

// Define a Raw AES keyring. For details, see [raw\_aes\_keyring\_node\.ts](https://github.com/aws/aws-encryption-sdk-javascript/blob/master/modules/raw-aes-keyring-node/src/raw_aes_keyring_node.ts).
const aesKeyring = new RawAesKeyringNode({ keyName, keyNamespace, wrappingSuite, unencryptedMasterKey })
```

------

Next, create the multi\-keyring and specify its generator keyring, if any\. In this example, we create a multi\-keyring in which the AWS KMS keyring is the generator keyring and the AES keyring is the child keyring\.

------
#### [ C ]

In the multi\-keyring constructor in C, you specify only its generator keyring\.

```
struct aws_cryptosdk_keyring *multi_keyring = aws_cryptosdk_multi_keyring_new(alloc, kms_keyring);
```

To add a child keyring to your multi\-keyring, use the `aws_cryptosdk_multi_keyring_add_child` method\. You need to call the method once for each child keyring that you add\. 

```
// Add the Raw AES keyring (C only)
aws_cryptosdk_multi_keyring_add_child(multi_keyring, aes_keyring);
```

------
#### [ JavaScript Browser ]

JavaScript multi\-keyrings are immutable\. The JavaScript multi\-keyring constructor lets you specify the generator keyring and multiple child keyrings\. 

```
const clientProvider = getClient(KMS, { credentials })

const multiKeyring = new MultiKeyringWebCrypto(generator: kmsKeyring, children: [aesKeyring]);
```

------
#### [ JavaScript Node\.js ]

JavaScript multi\-keyrings are immutable\. The JavaScript multi\-keyring constructor lets you specify the generator keyring and multiple child keyrings\. 

```
const multiKeyring = new MultiKeyringNode(generator: kmsKeyring, children: [aesKeyring]);
```

------

Now, you can use the multi\-keyring to encrypt and decrypt data\. 

For a complete example of how to build and use a multi\-keyring in C, see [multi\_keyring\.cpp](https://github.com/aws/aws-encryption-sdk-c/blob/master/examples/multi_keyring.cpp)\.