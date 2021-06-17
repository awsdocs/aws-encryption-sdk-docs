# Versions of the AWS Encryption CLI<a name="crypto-cli-versions"></a>

Beginning in version 2\.1\.*x*, the AWS Encryption CLI includes significant new security features to support [AWS Encryption SDK best practices](best-practices.md)\. These new features cause breaking changes that will require updates to your commands and scripts\. To make migration to version 2\.1\.*x* easier, we provide a transition version, 1\.8\.*x* that lets you prepare your commands and scripts for 2\.1\.*x*\. Features that are deprecated in version 1\.8\.*x* are removed in version 2\.1\.*x*\.

**Note**  
New security features were originally released in AWS Encryption CLI versions 1\.7\.*x* and 2\.0\.*x*\. However, AWS Encryption CLI version 1\.8\.*x* replaces version 1\.7\.*x* and AWS Encryption CLI 2\.1\.*x* replaces 2\.0\.*x*\. For details, see the relevant [security advisory](https://github.com/aws/aws-encryption-sdk-cli/security/advisories/GHSA-2xwp-m7mq-7q3r) in the [aws\-encryption\-sdk\-cli](https://github.com/aws/aws-encryption-sdk-cli/) repository on GitHub\.

**Which version do I use?**

If a command or script will be decrypting data encrypted by an earlier version of the AWS Encryption SDK, start with version 1\.8\.*x*\. Make [all recommended changes](migration-guide.md) before updating to version 2\.1\.*x* or later\. Otherwise, you can begin with the latest available version of the AWS Encryption CLI\.

**Learn more**
+ For detailed information about the changes and guidance for migrating to these new versions, see [Migrating to version 2\.0\.*x*](migration.md)\.
+ For descriptions of the new AWS Encryption CLI parameters and attributes, see [AWS Encryption SDK CLI syntax and parameter reference](crypto-cli-reference.md)\.

The following lists describe the change to the AWS Encryption CLI in versions 1\.8\.*x* and 2\.1\.*x*\.

## Version 1\.8\.*x* changes to the AWS Encryption CLI<a name="cli-changes-1.7"></a>
+ Deprecates the `--master-keys` parameter\. Instead, use the `--wrapping-keys` parameter\.
+ Adds the `--wrapping-keys` \(`-w`\) parameter\. It supports all attributes of the `--master-keys` parameter\. It also adds the following optional attributes, which are valid only when decrypting with AWS KMS customer master keys \(CMKs\)\.
  + **discovery**
  + **discovery\-partition**
  + **discovery\-account**

  For custom master key providers, `--encrypt` and \-`-decrypt` commands require either a `--wrapping-keys` parameter or a `--master-keys` parameter \(but not both\)\. Also, an `--encrypt` command with AWS KMS CMKs requires either a `--wrapping-keys` parameter or a `--master-keys` parameter \(but not both\)\. 

  In a `--decrypt` command with AWS KMS CMKs, the `--wrapping-keys` parameter is optional, but recommended, because it is required in version 2\.1\.*x*\. If you use it, you must specify either the **key** attribute or the **discovery** attribute with a value of `true` \(but not both\)\.
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
+ Adds the `--max-encrypted-data-keys` parameter\. For details, see [Limiting encrypted data keys](configure.md#config-limit-keys)\.

## Version 3\.0\.*x* changes to the AWS Encryption CLI<a name="cli-changes-v3"></a>
+ Adds support for AWS KMS multi\-Region keys\. For details, see [Using multi\-Region KMS keys](configure.md#config-mrks)\.