# Updating AWS KMS keyrings<a name="migrate-keyrings-v2"></a>

The AWS KMS keyrings in the [AWS Encryption SDK for C](c-language.md), the [AWS Encryption SDK for \.NET](dot-net.md), and the [AWS Encryption SDK for JavaScript](javascript.md) support [best practices](best-practices.md) by allowing you to specify wrapping keys when encrypting and decrypting\. If you create an [AWS KMS discovery keyring](use-kms-keyring.md#kms-keyring-discovery), you do so explicitly\. 

**Note**  
The earliest version of the AWS Encryption SDK for \.NET is version 3\.0\.*x*\. All versions of the AWS Encryption SDK for \.NET support the security best practices introduced in 2\.0\.*x* of the AWS Encryption SDK\. You can safely upgrade to the latest version without any code or data changes\.

When you update to the latest 1\.*x* version of the AWS Encryption SDK, you can use a [discovery filter](use-kms-keyring.md#kms-keyring-discovery) to limit the wrapping keys that an [AWS KMS discovery keyring](use-kms-keyring.md#kms-keyring-discovery) or [AWS KMS regional discovery keyring](use-kms-keyring.md#kms-keyring-regional) uses when decrypting to those in particular AWS accounts\. Filtering a discovery keyring is an AWS Encryption SDK [best practice](best-practices.md)\.

The examples in this section will show you how to add the discovery filter to an AWS KMS regional discovery keyring\.

**Learn more about migration**

For all AWS Encryption SDK users, learn about setting your commitment policy in [Setting your commitment policy](migrate-commitment-policy.md)\.

For AWS Encryption SDK for Java, AWS Encryption SDK for Python, and AWS Encryption CLI users, learn about a required update to master key providers in [Updating AWS KMS master key providers](migrate-mkps-v2.md)\.

 

You might have code like the following in your application\. This example creates an AWS KMS regional discovery keyring that can only use wrapping keys in the US West \(Oregon\) \(us\-west\-2\) Region\. This example represents code in AWS Encryption SDK versions earlier than 1\.7\.*x*\. However, it is still valid in versions 1\.7\.*x* and later\. 

------
#### [ C ]

```
struct aws_cryptosdk_keyring *kms_regional_keyring = Aws::Cryptosdk::KmsKeyring::Builder()
       .WithKmsClient(create_kms_client(Aws::Region::US_WEST_2)).BuildDiscovery());
```

------
#### [ JavaScript Browser ]

```
const clientProvider = getClient(KMS, { credentials })

const discovery = true
const clientProvider = limitRegions(['us-west-2'], getKmsClient)
const keyring = new KmsKeyringBrowser({ clientProvider, discovery })
```

------
#### [ JavaScript Node\.js ]

```
const discovery = true
const clientProvider = limitRegions(['us-west-2'], getKmsClient)
const keyring = new KmsKeyringNode({ clientProvider, discovery })
```

------

Beginning in version 1\.7\.*x*, you can add a discovery filter to any AWS KMS discovery keyring\. This discovery filter limits the AWS KMS keys that the AWS Encryption SDK can use for decryption to those in the specified partition and accounts\. Before using this code, change the partition, if necessary, and replace the example account IDs with valid ones\.

------
#### [ C ]

For a complete example, see [kms\_discovery\.cpp](https://github.com/aws/aws-encryption-sdk-c/blob/master/examples/kms_discovery.cpp)\.

```
std::shared_ptr<KmsKeyring::DiscoveryFilter> discovery_filter(
    KmsKeyring::DiscoveryFilter::Builder("aws")
        .AddAccount("111122223333")
        .AddAccount("444455556666")
        .Build());

struct aws_cryptosdk_keyring *kms_regional_keyring = Aws::Cryptosdk::KmsKeyring::Builder()
       .WithKmsClient(create_kms_client(Aws::Region::US_WEST_2)).BuildDiscovery(discovery_filter));
```

------
#### [ JavaScript Browser ]

```
const clientProvider = getClient(KMS, { credentials })

const discovery = true
const clientProvider = limitRegions(['us-west-2'], getKmsClient)
const keyring = new KmsKeyringBrowser(clientProvider, {
    discovery,
    discoveryFilter: { accountIDs: ['111122223333', '444455556666'], partition: 'aws' }
})
```

------
#### [ JavaScript Node\.js ]

For a complete example, see [kms\_filtered\_discovery\.ts](https://github.com/aws/aws-encryption-sdk-javascript/blob/master/modules/example-node/src/kms_filtered_discovery.ts)\.

```
const discovery = true
const clientProvider = limitRegions(['us-west-2'], getKmsClient)
const keyring = new KmsKeyringNode({
    clientProvider,
    discovery,
    discoveryFilter: { accountIDs: ['111122223333', '444455556666'], partition: 'aws' }
})
```

------