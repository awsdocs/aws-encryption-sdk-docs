# Getting started with the AWS Encryption SDK<a name="getting-started"></a>

To use the AWS Encryption SDK, you need to configure [keyring](concepts.md#keyring) or [master key provider](concepts.md#master-key-provider) with encryption keys\. If you don't have a key infrastructure, we recommend using [AWS Key Management Service \(AWS KMS\)](https://aws.amazon.com/kms/)\. Many of the code examples in the AWS Encryption SDK require an AWS KMS [customer master key](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#master_keys) \(CMK\)\.

To interact with AWS KMS, you need to use the AWS SDK for your preferred programming language, such as the AWS SDK for Java, the AWS SDK for Python \(Boto\), the AWS SDK for JavaScript, or the AWS SDK for C\+\+, which you use with the AWS Encryption SDK for C\. The AWS Encryption SDK client library works with the AWS SDKs to support master keys stored in AWS KMS\. 

**To prepare to use the AWS Encryption SDK with AWS KMS**

1. Create an AWS account\. To learn how, see [How do I create and activate a new Amazon Web Services account?](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/) in the AWS Knowledge Center\.

1. Create a customer master key \(CMK\) in AWS KMS\. To learn how, see [Creating Keys](https://docs.aws.amazon.com/kms/latest/developerguide/create-keys.html) in the *AWS Key Management Service Developer Guide*\.
**Tip**  
To use the CMK programmatically, you will need the key ID or Amazon Resource Name \(ARN\) of the CMK\. For help finding the ID or ARN of a CMK, see [Finding the Key ID and ARN](https://docs.aws.amazon.com/kms/latest/developerguide/viewing-keys.html#find-cmk-id-arn) in the *AWS Key Management Service Developer Guide*\.

1. Create an IAM user with an access key\. To learn how, see [Creating IAM Users](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html#id_users_create_console) in the *IAM User Guide*\. When you create the user, for **Access type**, choose **Programmatic access**\. After you create the user, choose **Download\.csv** to save the AWS access key that represents your user credentials\. [Store the file in a secure location](https://docs.aws.amazon.com/general/latest/gr/aws-access-keys-best-practices.html#iam-user-access-keys)\. 

   We recommend that you use AWS Identity and Access Management \(IAM\) access keys instead of AWS \(root\) account access keys\. IAM lets you securely control access to AWS services and resources in your AWS account\. For detailed best practice guidance, see [Best Practices for Managing AWS Access Keys](https://docs.aws.amazon.com/general/latest/gr/aws-access-keys-best-practices.html#iam-user-access-keys)\.

   The `Download.csv` file contains an AWS access key ID and a secret access key that represents the AWS credentials of the user that you created\. When you write code without using an AWS SDK, you use your access key to sign your requests to AWS\. The signature assures AWS that the request came from you unchanged\. However, when you use an AWS SDK, such as the AWS SDK for Java, the SDK signs all requests to AWS for you\. 

1. Set your AWS credentials using the instructions for [Java](https://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/setup-credentials.html), [JavaScript](https://docs.aws.amazon.com/sdk-for-javascript/v2/developer-guide/getting-your-credentials.html), [Python](http://boto3.amazonaws.com/v1/documentation/api/latest/guide/configuration.html#guide-configuration) or [C\+\+](https://docs.aws.amazon.com/sdk-for-cpp/latest/developer-guide/credentials.html) \(for C\), and the AWS access key in the `Download.csv` file that you downloaded in step 3\. 

   This procedure allows AWS SDKs to sign requests to AWS for you\. Code samples in the AWS Encryption SDK that interact with AWS KMS assume that you have completed this step\.

1. Download and install the AWS Encryption SDK\. To learn how, see the installation instructions for the [programming language](programming-languages.md) that you want to use\.