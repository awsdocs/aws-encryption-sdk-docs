# Installing the AWS Encryption SDK for C<a name="c-language-installation"></a>

You can find detailed instructions for installing and building the AWS Encryption SDK for C in the [README file](https://github.com/aws/aws-encryption-sdk-c/#readme) of the [aws\-encryption\-sdk\-c](https://github.com/aws/aws-encryption-sdk-c/) repository\. It includes instructions for building on Amazon Linux, Ubuntu, macOS, and Windows platforms\. 

Before you begin, decide whether you want to use [AWS KMS keyrings](use-kms-keyring.md) in the AWS Encryption SDK\. If you use an AWS KMS keyring, you need to install the AWS SDK for C\+\+\. The AWS SDK is required to interact with [AWS Key Management Service](https://docs.aws.amazon.com/kms/latest/developerguide/) \(AWS KMS\)\. When you use AWS KMS keyrings, the AWS Encryption SDK uses AWS KMS to generate and protect the encryption keys that protect your data\. 

You do not need to install the AWS SDK for C\+\+ if you are using another keyring type, such as a raw AES keyring, a raw RSA keyring, or a multi\-keyring that doesn't include an AWS KMS keyring\. However, when using a raw keyring type, you need to generate and protect your own raw wrapping keys\.

For help deciding which keyring types to use, see [Choosing a keyring](which-keyring.md)\.

If you're having trouble with your installation, [file an issue](https://github.com/aws/aws-encryption-sdk-c/issues) in the `aws-encryption-sdk-c` repository or use any of the feedback links on this page\.