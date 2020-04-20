# How the AWS Encryption SDK works<a name="how-it-works"></a>

The AWS Encryption SDK uses *envelope encryption* to protect your data and the corresponding data keys\. 

**Topics**
+ [Symmetric key encryption](#symmetric-key-encryption)
+ [Envelope encryption](#envelope-encryption)
+ [AWS Encryption SDK encryption workflows](#encryption-workflows)

## Symmetric key encryption<a name="symmetric-key-encryption"></a>

To encrypt data, the AWS Encryption SDK submits an encryption key, known as a *data key* and the plaintext data that you provide to an encryption algorithm\. The encryption algorithm uses those inputs to encrypt the data\. Then, the AWS Encryption SDK returns an [encrypted message](concepts.md#message) that includes the encrypted data, an encrypted copy of the data key, and the [encryption context](concepts.md#encryption-context), if you used one\. 

To decrypt the encrypted message, the AWS Encryption SDK submits the data key and the encrypted message that the SDK returned to a decryption algorithm\. The decryption algorithm uses those inputs to return the plaintext data\.

Because the same data key is used to encrypt and decrypt the data, the operations are known as *symmetric key* encryption and decryption\. The following figure shows symmetric key encryption and decryption in the AWS Encryption SDK\.

![\[Symmetric key encryption and decryption\]](http://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/images/encryption-basics.png)

## Envelope encryption<a name="envelope-encryption"></a>

The security of your encrypted data depends in part on protecting the data key that can decrypt it\. One accepted best practice for protecting the data key is to encrypt it\. To do this, you need another encryption key, known as a [master key](concepts.md#master-key)\. This practice of using a master key to encrypt data keys is known as *envelope encryption*\. Some of the benefits of envelope encryption include the following\.

**Protecting data keys**  
When you encrypt a data key, you don't have to worry about where to store it because the data key is inherently protected by encryption\. You can safely store the encrypted data key with the encrypted data\. The AWS Encryption SDK does this for you\. It saves the encrypted data and the encrypted data key together in an [encrypted message](concepts.md#message)\.

**Encrypting the same data under multiple master keys**  
Encryption operations can be time\-consuming, particularly when the data being encrypted are large objects\. Instead of reencrypting raw data multiple times with different keys, you can reencrypt only the data keys that protect the raw data\. 

**Combining the strengths of multiple algorithms**  
In general, symmetric key encryption algorithms are faster and produce smaller ciphertexts than asymmetric or *public key encryption*\. But public key algorithms provide inherent separation of roles and easier key management\. You might want to combine the strengths of each\. For example, you might encrypt raw data with symmetric key encryption, and then encrypt the data key with public key encryption\.

The AWS Encryption SDK uses envelope encryption\. It encrypts your data with a data key\. Then, it encrypts the data key with a master key\. The AWS Encryption SDK returns the encrypted data and the encrypted data keys in a single encrypted message, as shown in the following diagram\. 

![\[Envelope encryption with the AWS Encryption SDK\]](http://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/images/envelope-encryption.png)

If you have multiple master keys or wrapping keys, each of them can encrypt the plaintext data key\. Then, the AWS Encryption SDK returns an encrypted message that contains the encrypted data and the collection of encrypted data keys\. Any one of the master keys can decrypt one of the encrypted data keys, which can then decrypt the data\. 

When you use envelope encryption, you must protect your master keys from unauthorized access\. You can do this in one of the following ways:
+ Use a web service designed for this purpose, such as [AWS Key Management Service \(AWS KMS\)](https://aws.amazon.com/kms/)\.
+ Use a [hardware security module \(HSM\)](https://en.wikipedia.org/wiki/Hardware_security_module) such as those offered by [AWS CloudHSM](https://aws.amazon.com/cloudhsm/)\.
+ Use your existing key management tools\.

If you don't have a key management system, we recommend AWS KMS\. The AWS Encryption SDK integrates with AWS KMS to help you protect and use your master keys\. You can also use the AWS Encryption SDK with other [keyrings](concepts.md#keyring) and[ master key providers](concepts.md#master-key-provider), including custom ones that you define\. Even if you don't use AWS, you can still use this AWS Encryption SDK\.

## AWS Encryption SDK encryption workflows<a name="encryption-workflows"></a>

The workflows in this section explain how the SDK encrypts data and decrypts [encrypted messages](concepts.md#message)\. They show how the SDK uses the components that you create, including a [master key provider](concepts.md#master-key-provider) and [master key](concepts.md#master-key) \(Java and Python\) or [keyring](concepts.md#keyring) \(C or JavaScript\) to respond to encryption and decryption requests from your application\.

### How the SDK encrypts data<a name="encrypt-workflow"></a>

The SDK provides methods that encrypt strings, byte arrays, and byte streams\. For code examples that encrypt and decrypt strings and byte streams in each supported programming languages, see the examples in the [Programming languages](programming-languages.md) section\.

1. Your application passes plaintext data to one of the encryption methods\. We also recommend that you pass in an optional, non\-secret [encryption context](concepts.md#encryption-context)\. 

1. The encryption method asks the [cryptographic materials manager](concepts.md#crypt-materials-manager) \(CMM\) for encryption materials\. 

   The CMM is a component that assembles the data keys, signing keys, and other encryption materials\. The AWS Encryption SDK provides a default CMM and a CMM that manages [data key caching](data-key-caching.md)\. You can also create custom CMMs for your applications\. Typically, you don't create a default CMM explicitly\. When you specify a master key provider or keyring, the AWS Encryption SDK creates a default CMM for you\.

1. The CMM requests encryption materials from the master key provider or keyring\. The response includes a plaintext data key and the same data key encrypted under the master keys\. The CMM returns these encryption materials to the encryption method\.

1. The encryption method uses the plaintext data key to encrypt the data, and then discards the plaintext data key\. If you provided an encryption context, the encryption method also cryptographically binds the encryption context to the encrypted data\.

1. The encryption method returns an [encrypted message](concepts.md#message) that contains the encrypted data, the encrypted data key, and other metadata, including the encryption context, if one was used\.

### How the SDK decrypts an encrypted message<a name="decrypt-workflow"></a>

The SDK provides methods that decrypt the [encrypted message](concepts.md#message) and return plaintext strings, byte arrays, or byte streams\. For code examples in each supported programming languages, see the examples in the [Programming languages](programming-languages.md) section\.

You must use the same master key provider and keyring to decrypt that you used to encrypt, or a compatible one\.

1. Your application passes an [encrypted message](concepts.md#message) to a decryption method\.

   To indicate the source of the [data keys](concepts.md#DEK) that were used to encrypt your data, your request specifies a cryptographic materials manager \(CMM\), or a master key provider or keyring\. If you specify a master key provider or keyring, the AWS Encryption SDK creates a default CMM for you\.

1. The decryption method asks the CMM for cryptographic materials to decrypt the encrypted message\. It passes in information from the encrypted message, including the encrypted data keys\.

1. In Java and Python, to get decryption materials, the Default CMM asks its master key provider for a master key that can decrypt one of the encrypted data keys\. Other CMMs might use different techniques to get the decryption materials\. In C and JavaScript, the CMM asks the keyring for decryption materials\. The keyring uses its wrapping keys to decrypt one of the encrypted data keys\. 

   The response includes the decryption materials, including the plaintext data key\.

1. The decryption method uses the plaintext data key to decrypt the data, then discards the plaintext data key\. 

1. The decryption method returns the plaintext data\.