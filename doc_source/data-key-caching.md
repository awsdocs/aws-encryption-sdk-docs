# Data key caching<a name="data-key-caching"></a>

*Data key caching* stores [data keys](concepts.md#DEK) and [related cryptographic material](data-caching-details.md#cache-entries) in a cache\. When you encrypt or decrypt data, the AWS Encryption SDK looks for a matching data key in the cache\. If it finds a match, it uses the cached data key rather than generating a new one\. Data key caching can improve performance, reduce cost, and help you stay within service limits as your application scales\. 

Your application can benefit from data key caching if:
+ It can reuse data keys\.
+ It generates numerous data keys\. 
+ Your cryptographic operations are unacceptably slow, expensive, limited, or resource\-intensive\.

Caching can reduce your use of cryptographic services, such as AWS Key Management Service \(AWS KMS\)\. If you are hitting your [AWS KMS requests\-per\-second limit](https://docs.aws.amazon.com/kms/latest/developerguide/limits.html#requests-per-second), caching can help\. Your application can use cached keys to service some of your data key requests instead of calling AWS KMS\. \(You can also create a case in the [AWS Support Center](https://console.aws.amazon.com/support/home#/) to raise the limit for your account\.\)

The AWS Encryption SDK helps you to create and manage your data key cache\. It provides a [local cache](data-caching-details.md#simplecache) and a [caching cryptographic materials manager](data-caching-details.md#caching-cmm) \(caching CMM\) that interacts with the cache and enforces [security thresholds](thresholds.md) that you set\. Working together, these components help you to benefit from the efficiency of reusing data keys while maintaining the security of your system\.

Data key caching is an optional feature of the AWS Encryption SDK that you should use cautiously\. By default, the AWS Encryption SDK generates a new data key for every encryption operation\. This technique supports cryptographic best practices, which discourage excessive reuse of data keys\. In general, use data key caching only when it is required to meet your performance goals\. Then, use the data key caching [security thresholds](thresholds.md) to ensure that you use the minimum amount of caching required to meet your cost and performance goals\. 

For a detailed discussion of these security tradeoffs, see [AWS Encryption SDK: How to Decide if Data Key Caching is Right for Your Application](http://aws.amazon.com/blogs/security/aws-encryption-sdk-how-to-decide-if-data-key-caching-is-right-for-your-application/) in the AWS Security Blog\.

**Topics**
+ [How to use data key caching](implement-caching.md)
+ [Setting cache security thresholds](thresholds.md)
+ [Data key caching details](data-caching-details.md)
+ [Data key caching example](sample-cache-example.md)