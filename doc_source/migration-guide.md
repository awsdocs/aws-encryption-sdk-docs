# How to migrate and deploy the AWS Encryption SDK<a name="migration-guide"></a>

When migrating from an AWS Encryption SDK version earlier than 1\.7\.*x* to version 2\.0\.*x* or later, you must transition safely to encrypting with [key commitment](concepts.md#key-commitment)\. Otherwise, your application will encounter ciphertexts that it cannot decrypt\. If you are using AWS KMS master key providers, you must update to new constructors that create master key providers in strict mode or discovery mode\.

**Note**  
This topic is designed for users migrating from earlier versions of the AWS Encryption SDK to version 2\.0\.*x* or later\. If you are new to the AWS Encryption SDK, you can begin using the latest available version immediately with the default settings\.

To avoid a critical situation in which you cannot decrypt ciphertext that you need to read, we recommend that you migrate and deploy in multiple distinct stages\. Verify that each stage is complete and fully deployed before starting the next stage\. This is particularly important for distributed applications with multiple hosts\.

## Stage 1: Update your application to the latest 1\.*x* version<a name="migrate-stage1"></a>

Update to the latest 1\.*x* version for your programming language\. Test carefully, deploy your changes, and confirm that the update has propagated to all destination hosts before starting stage 2\.

**Important**  
Verify that your latest 1\.*x* version is version 1\.7\.*x* or later of the AWS Encryption SDK\.

The latest 1\.*x* versions of the AWS Encryption SDK are backward compatible with legacy versions of the AWS Encryption SDK and forward compatible with versions 2\.0\.*x* and later\. They include the new features that are present in version 2\.0\.*x*, but include safe defaults designed for this migration\. They allow you to upgrade your AWS KMS master key providers, if necessary, and to fully deploy with algorithm suites that can decrypt ciphertext with key commitment\.
+ Replace deprecated elements, including constructors for legacy AWS KMS master key providers\. In [Python](https://docs.python.org/3/library/warnings.html), be sure to turn on deprecation warnings\. Code elements that are deprecated in the latest 1\.*x* versions are removed from versions 2\.0\.*x* and later\. 
+ Explicitly set your commitment policy to `ForbidEncryptAllowDecrypt`\. Although this is the only valid value in the latest 1\.*x* versions, this setting is required when you use the APIs introduced in this release\. It prevents your application from rejecting ciphertext encrypted without key commitment when you migrate to version 2\.0\.*x* and later\. For details, see [Setting your commitment policy](migrate-commitment-policy.md)\.
+ If you use AWS KMS master key providers, you must update your legacy master key providers to master key providers that support *strict mode* and *discovery mode*\. This update is required for the AWS Encryption SDK for Java, AWS Encryption SDK for Python, and the AWS Encryption CLI\. If you use master key providers in discovery mode, we recommend that you implement the discovery filter that limits the wrapping keys used to those in particular AWS accounts\. This update is optional, but it's a [best practice](best-practices.md) that we recommend\. For details, see [Updating AWS KMS master key providers](migrate-mkps-v2.md)\. 
+ If you use [AWS KMS discovery keyrings](use-kms-keyring.md#kms-keyring-discovery), we recommend that you include a discovery filter that limits the wrapping keys used in decryption to those in particular AWS accounts\. This update is optional, but it's a [best practice](best-practices.md) that we recommend\. For details, see [Updating AWS KMS keyrings](migrate-keyrings-v2.md)\.

## Stage 2: Update your application to the latest version<a name="migrate-stage2"></a>

After deploying the latest 1\.*x* version successfully to all hosts, you can upgrade to versions 2\.0\.*x* and later\. Version 2\.0\.*x* includes breaking changes for all earlier versions of the AWS Encryption SDK\. However, if you make the code changes recommended in Stage 1, you can avoid errors when you migrate to the latest version\.

Before you update to the latest version, verify that your commitment policy is consistently set to `ForbidEncryptAllowDecrypt`\. Then, depending on your data configuration, you can migrate at your own pace to `RequireEncryptAllowDecrypt` and then to the default setting, `RequireEncryptRequireDecrypt`\. We recommend a series of transition steps like the following pattern\.

1. Begin with your [commitment policy](migrate-commitment-policy.md) set to `ForbidEncryptAllowDecrypt`\. The AWS Encryption SDK can decrypt messages with key commitment, but it doesn't yet encrypt with key commitment\.

1. When you are ready, update your commitment policy to `RequireEncryptAllowDecrypt`\. The AWS Encryption SDK begins to encrypt your data with [key commitment](concepts.md#key-commitment)\. It can decrypt ciphertext with and without key commitment\. 

   Before updating your commitment policy to `RequireEncryptAllowDecrypt`, verify that your latest 1\.*x* version is deployed to all hosts, including hosts of any applications that decrypt the ciphertext you produce\. Versions of the AWS Encryption SDK prior to version 1\.7\.*x* cannot decrypt messages encrypted with key commitment\.

   This is also a good time to add metrics to your application to measure whether you are still processing ciphertext without key commitment\. This will help you determine when it's safe to update your commitment policy setting to `RequireEncryptRequireDecrypt`\. For some applications, such as those that encrypt messages in an Amazon SQS queue, this might mean waiting long enough that all ciphertext encrypted under old versions have been re\-encrypted or deleted\. For other applications, such as encrypted S3 objects, you might need to download, re\-encrypt, and re\-upload all objects\.

1. When you are certain that you don't have any messages encrypted without key commitment, you can update your commitment policy to `RequireEncryptRequireDecrypt`\. This value assures that your data is always encrypted and decrypted with key commitment\. This setting is the default, so you aren't required to set it explicitly, but we recommend it\. An explicit setting will [aid debugging](troubleshooting-migration.md) and any potential rollbacks that might be required if your application encounters ciphertext encrypted without key commitment\. 