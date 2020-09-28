# Concepts in the AWS Encryption SDK<a name="concepts"></a>

This section introduces the concepts used in the AWS Encryption SDK, and provides a glossary and reference\. It's designed to help you understand how the AWS Encryption SDK works and the terms we use to describe it\.

Need help? 
+ Learn how the AWS Encryption SDK uses [envelope encryption](#envelope-encryption) to protect your data\.
+ Learn about the elements of envelope encryption: the [data keys](#DEK) that protect your data and the [wrapping keys](#master-key) that protect your data keys\. 
+ Learn about the [keyrings](#keyring) and [master key providers](#master-key-provider) that determine which wrapping keys you use\.
+ Learn about the [encryption context](#encryption-context) that adds integrity to your encryption process\. It's optional, but it's a best practice that we recommend\.
+ Learn about the [encrypted message](#message) that the encryption methods return\. 
+ Then you're ready to use the AWS Encryption SDK in your preferred [programming language](programming-languages.md)\.

**Topics**
+ [Envelope encryption](#envelope-encryption)
+ [Data key](#DEK)
+ [Wrapping key](#master-key)
+ [Keyring \(C and JavaScript\)](#keyring)
+ [Master key provider \(Java and Python\)](#master-key-provider)
+ [Encryption context](#encryption-context)
+ [Encrypted message](#message)
+ [Algorithm suite](#crypto-algorithm)
+ [Cryptographic materials manager](#crypt-materials-manager)
+ [Symmetric and asymmetric encryption](#symmetric-key-encryption)
+ [Key commitment](#key-commitment)
+ [Commitment policy](#commitment-policy)

## Envelope encryption<a name="envelope-encryption"></a>

The security of your encrypted data depends in part on protecting the data key that can decrypt it\. One accepted best practice for protecting the data key is to encrypt it\. To do this, you need another encryption key, known as a *key\-encryption key* or [wrapping key](#master-key)\. The practice of using a wrapping key to encrypt data keys is known as *envelope encryption*\. The benefits of envelope encryption include the following\.

**Protecting data keys**  
When you encrypt a data key, you don't have to worry about where to store it because the data key is inherently protected by encryption\. You can safely store the encrypted data key with the encrypted data\. The AWS Encryption SDK does this for you\. It saves the encrypted data and the encrypted data key together in an [encrypted message](#message)\.

**Encrypting the same data under multiple wrapping keys**  
Encryption operations can be time\-consuming, particularly when the data being encrypted are large objects\. Instead of reencrypting raw data multiple times with different keys, you can reencrypt only the data keys that protect the raw data\. 

**Combining the strengths of multiple algorithms**  
In general, symmetric key encryption algorithms are faster and produce smaller ciphertexts than asymmetric or *public key encryption*\. But public key algorithms provide inherent separation of roles and easier key management\. You might want to combine the strengths of each\. For example, you might encrypt raw data with symmetric key encryption, and then encrypt the data key with public key encryption\.

The AWS Encryption SDK uses envelope encryption\. It encrypts your data with a data key\. Then, it encrypts the data key with a wrapping key\. The AWS Encryption SDK returns the encrypted data and the encrypted data keys in a single [encrypted message](#message), as shown in the following diagram\. 

![\[Envelope encryption with the AWS Encryption SDK\]](http://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/images/envelope-encryption.png)

If you have multiple wrapping keys, each of them can encrypt the plaintext data key\. The encrypted message that the AWS Encryption SDK returns contains the encrypted data and the collection of encrypted data keys\. Any one of the wrapping keys can decrypt one of the encrypted data keys, which can then decrypt the data\. 

When you use envelope encryption, you need to protect your wrapping keys from unauthorized access\. You can do this in any of the following ways:
+ Use a web service designed for this purpose, such as [AWS Key Management Service \(AWS KMS\)](https://aws.amazon.com/kms/)\.
+ Use a [hardware security module \(HSM\)](https://en.wikipedia.org/wiki/Hardware_security_module) such as those offered by [AWS CloudHSM](https://aws.amazon.com/cloudhsm/)\.
+ Use other key management tools and services\.

If you don't have a key management system, we recommend AWS KMS\. The AWS Encryption SDK integrates with AWS KMS to help you protect and use your wrapping keys\. However, the AWS Encryption SDK does not require AWS or any AWS service\.

## Data key<a name="DEK"></a>

A *data key* is an encryption key that the AWS Encryption SDK uses to encrypt your data\. Each data key is a byte array that conforms to the requirements for cryptographic keys\. Unless you're using [data key caching](data-key-caching.md), the AWS Encryption SDK uses a unique data key to encrypt each message\.

You don't need to specify, generate, implement, extend, protect or use data keys\. The AWS Encryption SDK does that work for you when you call the encrypt and decrypt operations\. 

To protect your data keys, the AWS Encryption SDK encrypts them under one or more *key\-encryption keys* known as [master keys or wrapping keys](#master-key)\. After the AWS Encryption SDK uses your plaintext data keys to encrypt your data, it removes them from memory as soon as possible\. Then it stores the encrypted data keys with the encrypted data in the [encrypted message](#message) that the encrypt operations return\. For details, see [How the AWS Encryption SDK works](how-it-works.md)\.

**Tip**  
In the AWS Encryption SDK, we distinguish *data keys* from *data encryption keys*\. Several of the supported [algorithm suites](#crypto-algorithm), including the default suite, use a [key derivation function](https://en.wikipedia.org/wiki/Key_derivation_function) that prevents the data key from hitting its cryptographic limits\. The key derivation function takes the data key as input and returns a data encryption key that is actually used to encrypt the data\. For this reason, we often say that data is encrypted "under" a data key rather than "by" the data key\.

## Wrapping key<a name="master-key"></a>

A *wrapping key* \(or *master key*\) is a key\-encryption key that the AWS Encryption SDK uses to encrypt the [data key](#DEK) that encrypts your data\. Each plaintext data key can be encrypted under one or more wrapping keys\. You determine which wrapping keys are used to protect your data when you configure a [keyring](#keyring) or [master key provider](#master-key-provider)\.

**Note**  
*Wrapping key* refers to the keys in a keyring or master key provider\. *Master key* is typically associated with the `MasterKey` class that you instantiate when you use a master key provider\. master keys and wrapping keys are key\-encryption keys that the AWS Encryption SDK uses to protect the data key that protects your data\.

The AWS Encryption SDK supports several commonly used wrapping keys, such as AWS Key Management Service \(AWS KMS\) symmetric [customer master keys](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#master_keys) \(CMKs\), raw AES\-GCM \(Advanced Encryption Standard/Galois Counter Mode\) keys, and raw RSA keys\. You can also extend or implement your own wrapping keys\. 

## Keyring \(C and JavaScript\)<a name="keyring"></a>

A *keyring* generates, encrypts, and decrypts data keys\. When you define a keyring, you can specify the wrapping keys that encrypt your data keys\. Most keyrings specify at least one wrapping key or a service that provides and protects wrapping keys\. You can also define a keyring with no wrapping keys or a more complex keyring with additional configuration options\. For help choosing and using the keyrings that the AWS Encryption SDK defines, see [Using keyrings](choose-keyring.md)\.

You can use the keyrings that the AWS Encryption SDK provides or write your own compatible custom keyrings\. You can use a single keyring with one or more wrapping keys of the same type\. Or, you can combine keyrings of the same type or a different type into a [multi\-keyring](choose-keyring.md#use-multi-keyring)\. The multi\-keyring returns a copy of the data key encrypted by each of the wrapping keys in each of the keyrings that comprise the multi\-keyring\. When you use a multi\-keyring to encrypt data, you can decrypt the data using a keyring configured with any one of the wrapping keys in the multi\-keyring\.

The keyrings that the AWS Encryption SDK provides are compatible with master key providers in the AWS Encryption SDK for Java and the AWS Encryption SDK for Python\. However, you must specify the same key material and use a keyring that is compatible with the master key provider, subject to language constraints\. Any minor incompatibility due to language constraints is explained in the topic about the language implementation\. For details, see [Keyring compatibility](choose-keyring.md#keyring-compatibility)\. 

## Master key provider \(Java and Python\)<a name="master-key-provider"></a>

A *master key provider* returns master keys or objects that identify or represent master keys\. Each master key is associated with one master key provider, but a master key provider typically provides multiple master keys\. You can use the master key providers that the AWS Encryption SDK provides or design your own custom master key providers\.

Master key providers in the AWS Encryption SDK for Java and AWS Encryption SDK for Python are compatible with keyrings in the all language implementations, subject to language constraints\. However, you must specify the same key material and use a keyring that is compatible with the master key provider\. For details, see [Keyring compatibility](choose-keyring.md#keyring-compatibility)\.

When you use a master key and master key provider, you can also specify a [cryptographic materials manager](#crypt-materials-manager) \(CMM\)\. However, if you specify a master key provider, the AWS Encryption SDK creates a Default CMM for you\. 

## Encryption context<a name="encryption-context"></a>

To improve the security of your cryptographic operations, include an [encryption context](https://docs.aws.amazon.com/crypto/latest/userguide/cryptography-concepts.html#define-encryption-context) in all requests to encrypt data\. Using an encryption context is optional, but it is a cryptographic best practice that we recommend\.

An *encryption context* is a set of name\-value pairs that contain arbitrary, non\-secret additional authenticated data\. The encryption context can contain any data you choose, but it typically consists of data that is useful in logging and tracking, such as data about the file type, purpose, or ownership\. When you encrypt data, the encryption context is cryptographically bound to the encrypted data so that the same encryption context is required to decrypt the data\. The AWS Encryption SDK includes the encryption context in plaintext in the header of the [encrypted message](#message) that it returns\.

The encryption context that the AWS Encryption SDK uses consists of the encryption context that you specify and a public key pair that the [cryptographic materials manager](#crypt-materials-manager) \(CMM\) adds\. Specifically, whenever you use an [encryption algorithm with signing](algorithms-reference.md), the CMM adds a name\-value pair to the encryption context that consists of a reserved name, `aws-crypto-public-key`, and a value that represents the public verification key\. The `aws-crypto-public-key` name in the encryption context is reserved by the AWS Encryption SDK and cannot be used as a name in any other pair in the encryption context\. For details, see [AAD](message-format.md#header-aad) in the *Message Format Reference*\.

The following example encryption context consists of two encryption context pairs specified in the request and the public key pair that the CMM adds\.

```
"Purpose"="Test", "Department"="IT", aws-crypto-public-key=<public key>
```

To decrypt the data, you pass in the encrypted message\. Because the AWS Encryption SDK can extract the encryption context from the encrypted message header, you are not required to provide the encryption context separately\. However, the encryption context can help you to confirm that you are decrypting the correct encrypted message\. 
+ In the [AWS Encryption SDK Command Line Interface](crypto-cli.md) \(CLI\), if you provide an encryption context in a decrypt command, the CLI verifies that the values are present in the encryption context of the encrypted message before it returns the plaintext data\. 
+ In other languages, the decrypt response includes the encryption context and the plaintext data\. The decrypt function in your application should always verify that the encryption context in the decrypt response includes the encryption context in the encrypt request \(or a subset\) before it returns the plaintext data\.

When choosing an encryption context, remember that it is not a secret\. The encryption context is displayed in plaintext in the header of the [encrypted message](#message) that the AWS Encryption SDK returns\. If you are using AWS Key Management Service, the encryption context also might appear in plaintext in audit records and logs, such as AWS CloudTrail\.

For examples of submitting and verifying an encryption context in your code, see the examples for your preferred [programming language](programming-languages.md)\.

## Encrypted message<a name="message"></a>

When you encrypt data with the AWS Encryption SDK, it returns an encrypted message\.

An *encrypted message* is a portable [formatted data structure](message-format.md) that includes the encrypted data along with encrypted copies of the data keys, the algorithm ID, and, optionally, an encryption context and a message signature\. Encrypt operations in the AWS Encryption SDK return an encrypted message and decrypt operations take an encrypted message as input\. 

Combining the encrypted data and its encrypted data keys streamlines the decryption operation and frees you from having to store and manage encrypted data keys independently of the data that they encrypt\.

For technical information about the encrypted message, see [Encrypted Message Format](message-format.md)\.

## Algorithm suite<a name="crypto-algorithm"></a>

The AWS Encryption SDK uses an algorithm suite to encrypt and sign the data in the [encrypted message](#message) that the encrypt and decrypt operations return\. The AWS Encryption SDK supports several [algorithm suites](supported-algorithms.md)\. All of the supported suites use Advanced Encryption Standard \(AES\) as the primary algorithm, and combine it with other algorithms and values\. 

The AWS Encryption SDK establishes a recommended algorithm suite as the default for all encryption operations\. The default might change as standards and best practices improve\. You can specify an alternate algorithm suite in requests to encrypt data or when creating a [cryptographic materials manager \(CMM\)](#crypt-materials-manager), but unless an alternate is required for your situation, it is best to use the default\. The current default is AES\-GCM with an HMAC\-based extract\-and\-expand key derivation function \([HKDF](https://en.wikipedia.org/wiki/HKDF)\), Elliptic Curve Digital Signature Algorithm \(ECDSA\) signing, and a 256\-bit encryption key\. 

If you specify an algorithm suite, we recommend an algorithm suite that uses a [key derivation function](https://en.wikipedia.org/wiki/HKDF) and a message signing algorithm\. Algorithm suites that have neither feature are supported only for backward compatibility\.

## Cryptographic materials manager<a name="crypt-materials-manager"></a>

The cryptographic materials manager \(CMM\) assembles the cryptographic materials that are used to encrypt and decrypt data\. The *cryptographic materials* include plaintext and encrypted data keys, and an optional message signing key\. You never interact with the CMM directly\. The encryption and decryption methods handle it for you\.

You can use the default CMM or the [caching CMM](data-key-caching.md) that the AWS Encryption SDK provides, or write a custom CMM\. And you can specify a CMM, but it's not required\. When you specify a keyring or master key provider, the AWS Encryption SDK creates a default CMM for you\. The default CMM gets the encryption or decryption materials from the keyring or master key provider that you specify\. This might involve a call to a cryptographic service, such as [AWS Key Management Service ](https://docs.aws.amazon.com/kms/latest/developerguide/)\(AWS KMS\)\.

Because the CMM acts as a liaison between the AWS Encryption SDK and a keyring \(or master key provider\), it is an ideal point for customization and extension, such as support for policy enforcement and caching\. The AWS Encryption SDK provides a caching CMM to support [data key caching\.](data-key-caching.md) 

## Symmetric and asymmetric encryption<a name="symmetric-key-encryption"></a>

*Symmetric encryption* uses the same key to encrypt and decrypt data\. 

*Asymmetric encryption* uses a mathematically related data key pair\. One key in the pair encrypts the data; only the other key in the pair can decrypt the data\. For details, see [Cryptographic algorithms](https://docs.aws.amazon.com/crypto/latest/userguide/concepts-algorithms.html) in the *AWS Cryptographic Services and Tools Guide*\.

The AWS Encryption SDK uses [envelope encryption](#envelope-encryption)\. It encrypts your data with a symmetric data key\. It encrypts the symmetric data key with one or more symmetric or asymmetric wrapping keys\. It returns an [encrypted message](#message) that includes the encrypted data and at least one encrypted copy of the data key\. 

**Encrypting your data \(symmetric encryption\)**  
To encrypt your data, the AWS Encryption SDK uses a symmetric [data key](#DEK) and an [algorithm suite](#crypto-algorithm) that includes a symmetric encryption algorithm\. To decrypt the data, the AWS Encryption SDK uses the same data key and the same algorithm suite\.

**Encrypting your data key \(symmetric or asymmetric encryption\)**  
The [keyring](#keyring) or [master key provider](#master-key-provider) that you supply to an encrypt and decrypt operation determines how the symmetric data key is encrypted and decrypted\. You can choose a keyring or master key provider that uses symmetric encryption, such as a AWS KMS keyring, or one that uses asymmetric encryption, such as a raw RSA keyring or `JceMasterKey`\.

## Key commitment<a name="key-commitment"></a>

The AWS Encryption SDK supports *key commitment* \(sometimes known as *robustness*\), a security property that guarantees that each ciphertext can be decrypted only to a single plaintext\. To do this, key commitment guarantees that only the data key that encrypted your message will be used to decrypt it\. Encrypting and decrypting with key commitment is an [AWS Encryption SDK best practice](best-practices.md)\.

Most modern symmetric ciphers \(including AES\) encrypt a plaintext under a single secret key, such as the [unique data key](#DEK) that the AWS Encryption SDK uses to encrypt each plaintext message\. Decrypting this data with the same data key returns a plaintext that is identical to the original\. Decrypting with a different key will usually fail\. However, it's possible to decrypt a ciphertext under two different keys\. In rare cases, it is feasible to find a key that can decrypt a few bytes of ciphertext into a different, but still intelligible, plaintext\. 

The AWS Encryption SDK always encrypts each plaintext message under one unique data key\. It might encrypt that data key under multiple wrapping keys \(or master keys\), but the wrapping keys always encrypt the same data key\. Nonetheless, a sophisticated, manually crafted [encrypted message](#message) might actually contain different data keys, each encrypted by a different wrapping key\. For example, if one user decrypts the encrypted message it returns 0x0 \(false\) while another user decrypting the same encrypted message gets 0x1 \(true\)\.

To prevent this scenario, the AWS Encryption SDK supports key commitment when encrypting and decrypting\. When the AWS Encryption SDK encrypts a message with key commitment, it cryptographically binds the unique data key that produced the ciphertext to the *key commitment string*, a non\-secret data key identifier\. Then it stores key commitment string in the metadata of the encrypted message\. When it decrypts a message with key commitment, the AWS Encryption SDK verifies that the data key is the one and only key for that encrypted message\. If data key verification fails, the decrypt operation fails\. 

Support for key commitment is introduced in version 1\.7\.*x*, which can decrypt messages with key commitment, but won't encrypt with key commitment\. You can use this version to fully deploy the ability to decrypt ciphertext with key commitment\. Version 2\.0\.*x* includes full support for key commitment\. By default, it encrypts and decrypts only with key commitment\. This is an ideal configuration for applications that don't need to decrypt ciphertext encrypted by earlier versions of the AWS Encryption SDK\. 

Although encrypting and decrypting with key commitment is a best practice, we let you decide when it's used, and let you adjust the pace at which you adopt it\. Beginning in version 1\.7\.*x*, AWS Encryption SDK supports a [commitment policy](#commitment-policy) that sets the [default algorithm suite](supported-algorithms.md) and limits the algorithm suites that may be used\. This policy determines whether your data is encrypted and decrypted with key commitment\. 

Key commitment results in a [slightly larger \(\+ 30 bytes\) encrypted message](message-format.md) and takes more time to process\. If your application is very sensitive to size or performance, you might choose to opt out of key commitment\. But do so only if you must\. 

For more information about migrating to versions 1\.7\.*x* and 2\.0\.*x*, including their key commitment features, see [Migrating to version 2\.0\.*x*](migration.md)\. For technical information about key commitment, see [AWS Encryption SDK algorithms reference](algorithms-reference.md) and [AWS Encryption SDK message format reference](message-format.md)\.

## Commitment policy<a name="commitment-policy"></a>

A *commitment policy* is a configuration setting that determines whether your application encrypts and decrypts with [key commitment](#key-commitment)\. Encrypting and decrypting with key commitment is an [AWS Encryption SDK best practice](best-practices.md)\. 

Commitment policy has three values\.

**Note**  
You might have to scroll horizontally or vertically to see the entire table\.


**Commitment policy values**  

| Value | Encrypts with key commitment | Encrypts without key commitment | Decrypts with key commitment | Decrypts without key commitment | 
| --- | --- | --- | --- | --- | 
| ForbidEncryptAllowDecrypt | ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/images/icon-no.png)  | ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/images/icon-yes.png)  | ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/images/icon-yes.png) | ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/images/icon-yes.png) | 
| RequireEncryptAllowDecrypt | ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/images/icon-yes.png) | ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/images/icon-no.png) | ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/images/icon-yes.png) | ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/images/icon-yes.png) | 
| RequireEncryptRequireDecrypt | ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/images/icon-yes.png) | ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/images/icon-no.png) | ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/images/icon-yes.png) | ![\[Image NOT FOUND\]](http://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/images/icon-no.png) | 

The commitment policy setting is introduced in AWS Encryption SDK version 1\.7\.*x*\. It's valid in all supported [programming languages](programming-languages.md)\.
+ `ForbidEncryptAllowDecrypt` decrypts with or without key commitment, but it won't encrypt with key commitment\. This is the only valid value for commitment policy in version 1\.7\.*x*\. It's designed to prepare all hosts running your application to decrypt with key commitment before they ever encounter a ciphertext encrypted with key commitment\.
+ `RequireEncryptAllowDecrypt` always encrypts with key commitment\. It can decrypt with or without key commitment\. This value, introduced in version 2\.0\.*x*, lets you start encrypting with key commitment, but still decrypt legacy ciphertexts without key commitment\.
+ `RequireEncryptRequireDecrypt` encrypts and decrypts only with key commitment\. This value is the default for version 2\.0\.*x*\. Use this value when you are certain that all of your ciphertexts are encrypted with key commitment\.

The commitment policy setting determines which algorithm suites you can use\. Beginning in version 1\.7\.*x*, the AWS Encryption SDK supports [algorithm suites](supported-algorithms.md) for key commitment; with and without signing\. If you specify an algorithm suite that conflicts with your commitment policy, the AWS Encryption SDK returns an error\. 

For help setting your commitment policy, see [Setting your commitment policy](migrate-commitment-policy.md)\.