# AWS Encryption SDK initialization vector reference<a name="IV-reference"></a>


|  | 
| --- |
|  The information on this page is a reference for building your own encryption library that is compatible with the AWS Encryption SDK\. If you are not building your own compatible encryption library, you likely do not need this information\. To use the AWS Encryption SDK in one of the supported programming languages, see [Programming languages](programming-languages.md)\. For the specification that defines the elements of a proper AWS Encryption SDK implementation, see the *AWS Encryption SDK Specification* in the [aws\-encryption\-sdk\-specification](https://github.com/awslabs/aws-encryption-sdk-specification/) repository in GitHub\.  | 

The AWS Encryption SDK supplies the [initialization vectors](https://en.wikipedia.org/wiki/Initialization_vector) \(IVs\) that are required by all supported [algorithm suites](algorithms-reference.md)\. The SDK uses frame sequence numbers to construct an IV so that no two frames in the same message can have the same IV\. 

Each 96\-bit \(12\-byte\) IV is constructed from two big\-endian byte arrays concatenated in the following order:
+ 64 bits: 0 \(reserved for future use\)
+ 32 bits: Frame sequence number\. For the header authentication tag, this value is all zeroes\.

Before the introduction of [data key caching](data-key-caching.md), the AWS Encryption SDK always used a new data key to encrypt each message, and it generated all IVs randomly\. Randomly generated IVs were cryptographically safe because data keys were never reused\. When the SDK introduced data key caching, which intentionally reuses data keys, we changed the way the SDK generates IVs\. 

Using deterministic IVs that cannot repeat within a message significantly increases the number of invocations that can safely be executed under a single data key\. In addition, data keys that are cached always use an algorithm suite with a [key derivation function](https://en.wikipedia.org/wiki/Key_derivation_function)\. Using a deterministic IV with a pseudo\-random key derivation function to derive encryption keys from a data key allows the AWS Encryption SDK to encrypt 2^32 messages without exceeding cryptographic bounds\. 