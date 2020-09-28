# AWS Encryption SDK command line interface<a name="crypto-cli"></a>

The AWS Encryption SDK Command Line Interface \(AWS Encryption CLI\) enables you to use the AWS Encryption SDK to encrypt and decrypt data interactively at the command line and in scripts\. You don't need cryptography or programming expertise\. 

**Note**  
Version 2\.0\.*x* of the AWS Encryption CLI introduces new security features to support [AWS Encryption SDK best practices](best-practices.md)\. However, version 2\.0\.*x* is not backward\-compatible; it will cause commands and scripts designed for earlier versions of the AWS Encryption CLI to fail\. To mitigate the effect of these changes, we provide a transition version, 1\.7\.*x*\.   
For information about the changes and for help migrating from your current version to version 1\.7\.*x* and 2\.0\.*x*, see [Migrating to version 2\.0\.*x*](migration.md)\.

Like all implementations of the AWS Encryption SDK, the AWS Encryption CLI offers advanced data protection features\. These include [envelope encryption](https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/how-it-works.html#envelope-encryption), additional authenticated data \(AAD\), and secure, authenticated, symmetric key [algorithm suites](https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/supported-algorithms.html), such as 256\-bit AES\-GCM with key derivation, [key commitment](concepts.md#key-commitment), and signing\. 

The AWS Encryption CLI is built on the [AWS Encryption SDK for Python](python.md) and is supported on Linux, macOS, and Windows\. You can run commands and scripts to encrypt and decrypt your data in your preferred shell on Linux or macOS, in a Command Prompt window \(cmd\.exe\) on Windows, and in a PowerShell console on any system\. 

All language\-specific implementations of the AWS Encryption SDK, including the AWS Encryption CLI, are interoperable\. For example, you can encrypt data with the [AWS Encryption SDK for Java](java.md) and decrypt it with the AWS Encryption CLI\. 

This topic introduces the AWS Encryption CLI, explains how to install and use it, and provides several examples to help you get started\. For a quick start, see [How to Encrypt and Decrypt Your Data with the AWS Encryption CLI](http://aws.amazon.com/blogs/security/how-to-encrypt-and-decrypt-your-data-with-the-aws-encryption-cli/) in the AWS Security Blog\. For more detailed information, see [Read The Docs](https://aws-encryption-sdk-cli.readthedocs.io/en/latest/), and join us in developing the AWS Encryption CLI in the [aws\-encryption\-sdk\-cli](https://github.com/aws/aws-encryption-sdk-cli/) repository on GitHub\.

**Performance**  
The AWS Encryption CLI is built on the AWS Encryption SDK for Python\. Each time you run the CLI, you start a new instance of the Python runtime\. To improve performance, whenever possible, use a single command instead of a series of independent commands\. For example, run one command that processes the files in a directory recursively instead of running separate commands for each file\.

**Topics**
+ [Installing the CLI](crypto-cli-install.md)
+ [How to use the CLI](crypto-cli-how-to.md)
+ [Examples](crypto-cli-examples.md)
+ [Syntax and parameter reference](crypto-cli-reference.md)
+ [Versions](crypto-cli-versions.md)