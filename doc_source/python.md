# AWS Encryption SDK for Python<a name="python"></a>

This topic explains how to install and use the AWS Encryption SDK for Python\. For details about programming with the AWS Encryption SDK for Python, see the [aws\-encryption\-sdk\-python](https://github.com/aws/aws-encryption-sdk-python/) repository on GitHub\. For API documentation, see [Read the Docs](https://aws-encryption-sdk-python.readthedocs.io/en/latest/)\.

**Topics**
+ [Prerequisites](#python-prerequisites)
+ [Installation](#python-installation)
+ [Example code](python-example-code.md)

## Prerequisites<a name="python-prerequisites"></a>

Before you install the AWS Encryption SDK for Python, be sure you have the following prerequisites\.

**A supported version of Python**  
To use this SDK, you need Python 2\.7, or Python 3\.4 or later\. To download Python, see [Python downloads](https://www.python.org/downloads/)\.

**The pip installation tool for Python**  
If you have Python 2\.7\.9 or later, or Python 3\.4 or later, you already have pip, although you might want to upgrade it\. For more information about upgrading or installing pip, see [Installation](https://pip.pypa.io/en/latest/installing/) in the pip documentation\.

## Installation<a name="python-installation"></a>

Use pip to install the AWS Encryption SDK for Python, as shown in the following examples\.

**To install the latest version**  

```
pip install aws-encryption-sdk
```

For more details about using pip to install and upgrade packages, see [Installing Packages](https://packaging.python.org/tutorials/installing-packages/)\.

The SDK requires the [cryptography library](https://cryptography.io/en/latest/) on all platforms\. All versions of `pip` install and build the **cryptography** library on Windows\. `pip` 8\.1 and later installs and builds **cryptography** on Linux\. If you are using an earlier version of `pip` and your Linux environment doesn't have the tools needed to build the **cryptography** library, you need to install them\. For more information, see [Building Cryptography on Linux](https://cryptography.io/en/latest/installation/#building-cryptography-on-linux)\.

For the latest development version of this SDK, go to the [aws\-encryption\-sdk\-python GitHub repository](https://github.com/aws/aws-encryption-sdk-python/)\.

After you install the SDK, get started by looking at the [example Python code](python-example-code.md) in this guide\.