# Versions of the AWS Encryption CLI<a name="crypto-cli-versions"></a>

Beginning in version 2\.0\.*x*, the AWS Encryption CLI includes significant new security features to support [AWS Encryption SDK best practices](best-practices.md)\. These new features cause breaking changes that will require updates your commands and scripts\. To make migration to version 2\.0\.*x* easier, we provide a transition version, 1\.7\.*x* that lets you prepare your commands and scripts for 2\.0\.*x*\. Features that are deprecated in version 1\.7\.*x* are removed in version 2\.0\.*x*\. 

**Which version do I use?**

If a command or script will be decrypting data encrypted by an earlier version of the AWS Encryption SDK, start with version 1\.7\.*x*\. Make [all recommended changes](migration-guide.md) before updating to version 2\.0\.*x* or later\. Otherwise, you can begin with the latest available version of the AWS Encryption CLI\.

**Learn more**
+ For detailed information about the changes and guidance for migrating to these new versions, see [Migrating to version 2\.0\.*x*](migration.md)\.
+ For descriptions of the new AWS Encryption CLI parameters and attributes, see [AWS Encryption SDK CLI syntax and parameter reference](crypto-cli-reference.md)\.

The following lists describe the change to the AWS Encryption CLI in versions 1\.7\.*x* and 2\.0\.*x*\.

## Version 1\.7\.*x* changes to the AWS Encryption CLI<a name="cli-changes-1.7"></a>
+ Deprecates the `--master-keys` parameter\. Instead, use the `--wrapping-keys` parameter\.
+ Adds the `--wrapping-keys` \(`-w`\) parameter\. It supports all attributes of the `--master-keys` parameter\. It also adds the following optional attributes, which are valid only when decrypting with AWS KMS customer master keys \(CMKs\)\.
  + **discovery**
  + **discovery\-partition**
  + **discovery\-account**

  For custom master key providers, `--encrypt` and \-`-decrypt` commands require either a `--wrapping-keys` parameter or a `--master-keys` parameter \(but not both\)\. Also, an `--encrypt` command with AWS KMS CMKs requires either a `--wrapping-keys` parameter or a `--master-keys` parameter \(but not both\)\. 

  In a `--decrypt` command with AWS KMS CMKs, the `--wrapping-keys` parameter is optional, but recommended, because it is required in version 2\.0\.*x*\. If you use it, you must specify either the **key** attribute or the **discovery** attribute \(but not both\)\.
+ Adds the `--commitment-policy` parameter\. The only valid value is `forbid-encrypt-allow-decrypt`\. 

  The `--commitment-policy` parameter is required in commands with the `--wrapping-keys` parameter\. Otherwise it is valid, but optional\. We require the `--commitment-policy` parameter with `--wrapping keys` even though there is only one value\. This configuration prevents your [commitment policy](concepts.md#commitment-policy) from changing automatically to `require-encrypt-require-decrypt` when you upgrade to version 2\.0\.*x*\.

## Version 2\.0\.*x* changes to the AWS Encryption CLI<a name="cli-changes-2.x"></a>
+ Removes the `--master-keys` parameter\. Instead, use the `--wrapping-keys` parameter\.
+ The `--wrapping-keys` parameter is required in all encrypt and decrypt commands\. You must specify either a **key** attribute or a **discovery** attribute \(but not both\)\.
+ The `--commitment-policy` parameter supports the following values\.
  + `forbid-encrypt-allow-decrypt`
  + `require-encrypt-allow-decrypt`
  + `require-encrypt-require decrypt` \(Default\)
+ The `--commitment-policy` parameter is now optional\. The default value is `require-encrypt-require decrypt`\.