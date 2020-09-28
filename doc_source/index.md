# AWS Encryption SDK Developer Guide

-----
*****Copyright &copy; 2020 Amazon Web Services, Inc. and/or its affiliates. All rights reserved.*****

-----
Amazon's trademarks and trade dress may not be used in 
     connection with any product or service that is not Amazon's, 
     in any manner that is likely to cause confusion among customers, 
     or in any manner that disparages or discredits Amazon. All other 
     trademarks not owned by Amazon are the property of their respective
     owners, who may or may not be affiliated with, connected to, or 
     sponsored by Amazon.

-----
## Contents
+ [What is the AWS Encryption SDK?](introduction.md)
   + [Concepts in the AWS Encryption SDK](concepts.md)
   + [How the AWS Encryption SDK works](how-it-works.md)
   + [Supported algorithm suites in the AWS Encryption SDK](supported-algorithms.md)
+ [Getting started with the AWS Encryption SDK](getting-started.md)
+ [Best practices for the AWS Encryption SDK](best-practices.md)
+ [Using keyrings](choose-keyring.md)
+ [AWS Encryption SDK programming languages](programming-languages.md)
   + [AWS Encryption SDK for C](c-language.md)
      + [Installing the AWS Encryption SDK for C](c-language-installation.md)
      + [Using the AWS Encryption SDK for C](c-language-using.md)
      + [AWS Encryption SDK for C examples](c-examples.md)
   + [AWS Encryption SDK for Java](java.md)
      + [AWS Encryption SDK for Java example code](java-example-code.md)
   + [AWS Encryption SDK for JavaScript](javascript.md)
      + [Compatibility of the AWS Encryption SDK for JavaScript](javascript-compatibility.md)
      + [Installing the AWS Encryption SDK for JavaScript](javascript-installation.md)
      + [Modules in the AWS Encryption SDK for JavaScript](javascript-modules.md)
      + [AWS Encryption SDK for JavaScript examples](js-examples.md)
   + [AWS Encryption SDK for Python](python.md)
      + [AWS Encryption SDK for Python example code](python-example-code.md)
   + [AWS Encryption SDK command line interface](crypto-cli.md)
      + [Installing the AWS Encryption SDK command line interface](crypto-cli-install.md)
      + [How to use the AWS Encryption CLI](crypto-cli-how-to.md)
      + [Examples of the AWS Encryption CLI](crypto-cli-examples.md)
      + [AWS Encryption SDK CLI syntax and parameter reference](crypto-cli-reference.md)
      + [Versions of the AWS Encryption CLI](crypto-cli-versions.md)
+ [Data key caching](data-key-caching.md)
   + [How to use data key caching](implement-caching.md)
   + [Setting cache security thresholds](thresholds.md)
   + [Data key caching details](data-caching-details.md)
   + [Data key caching example](sample-cache-example.md)
      + [Data key caching example in Java](sample-cache-example-java.md)
      + [Data key caching example in Python](sample-cache-example-python.md)
      + [Local cache example AWS CloudFormation template](sample-cache-example-cloudformation.md)
+ [Migrating to AWS Encryption SDK Version 2.0.x](migration.md)
   + [Versions of the AWS Encryption SDK](about-versions.md)
   + [How to migrate and deploy the AWS Encryption SDK](migration-guide.md)
   + [Updating AWS KMS master key providers](migrate-mkps-v2.md)
   + [Updating AWS KMS keyrings](migrate-keyrings-v2.md)
   + [Setting your commitment policy](migrate-commitment-policy.md)
   + [Troubleshooting migration to version 2.0.x](troubleshooting-migration.md)
+ [Frequently asked questions](faq.md)
+ [AWS Encryption SDK reference](reference.md)
   + [AWS Encryption SDK message format reference](message-format.md)
   + [AWS Encryption SDK message format examples](message-format-examples.md)
   + [Body additional authenticated data (AAD) reference for the AWS Encryption SDK](body-aad-reference.md)
   + [AWS Encryption SDK algorithms reference](algorithms-reference.md)
   + [AWS Encryption SDK initialization vector reference](IV-reference.md)
+ [Document history for the AWS Encryption SDK Developer Guide](document-history.md)