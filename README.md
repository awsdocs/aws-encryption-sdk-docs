# AWS Encryption SDK Developer Guide

This repository contains the open source version of the [AWS Encryption SDK Developer
Guide](https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/). We welcome your contributions! Please start by reading the
[Contributions](https://github.com/awsdocs/aws-encryption-sdk-docs/blob/master/CONTRIBUTING.md) page in this
repo.

## How you can help
We welcome all contributions, including help with conceptual explanations and diagrams. But because 
examples are the heart of every Developer Guide, we want more of them, especially real-world
examples that show common use patterns.

## What is the AWS Encryption SDK? 

The AWS Encryption SDK is a
[client-side](https://docs.aws.amazon.com/dynamodb-encryption-client/latest/devguide/client-server-side.html)
encryption library that makes it easier for you to encrypt and decrypt data securely in your
application. It can be used on any type of data. The `encrypt`
method returns a single, portable formatted message that is easy to store and manage. 

The AWS Encryption SDK is available in [Java](https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/java.html),
[Python](https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/python.html),
[C](https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/c-language.html), [JavaScript](https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/javascript.html), and a [command-line
interface](https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/crypto-cli.html) that runs on Linux, macOS and Windows.


To protect data, the Encryption SDK uses *envelope encryption*. Each item of data is encrypted under a unique data key. Then, the data key is encrypted under a master key so it can be safely stored with the data. Your application does not have to generate or manage the data keys.

To protect the master key that encrypts the data keys, you can use a web service, such as [AWS Key
Management Service](https://docs.aws.amazon.com/kms/latest/developerguide/) (AWS KMS), a hardware
security module (HSM), such as those offered by [AWS
CloudHSM](https://docs.aws.amazon.com/cloudhsm/latest/userguide/), or your existing key management tools. The AWS Encryption SDK does not require an AWS
account or any AWS service.

## What is the AWS Encryption SDK Developer Guide?

The AWS Encryption SDK Developer Guide provides a
conceptual overview of the AWS Encryption SDK, including an introduction to its
architecture, an explanation of how it protects data, a reference section of cryptographic details,
and examples in each programming language to help you get started.

## Where is the code?
We are developing the AWS Encryption SDK in the following open source projects on GitHub. 

* C - [aws-encryption-sdk-c](https://github.com/aws/aws-encryption-sdk-c)
* Java - [aws-encryption-sdk-java](https://github.com/aws/aws-encryption-sdk-java)
* JavaScript - [aws-encryption-sdk-javascript](https://github.com/aws/aws-encryption-sdk-javascript)
* Python -
[aws-encryption-sdk-python](https://github.com/aws/aws-encryption-sdk-python)
* Command-line interface - [aws-encryption-sdk-cli](https://github.com/awslabs/aws-encryption-sdk-cli)

## Who is the audience?
Any developer who needs to protect sensitive data at its source. The developers who use this library
don't need any cryptographic expertise. The library is designed for general use and includes strong and secure default
implementations. If anyone needs a custom design, the libraries provide extension points for custom implementations.


## License Summary

The documentation is made available under the Creative Commons Attribution-ShareAlike 4.0 International License. See the LICENSE file.

The sample code within this documentation is made available under a modified MIT license. See the LICENSE-SAMPLECODE file.
