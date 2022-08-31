# Versions of the AWS Encryption CLI<a name="crypto-cli-versions"></a>

We recommend that you use the latest version of the AWS Encryption CLI\.

**Note**  
Versions of the AWS Encryption CLI earlier than 4\.0\.0 are in the [end\-of\-support phase](https://docs.aws.amazon.com/sdkref/latest/guide/maint-policy.html#version-life-cycle)\.  
You can safely update from version 2\.1\.*x* and later to the latest version of the AWS Encryption CLI without any code or data changes\. However, [ new security features](about-versions.md#version-2) introduced in version 2\.1\.*x* are not backward\-compatible\. To update from version 1\.7\.*x* or earlier, you must first update to the latest 1\.*x* version of the AWS Encryption CLI\. For details, see [Migrating your AWS Encryption SDK](migration.md)\.  
New security features were originally released in AWS Encryption CLI versions 1\.7\.*x* and 2\.0\.*x*\. However, AWS Encryption CLI version 1\.8\.*x* replaces version 1\.7\.*x* and AWS Encryption CLI 2\.1\.*x* replaces 2\.0\.*x*\. For details, see the relevant [security advisory](https://github.com/aws/aws-encryption-sdk-cli/security/advisories/GHSA-2xwp-m7mq-7q3r) in the [aws\-encryption\-sdk\-cli](https://github.com/aws/aws-encryption-sdk-cli/) repository on GitHub\.

For information about significant versions of the AWS Encryption SDK, see [Versions of the AWS Encryption SDK](about-versions.md)\.

**Which version do I use?**

If you're new to the AWS Encryption CLI, use the latest version\.

To decrypt data encrypted by a version of the AWS Encryption SDK earlier than version 1\.7\.*x*, migrate first to the latest version of the AWS Encryption CLI\. Make [all recommended changes](migration-guide.md) before updating to version 2\.1\.*x* or later\. For details, see [Migrating your AWS Encryption SDK](migration.md)\.

**Learn more**
+ For detailed information about the changes and guidance for migrating to these new versions, see [Migrating your AWS Encryption SDK](migration.md)\.
+ For descriptions of the new AWS Encryption CLI parameters and attributes, see [AWS Encryption SDK CLI syntax and parameter reference](crypto-cli-reference.md)\.

The following lists describe the change to the AWS Encryption CLI in versions 1\.8\.*x* and 2\.1\.*x*\.

## Version 1\.8\.*x* changes to the AWS Encryption CLI<a name="cli-changes-1.7"></a>
+ Deprecates the `--master-keys` parameter\. Instead, use the `--wrapping-keys` parameter\.
+ Adds the `--wrapping-keys` \(`-w`\) parameter\. It supports all attributes of the `--master-keys` parameter\. It also adds the following optional attributes, which are valid only when decrypting with AWS KMS keys\.
  + **discovery**
  + **discovery\-partition**
  + **discovery\-account**

  For custom master key providers, `--encrypt` and \-`-decrypt` commands require either a `--wrapping-keys` parameter or a `--master-keys` parameter \(but not both\)\. Also, an `--encrypt` command with AWS KMS keys requires either a `--wrapping-keys` parameter or a `--master-keys` parameter \(but not both\)\. 

  In a `--decrypt` command with AWS KMS keys, the `--wrapping-keys` parameter is optional, but recommended, because it is required in version 2\.1\.*x*\. If you use it, you must specify either the **key** attribute or the **discovery** attribute with a value of `true` \(but not both\)\.
+ Adds the `--commitment-policy` parameter\. The only valid value is `forbid-encrypt-allow-decrypt`\. The `forbid-encrypt-allow-decrypt` commitment policy is used in all encrypt and decrypt commands\.

  In version 1\.8\.*x*, when you use the `--wrapping-keys` parameter, a `--commitment-policy` parameter with the `forbid-encrypt-allow-decrypt` value is required\. Setting the value explicitly prevents your [commitment policy](concepts.md#commitment-policy) from changing automatically to `require-encrypt-require-decrypt` when you upgrade to version 2\.1\.*x*\.

## Version 2\.1\.*x* changes to the AWS Encryption CLI<a name="cli-changes-2.x"></a>
+ Removes the `--master-keys` parameter\. Instead, use the `--wrapping-keys` parameter\.
+ The `--wrapping-keys` parameter is required in all encrypt and decrypt commands\. You must specify either a **key** attribute or a **discovery** attribute with a value of `true` \(but not both\)\.
+ The `--commitment-policy` parameter supports the following values\. For details, see [Setting your commitment policy](migrate-commitment-policy.md)\.
  + `forbid-encrypt-allow-decrypt`
  + `require-encrypt-allow-decrypt`
  + `require-encrypt-require decrypt` \(Default\)
+ The `--commitment-policy` parameter is optional in version 2\.1\.*x*\. The default value is `require-encrypt-require-decrypt`\.

## Version 1\.9\.*x* and 2\.2\.*x* changes to the AWS Encryption CLI<a name="cli-changes-2.2"></a>
+ Adds the `--decrypt-unsigned` parameter\. For details, see [Version 2\.2\.*x*](about-versions.md#version2.2.x)\.
+ Adds the `--buffer` parameter\. For details, see [Version 2\.2\.*x*](about-versions.md#version2.2.x)\.
+ Adds the `--max-encrypted-data-keys` parameter\. For details, see [Limit encrypted data keys](configure.md#config-limit-keys)\.

## Version 3\.0\.*x* changes to the AWS Encryption CLI<a name="cli-changes-v3"></a>
+ Adds support for AWS KMS multi\-Region keys\. For details, see [Use multi\-Region AWS KMS keys](configure.md#config-mrks)\.