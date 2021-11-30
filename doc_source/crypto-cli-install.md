# Installing the AWS Encryption SDK command line interface<a name="crypto-cli-install"></a>

This topic explains how to install the AWS Encryption CLI\. For detailed information, see the [aws\-encryption\-sdk\-cli](https://github.com/aws/aws-encryption-sdk-cli/) repository on GitHub and [Read the Docs](https://aws-encryption-sdk-cli.readthedocs.io/en/latest/)\.

**Topics**
+ [Installing the prerequisites](#crypto-cli-prerequisites)
+ [Installing the AWS Encryption CLI](#install-sdk-cli)

## Installing the prerequisites<a name="crypto-cli-prerequisites"></a>

The AWS Encryption CLI is built on the AWS Encryption SDK for Python\. To install the AWS Encryption CLI, you need Python and `pip`, the Python package management tool\. Python and `pip` are available on all supported platforms\.

Install the following prerequisites before you install the AWS Encryption CLI, 

**Python**  
Python 3\.6 or later is required by the AWS Encryption CLI versions 4\.1\.0 and later\.  
Earlier versions of the AWS Encryption CLI support Python 2\.7 and 3\.4 and later, but we recommend that you use the latest version of the AWS Encryption CLI\.  
Python is included in most Linux and macOS installations, but you need to upgrade to Python 3\.6 or later\. We recommend that you use the latest version of Python\. On Windows, you have to install Python; it is not installed by default\. To download and install Python, see [Python downloads](https://www.python.org/downloads/)\.  
To determine whether Python is installed, at the command line, type the following\.  

```
python
```
To check the Python version, use the `-V` \(uppercase V\) parameter\.  

```
python -V
```
On Windows, after you install Python, add the path to the `Python.exe` file to the value of the **Path** environment variable\.   
By default, Python is installed in the all users directory or in a user profile directory \(`$home` or `%userprofile%`\) in the `AppData\Local\Programs\Python` subdirectory\. To find the location of the `Python.exe` file on your system, check one of the following registry keys\. You can use PowerShell to search the registry\.   

```
PS C:\> dir HKLM:\Software\Python\PythonCore\version\InstallPath
# -or-
PS C:\> dir HKCU:\Software\Python\PythonCore\version\InstallPath
```

**pip**  
`pip` is the Python package manager\. To install the AWS Encryption CLI and its dependencies, you need `pip` 8\.1 or later\. For help installing or upgrading `pip`, see [Installation](https://pip.pypa.io/en/latest/installing/) in the `pip` documentation\.  
On Linux installations, versions of `pip` earlier than 8\.1 can't build the **cryptography** library that the AWS Encryption CLI requires\. If you choose not to update your `pip` version, you can install the build tools separately\. For more information, see [Building cryptography on Linux](https://cryptography.io/en/latest/installation.html#building-cryptography-on-linux)\.

**AWS Command Line Interface**  
The AWS Command Line Interface \(AWS CLI\) is required only if you are using AWS KMS keys in AWS Key Management Service \(AWS KMS\) with the AWS Encryption CLI\. If you are using a different [master key provider](concepts.md#master-key-provider), the AWS CLI is not required\.  
To use AWS KMS keys with the AWS Encryption CLI, you need to [install](https://docs.aws.amazon.com/cli/latest/userguide/installing.html) and [configure ](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html#cli-quick-configuration) the AWS CLI\. The configuration makes the credentials that you use to authenticate to AWS KMS available to the AWS Encryption CLI\. 

## Installing the AWS Encryption CLI<a name="install-sdk-cli"></a>

When you use `pip` to install the AWS Encryption CLI, it automatically installs the libraries that the CLI needs, including the [AWS Encryption SDK for Python](python.md), the Python [cryptography library](https://cryptography.io/en/latest/), and the [AWS SDK for Python \(Boto3\)](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html)\.

**Note**  
If you are new to the AWS Encryption CLI, install the latest available version\.   
If you support commands and scripts designed for version of the AWS Encryption CLI before 1\.8\.*x*, we recommend that you upgrade first to version 1\.8\.*x* before upgrading to version 2\.1\.*x* or later\. Version 2\.1\.*x* of the AWS Encryption CLI introduces new security features to support [AWS Encryption SDK best practices](best-practices.md)\. However, version 2\.1\.*x* is not backward\-compatible; it will cause scripts designed for earlier versions of the AWS Encryption CLI to fail\.   
New security features were originally released in AWS Encryption CLI versions 1\.7\.*x* and 2\.0\.*x*\. However, AWS Encryption CLI version 1\.8\.*x* replaces version 1\.7\.*x* and AWS Encryption CLI 2\.1\.*x* replaces 2\.0\.*x*\. For details, see the relevant [security advisory](https://github.com/aws/aws-encryption-sdk-cli/security/advisories/GHSA-2xwp-m7mq-7q3r) in the [aws\-encryption\-sdk\-cli](https://github.com/aws/aws-encryption-sdk-cli/) repository on GitHub\.  
For information about the changes and for help migrating from your current version to version 1\.8\.*x* and 2\.1\.*x*, see [Migrating to version 2\.0\.*x*](migration.md)\.

**To install the latest version of the AWS Encryption CLI**  

```
pip install aws-encryption-sdk-cli
```

**To upgrade to the latest version of the AWS Encryption CLI**  

```
pip install --upgrade aws-encryption-sdk-cli
```

**To find the version numbers of your AWS Encryption CLI and AWS Encryption SDK**  

```
aws-encryption-cli --version
```
The output lists the version numbers of both libraries\.  

```
aws-encryption-sdk-cli/2.1.0 aws-encryption-sdk/2.0.0
```

**To upgrade to the latest version of the AWS Encryption CLI**  

```
pip install --upgrade aws-encryption-sdk-cli
```

Installing the AWS Encryption CLI also installs the latest version of the AWS SDK for Python \(Boto3\), if it's not already installed\. If Boto3 is installed, the installer verifies the Boto3 version and updates it if required\.

**To find your installed version of Boto3**  

```
pip show boto3
```

**To update to the latest version of Boto3**  

```
pip install --upgrade boto3
```

To install the version of the AWS Encryption CLI currently in development, see the [aws\-encryption\-sdk\-cli](https://github.com/aws/aws-encryption-sdk-cli/) repository on GitHub\.

For more details about using `pip` to install and upgrade Python packages, see the [pip documentation](https://pip.pypa.io/en/stable/quickstart/)\.