# Migrating your AWS Encryption SDK<a name="migration"></a>

The AWS Encryption SDK supports multiple interoperable [programming language implementations](programming-languages.md), each of which is developed in an open\-source repository on GitHub\. As a [best practice](best-practices.md), we recommend that you use the latest version of the AWS Encryption SDK for each language\. 

You can safely upgrade from version 2\.0\.*x* or later of AWS Encryption SDK to the latest version\. However, the 2\.0\.*x* version of the AWS Encryption SDK introduces significant new security features, some of which are breaking changes\. To upgrade from versions earlier than 1\.7\.*x* to versions 2\.0\.*x* and later, you must first upgrade to the latest 1\.*x* version\. The topics in this section are designed to help you understand the changes, select the correct version for your application, and migrate safely and successfully to the newest versions of the AWS Encryption SDK\.

For information about significant versions of the AWS Encryption SDK, see [Versions of the AWS Encryption SDK](about-versions.md)\.

**Important**  
Do not upgrade directly from a version earlier than 1\.7\.*x* to version 2\.0\.*x* or later without first upgrading to the latest 1\.*x* version\. If you upgrade directly to version 2\.0\.*x* or later and enable all new features immediately, the AWS Encryption SDK won't be able to decrypt ciphertext encrypted under older versions of the AWS Encryption SDK\.

**Note**  
The earliest version of the AWS Encryption SDK for \.NET is version 3\.0\.*x*\. All versions of the AWS Encryption SDK for \.NET support the security best practices introduced in 2\.0\.*x* of the AWS Encryption SDK\. You can safely upgrade to the latest version without any code or data changes\.  
AWS Encryption CLI: When reading this migration guide, use the 1\.7\.*x* migration instructions for AWS Encryption CLI 1\.8\.*x* and use the 2\.0\.*x* migration instructions for AWS Encryption CLI 2\.1\.*x*\. For details, see [Versions of the AWS Encryption CLI](crypto-cli-versions.md)\.  
New security features were originally released in AWS Encryption CLI versions 1\.7\.*x* and 2\.0\.*x*\. However, AWS Encryption CLI version 1\.8\.*x* replaces version 1\.7\.*x* and AWS Encryption CLI 2\.1\.*x* replaces 2\.0\.*x*\. For details, see the relevant [security advisory](https://github.com/aws/aws-encryption-sdk-cli/security/advisories/GHSA-2xwp-m7mq-7q3r) in the [aws\-encryption\-sdk\-cli](https://github.com/aws/aws-encryption-sdk-cli/) repository on GitHub\.

**New users**  
If you're new to the AWS Encryption SDK, install the latest version of the AWS Encryption SDK for your programming language\. The default values enable all security features of the AWS Encryption SDK, including encryption with signing, key derivation, and [key commitment](concepts.md#key-commitment)\. of the AWS Encryption SDK

**Current users**  
We recommend that you upgrade from your current version to the latest available version as soon as possible\. All 1\.*x* versions of the AWS Encryption SDK are in the [end\-of\-support phase](https://docs.aws.amazon.com/sdkref/latest/guide/maint-policy.html#version-life-cycle), as are later versions in some programming languages\. For details about the support and maintenance status of the AWS Encryption SDK in your programming language, see [Support and maintenance](introduction.md#support)\.  
AWS Encryption SDK versions 2\.0\.*x* and later provide new security features to help protect your data\. However, AWS Encryption SDK version 2\.0\.*x* includes breaking changes that are not backwards compatible\. To assure a safe transition, begin by migrating from your current version to the latest 1\.*x* in your programming language\. When your latest 1\.*x* version is fully deployed and operating successfully, you can safely migrate to versions 2\.0\.*x* and later\. This [two\-step process](migration-guide.md) is critical especially for distributed applications\.

For more information about the AWS Encryption SDK security features that underlie these changes, see [Improved client\-side encryption: Explicit KeyIds and key commitment](http://aws.amazon.com/blogs/security/improved-client-side-encryption-explicit-keyids-and-key-commitment/) in the *AWS Security Blog*\.

Looking for help with using the AWS Encryption SDK for Java with the AWS SDK for Java 2\.x? See [Prerequisites](java.md#java-prerequisites)\.

**Topics**
+ [How to migrate and deploy the AWS Encryption SDK](migration-guide.md)
+ [Updating AWS KMS master key providers](migrate-mkps-v2.md)
+ [Updating AWS KMS keyrings](migrate-keyrings-v2.md)
+ [Setting your commitment policy](migrate-commitment-policy.md)
+ [Troubleshooting migration to the latest versions](troubleshooting-migration.md)