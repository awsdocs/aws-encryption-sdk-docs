# Migrating to AWS Encryption SDK Version 2\.0\.*x*<a name="migration"></a>

The AWS Encryption SDK supports multiple interoperable [programming language implementations](programming-languages.md), each of which is developed in an open\-source repository on GitHub\. As a [best practice](best-practices.md), we recommend that you use the latest version of the AWS Encryption SDK for each language\. However, the 2\.0\.*x* version of the AWS Encryption SDK introduces significant new security features, some of which are breaking changes\. To provide a safe upgrade path to version 2\.0\.*x*, we provide a transition version, 1\.7\.*x*, in each programming language\. The topics in this section are designed to help you understand the changes, select the correct version for your application, and migrate safely and successfully to version 2\.0\.*x*\.

**Important**  
If you upgrade directly to version 2\.0\.*x* and enable all new features immediately, the AWS Encryption SDK won't be able to decrypt ciphertext encrypted under older versions of the AWS Encryption SDK\.

**New users**  
If you're new to the AWS Encryption SDK, start with the latest version of the AWS Encryption SDK for your programming language\. The default values enable all security features of the AWS Encryption SDK, including encryption with signing, key derivation, and [key commitment](concepts.md#key-commitment)\.

**Current users**  
We recommend that you upgrade from your current version to the newer versions as soon as possible\. Version 2\.0\.*x* of the AWS Encryption SDK provides new security features to help protect your data\. All future releases, and their improvements, will build on this version\.  
However, version 2\.0\.*x* of the AWS Encryption SDK includes breaking changes that are not backwards compatible\. To assure a safe transition, begin by migrating from your current version to version 1\.7\.*x*\. When version 1\.7\.*x* is fully deployed and operating successfully, you can safely migrate to version 2\.0\.*x*\. This [two\-step process](migration-guide.md) is critical especially for distributed applications\.

For more information about the AWS Encryption SDK security features that underlie these changes, see [Improved client\-side encryption: Explicit KeyIds and key commitment](http://aws.amazon.com/blogs/security/improved-client-side-encryption-explicit-keyids-and-key-commitment/) in the *AWS Security Blog*\.

**Topics**
+ [Versions of the AWS Encryption SDK](about-versions.md)
+ [How to migrate and deploy the AWS Encryption SDK](migration-guide.md)
+ [Updating AWS KMS master key providers](migrate-mkps-v2.md)
+ [Updating AWS KMS keyrings](migrate-keyrings-v2.md)
+ [Setting your commitment policy](migrate-commitment-policy.md)
+ [Troubleshooting migration to version 2\.0\.*x*](troubleshooting-migration.md)