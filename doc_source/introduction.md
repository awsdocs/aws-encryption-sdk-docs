# What Is the AWS Encryption SDK?<a name="introduction"></a>

The AWS Encryption SDK is an encryption library that helps make it easier for you to implement encryption best practices in your application\. It enables you to focus on the core functionality of your application, rather than on how to best encrypt and decrypt your data\.

The AWS Encryption SDK answers questions like the following for you:
+ Which encryption algorithm should I use?
+ How, or in which mode, should I use that algorithm?
+ How do I generate the encryption key?
+ How do I protect the encryption key, and where should I store it?
+ How can I make my encrypted data portable?
+ How do I ensure that the intended recipient can read my encrypted data?
+ How can I ensure my encrypted data is not modified between the time it is written and when it is read?

Without the AWS Encryption SDK, you might spend more effort on building an encryption solution than on the core functionality of your application\. The AWS Encryption SDK answers these questions by providing the following things\.

**A Default Implementation that Adheres to Cryptography Best Practices**  
By default, the AWS Encryption SDK generates a unique data key for each data object that it encrypts\. This follows the cryptography best practice of using unique data keys for each encryption operation\.  
The AWS Encryption SDK encrypts your data using a secure, authenticated, symmetric key algorithm\. For more information, see [Supported Algorithm Suites in the AWS Encryption SDK](supported-algorithms.md)\.

**A Framework for Protecting Data Keys with Master Keys**  
The AWS Encryption SDK protects the data keys that encrypt your data by encrypting them under one or more master keys\. By providing a framework to encrypt data keys with more than one master key, the AWS Encryption SDK helps make your encrypted data portable\.   
For example, you can encrypt data under multiple AWS Key Management Service \(AWS KMS\) customer master keys \(CMKs\), each in a different AWS Region\. Then you can copy the encrypted data to any of the regions and use the CMK in that region to decrypt it\. You can also encrypt data under a CMK in AWS KMS and a master key in an on\-premises HSM, enabling you to later decrypt the data even if one of the options is unavailable\.

**A Formatted Message that Stores Encrypted Data Keys with the Encrypted Data**  
The AWS Encryption SDK stores the encrypted data and encrypted data key together in an [encrypted message](concepts.md#message) that uses a defined data format\. This means you don't need to keep track of or protect the data keys that encrypt your data because the AWS Encryption SDK does it for you\.

With the AWS Encryption SDK, you define a [master key provider](concepts.md#master-key-provider) that returns one or more [master keys](concepts.md#master-key)\. Then you encrypt and decrypt your data using straightforward methods provided by the AWS Encryption SDK\. The AWS Encryption SDK does the rest\.

## Where to find more information<a name="intro-see-also"></a>

If you're looking for more information about the AWS Encryption SDK and client\-side encryption, try these sources\.
+ To get started quickly, see [Getting Started](getting-started.md)\.
+ For information about how this SDK works, see [How the SDK Works](how-it-works.md)\.
+ For help with the terms and concepts used in this SDK, see [Concepts in the AWS Encryption SDK](concepts.md)\.
+ For detailed technical information, see the [AWS Encryption SDK Reference](reference.md)\.
+ For help with questions about using the AWS Encryption SDK, read and post on the [AWS Key Management Service \(KMS\) Discussion Forum](https://forums.aws.amazon.com/forum.jspa?forumID=182) that the AWS Encryption SDK shares with KMS\.

For information about implementations of the AWS Encryption SDK in different programming languages\.
+ **Java**: See [AWS Encryption SDK for Java](java.md), the AWS Encryption SDK [Javadocs](https://awslabs.github.io/aws-encryption-sdk-java/javadoc/), and the [aws\-encryption\-sdk\-java](https://github.com/awslabs/aws-encryption-sdk-java) repository on GitHub\.
+ **Python**: See [AWS Encryption SDK for Python](python.md), the AWS Encryption SDK [Python documentation](http://aws-encryption-sdk-python.readthedocs.io/en/latest/), and the [aws\-encryption\-sdk\-python](https://github.com/awslabs/aws-encryption-sdk-python) repository on GitHub\.
+ **Command Line Interface**: See [AWS Encryption SDK Command Line Interface](crypto-cli.md), [Read the Docs](http://aws-encryption-sdk-cli.readthedocs.io/en/latest/) for the AWS Encryption CLI, and the [aws\-encryption\-sdk\-cli](https://github.com/awslabs/aws-encryption-sdk-cli/) repository on GitHub\.

If you have questions or comments about this guide, let us know\! Use the feedback links in the lower right corner of this page\.

The AWS Encryption SDK is provided for free under the Apache license\.