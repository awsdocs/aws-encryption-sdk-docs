# How the AWS Encryption SDK works<a name="how-it-works"></a>

The AWS Encryption SDK uses [envelope encryption](concepts.md#envelope-encryption) to protect your data\. The workflows in this section explain how the SDK encrypts data and decrypts [encrypted messages](concepts.md#message)\. They show how the SDK uses the components that you create, including a [master key provider](concepts.md#master-key-provider) and [master key](concepts.md#master-key) \(Java and Python\) or [keyring](concepts.md#keyring) \(C or JavaScript\) to respond to encryption and decryption requests from your application\. 

These workflows describe the basic process using the default features\. For details about defining and using custom components, see the GitHub repository for each supported [language implementation](programming-languages.md)\.

Need help with the terminology we use in the AWS Encryption SDK, such as [envelope encryption](concepts.md#envelope-encryption) and [symmetric key encryption](concepts.md#symmetric-key-encryption)? See [Concepts in the AWS Encryption SDK](concepts.md)\.

**Topics**
+ [How the AWS Encryption SDK encrypts data](#encrypt-workflow)
+ [How the AWS Encryption SDK decrypts an encrypted message](#decrypt-workflow)

## How the AWS Encryption SDK encrypts data<a name="encrypt-workflow"></a>

The AWS Encryption SDK provides methods that encrypt strings, byte arrays, and byte streams\. For code examples that encrypt and decrypt strings and byte streams in each supported programming languages, see the examples in the [Programming languages](programming-languages.md) section\.

1. Your application passes your plaintext data and a [keyring](concepts.md#keyring) or [master key provider](concepts.md#master-key-provider) to an encryption method\. We also recommend that you pass in an optional, non\-secret [encryption context](concepts.md#encryption-context)\. 

1. The encryption method asks the [cryptographic materials manager](concepts.md#crypt-materials-manager) \(CMM\) for encryption materials\. 

   The CMM assembles the data key, signing keys, and other encryption materials\. The AWS Encryption SDK provides a default CMM and a CMM that manages [data key caching](data-key-caching.md)\. You can also create custom CMMs for your applications\. Typically, you don't create a default CMM explicitly\. When you specify a master key provider or keyring, the AWS Encryption SDK creates a default CMM for you\.

1. The CMM requests encryption materials from the keyring or master key provider\. The response includes a unique plaintext data key and the same data key encrypted under each of the wrapping keys in your keyring or master key provider\. The CMM returns these encryption materials to the encryption method\.

1. The encryption method uses the plaintext data key to encrypt the data, and then discards the plaintext data key\. If you provided an encryption context, the encryption method also cryptographically binds the encryption context to the encrypted data\.

1. The encryption method returns an [encrypted message](concepts.md#message) that contains the encrypted data, the encrypted data key, and other metadata, including the encryption context, if one was used\.

## How the AWS Encryption SDK decrypts an encrypted message<a name="decrypt-workflow"></a>

The AWS Encryption SDK provides methods that decrypt the [encrypted message](concepts.md#message) and return plaintext strings, byte arrays, or byte streams\. For code examples in each supported programming languages, see the examples in the [Programming languages](programming-languages.md) section\.

1. Create a keyring or master key provider with the wrapping keys that can decrypt your data\. You can use the same keyring or master key provider that you provided to the encryption method or [a compatible one](choose-keyring.md#keyring-compatibility)\. To be successful, your keyring or master key provider must include at least one wrapping key that can decrypt an encrypted data key\.

1. Your application passes the [encrypted message](concepts.md#message) and your keyring or master key provider to a decryption method\.

   You can specify a CMM, but if you don't, the AWS Encryption SDK creates a default CMM for you\.

1. The decryption method asks the CMM for cryptographic materials to decrypt the encrypted message\. It passes information from the encrypted message to the CMM, including the encrypted data keys\.

1. In Java and Python, to get decryption materials, the Default CMM asks its master key provider for a master key that can decrypt one of the encrypted data keys\. Other CMMs might use different techniques to get the decryption materials\. In C and JavaScript, the CMM asks the keyring for decryption materials\. The keyring uses its wrapping keys to decrypt one of the encrypted data keys\. The response includes the decryption materials, including the plaintext data key\.

   Beginning in [version 1\.7\.*x*](about-versions.md#version-1.7) of the AWS Encryption SDK, if you use an AWS Key Management Service \(AWS KMS\) keyring or master key provider, the AWS Encryption SDK always passes the key ARN of the CMK wrapping key to the `KeyId` parameter of the AWS KMS [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operation\. This is an AWS KMS best practice that assures that you decrypt the encrypted data key with the wrapping key you intend to use\.

1. The decryption method uses the plaintext data key to decrypt the data, then discards the plaintext data key\. 

1. The decryption method returns the plaintext data\.