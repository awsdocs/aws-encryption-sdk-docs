# What is the AWS Encryption SDK?<a name="introduction"></a>

The AWS Encryption SDK is a client\-side encryption library designed to make it easy for everyone to encrypt and decrypt data using industry standards and best practices\. It enables you to focus on the core functionality of your application, rather than on how to best encrypt and decrypt your data\. The AWS Encryption SDK is provided free of charge under the Apache 2\.0 license\.

The AWS Encryption SDK answers questions like the following for you:
+ Which encryption algorithm should I use?
+ How, or in which mode, should I use that algorithm?
+ How do I generate the encryption key?
+ How do I protect the encryption key, and where should I store it?
+ How can I make my encrypted data portable?
+ How do I ensure that the intended recipient can read my encrypted data?
+ How can I ensure my encrypted data is not modified between the time it is written and when it is read?

With the AWS Encryption SDK, you define a [master key provider](concepts.md#master-key-provider) \(Java or Python\) or a [keyring](concepts.md#keyring) \(C or JavaScript\) that determines which master keys you use to protect your data\. Then you encrypt and decrypt your data using straightforward methods provided by the AWS Encryption SDK\. The AWS Encryption SDK does the rest\.

Without the AWS Encryption SDK, you might spend more effort on building an encryption solution than on the core functionality of your application\. The AWS Encryption SDK answers these questions by providing the following things\.

**A default implementation that adheres to cryptography best practices**  
By default, the AWS Encryption SDK generates a unique data key for each data object that it encrypts\. This follows the cryptography best practice of using unique data keys for each encryption operation\.  
The AWS Encryption SDK encrypts your data using a secure, authenticated, symmetric key algorithm\. For more information, see [Supported algorithm suites in the AWS Encryption SDK](supported-algorithms.md)\.

**A framework for protecting data keys with master keys**  
The AWS Encryption SDK protects the data keys that encrypt your data by encrypting them under one or more master keys\. By providing a framework to encrypt data keys with more than one master key, the AWS Encryption SDK helps make your encrypted data portable\.   
For example, you can encrypt data under multiple AWS KMS keys, each in a different AWS Region\. Then you can copy the encrypted data to any of the regions and use the AWS KMS key in that region to decrypt it\. You can also encrypt data under an AWS KMS key in AWS KMS and a master key in an on\-premises HSM, enabling you to later decrypt the data even if one of the options is unavailable\.

**A formatted message that stores encrypted data keys with the encrypted data**  
The AWS Encryption SDK stores the encrypted data and encrypted data key together in an [encrypted message](concepts.md#message) that uses a defined data format\. This means you don't need to keep track of or protect the data keys that encrypt your data because the AWS Encryption SDK does it for you\.

Some language implementations of the AWS Encryption SDK require an AWS SDK, but the AWS Encryption SDK doesn't require an AWS account and it doesn't depend on any AWS service\. You need an AWS account only if you choose to use [AWS KMS keys](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#kms-keys) to protect your data\.

## Developed in open\-source repositories<a name="esdk-repos"></a>

The AWS Encryption SDK is developed in open\-source repositories on GitHub\. You can use these repositories to view the code, read and submit issues, and find information that is specific to your language implementation\.
+ AWS Encryption SDK for C — [aws\-encryption\-sdk\-c](https://github.com/aws/aws-encryption-sdk-c/)
+ AWS Encryption CLI — [aws\-encryption\-sdk\-cli](https://github.com/aws/aws-encryption-sdk-cli/)
+ AWS Encryption SDK for Java — [aws\-encryption\-sdk\-java](https://github.com/aws/aws-encryption-sdk-java/)
+ AWS Encryption SDK for JavaScript — [aws\-encryption\-sdk\-javascript](https://github.com/aws/aws-encryption-sdk-javascript/)
+ AWS Encryption SDK for Python — [aws\-encryption\-sdk\-python](https://github.com/aws/aws-encryption-sdk-python/)

## Compatibility with encryption libraries and services<a name="intro-compatibility"></a>

The AWS Encryption SDK is supported in several [programming languages](programming-languages.md)\. All language implementations are interoperable\. You can encrypt with one language implementation and decrypt with another\. Interoperability might be subject to language constraints\. If so, these constraints are described in the topic about the language implementation\. Also, when encrypting and decrypting, you must use compatible keyrings, or master keys and master key providers\. For details, see [Keyring compatibility](keyring-compatibility.md)\.

However, the AWS Encryption SDK cannot interoperate with other libraries\. Because each library returns encrypted data in a different format, you cannot encrypt with one library and decrypt with another\.

**DynamoDB Encryption Client and Amazon S3 client\-side encryption**  <a name="ESDK-DDBEC"></a>
The AWS Encryption SDK cannot decrypt data encrypted by the [DynamoDB Encryption Client](https://docs.aws.amazon.com/dynamodb-encryption-client/latest/devguide/) or [Amazon S3 client\-side encryption](https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingClientSideEncryption.html)\. And these libraries cannot decrypt the [encrypted message](concepts.md#message) the AWS Encryption SDK returns\. 

**AWS Key Management Service \(AWS KMS\)**  <a name="ESDK-KMS"></a>
The AWS Encryption SDK can use [AWS KMS keys](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#master_keys) and [data keys](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#data-keys) to protect your data\. For example, you can configure the AWS Encryption SDK to encrypt your data under one or more AWS KMS keys in your AWS account\. However, you must use the AWS Encryption SDK to decrypt that data\.   
The AWS Encryption SDK cannot decrypt the ciphertext that the AWS KMS [Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html) or [ReEncrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_ReEncrypt.html) operations return\. Similarly, the AWS KMS [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operation cannot decrypt the [encrypted message](concepts.md#message) the AWS Encryption SDK returns\.  
The AWS Encryption SDK supports only [symmetric encryption KMS keys](https://docs.aws.amazon.com/kms/latest/developerguide/symm-asymm-concepts.html#symmetric-cmks)\. You cannot use an [asymmetric KMS key](https://docs.aws.amazon.com/kms/latest/developerguide/symm-asymm-concepts.html#asymmetric-cmks) for encryption or signing in the AWS Encryption SDK\. The AWS Encryption SDK generates its own ECDSA signing keys for [algorithm suites](supported-algorithms.md) that sign messages\.

For help deciding which library or service to use, see [How to Choose an Encryption Tool or Service](https://docs.aws.amazon.com/crypto/latest/userguide/awscryp-overview.html) in *AWS Cryptographic Services and Tools*\.

## Support and maintenance<a name="support"></a>

The AWS Encryption SDK uses the same [maintenance policy](https://docs.aws.amazon.com/sdkref/latest/guide/maint-policy.html) that the AWS SDK and Tools use, including its versioning and life\-cycle phases\. As a [best practice](best-practices.md), we recommend that you use the latest available version of the AWS Encryption SDK for your programming language, and upgrade as new versions are released\. When a version requires significant changes, such as the upgrade from AWS Encryption SDK versions earlier than 1\.7\.*x* to versions 2\.0\.*x* and later, we provide [detailed instructions](migration.md) to help you\.

Each programming language implementation of the AWS Encryption SDK is developed in a separate open\-source GitHub repository\. The life\-cycle and support phase of each version is likely to vary among repositories\. For example, a given version of the AWS Encryption SDK might be in the general availability \(full support\) phase in one programming language, but the end\-of\-support phase in a different programming language\. We recommend that you use a fully supported version whenever possible and avoid versions that are no longer supported\.

To find the life\-cycle phase of AWS Encryption SDK versions for your programming language, see the `SUPPORT_POLICY.rst` file in each AWS Encryption SDK repository\.
+ AWS Encryption SDK for C — [SUPPORT\_POLICY\.rst](https://github.com/aws/aws-encryption-sdk-c/blob/master/SUPPORT_POLICY.rst)
+ AWS Encryption CLI — [SUPPORT\_POLICY\.rst](https://github.com/aws/aws-encryption-sdk-cli/blob/master/SUPPORT_POLICY.rst)
+ AWS Encryption SDK for Java — [SUPPORT\_POLICY\.rst](https://github.com/aws/aws-encryption-sdk-java/blob/master/SUPPORT_POLICY.rst)
+ AWS Encryption SDK for JavaScript — [SUPPORT\_POLICY\.rst](https://github.com/aws/aws-encryption-sdk-javascript/blob/master/SUPPORT_POLICY.rst)
+ AWS Encryption SDK for Python — [SUPPORT\_POLICY\.rst](https://github.com/aws/aws-encryption-sdk-python/blob/master/SUPPORT_POLICY.rst)

For more information, see [Versions of the AWS Encryption SDK](about-versions.md) and [AWS SDKs and Tools maintenance policy](https://docs.aws.amazon.com/sdkref/latest/guide/maint-policy.html) in the AWS SDKs and Tools Reference Guide\.

## Learning more<a name="intro-see-also"></a>

For more information about the AWS Encryption SDK and client\-side encryption, try these sources\.
+ To get started quickly, see [Getting started](getting-started.md)\.
+ For best practice guidelines, see [Best practices for the AWS Encryption SDK](best-practices.md)\.
+ For information about how this SDK works, see [How the SDK works](how-it-works.md)\.
+ For help with the terms and concepts used in this SDK, see [Concepts in the AWS Encryption SDK](concepts.md)\.
+ For detailed technical information, see the [AWS Encryption SDK reference](reference.md)\.
+ For the technical specification for the AWS Encryption SDK, see the [AWS Encryption SDK Specification](https://github.com/awslabs/aws-encryption-sdk-specification/) in GitHub\.
+ For answers to your questions about using the AWS Encryption SDK, read and post on the [AWS Crypto Tools Discussion Forum](https://forums.aws.amazon.com/forum.jspa?forumID=302)\.

For information about implementations of the AWS Encryption SDK in different programming languages\.
+ **C**: See [AWS Encryption SDK for C](c-language.md), the AWS Encryption SDK [C documentation](https://aws.github.io/aws-encryption-sdk-c/html/), and the [aws\-encryption\-sdk\-c](https://github.com/aws/aws-encryption-sdk-c/) repository on GitHub\.
+ **Command Line Interface**: See [AWS Encryption SDK command line interface](crypto-cli.md), [Read the Docs](https://aws-encryption-sdk-cli.readthedocs.io/en/latest/) for the AWS Encryption CLI, and the [aws\-encryption\-sdk\-cli](https://github.com/aws/aws-encryption-sdk-cli/) repository on GitHub\.
+ **Java**: See [AWS Encryption SDK for Java](java.md), the AWS Encryption SDK [Javadoc](https://aws.github.io/aws-encryption-sdk-java/), and the [aws\-encryption\-sdk\-java](https://github.com/aws/aws-encryption-sdk-java/) repository on GitHub\.

  **JavaScript**: See [AWS Encryption SDK for JavaScript](javascript.md) and the [aws\-encryption\-sdk\-javascript](https://github.com/aws/aws-encryption-sdk-javascript/) repository on GitHub\. 
+ **Python**: See [AWS Encryption SDK for Python](python.md), the AWS Encryption SDK [Python documentation](https://aws-encryption-sdk-python.readthedocs.io/en/latest/), and the [aws\-encryption\-sdk\-python](https://github.com/aws/aws-encryption-sdk-python/) repository on GitHub\.

## Sending feedback<a name="report-issues"></a>

We welcome your feedback\! If you have a question or comment, or an issue to report, please use the following resources\.
+ If you discover a potential security vulnerability in the AWS Encryption SDK, please [notify AWS security](https://aws.amazon.com/security/vulnerability-reporting/)\. Do not create a public GitHub issue\.
+ To provide feedback on the AWS Encryption SDK, file an issue in the GitHub repository for the programming language you are using\. 
+ To provide feedback on this documentation, use the **Feedback** link on this page\. You can also file an issue or contribute to [aws\-encryption\-sdk\-docs](https://github.com/awsdocs/aws-encryption-sdk-docs), the open\-source repository for this documentation on GitHub\.