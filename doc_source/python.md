# AWS Encryption SDK for Python<a name="python"></a>

This topic explains how to install and use the AWS Encryption SDK for Python\. For details about programming with the AWS Encryption SDK for Python, see the [aws\-encryption\-sdk\-python](https://github.com/aws/aws-encryption-sdk-python/) repository on GitHub\. For API documentation, see [Read the Docs](https://aws-encryption-sdk-python.readthedocs.io/en/latest/)\.

**Topics**
+ [Prerequisites](#python-prerequisites)
+ [Installation](#python-installation)
+ [Examples](python-example-code.md)

## Prerequisites<a name="python-prerequisites"></a>

Before you install the AWS Encryption SDK for Python, be sure you have the following prerequisites\.

**A supported version of Python**  
Python 3\.6 or later is required by the AWS Encryption SDK for Python versions 3\.1\.0 and later\.  
Earlier versions of the AWS Encryption SDK support Python 2\.7 and Python 3\.4 and later, but we recommend that you use the latest version of the AWS Encryption SDK\.   
To download Python, see [Python downloads](https://www.python.org/downloads/)\.

**The pip installation tool for Python**  
`pip` is included in Python 3\.6 and later versions, although you might want to upgrade it\. For more information about upgrading or installing `pip`, see [Installation](https://pip.pypa.io/en/latest/installation/) in the `pip` documentation\.

## Installation<a name="python-installation"></a>

Install the latest version of the AWS Encryption SDK for Python\.

**Note**  
All versions of the AWS Encryption SDK for Python earlier than 3\.0\.0 are in the [end\-of\-support phase](https://docs.aws.amazon.com/sdkref/latest/guide/maint-policy.html#version-life-cycle)\.  
You can safely update from version 2\.0\.*x* and later to the latest version of the AWS Encryption SDK without any code or data changes\. However, [ new security features](about-versions.md#version-2) introduced in version 2\.0\.*x* are not backward\-compatible\. To update from versions earlier than 1\.7\.*x* to version 2\.0\.*x* and later, you must first update to the latest 1\.*x* version of the AWS Encryption SDK\. For details, see [Migrating your AWS Encryption SDK](migration.md)\.

Use `pip` to install the AWS Encryption SDK for Python, as shown in the following examples\.

**To install the latest version**  

```
pip install aws-encryption-sdk
```

For more details about using pip to install and upgrade packages, see [Installing Packages](https://packaging.python.org/tutorials/installing-packages/)\.

The AWS Encryption SDK for Python requires the [cryptography library](https://cryptography.io/en/latest/) \(pyca/cryptography\) on all platforms\. All versions of `pip` automatically install and build the `cryptography` library on Windows\. `pip` 8\.1 and later automatically installs and builds `cryptography` on Linux\. If you are using an earlier version of `pip` and your Linux environment doesn't have the tools needed to build the `cryptography` library, you need to install them\. For more information, see [Building Cryptography on Linux](https://cryptography.io/en/latest/installation.html#building-cryptography-on-linux)\.

Versions 1\.10\.0 and 2\.5\.0 of the AWS Encryption SDK for Python pin the [cryptography](https://cryptography.io/en/latest/) dependency between 2\.5\.0 and 3\.3\.2\. Other versions of the AWS Encryption SDK for Python install the latest version of cryptography\. If you require a version of cryptography later than 3\.3\.2, we recommend that you use the latest major version of the AWS Encryption SDK for Python\.

For the latest development version of the AWS Encryption SDK for Python, go to the [aws\-encryption\-sdk\-python](https://github.com/aws/aws-encryption-sdk-python/) repository in GitHub\.

After you install the AWS Encryption SDK for Python, get started by looking at the [Python example code](python-example-code.md) in this guide\.