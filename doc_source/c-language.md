# AWS Encryption SDK for C<a name="c-language"></a>

The AWS Encryption SDK for C provides a client\-side encryption library for developers who are writing applications in C\. It also serves as a foundation for implementations of the AWS Encryption SDK in higher\-level programming languages\.

Like all implementations of the AWS Encryption SDK, the AWS Encryption SDK for C offers advanced data protection features\. These include [envelope encryption](https://docs.aws.amazon.com/crypto/latest/userguide/cryptography-concepts.html#define-envelope-encryption), additional authenticated data \(AAD\), and secure, authenticated, symmetric key [algorithm suites](concepts.md#crypto-algorithm), such as 256\-bit AES\-GCM with key derivation and signing\.

All language\-specific implementations of the AWS Encryption SDK are fully interoperable\. For example, you can encrypt data with the AWS Encryption SDK for C and decrypt it with [any supported language implementation](programming-languages.md), including the [AWS Encryption CLI](crypto-cli.md)\.

The AWS Encryption SDK for C uses the AWS SDK for C\+\+ to interact with AWS Key Management Service \(AWS KMS\) so it can support the optional [AWS KMS keyring](choose-keyring.md#use-kms-keyring)\. However, the AWS Encryption SDK doesn't require AWS KMS or any other AWS service\.

**Learn More**
+ For details about programming with the AWS Encryption SDK for C, see the [C examples](c-examples.md), the [examples](https://github.com/aws/aws-encryption-sdk-c/tree/master/examples) in the [aws\-encryption\-sdk\-c repository](https://github.com/aws/aws-encryption-sdk-c/) on GitHub, and the [AWS Encryption SDK for C API documentation](https://aws.github.io/aws-encryption-sdk-c/html/)\.
+ For a discussion about how to use the AWS Encryption SDK for C to encrypt data so that you can decrypt it in multiple AWS Regions, see [How to decrypt ciphertexts in multiple regions with the AWS Encryption SDK in C](http://aws.amazon.com/blogs/security/how-to-decrypt-ciphertexts-multiple-regions-aws-encryption-sdk-in-c/) in the AWS Security Blog\.

**Topics**
+ [Install and build](c-language-installation.md)
+ [Using the C SDK](c-language-using.md)
+ [Examples](c-examples.md)