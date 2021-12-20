# How keyrings work<a name="using-keyrings"></a>

When you encrypt data, the AWS Encryption SDK asks the keyring for encryption materials\. The keyring returns a plaintext data key and a copy of the data key that's encrypted by each of the wrapping keys in the keyring\. The AWS Encryption SDK uses the plaintext key to encrypt the data, and then destroys the plaintext data key\. Then, the AWS Encryption SDK returns an [encrypted message](concepts.md#message) that includes the encrypted data keys and the encrypted data\.

![\[Encrypting with a keyring with multiple wrapping keys.\]](http://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/images/keyring-encrypt.png)

When you decrypt data, you can use the same keyring that you used to encrypt the data, or a different one\. To decrypt the data, a decryption keyring must include \(or have access to\) at least one wrapping key in the encryption keyring\. 

The AWS Encryption SDK passes the encrypted data keys from the encrypted message to the keyring, and asks the keyring to decrypt any one of them\. The keyring uses its wrapping keys to decrypt one of the encrypted data keys and returns a plaintext data key\. The AWS Encryption SDK uses the plaintext data key to decrypt the data\. If none of the wrapping keys in the keyring can decrypt any of the encrypted data keys, the decrypt operation fails\.

![\[Decrypting with a keyring.\]](http://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/images/keyring-decrypt.png)

You can use a single keyring or also combine keyrings of the same type or a different type into a [multi\-keyring](use-multi-keyring.md)\. When you encrypt data, the multi\-keyring returns a copy of the data key encrypted by all of the wrapping keys in all of the keyrings that comprise the multi\-keyring\. You can decrypt the data using a keyring with any one of the wrapping keys in the multi\-keyring\.