# How the AWS Encryption SDK works<a name="how-it-works"></a>

The workflows in this section explain how the AWS Encryption SDK encrypts data and decrypts [encrypted messages](concepts.md#message)\. These workflows describes the basic process using the default features\. For details about defining and using custom components, see the GitHub repository for each supported [language implementation](programming-languages.md)\.

The AWS Encryption SDK uses envelope encryption to protect your data\. Each message is encrypted under a unique data key\. Then the data key is encrypted by the wrapping keys you specify\. To decrypt the encrypted message, the AWS Encryption SDK uses the wrapping keys you specify to decrypt at least one encrypted data key\. Then it can decrypt the ciphertext and return a plaintext message\.

Need help with the terminology we use in the AWS Encryption SDK? See [Concepts in the AWS Encryption SDK](concepts.md)\.

## How the AWS Encryption SDK encrypts data<a name="encrypt-workflow"></a>

The AWS Encryption SDK provides methods that encrypt strings, byte arrays, and byte streams\. For code examples, see the Examples topic in each [Programming languages](programming-languages.md) section\.

1. Create a [keyring](choose-keyring.md) \(or [master key provider](concepts.md#master-key-provider)\) that specifies the wrapping keys that protect your data\.

1. Pass the keyring and plaintext data to an encryption method\. We recommend that you pass in an optional, non\-secret [encryption context](concepts.md#encryption-context)\.

1. The encryption method asks the keyring for encryption materials\. The keyring returns unique data encryption keys for the message: one plaintext data key and one copy of that data key encrypted by the each of the specified wrapping keys\.

1. The encryption method uses the plaintext data key to encrypt the data, and then discards the plaintext data key\. If you provide an encryption context \(an AWS Encryption SDK [best practice](best-practices.md)\), the encryption method cryptographically binds the encryption context to the encrypted data\.

1. The encryption method returns an [encrypted message](concepts.md#message) that contains the encrypted data, the encrypted data keys, and other metadata, including the encryption context, if you used one\.

## How the AWS Encryption SDK decrypts an encrypted message<a name="decrypt-workflow"></a>

The AWS Encryption SDK provides methods that decrypt the [encrypted message](concepts.md#message) and return plaintext\. For code examples, see the Examples topic in each [Programming languages](programming-languages.md) section\.

The [keyring](choose-keyring.md) \(or [master key provider](concepts.md#master-key-provider)\) that decrypts the encrypted message must be compatible with the one used to encrypt the message\. One of its wrapping keys must be able to decrypt an encrypted data key in the encrypted message\. For information about compatibility with keyrings and master key providers, see [Keyring compatibility](keyring-compatibility.md)\.

1. Create a keyring or master key provider with wrapping keys that can decrypt your data\. You can use the same keyring that you provided to the encryption method or a different one\.

1. Pass the [encrypted message](concepts.md#message) and the keyring to a decryption method\.

1. The decryption method asks the keyring or master key provider to decrypt one of the encrypted data keys in the encrypted message\. It passes in information from the encrypted message, including the encrypted data keys\.

1. The keyring uses its wrapping keys to decrypt one of the encrypted data keys\. If it's successful, the response includes the plaintext data key\. If none of the wrapping keys specified by the keyring or master key provider can decrypt an encrypted data key, the decrypt call fails\.

1. The decryption method uses the plaintext data key to decrypt the data, discards the plaintext data key, and returns the plaintext data\.