# AWS Encryption SDK for \.NET<a name="dot-net"></a>

The AWS Encryption SDK for \.NET is a client\-side encryption library for developers who are writing applications in C\# and other \.NET programming languages\. It is supported on Windows, macOS, and Linux\.

All [programming language](programming-languages.md) implementations of the AWS Encryption SDK are fully interoperable\. For example, you can encrypt data with AWS Encryption SDK for \.NET and decrypt it with any [supported programming language implementation](programming-languages.md), including the [AWS Encryption CLI](crypto-cli.md), provided that you use [compatible keyrings and master key providers](keyring-compatibility.md)\.

The AWS Encryption SDK for \.NET differs from some of the other programming language implementations of the AWS Encryption SDK in the following ways:
+ No support for [data key caching](data-key-caching.md)
+ No support for streaming data
+ [No logging or stack traces](#dot-net-debugging) from the AWS Encryption SDK for \.NET
+ [Requires the AWS SDK for \.NET](#dot-net-install)

The AWS Encryption SDK for \.NET includes all of the security features introduced in versions 2\.0\.*x* and later of other language implementations of the AWS Encryption SDK\. However, if you are using the AWS Encryption SDK for \.NET to decrypt data that was encrypted by a pre\-2\.0\.*x* version another language implementation of the AWS Encryption SDK, you might need to adjust your [commitment policy](concepts.md#commitment-policy)\. For details, see [How to set your commitment policy](migrate-commitment-policy.md#migrate-commitment-step1)\.

The AWS Encryption SDK for \.NET is a product of the AWS Encryption SDK in [Dafny](https://github.com/dafny-lang/dafny/blob/master/README.md), a formal verification language in which you write specifications, the code to implement them, and the proofs to test them\. The result is a library that implements the features of the AWS Encryption SDK in a framework that assures functional correctness\.

**Learn More**
+ For examples showing how to configure options in the AWS Encryption SDK, such as specifying an alternate algorithm suite, limiting encrypted data keys, and using AWS KMS multi\-Region keys, see [Configuring the AWS Encryption SDK](configure.md)\.
+ For details about programming with the AWS Encryption SDK for \.NET, see the [https://github.com/aws/aws-encryption-sdk-dafny/blob/mainline/aws-encryption-sdk-net/](https://github.com/aws/aws-encryption-sdk-dafny/blob/mainline/aws-encryption-sdk-net/) directory of the aws\-encryption\-sdk\-dafny repository on GitHub\.

**Topics**
+ [Install and build](#dot-net-install)
+ [Debugging](#dot-net-debugging)
+ [AWS KMS keyrings](#net-kms-keyrings)
+ [Examples](dot-net-examples.md)

## Installing the AWS Encryption SDK for \.NET<a name="dot-net-install"></a>

The AWS Encryption SDK for \.NET is available as the [https://www.nuget.org/packages/AWS.EncryptionSDK/](https://www.nuget.org/packages/AWS.EncryptionSDK/) package in NuGet\. For details about installing and building the AWS Encryption SDK for \.NET, see the [README\.md](https://github.com/aws/aws-encryption-sdk-dafny/blob/mainline/aws-encryption-sdk-net/#readme) file in the `aws-encryption-sdk-net` repository\.

The AWS Encryption SDK for \.NET supports \.NET Framework 4\.5\.2 – 4\.8 only on Windows\. It supports \.NET Core 3\.0\+ and \.NET 5\.0 and later on all supported operating systems\.

The AWS Encryption SDK for \.NET requires the AWS SDK for \.NET even if you aren't using AWS Key Management Service \(AWS KMS\) keys\. It's installed with the NuGet package\. However, unless you are using AWS KMS keys, AWS Encryption SDK for \.NET does not require an AWS account, AWS credentials, or interaction with any AWS service\. For help setting up an AWS account if you need it, see [Using the AWS Encryption SDK with AWS KMS](getting-started.md)\.

## Debugging the AWS Encryption SDK for \.NET<a name="dot-net-debugging"></a>

The AWS Encryption SDK for \.NET does not generate any logs\. Exceptions in the AWS Encryption SDK for \.NET generate an exception message, but no stack traces\.

To help you debug, be sure to enable logging in the AWS SDK for \.NET\. The logs and error messages from the AWS SDK for \.NET can help you distinguish errors arising in the AWS SDK for \.NET from those in the AWS Encryption SDK for \.NET\. For help with AWS SDK for \.NET logging, see [AWSLogging](https://docs.aws.amazon.com/sdk-for-net/v3/developer-guide/net-dg-config-other.html#config-setting-awslogging) in the *AWS SDK for \.NET Developer Guide*\. \(To see the topic, expand the **Open to view \.NET Framework content** section\.\)

## AWS KMS keyrings in the AWS Encryption SDK for \.NET<a name="net-kms-keyrings"></a>

The basic AWS KMS keyrings in the AWS Encryption SDK for \.NET take only one KMS key\. They also require an AWS KMS client, which gives you an opportunity to configure the client for the AWS Region of the KMS key\. 

To create a AWS KMS keyring with one or more wrapping keys, use a multi\-keyring\. The AWS Encryption SDK for \.NET has a special multi\-keyring that takes one or more AWS KMS keys, and a standard multi\-keyring that takes one or more keyrings of any supported type\. Some programmers prefer to use a multi\-keyring method to create all of their keyrings, and the AWS Encryption SDK for \.NET supports that strategy\.

The AWS Encryption SDK for \.NET provides basic single\-key keyrings and multi\-keyrings for all typical use\-cases, including AWS KMS [multi\-Region keys](configure.md#config-mrks)\.

For example, to create a AWS KMS keyring with one AWS KMS key, you can use the `CreateAwsKmsKeyring()` method\. This example creates a default AWS KMS client for the Region that contains the specified key\.

```
// Instantiate the AWS Encryption SDK and material providers
var encryptionSdk = AwsEncryptionSdkFactory.CreateDefaultAwsEncryptionSdk();
var materialProviders =
    AwsCryptographicMaterialProvidersFactory.CreateDefaultAwsCryptographicMaterialProviders();

string keyArn = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";

// Instantiate the keyring input object
var kmsKeyringInput = new CreateAwsKmsKeyringInput
{
    KmsClient = new AmazonKeyManagementServiceClient(),
    KmsKeyId = keyArn
};

// Create the keyring
var keyring = materialProviders.CreateAwsKmsKeyring(kmsKeyringInput);
```

To create a keyring with one or more AWS KMS keys, use the `CreateAwsKmsMultiKeyring()` method\. This example uses two AWS KMS keys\. To specify one KMS key, use only the `Generator` parameter\. The `KmsKeyIds` parameter that specifies additional KMS keys is optional\.

The input for this keyring doesn't take an AWS KMS client\. Instead, the AWS Encryption SDK uses the default AWS KMS client for each Region represented by a KMS key in the keyring\. For example, if the KMS key identified by the value of the `Generator` parameter is in the US West \(Oregon\) Region \(`us-west-2`\), the AWS Encryption SDK creates a default AWS KMS client for the `us-west-2` Region\. If you need to customize the AWS KMS client, use the `CreateAwsKmsKeyring()` method\.

```
// Instantiate the AWS Encryption SDK and material providers
var encryptionSdk = AwsEncryptionSdkFactory.CreateDefaultAwsEncryptionSdk();
var materialProviders =
    AwsCryptographicMaterialProvidersFactory.CreateDefaultAwsCryptographicMaterialProviders();

string generatorKey = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";
List<string> additionalKeys = new List<string> { "arn:aws:kms:us-west-2:111122223333:key/0987dcba-09fe-87dc-65ba-ab0987654321" };

// Instantiate the keyring input object
var createEncryptKeyringInput = new CreateAwsKmsMultiKeyringInput
{
    Generator = generatorKey,
    KmsKeyIds = additionalKeys
};

var kmsEncryptKeyring = materialProviders.CreateAwsKmsMultiKeyring(createEncryptKeyringInput);
```