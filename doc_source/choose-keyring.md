# Using keyrings<a name="choose-keyring"></a>

The AWS Encryption SDK for C and AWS Encryption SDK for JavaScript use *keyrings* to perform [envelope encryption](https://docs.aws.amazon.com/crypto/latest/userguide/cryptography-concepts.html#define-envelope-encryption)\. Keyrings generate, encrypt, and decrypt the unique data keys that encrypt your data\. Keyrings also define the wrapping keys that are used to encrypt and decrypt the data keys\. You can use the keyrings that the AWS Encryption SDK provides or write your own compatible custom keyrings\.

You can use each keyring individually or combine keyrings into a [multi\-keyring](use-multi-keyring.md)\. Although most keyrings can generate, encrypt, and decrypt data keys, you might create a keyring that performs only one particular operation, such as a keyring that only generates data keys, and use that keyring in combination with others\.

We recommend that you use a keyring that protects your wrapping keys and performs cryptographic operations within a secure boundary, such as the AWS KMS keyring, which uses AWS KMS keys that never leave [AWS Key Management Service](https://docs.aws.amazon.com/kms/latest/developerguide/) \(AWS KMS\) unencrypted\. You can also write a keyring that uses wrapping keys that are stored in your hardware security modules \(HSMs\) or protected by other master key services\. For details, see the [Keyring Interface](https://github.com/awslabs/aws-encryption-sdk-specification/blob/master/framework/keyring-interface.md) topic in the *AWS Encryption SDK Specification*\. 

Keyrings play the role of [master keys](concepts.md#master-key) and [master key providers](concepts.md#master-key-provider) in the AWS Encryption SDK for Java, AWS Encryption SDK for Python, and the AWS Encryption CLI\. If you use different language implementations of the AWS Encryption SDK to encrypt and decrypt your data, be sure to use compatible keyrings and master key providers\. For details, see [Keyring compatibility](keyring-compatibility.md)\.

This topic explains how to use the keyring feature of the AWS Encryption SDK and how to choose a keyring\. For examples of creating and using keyrings, see the [C](c-language.md) and [JavaScript](javascript.md) topics\.

**Topics**
+ [How keyrings work](using-keyrings.md)
+ [Keyring compatibility](keyring-compatibility.md)
+ [Choosing a keyring](which-keyring.md)