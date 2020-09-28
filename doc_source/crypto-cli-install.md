# Installing the AWS Encryption SDK command line interface<a name="crypto-cli-install"></a>

This topic explains how to install the AWS Encryption CLI\. For detailed information, see the [aws\-encryption\-sdk\-cli](https://github.com/aws/aws-encryption-sdk-cli/) repository on GitHub and [Read the Docs](https://aws-encryption-sdk-cli.readthedocs.io/en/latest/)\.

**Topics**
+ [Installing the prerequisites](#crypto-cli-prerequisites)
+ [Installing the AWS Encryption CLI](#install-sdk-cli)

## Installing the prerequisites<a name="crypto-cli-prerequisites"></a>

The AWS Encryption CLI is built on the AWS Encryption SDK for Python\. To use the AWS Encryption CLI, you need Python and `pip`, the Python package management tool\. Python and `pip` are available on all supported platforms\.

Before you install the AWS Encryption CLI, be sure that you have the following prerequisites\.

**Python**  
The AWS Encryption CLI requires Python 2\.7, or Python 3\.4 or later\. Python is included in most Linux and macOS installations, although you might need to upgrade to one of the required versions\. However, you have to install Python on Windows, if it is not already installed\. To download Python, see [Python downloads](https://www.python.org/downloads/)\.  
To determine whether Python is installed, at the command line, type the following\.  

```
python
```
To check the Python version, use the `-V` \(uppercase V\) parameter\.  

```
python -V
```
On Windows, you need to install Python\. Then, add the path to the `Python.exe` file to the value of the **Path** environment variable\.   
By default, Python is installed in the all users directory or in a user profile directory \(`$home` or `%userprofile%`\) in the `AppData\Local\Programs\Python` subdirectory\. To find the location of the `Python.exe` file on your system, check one of the following registry keys\. You can use PowerShell to search the registry\.   

```
PS C:\> dir HKLM:\Software\Python\PythonCore\version\InstallPath
# -or-
PS C:\> dir HKCU:\Software\Python\PythonCore\version\InstallPath
```

**pip**  
`pip` is the Python package manager\. To install the AWS Encryption CLI and its dependencies, you need `pip` 8\.1 or later\.   
For help installing or upgrading `pip`, see [Installation](https://pip.pypa.io/en/latest/installing/) in the `pip` documentation\.

**AWS Command Line Interface**  
The AWS Command Line Interface \(AWS CLI\) is required only if you are using AWS Key Management Service \(AWS KMS\) customer master keys \(CMKs\) with the AWS Encryption CLI\. If you are using a different [master key provider](concepts.md#master-key-provider), the AWS CLI is not required\.  
To use AWS KMS CMKs with the AWS Encryption CLI, you need to [install](https://docs.aws.amazon.com/cli/latest/userguide/installing.html) and [configure ](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html#cli-quick-configuration) the AWS CLI\. The configuration makes the credentials that you use to authenticate to AWS KMS available to the AWS Encryption CLI\. 

## Installing the AWS Encryption CLI<a name="install-sdk-cli"></a>

Use `pip` to install the AWS Encryption CLI and the Python [cryptography library](https://cryptography.io/en/latest/) that it requires\. 

**Note**  
If you are new to the AWS Encryption CLI, install the latest available version\.   
If you support commands and scripts designed for version of the AWS Encryption CLI before 1\.7\.*x*, we recommend that you upgrade first to version 1\.7\.*x* before upgrading to version 2\.0\.*x* or later\. Version 2\.0\.*x* of the AWS Encryption CLI introduces new security features to support [AWS Encryption SDK best practices](best-practices.md)\. However, version 2\.0\.*x* is not backward\-compatible; it will cause scripts designed for earlier versions of the AWS Encryption CLI to fail\.   
For information about the changes and for help migrating from your current version to version 1\.7\.*x* and 2\.0\.*x*, see [Migrating to version 2\.0\.*x*](migration.md)\.

The AWS Encryption CLI requires the **cryptography** library on all platforms\. All versions of `pip` install and build the **cryptography** library on Windows and OS X\. 

On Linux, `pip` 8\.1 and later installs and builds the **cryptography** library\. If you are using an earlier version of `pip` and your Linux environment doesn't have the tools needed to build the **cryptography** library, you must install them\. For more information, see [Building cryptography on Linux](https://cryptography.io/en/latest/installation/#building-cryptography-on-linux)\.

**To install the latest version**  

```
pip install aws-encryption-sdk-cli
```

**To upgrade to the latest version**  

```
pip install --upgrade aws-encryption-sdk-cli
```

**To find the version number of your AWS Encryption CLI and AWS Encryption SDK**  

```
aws-encryption-cli --version

aws-encryption-sdk-cli/1.1.0 aws-encryption-sdk/1.3.2
```

To install the version of the AWS Encryption CLI currently in development, see the [aws\-encryption\-sdk\-cli](https://github.com/aws/aws-encryption-sdk-cli/) repository on GitHub\.

For more details about using `pip` to install and upgrade Python packages, see the [pip documentation](https://pip.pypa.io/en/stable/quickstart/)\.