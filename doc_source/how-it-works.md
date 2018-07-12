# How the AWS Encryption SDK Works<a name="how-it-works"></a>

The AWS Encryption SDK uses *envelope encryption* to protect your data and the corresponding data keys\. For more information, see the following topics\.

**Topics**
+ [Symmetric Key Encryption](#symmetric-key-encryption)
+ [Envelope Encryption](#envelope-encryption)
+ [AWS Encryption SDK Encryption Workflows](#encryption-workflows)

## Symmetric Key Encryption<a name="symmetric-key-encryption"></a>

To encrypt data, the AWS Encryption SDK provides raw data, known as *plaintext data*, and a data key to an encryption algorithm\. The encryption algorithm uses those inputs to encrypt the data\. Then, the AWS Encryption SDK returns an [encrypted message](concepts.md#message) that includes the encrypted data and an encrypted copy of the data key\. 

To decrypt the encrypted message, the AWS Encryption SDK provides the encrypted message to a decryption algorithm that uses those inputs to return the plaintext data\.

Because the same data key is used to encrypt and decrypt the data, the operations are known as *symmetric key* encryption and decryption\. The following figure shows symmetric key encryption and decryption in the AWS Encryption SDK\.

![\[Symmetric key encryption and decryption\]](http://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/images/encryption-basics.png)

## Envelope Encryption<a name="envelope-encryption"></a>

The security of your encrypted data depends on protecting the data key that can decrypt it\. One accepted best practice for protecting the data key is to encrypt it\. To do this, you need another encryption key, known as a [master key](concepts.md#master-key)\. This practice of using a master key to encrypt data keys is known as *envelope encryption*\. Some of the benefits of envelope encryption include the following\.

**Protecting Data Keys**  
When you encrypt a data key, you don't have to worry about where to store it because the data key is inherently protected by encryption\. You can safely store the encrypted data key with the encrypted data\. The AWS Encryption SDK does this for you\. It saves the encrypted data and the encrypted data key together in an [encrypted message](concepts.md#message)\.

**Encrypting the Same Data Under Multiple Master Keys**  
Encryption operations can be time\-consuming, particularly when the data being encrypted are large objects\. Instead of reencrypting raw data multiple times with different keys, you can reencrypt only the data keys that protect the raw data\. 

**Combining the Strengths of Multiple Algorithms**  
In general, symmetric key encryption algorithms are faster and produce smaller ciphertexts than assymetric or *public key encryption*\. But, public key algorithms provide inherent separation of roles and easier key management\. You might want to combine the strengths of each\. For example, you might encrypt raw data with symmetric key encryption, and then encrypt the data key with public key encryption\.

The AWS Encryption SDK uses envelope encryption\. It encrypts your data with a data key\. Then, it encrypts the data key with a master key\. The AWS Encryption SDK returns the encrypted data and the encrypted data keys in a single encrypted message, as shown in the following diagram\. 

![\[Envelope encryption with the AWS Encryption SDK\]](http://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/images/envelope-encryption.png)

If you have multiple master keys, each of them can encrypt the plaintext data key\. Then, the AWS Encryption SDK returns an encrypted message that contains the encrypted data and the collection of encrypted data keys\. Any one of the master keys can decrypt one of the encrypted data keys, which can then decrypt the data\. 

When you use envelope encryption, you must protect your master keys from unauthorized access\. You can do this in one of the following ways:
+ Use a web service designed for this purpose, such as [AWS Key Management Service \(AWS KMS\)](https://aws.amazon.com/kms/)\.
+ Use a [hardware security module \(HSM\)](https://en.wikipedia.org/wiki/Hardware_security_module) such as those offered by [AWS CloudHSM](https://aws.amazon.com/cloudhsm/)\.
+ Use your existing key management tools\.

If you don't have a key management system, we recommend AWS KMS\. The AWS Encryption SDK integrates with AWS KMS to help you protect and use your master keys\. You can also use the AWS Encryption SDK with other master key providers, including custom ones that you define\. Even if you don't use AWS, you can still use this AWS Encryption SDK\.

## AWS Encryption SDK Encryption Workflows<a name="encryption-workflows"></a>

The workflows in this section explain how the SDK encrypts data and decrypts [encrypted messages](concepts.md#message)\. They show how the SDK uses the components that you create, including the [cryptographic materials manager](concepts.md#crypt-materials-manager) \(CMM\), [master key provider](concepts.md#master-key-provider), and [master key](concepts.md#master-key), to respond to encryption and decryption requests from your application\.

### How the SDK Encrypts Data<a name="encrypt-workflow"></a>

The SDK provides methods that encrypt strings, byte arrays, and byte streams\. For code examples showing calls to encrypt and decrypt strings and byte streams in each supported programming languages, see the examples in the [Programming Languages](programming-languages.md) section\.

1. Your application passes plaintext data to one of the encryption methods\. 

   To indicate the source of the [data keys](concepts.md#DEK) that you want to use to encrypt your data, your request specifies a cryptographic materials manager \(CMM\) or a master key provider\. \(If you specify a master key provider, the AWS Encryption SDK creates a default CMM that interacts with your chosen master key provider\.\)

1. The encryption method asks the CMM for data keys \(and related cryptographic material\)\.

1. The CMM gets a [master key](concepts.md#master-key) from its master key provider\.
**Note**  
If you are using AWS Key Management Service \(AWS KMS\), the KMS master key object that is returned identifies the CMK, but the actual CMK never leaves the AWS KMS service\.

1. The CMM asks the master key to generate a data key\. The master key returns two copies of the data key, one in plaintext and one encrypted under the master key\. 

1. The CMM returns the plaintext and encrypted data keys to the encryption method\.

1. The encryption method uses the plaintext data key to encrypt the data, and then discards the plaintext data key\.

1. The encryption method returns an [encrypted message](concepts.md#message) that contains the encrypted data and the encrypted data key\.

### How the SDK Decrypts an Encrypted Message<a name="decrypt-workflow"></a>

The SDK provides methods that decrypt an encrypted message and return plaintext strings, byte arrays, or byte streams\. For code examples in each supported programming languages, see the examples in the [Programming Languages](programming-languages.md) section\.

1. Your application passes an encrypted message to a decryption method\.

   To indicate the source of the [data keys](concepts.md#DEK) that were used to encrypt your data, your request specifies a cryptographic materials manager \(CMM\) or a master key provider\. \(If you specify a master key provider, the AWS Encryption SDK creates a default CMM that interacts with the specified master key provider\.\)

1. The decryption method extracts the encrypted data key from the encrypted message\. Then, it asks the cryptographic materials manager \(CMM\) for a data key to decrypt the encrypted data key\.

1. The CMM asks its master key provider for a master key that can decrypt the encrypted data key\. 

1. The CMM uses the master key to decrypt the encrypted data key\. Then, it returns the plaintext data key to the decryption method\.

1. The decryption method uses the plaintext data key to decrypt the data, then discards the plaintext data key\. 

1. The decryption method returns the plaintext data\.
