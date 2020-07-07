# Concepts in the AWS Encryption SDK<a name="concepts"></a>

This section introduces the concepts used in the AWS Encryption SDK, and provides a glossary and reference\.

**Topics**
+ [Data keys](#DEK)
+ [Master key](#master-key)
+ [Cryptographic materials manager](#crypt-materials-manager)
+ [Master key provider \(Java and Python\)](#master-key-provider)
+ [Keyring \(C and JavaScript\)](#keyring)
+ [Algorithm suite](#crypto-algorithm)
+ [Encryption context](#encryption-context)
+ [Encrypted message](#message)

## Data keys<a name="DEK"></a>

A *data key* is an encryption key that the AWS Encryption SDK uses to encrypt your data\. Each data key is a byte array that conforms to the requirements for cryptographic keys\. Unless you're using [data key caching](data-key-caching.md), the AWS Encryption SDK uses a unique data key to encrypt each message\. 

To protect your data keys, the AWS Encryption SDK encrypts them under one or more key encryption keys known as [master keys](#master-key) or wrapping keys\. After the AWS Encryption SDK uses your plaintext data keys to encrypt your data, it removes them from memory as soon as possible\. Then it stores the encrypted data keys with the encrypted data in the [encrypted message](#message) that the encrypt operations return\. 

When you use the AWS Encryption SDK, you do not need to generate, implement, extend, protect or use data keys or master keys\. The AWS Encryption SDK does that work for you when you call the encrypt and decrypt operations\. 

However, when you select your master key provider \(Java and Python\) or keyring \(C and JavaScript\), you determine the source of your master keys\. You also determine whether your plaintext data key is encrypted by one or multiple master keys\. 

**Tip**  
In the AWS Encryption SDK, we distinguish *data keys* from *data encryption keys*\. Several of the supported [algorithm suites](#crypto-algorithm), including the default suite, use a [key derivation function](https://en.wikipedia.org/wiki/Key_derivation_function) that prevents the data key from hitting its cryptographic limits\. The key derivation function takes the data key as input and returns a data encryption key that is actually used to encrypt the data\. For this reason, we often say that data is encrypted "under" a data key rather than "by" the data key\.

## Master key<a name="master-key"></a>

A *master key*, also known as a *wrapping key*, is an encryption key that is used to encrypt data keys\. Each plaintext data key can be encrypted under one or more master keys\. 

When you use the AWS Encryption SDK, you do not need to generate, implement, extend, protect or use data keys or master keys\. The AWS Encryption SDK does that work for you when you call the encrypt and decrypt operations\. However, in the Java and Python implementations, you need to specify a [cryptographic materials manager](#crypt-materials-manager) or a [master key provider](#master-key-provider) that supplies master keys\. In the C and JavaScript implementations, you specify a [keyring](#keyring) that interacts with wrapping keys for you and returns data keys\.

The AWS Encryption SDK provides several commonly used master keys or wrapping keys, such as AWS Key Management Service \(AWS KMS\) symmetric customer master keys \(CMKs\), raw AES\-GCM \(Advanced Encryption Standard/Galois Counter Mode\) keys, and RSA keys\. You can also extend or implement your own master keys and wrapping keys\. 

## Cryptographic materials manager<a name="crypt-materials-manager"></a>

The cryptographic materials manager \(CMM\) assembles the cryptographic materials that are used to encrypt and decrypt data\. The *cryptographic materials* include plaintext and encrypted data keys, and an optional message signing key\. You can use the default CMM that the AWS Encryption SDK provides or write a custom CMM\. You can specify a CMM, but you never interact with it directly\. The encryption and decryption methods handle it for you\.

The default CMM gets the encryption or decryption materials from the master key provider \(Java and Python\) or keyring \(C or JavaScript\) that you specify\. This might involve a call to a cryptographic service, such as AWS Key Management Service \(AWS KMS\)\.

You can specify a CMM and master key provider or keyring, but it's not required\. If you specify a master key provider or keyring, the AWS Encryption SDK creates a Default CMM for you\.

Because the CMM acts as a liaison between the SDK and a keyring or master key provider, it is an ideal point for customization and extension, such as support for policy enforcement and caching\. The AWS Encryption SDK provides a caching CMM to support [data key caching\.](data-key-caching.md) 

## Master key provider \(Java and Python\)<a name="master-key-provider"></a>

In the AWS Encryption SDK for Java and the AWS Encryption SDK for Python, a *master key provider* returns master keys, or objects that identify or represent master keys\. Each master key is associated with one master key provider, but a master key provider typically provides multiple master keys\.

When you use the Java and Python implementations of the AWS Encryption SDK, you need to specify a [cryptographic materials manager](#crypt-materials-manager) \(CMM\) or a master key provider, but you do not need to design or implement your own master key provider\. If you specify a master key provider, the SDK creates a Default CMM for you based on the master key provider that you specify\. 

master key providers\-upper; in Java and Python are compatible with keyrings in the AWS Encryption SDK for C and the AWS Encryption SDK for JavaScript, subject to language constraints\. However, you must specify the same key material and use a keyring that is compatible with the master key provider\. For details, see [Keyring compatibility](choose-keyring.md#keyring-compatibility)\.

## Keyring \(C and JavaScript\)<a name="keyring"></a>

A *keyring* generates, encrypts, and decrypts data keys\. Each [keyring](choose-keyring.md#using-keyrings) is typically associated with a wrapping key or a service that provides and protects wrapping keys\. You can use the keyrings that the AWS Encryption SDK provides or write your own compatible custom keyrings\. Keyrings are supported only in the AWS Encryption SDK for C and the AWS Encryption SDK for JavaScript\. 

You can use a single keyring or combine keyrings of the same type or a different type into a *multi\-keyring*\. The [multi\-keyring](choose-keyring.md#use-multi-keyring) returns a copy of the data key encrypted by each of the wrapping keys in each of the keyrings that comprise the multi\-keyring\. When you use a multi\-keyring to encrypt data, you can decrypt the data using a keyring configured with any one of the wrapping keys in the multi\-keyring\.

For details about working with keyrings, see [Using keyrings](choose-keyring.md)\.

Keyrings in C and JavaScript are compatible with master key providers in the AWS Encryption SDK for Java and the AWS Encryption SDK for Python\. However, you must specify the same key material and use a keyring that is compatible with the master key provider, subject to language constraints\. Any minor incompatibility due to language constraints is explained in the topic about the language implementation\. For details, see [Keyring compatibility](choose-keyring.md#keyring-compatibility)\. 

## Algorithm suite<a name="crypto-algorithm"></a>

The AWS Encryption SDK supports several [algorithm suites](supported-algorithms.md)\. All of the supported suites use Advanced Encryption Standard \(AES\) as the primary algorithm, and combine it with other algorithms and values\. 

The AWS Encryption SDK establishes a recommended algorithm suite as the default for all encryption operations\. The default might change as standards and best practices improve\. You can specify an alternate algorithm suite in requests to encrypt data or when creating a [cryptographic materials manager \(CMM\)](#crypt-materials-manager), but unless an alternate is required for your situation, it is best to use the default\. The current default is AES\-GCM with an HMAC\-based extract\-and\-expand key derivation function \([HKDF](https://en.wikipedia.org/wiki/HKDF)\), Elliptic Curve Digital Signature Algorithm \(ECDSA\) signing, and a 256\-bit encryption key\. 

If you specify an algorithm suite, we recommend an algorithm suite that uses a [key derivation function](https://en.wikipedia.org/wiki/HKDF) and a message signing algorithm\. Algorithm suites that have neither feature are supported only for backward compatibility\.

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

When choosing an encryption context, remember that it is not a secret\. The encryption context is displayed in plaintext in the header of the [encrypted message](#message) that the SDK returns\. If you are using AWS Key Management Service, the encryption context also might appear in plaintext in audit records and logs, such as AWS CloudTrail\.

For examples of verifying an encryption context in your code, see:
+ C – [Encrypting and Decrypting Strings](c-examples.md#c-example-strings)
+ CLI – All [AWS Encryption SDK CLI examples](crypto-cli-examples.md) include an encryption context\. 
+ Java – [Strings](java-example-code.md#java-example-strings), [Byte Streams](java-example-code.md#java-example-streams)
+ JavaScript Node\.js – [kms\-simple\.ts](https://github.com/aws/aws-encryption-sdk-javascript/blob/master/modules/example-node/src/kms_simple.ts)
+ JavaScript Browser – [kms\-simple\.ts](https://github.com/aws/aws-encryption-sdk-javascript/blob/master/modules/example-browser/src/kms_simple.ts)
+ Python – [Using data key caching to encrypt messages](python-example-code.md#python-example-caching)

## Encrypted message<a name="message"></a>

When you encrypt data with the AWS Encryption SDK, it returns an encrypted message\.

An *encrypted message* is a portable [formatted data structure](message-format.md) that includes the encrypted data along with encrypted copies of the data keys, the algorithm ID, and, optionally, an encryption context and a message signature\. Encrypt operations in the AWS Encryption SDK return an encrypted message and decrypt operations take an encrypted message as input\. 

Combining the encrypted data and its encrypted data keys streamlines the decryption operation and frees you from having to store and manage encrypted data keys independently of the data that they encrypt\.

For technical information about the encrypted message, see [Encrypted Message Format](message-format.md)\.