# Choosing a keyring<a name="which-keyring"></a>

Your keyring determines the wrapping keys that protect your data keys, and ultimately, your data\. Use the most secure wrapping keys that are practical for your task\. Whenever possible use wrapping keys that are protected by a hardware security module or a key management infrastructure, such as KMS keys in [AWS Key Management Service](https://docs.aws.amazon.com/kms/latest/developerguide/) \(AWS KMS\) or encryption keys [AWS CloudHSM](https://docs.aws.amazon.com/cloudhsm/latest/userguide/)\.

The AWS Encryption SDK provides several keyrings and keyring configurations in multiple programming languages, and you can create your own custom keyrings\. You can also create a [multi\-keyring](use-multi-keyring.md) that includes one or more keyrings of the same or a different type\.

**Topics**
+ [AWS KMS keyrings](use-kms-keyring.md)
+ [Raw AES keyrings](use-raw-aes-keyring.md)
+ [Raw RSA keyrings](use-raw-rsa-keyring.md)
+ [Multi\-keyrings](use-multi-keyring.md)