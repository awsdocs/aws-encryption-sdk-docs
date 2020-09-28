# AWS Encryption SDK reference<a name="reference"></a>


|  | 
| --- |
|  The information on this page is a reference for building your own encryption library that is compatible with the AWS Encryption SDK\. If you are not building your own compatible encryption library, you likely do not need this information\. To use the AWS Encryption SDK in one of the supported programming languages, see [Programming languages](programming-languages.md)\. For the specification that defines the elements of a proper AWS Encryption SDK implementation, see the *AWS Encryption SDK Specification* in the [aws\-encryption\-sdk\-specification](https://github.com/awslabs/aws-encryption-sdk-specification/) repository in GitHub\.  | 

The AWS Encryption SDK uses the [supported algorithms](supported-algorithms.md) to return a single data structure or *message* that contains encrypted data and the corresponding encrypted data keys\. The following topics explain the algorithms and the data structure\. Use this information to build libraries that can read and write ciphertexts that are compatible with this SDK\.

**Topics**
+ [Message format reference](message-format.md)
+ [Message format examples](message-format-examples.md)
+ [Body AAD reference](body-aad-reference.md)
+ [Algorithms reference](algorithms-reference.md)
+ [Initialization vector reference](IV-reference.md)