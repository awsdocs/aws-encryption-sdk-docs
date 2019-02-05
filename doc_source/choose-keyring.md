# Choosing a Keyring<a name="choose-keyring"></a>


|  | 
| --- |
|   The AWS Encryption SDK for C is a preview release\. The code and the documentation are subject to change\.  | 

The AWS Encryption SDK for C uses *keyrings* to help it perform [envelope encryption](https://docs.aws.amazon.com/crypto/latest/userguide/cryptography-concepts.html#define-envelope-encryption)\. A keyring is used to generate, encrypt, and decrypt data keys\. The keyring that you use determines the source of the unique data keys that protect each message, and the wrapping keys that encrypt that data key\. You can use the keyrings that the SDK provides or write your own compatible custom keyrings\. 

You can use each keyring individually or combine keyrings into a [multi\-keyring](#use-multi-keyring)\. Although most keyrings can generate, encrypt, and decrypt data keys, you might create a keyring that performs only one particular operation, such as a keyring that only generates data keys, and use that keyring in combination with others\.

You specify a keyring for encrypting and decrypting data\. 
+ On encrypt, the keyring generates a unique plaintext data key\. Then, each of the wrapping keys in the keyring encrypts the same plaintext data key\. The keyring returns the plaintext data key and one encrypted data key for each wrapping key in the keyring\. The SDK uses the plaintext data key to encrypt the data, and it returns an [encrypted message](concepts.md#message) that includes the encrypted data and the encrypted data keys\.

   
+ On decrypt, the keyring that you specify must contain at least one wrapping key that can decrypt one of the encrypted data keys in the encrypted message\. Otherwise, the attempt to decrypt will fail\.

We recommend that you use a keyring that protects your master keys and performs cryptographic operations within a secure boundary, such as the KMS keyring, which uses [AWS Key Management Service](https://docs.aws.amazon.com/kms/latest/developerguide/) \(AWS KMS\) [customer master keys](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#master_keys) \(CMKs\) that never leave AWS KMS unencrypted\. You can also write a keyring that uses master keys that are stored in your hardware security modules \(HSMs\) or protected by other master key services\. 

This topic explains how to use the keyring feature of the AWS Encryption SDK\. For details about programming with the AWS Encryption SDK, see the [C examples](c-examples.md), the [examples](https://github.com/awslabs/aws-encryption-sdk-c/tree/master/examples) in the aws\-encryption\-sdk\-c repository on GitHub, and the [C API documentation](https://awslabs.github.io/aws-encryption-sdk-c/html/)\.

**Topics**
+ [Keyring Compatibility](#keyring-compatibility)
+ [KMS Keyring](#use-kms-keyring)
+ [Raw AES Keyring](#use-raw-aes-keyring)
+ [Raw RSA Keyring](#use-raw-rsa-keyring)
+ [Creating a Multi\-Keyring](#use-multi-keyring)

## Keyring Compatibility<a name="keyring-compatibility"></a>

Although the Java, Python, and C implementations of the AWS Encryption SDK have some architectural differences, they are designed to be fully compatible\. You can encrypt your data using one programming language implementation and decrypt it with any other language implementation\. However, you must use the same or corresponding master keys to encrypt and decrypt your data keys\. 

The following table shows which master keys and master key providers are compatible with the keyrings provided in the AWS Encryption SDK in C\. 


**Compatible Keyrings and Master Key Providers**  

| Keyring: C | Master Key Provider: Java and Python | 
| --- | --- | 
| [KMS keyring](#use-kms-keyring) |  [KMS master key \(Java\)](https://aws.github.io/aws-encryption-sdk-java/javadoc/com/amazonaws/encryptionsdk/kms/KmsMasterKey.html) [KMS master key provider \(Java\)](https://aws.github.io/aws-encryption-sdk-java/javadoc/com/amazonaws/encryptionsdk/kms/KmsMasterKeyProvider.html) [KMS master key \(Python\)](https://aws-encryption-sdk-python.readthedocs.io/en/latest/generated/aws_encryption_sdk.key_providers.kms.html) [KMS master key provider \(Python\)](https://aws-encryption-sdk-python.readthedocs.io/en/latest/generated/aws_encryption_sdk.key_providers.kms.html#aws_encryption_sdk.key_providers.kms.KMSMasterKeyProvider)  | 
| [Raw AES keyring](#use-raw-aes-keyring) | When they are used with symmetric encryption keys:[JceMasterKey](https://aws.github.io/aws-encryption-sdk-java/javadoc/com/amazonaws/encryptionsdk/jce/JceMasterKey.html) \(Java\)[RawMasterKey](https://aws-encryption-sdk-python.readthedocs.io/en/latest/generated/aws_encryption_sdk.key_providers.raw.html#aws_encryption_sdk.key_providers.raw.RawMasterKey) \(Python\) | 
| [Raw RSA keyring](#use-raw-rsa-keyring) | When they are used with asymmetric encryption keys:[JceMasterKey](https://aws.github.io/aws-encryption-sdk-java/javadoc/com/amazonaws/encryptionsdk/jce/JceMasterKey.html) \(Java\)[RawMasterKey](https://aws-encryption-sdk-python.readthedocs.io/en/latest/generated/aws_encryption_sdk.key_providers.raw.html#aws_encryption_sdk.key_providers.raw.RawMasterKey) \(Python\) | 

When decrypting, the Java and Python implementations behave like the [KMS Discovery keyring](#kms-keyring-discovery), that is, they don't limit the CMKs that can be used to decrypt the encrypted data keys\. Also, the Java and Python SDKs do not supply an equivalent for the [KMS Regional Discovery keyring](#kms-keyring-regional), although you can create your own custom keyrings\. 

## KMS Keyring<a name="use-kms-keyring"></a>

A KMS keyring uses AWS Key Management Service \(AWS KMS\) customer master keys \(CMKs\) to generate, encrypt, and decrypt data keys\. AWS KMS protects your master keys and performs cryptographic operations within the FIPS boundary\. We recommend that you use the KMS keyring, or a keyring with similar security properties, whenever possible\.

To use the CMKs in a KMS keyring, the caller must have the permission to perform the AWS KMS API calls required for each type of keyring operation, such as generating data keys, encrypting, or decrypting\. The required permissions are listed in the description of each type of KMS keyring\. For information about setting and changing AWS KMS permissions, see [Authentication and Access Control for AWS KMS](https://docs.aws.amazon.com/kms/latest/developerguide/control-access.html) in the *AWS Key Management Service Developer Guide*\.

**Topics**
+ [Encrypting with a KMS Keyring](#kms-keyring-encrypt)
+ [Decrypting with a KMS Keyring](#kms-keyring-decrypt)
+ [Using a KMS Discovery Keyring](#kms-keyring-discovery)
+ [Using a KMS Regional Discovery Keyring](#kms-keyring-regional)

### Encrypting with a KMS Keyring<a name="kms-keyring-encrypt"></a>

You can configure each KMS keyring with a single CMK or multiple CMKs in the same or different AWS accounts and AWS Regions\. As with all keyrings, you can use one or more KMS keyrings in a [multi\-keyring](#use-multi-keyring)\. 

When you use a KMS keyring with multiple CMKs to encrypt data, you specify a *generator key* that generates a plaintext data key and encrypts it\. Then, if you choose, you can specify additional CMKs that encrypt the same plaintext data key\. The [encrypted message](concepts.md#message) that the AWS Encryption SDK returns includes all of the encrypted data keys and the ciphertext\. The caller must have `kms:GenerateDataKey` permission for the generator CMK and `kms:Encrypt` permission for all additional CMKs\.

For example, in the following KMS keyring, CMK\_1 is the generator key and CMK\_2 is an additional key\. When you use this keyring to encrypt data, it returns one plaintext data key generated by the CMK\_1 and two encrypted data keys, one encrypted by using CMK\_1 and one encrypted by using CMK\_2\. To decrypt the data protected by this keyring, the keyring that you use must include either CMK\_1 or CMK\_2, or both\.

```
const char * CMK_1 = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab"
const char * CMK_2 = "arn:aws:kms:us-west-2:111122223333:key/0987dcba-09fe-87dc-65ba-ab0987654321"    

struct aws_cryptosdk_keyring *kms_encrypt_keyring = 
       Aws::Cryptosdk::KmsKeyring::Builder().Build(CMK_1,{CMK_2});
```

### Decrypting with a KMS Keyring<a name="kms-keyring-decrypt"></a>

Although the AWS KMS [Decrypt API](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) doesn't allow you to specify a CMK, you can specify a KMS keyring when decrypting data in the AWS Encryption SDK\. The decryption keyring limits the CMKs used for decryption to only those that you specify\. 

When decrypting with a KMS keyring, the AWS Encryption SDK searches the decryption keyring for a CMK that can decrypt one of the encrypted data keys\. If the keyring contains the CMK it needs, the SDK asks AWS KMS to decrypt the encrypted data key\. Otherwise, it skips to the next encrypted data key, if any\. To succeed, the caller must have `kms:Decrypt` permission on at least one of the CMKs that was used to encrypt a data key\.

The AWS Encryption SDK never attempts to decrypt an encrypted data key unless the CMK that encrypted that data key is included in the decryption keyring\. If the decryption keyring doesn't include any of the CMKs that encrypted any of the data keys, the AWS Encryption SDK fails the decrypt call without ever calling AWS KMS\.

A decrypt call with a KMS keyring succeeds when at least one CMK in the decryption keyring can decrypt one of the encrypted data keys in the [encrypted message](concepts.md#message)\. This behavior enables you to encrypt data under multiple CMKs in different AWS Regions and accounts, but provide a more limited decryption keyring tailored to a particular account, Region, user, group, or role\. 

For example, the following KMS keyring includes only CMK\_2\. You can use this keyring to decrypt a message encrypted under both CMK\_1 and CMK\_2, provided that you have permission to use CMK\_2 to decrypt data\.

```
const char * CMK_2 = "arn:aws:kms:us-west-2:111122223333:key/0987dcba-09fe-87dc-65ba-ab0987654321"    

struct aws_cryptosdk_keyring *kms_decrypt_keyring = 
       Aws::Cryptosdk::KmsKeyring::Builder().Build(CMK_2);
```

You can also use a KMS keyring that specifies a generator key for decrypting, such as the following one\. When decrypting, the AWS Encryption SDK ignores the distinction between generator keys and additional keys\. It can use any of the specified CMKs to decrypt an encrypted data key\. The call to AWS KMS succeeds only when the caller has permission to use that CMK to decrypt data\.

```
struct aws_cryptosdk_keyring *kms_decrypt_keyring = 
       Aws::Cryptosdk::KmsKeyring::Builder().Build(CMK_1, {CMK_2, CMK_3});
```

Unlike an encryption keyring that uses all of the specified CMKs, you can decrypt an encrypted message using a decryption keyring that includes CMKs unrelated to the encrypted message, and CMKs that the caller doesn't have permission to use\. If a Decrypt call to AWS KMS fails, such as when the caller doesn't have the required permission, the AWS Encryption SDK just skips to the next encrypted data key\. 

### Using a KMS Discovery Keyring<a name="kms-keyring-discovery"></a>

Typically, when decrypting, you provide a KMS keyring that limits the CMKs that the AWS Encryption SDK can use to those that you specify\. However, you can also create a *KMS Discovery keyring*, that is, a KMS keyring that doesn't specify any CMKs\. It allows the AWS Encryption SDK to ask AWS KMS to decrypt any encrypted data key in an encrypted message using whichever CMK encrypted it, regardless of who owns or has access to that CMK\.

The following code instantiates a KMS Discovery keyring\.

```
struct kms_discovery_keyring = Aws::Cryptosdk::KmsKeyring::Builder().BuildDiscovery();
```

When encrypting, a KMS Discovery keyring has no effect\. It doesn't return any encrypted data keys\. However, you might want to include a KMS Discovery keyring in a multi\-keyring that will be used for encrypting and decrypting\.

When decrypting with a KMS Discovery keyring, the AWS Encryption SDK calls AWS KMS to decrypt each of the encrypted data keys in the encrypted message in the order that they appear until one succeeds\. To succeed, the caller must have `kms:Decrypt` permission on at least one of the CMKs that encrypted one of the encrypted data keys\.

The AWS Encryption SDK provides a KMS Discovery keyring for convenience\. However, we recommend that you use a more limited keyring whenever possible for the following reasons\.
+ **Authenticity** — The KMS Discovery keyring can use any CMK that was used to encrypt a data key in the encrypted message, just so the caller has permission to use that CMK to decrypt\. This might not be the CMK that the caller intends to use\. For example, one of the encrypted data keys might have been encrypted under a less secure CMK that anyone can use\. 
+ **Latency and performance** — A KMS Discovery keyring might be perceptibly slower than other keyrings because the AWS Encryption SDK tries to decrypt all of the encrypted data keys, including those encrypted by CMKs in other AWS accounts and Regions, and CMKs that the caller doesn't have permission to use for decryption\. 

When you use a KMS Discovery keyring in a multi\-keyring, it has no effect on encryption\. When decrypting, by allowing the use of any CMK, a KMS Discovery keyring overrides any CMK limits that other KMS keyrings in the multi\-keyring might impose\. For example, if you combine a KMS keyring that lets you use only CMK\_1 and a KMS Discovery keyring, the resulting multi\-keyring behaves just like the KMS Discovery keyring\. It lets the AWS Encryption SDK ask AWS KMS to decrypt any encrypted data key, even if it was encrypted by a different CMK\. 

### Using a KMS Regional Discovery Keyring<a name="kms-keyring-regional"></a>

Instead of using a KMS keyring that specifies particular CMKs, or a KMS Discovery keyring that can use any CMK, you might want to use a KMS Regional Discovery keyring that limits the AWS Encryption SDK to CMKs in a particular AWS Region\. 

For example, the following code creates a KMS Regional Discovery keyring for the US West \(Oregon\) Region \(us\-west\-2\)\. To view this keyring, and the `create_kms_client` method, in a working example, see [kms\_discovery\.cpp](https://github.com/awslabs/aws-encryption-sdk-c/blob/master/examples/kms_discovery.cpp)\.

```
struct aws_cryptosdk_keyring *kms_regional_keyring = Aws::Cryptosdk::KmsKeyring::Builder()
       .WithKmsClient(create_kms_client(Aws::Region::US_WEST_2)).BuildDiscovery());
```

When encrypting, a KMS Regional Discovery keyring has no effect\. Because it doesn't specify any CMKs, it can't generate or encrypt data keys\. However, you can include a KMS Regional Discovery keyring in a multi\-keyring that will be used for encrypting and decrypting\. 

When decrypting with a KMS Regional Discovery keyring, the AWS Encryption SDK can ask AWS KMS to decrypt any encrypted data key that was encrypted under a CMK in the specified AWS Region\. To succeed, the caller must have `kms:Decrypt` permission on at least one of the CMKs in the specified AWS Region that encrypted one of the data keys in the encrypted message\.

In a multi\-keyring, because it allows the use of any CMK in the AWS Region, a KMS Regional Discovery keyring can override CMK limits that other KMS keyrings in the multi\-keyring might impose\. For example, if you combine a KMS keyring that lets you use a particular CMK in the EU \(London\) Region and a KMS Regional Discovery keyring for the EU \(London\) Region, the resulting multi\-keyring behaves just like the KMS Regional Discovery keyring alone\. It lets the AWS Encryption SDK ask AWS KMS to decrypt any encrypted data key that was encrypted by any CMK in the EU \(London\) Region\. 

## Raw AES Keyring<a name="use-raw-aes-keyring"></a>

The Raw AES keyring performs AES\-GCM encryption of data keys using the AES\-128, AES\-192, or AES\-256 algorithm and a wrapping key that you specify as a byte array\. You can specify only one wrapping key in each Raw AES keyring, but you can include multiple Raw AES keyrings in each [multi\-keyring](#use-multi-keyring)\. The Raw AES keyring is equivalent to and interoperates with the [JceMasterKey](https://aws.github.io/aws-encryption-sdk-java/javadoc/com/amazonaws/encryptionsdk/jce/JceMasterKey.html) in the AWS Encryption SDK for Java and the [RawMasterKey](https://aws-encryption-sdk-python.readthedocs.io/en/latest/generated/aws_encryption_sdk.key_providers.raw.html#aws_encryption_sdk.key_providers.raw.RawMasterKey) in the AWS Encryption SDK for Python when they are used with symmetric encryption keys\. You can encrypt data with one implementation and decrypt the data with any other implementation using the same wrapping key\.

Use a Raw AES keyring when you need to provide the wrapping key and encrypt the data keys locally, or you need to write an application that is compatible with the AWS Encryption SDK for Java or Python\. Whenever possible, we recommend using a hardware security module \(HSM\) or a service, such as AWS KMS, that doesn't expose wrapping keys and encrypts data keys within a secure boundary\.

To identify the wrapping key, the Raw AES keyring uses a namespace and name that you provide\. These are equivalent to the `Provider ID` and `Key ID` fields in the AWS Encryption SDK for Java and Python\. These values are not secret; they appear in plaintext in the header of the [encrypted message](concepts.md#message) that the AWS Encryption SDK returns\. However, they are critical\. You must use the same namespace and name for a key in your decryption keyring that you used for that key in your encryption keyring\. If the namespace and name are not an exact, case\-sensitive match, the AWS Encryption SDK will not recognize that the wrapping keys are the same, even if the bytes are identical, and it will not be able to decrypt the encrypted data keys\.

For an example of how to use a Raw AES keyring, see [raw\_aes\_keyring\.c](https://github.com/awslabs/aws-encryption-sdk-c/blob/master/examples/raw_aes_keyring.c)\.

## Raw RSA Keyring<a name="use-raw-rsa-keyring"></a>

The Raw RSA keyring performs asymmetric encryption and decryption of data keys in local memory with wrapping and unwrapping keys that you specify\. The encryption function encrypts the data key under the RSA public key\. The decryption function decrypts the data key using the private key\. You can select from among the several [RSA padding modes](https://github.com/awslabs/aws-encryption-sdk-c/blob/master/include/aws/cryptosdk/cipher.h)\.

A Raw RSA keyring that encrypts and decrypts must include an asymmetric public key and private key pair\. But, you can create a Raw RSA encryption keyring with only a public key or a Raw RSA decryption keyring with only a private key\. And, you can include any Raw RSA keyring in a [multi\-keyring](#use-multi-keyring)\. The Raw RSA keyring is equivalent to and interoperates with the [JceMasterKey](https://aws.github.io/aws-encryption-sdk-java/javadoc/com/amazonaws/encryptionsdk/jce/JceMasterKey.html) in the AWS Encryption SDK for Java and the [RawMasterKey](https://aws-encryption-sdk-python.readthedocs.io/en/latest/generated/aws_encryption_sdk.key_providers.raw.html#aws_encryption_sdk.key_providers.raw.RawMasterKey) in the AWS Encryption SDK for Python when they are used with asymmetric encryption keys\. You can encrypt data with one implementation and decrypt the data with any other implementation using the same wrapping key\.

Use a Raw RSA keyring when you want to use an asymmetric key pair and provide the wrapping key and unwrapping keys, or you need to be compatible with the AWS Encryption SDK in another programming language\. 

To identify the key pair, the Raw RSA keyring uses a namespace and name that you provide\. These values are not secret; they appear in plaintext in the header of the [encrypted message](concepts.md#message) that the AWS Encryption SDK returns\. However, they are critical\. If you use the same key pair in an encryption keyring and a decryption keyring, be sure to use the same namespace and name for the key pair in both keyrings\. If the namespace and name are not an exact, case\-sensitive match, the AWS Encryption SDK will not recognize that the asymmetric keys are a pair, and it will not be able to decrypt the encrypted data keys\.

When constructing a Raw RSA keyring, be sure to provide the *contents* of the PEM file that includes each key as a null\-terminated C\-string, not as a path or file name\. For an example of how to use a Raw RSA keyring, see [raw\_rsa\_keyring\.c](https://github.com/awslabs/aws-encryption-sdk-c/blob/master/examples/raw_rsa_keyring.c)\.

## Creating a Multi\-Keyring<a name="use-multi-keyring"></a>

You can combine keyrings into a multi\-keyring\. A *multi\-keyring* is a keyring that consists of one or more individual keyrings of the same or a different type\. The effect is like using several keyrings in a series\. When you use a multi\-keyring to encrypt data, any of the wrapping keys in any of its keyrings can decrypt that data\.

When you create a multi\-keyring to encrypt data, you designate one of the child keyrings as the *generator keyring*\. The generator keyring generates and encrypts the plaintext data key\. Then, all of the wrapping keys in all of the child keyrings encrypt the same plaintext data key\. The multi\-keyring returns the plaintext key and one encrypted data key for each wrapping key in the multi\-keyring\. If you create a multi\-keyring with no generator keyring, you can use it to decrypt data, but not to encrypt\. If the generator keyring is a [KMS keyring](#use-kms-keyring), the AWS Encryption SDK asks the generator key in the KMS keyring to generate and encrypt the plaintext key\. Then, all additional CMKs in the KMS keyring, and all wrapping keys in all child keyrings in the multi\-keyring encrypt the same plaintext key\. 

When decrypting, the AWS Encryption SDK uses the keyrings to try to decrypt one of the encrypted data keys\. The keyrings are called in the order that they are specified in the multi\-keyring, and processing stops as soon as a keyring can decrypt an encrypted data key\. 

To create a multi\-keyring, first instantiate the child keyrings\. In this example, we use a KMS keyring and a Raw AES keyring, but you can combine any supported keyrings in a multi\-keyring\.

```
// Define a KMS keyring. For details, see [string\.cpp](https://github.com/awslabs/aws-encryption-sdk-c/blob/master/examples/string.cpp).
struct aws_cryptosdk_keyring *kms_keyring = Aws::Cryptosdk::KmsKeyring::Builder().Build(example_CMK);

// Define a Raw AES keyring. For details, see [raw\_aes\_keyring\.c](https://github.com/awslabs/aws-encryption-sdk-c/blob/master/examples/raw_aes_keyring.c).
struct aws_cryptosdk_keyring *aes_keyring = aws_cryptosdk_raw_aes_keyring_new(
        alloc, wrapping_key_namespace, wrapping_key_name, wrapping_key, AWS_CRYPTOSDK_AES256);
```

Next, create the multi\-keyring and specify its generator keyring, if any\. In this example, we create a multi\-keyring in which the KMS keyring is the generator keyring\.

```
struct aws_cryptosdk_keyring *multi_keyring = aws_cryptosdk_multi_keyring_new(alloc, kms_keyring);
```

Then, to add each additional child keyring, call the `aws_cryptosdk_multi_keyring_add_child` method\. In this example, we'll add the Raw AES keyring to the multi\-keyring\. 

The result is a multi\-keyring that includes the KMS keyring as the generator keyring and the Raw AES keyring as a child keyring\. 

```
/* Add the Raw AES keyring
int aws_cryptosdk_multi_keyring_add_child(struct aws_cryptosdk_keyring *multi, struct aws_cryptosdk_keyring *child);
```

Now, you can use the multi\-keyring to encrypt and decrypt data\. For a complete example of how to build and use a multi\-keyring, see [multi\_keyring\.cpp](https://github.com/awslabs/aws-encryption-sdk-c/blob/master/examples/multi_keyring.cpp)\.