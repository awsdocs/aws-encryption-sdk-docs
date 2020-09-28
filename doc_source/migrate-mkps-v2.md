# Updating AWS KMS master key providers<a name="migrate-mkps-v2"></a>

To migrate to versions 1\.7\.*x* and 2\.0\.*x*, you must replace legacy AWS KMS master key providers with master key providers created explicitly in [*strict mode* or *discovery mode*](about-versions.md#changes-to-mkps)\. Legacy master key providers are deprecated in version 1\.7\.*x* and removed in version 2\.0\.*x*\. This change is required for applications and scripts that use the [AWS Encryption SDK for Java](java.md), [AWS Encryption SDK for Python](python.md), and the [AWS Encryption CLI](crypto-cli.md)\. The examples in this section will show you how to update your code\. 

**Note**  
In Python, [turn on deprecation warnings](https://docs.python.org/3/library/warnings.html)\. This will help you identify the parts of your code that you need to update\.

If you are using an AWS KMS master key \(not a master key provider\), you can skip this step\. AWS KMS master keys are not deprecated or removed\. They encrypt and decrypt only with the wrapping keys that you specify\.

The examples in this section focus on the elements of your code that you need to change\. For a complete example of the updated code, see the Examples section of the GitHub repository for your [programming language](programming-languages.md)\. Also, these examples typically use key ARNs to represent customer master keys \(CMKs\)\. When you create a master key provider for encrypting, you can use any valid AWS KMS [key identifier](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#key-id) to represent a CMK \. When you create a master key provider for decrypting, you must use a key ARN\.

**Learn more about migration**

For all AWS Encryption SDK users, learn about setting your commitment policy in [Setting your commitment policy](migrate-commitment-policy.md)\.

For AWS Encryption SDK for C and AWS Encryption SDK for JavaScript users, learn about an optional update to keyrings in [Updating AWS KMS keyrings](migrate-keyrings-v2.md)\.

**Topics**
+ [Migrating to strict mode](#migrate-mkp-strict-mode)
+ [Migrating to discovery mode](#migrate-mkp-discovery-mode)

## Migrating to strict mode<a name="migrate-mkp-strict-mode"></a>

After updating to version 1\.7\.*x*, you can replace your legacy master key providers with master key providers in strict mode\. In strict mode, you must specify the wrapping keys to use when encrypting and decrypting\. The AWS Encryption SDK uses only the wrapping keys you specify\. Deprecated master key providers can decrypt data using any CMK that encrypted a data key, including CMKs in different AWS accounts and Regions\.

Master key providers in strict mode are introduced in the AWS Encryption SDK version 1\.7\.*x*\. They replace legacy master key providers, which are deprecated in 1\.7\.*x* and removed in 2\.0\.*x*\. Using master key providers in strict mode is an AWS Encryption SDK [ best practice](best-practices.md)\.

The following code creates a master key provider in strict mode that you can use for encrypting and decrypting\. 

------
#### [ Java ]

This example represents code in an application that uses the version 1\.6\.2 or earlier of the AWS Encryption SDK for Java\.

This code uses the `KmsMasterKeyProvider.builder()` method to instantiate an AWS KMS master key provider that uses one CMK as a wrapping key\. 

```
// Create a master key provider
// Replace the example key ARN with a valid one
String awsKmsCmk = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";

KmsMasterKeyProvider masterKeyProvider = KmsMasterKeyProvider.builder()
    .withKeysForEncryption(awsKmsCmk)
    .build();
```

This example represents code in an application that uses version 1\.7\.*x* or later of the AWS Encryption SDK for Java \. For a complete example, see [BasicEncryptionExample\.java](https://github.com/aws/aws-encryption-sdk-java/blob/master/src/examples/java/com/amazonaws/crypto/examples/BasicEncryptionExample.java)\.

The `Builder.build()` and `Builder.withKeysForEncryption()` methods used in the previous example are deprecated in version 1\.7\.*x* and are removed from version 2\.0\.*x*\.

To update to a strict mode master key provider, this code replaces calls to deprecated methods with a call to the new `Builder.buildStrict()` method\. This example specifies one customer master key \(CMK\) as the wrapping key, but the `Builder.buildStrict()` method can take a list of multiple CMKs\.

```
// Create a master key provider in strict mode
// Replace the example key ARN with a valid one from your AWS account.
String awsKmsCmk = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";

KmsMasterKeyProvider masterKeyProvider = KmsMasterKeyProvider.builder()
    .buildStrict(awsKmsCmk);
```

------
#### [ Python ]

This example represents code in an application that uses version 1\.4\.1 of the AWS Encryption SDK for Python\. This code uses `KMSMasterKeyProvider`, which is deprecated in version 1\.7\.*x* and removed from version 2\.0\.*x*\. When decrypting, it uses any CMK that encrypted a data key without regard to the CMKs you specify\. 

Note that `KMSMasterKey` is not deprecated or removed\. When encrypting and decrypting, it uses only the CMK you specify\.

```
# Create a master key provider
# Replace the example key ARN with a valid one
cmk_1 = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab"
cmk_2 = "arn:aws:kms:us-west-2:111122223333:key/0987dcba-09fe-87dc-65ba-ab0987654321"

aws_kms_master_key_provider = KMSMasterKeyProvider(
   key_ids=[cmk_1, cmk_2]
)
```

This example represents code in an application that uses version 1\.7\.*x* of the AWS Encryption SDK for Python\. For a complete example, see [basic\_encryption\.py](https://github.com/aws/aws-encryption-sdk-python/blob/master/examples/src/basic_encryption.py)\.

To update to a strict mode master key provider, this code replaces the call to `KMSMasterKeyProvider()` with a call to `StrictAwsKmsMasterKeyProvider()`\. 

```
# Create a master key provider in strict mode
# Replace the example key ARNs with valid values from your AWS account
cmk_1 = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab"
cmk_2 = "arn:aws:kms:us-west-2:111122223333:key/0987dcba-09fe-87dc-65ba-ab0987654321"

aws_kms_master_key_provider = StrictAwsKmsMasterKeyProvider(
    key_ids=[cmk_1, cmk_2]
)
```

------
#### [ AWS Encryption CLI ]

This example shows how to encrypt and decrypt using the AWS Encryption CLI version 1\.1\.7 or earlier\.

In version 1\.1\.7 and earlier, when encrypting, you specify one or more master keys \(or *wrapping keys*\), such as an AWS KMS customer master key \(CMK\)\. When decrypting, you can't specify any wrapping keys unless you are using a custom master key provider\. The AWS Encryption CLI can use any wrapping key that encrypted a data key\.

```
\\ Replace the example key ARN with a valid one
$ cmkArn=arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab

\\ Encrypt your plaintext data
$ aws-encryption-cli --encrypt \
                     --input hello.txt \
                     --master-keys key=$cmkArn \
                     --metadata-output ~/metadata \
                     --encryption-context purpose=test \
                     --output .

\\ Decrypt your ciphertext               
$ aws-encryption-cli --decrypt \
                     --input hello.txt.encrypted \
                     --encryption-context purpose=test \
                     --metadata-output ~/metadata \
                     --output .
```

This example shows how to encrypt and decrypt using the AWS Encryption CLI version 1\.7\.*x* or later\. For complete examples, see [Examples of the AWS Encryption CLI](crypto-cli-examples.md)\.

The `--master-keys` parameter is deprecated in version 1\.7\.*x* and removed in version 2\.0\.*x*\. It's replaced the by `--wrapping-keys` parameter, which is required in encrypt and decrypt commands\. This parameter supports strict mode and discovery mode\. Strict mode is an AWS Encryption SDK best practice that assures that you use the wrapping key that you intend\. 

To upgrade to *strict mode*, use the **key** attribute of the `--wrapping-keys` parameter to specify a wrapping key when encrypting and decrypting\. 

```
\\ Replace the example key ARN with a valid value
$ cmkArn=arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab

\\ Encrypt your plaintext data
$ aws-encryption-cli --encrypt \
                     --input hello.txt \
                     --wrapping-keys key=$cmkArn \
                     --metadata-output ~/metadata \
                     --encryption-context purpose=test \
                     --output .

\\ Decrypt your ciphertext               
$ aws-encryption-cli --decrypt \
                     --input hello.txt.encrypted \
                     --wrapping-keys key=$cmkArn \
                     --encryption-context purpose=test \
                     --metadata-output ~/metadata \
                     --output .
```

------

## Migrating to discovery mode<a name="migrate-mkp-discovery-mode"></a>

Beginning in version 1\.7\.*x*, it's an AWS Encryption SDK [best practice](best-practices.md) to use *strict mode* for AWS KMS master key providers, that is, to specify wrapping keys when encrypting and decrypting\. You must always specify wrapping keys when encrypting\. But there are situations in which specifying the key ARNs of CMKs for decrypting is impractical\. For example, if you're using aliases to identify CMKs when encrypting, you lose the benefit of aliases if you have to list key ARNs when decrypting\. Also, because master key providers in discovery mode behave like the original master key providers, you might use them temporarily as part of your migration strategy, and then upgrade to master key providers in strict mode later\.

In cases like this, you can use master key providers in *discovery mode*\. These master key providers don't let you specify wrapping keys, so you cannot use them for encrypting\. When decrypting, they can use any wrapping key that encrypted a data key\. But unlike legacy master key providers, which behave the same way, you create them in discovery mode explicitly\. When using master key providers in discovery mode, you can limit the wrapping keys that can be used to those in particular AWS accounts\. This filter is optional, but it's a best practice that we recommend\. For information about AWS partitions and accounts, see [Amazon Resource Names](https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html#arns-syntax) in the *AWS General Reference*\.

The following examples create an AWS KMS master key provider in strict mode for encrypting and an AWS KMS master key provider in discovery mode for decrypting\. The master key provider in discovery mode uses a discovery filter to limit the wrapping keys used for decrypting to the `aws` partition and to particular example AWS accounts\. Although the account filter is not necessary in this very simple example, it's a best practice that is very beneficial when one application encrypts data and a different application decrypts the data\.

------
#### [ Java ]

This example represents code in an application that uses version 1\.7\.*x* or later of the AWS Encryption SDK for Java\. For a complete example, see [DiscoveryDecryptionExample\.java](https://github.com/aws/aws-encryption-sdk-java/blob/master/src/examples/java/com/amazonaws/crypto/examples/)\.

To instantiate a master key provider in strict mode for encrypting, this example uses the `Builder.buildStrict()` method\. To instantiate a master key provider in discovery mode for decrypting, it uses the the `Builder.buildDiscovery()` method\. The `Builder.buildDiscovery()` method takes a `DiscoveryFilter` that limits the AWS Encryption SDK to CMKs in the specified AWS partition and accounts\. 

```
// Create a master key provider in strict mode for encrypting
// Replace the example alias ARN with a valid one from your AWS account.
String awsKmsCmk = "arn:aws:kms:us-west-2:111122223333:alias/ExampleAlias";

KmsMasterKeyProvider encryptingKeyProvider = KmsMasterKeyProvider.builder()
    .buildStrict(awsKmsCmk);

// Create a master key provider in discovery mode for decrypting
// Replace the example account IDs with valid values.
DiscoveryFilter accounts = new DiscoveryFilter("aws", Arrays.asList("111122223333", "444455556666"));

KmsMasterKeyProvider decryptingKeyProvider = KmsMasterKeyProvider.builder()
    .buildDiscovery(accounts);
```

------
#### [ Python ]

 This example represents code in an application that uses version 1\.7\.*x* or later of the AWS Encryption SDK for Python \. For a complete example, see [discovery\_kms\_provider\.py](https://github.com/aws/aws-encryption-sdk-python/blob/master/examples/src/discovery_kms_provider.py)\.

To create a master key provider in strict mode for encrypting, this example uses `StrictAwsKmsMasterKeyProvider`\. To create a master key provider in discovery mode for decrypting, it uses `DiscoveryAwsKmsMasterKeyProvider` with a `DiscoveryFilter` that limits the AWS Encryption SDK to CMKs in the specified AWS partition and accounts\. 

```
# Create a master key provider in strict mode
# Replace the example key ARN and alias ARNs with valid values from your AWS account.
cmk_1 = "arn:aws:kms:us-west-2:111122223333:alias/ExampleAlias"
cmk_2 = "arn:aws:kms:us-west-2:444455556666:key/1a2b3c4d-5e6f-1a2b-3c4d-5e6f1a2b3c4d"

aws_kms_master_key_provider = StrictAwsKmsMasterKeyProvider(
    key_ids=[cmk_1, cmk_2]
)

# Create a master key provider in discovery mode for decrypting
# Replace the example account IDs with valid values
accounts = DiscoveryFilter(
    partition="aws",
    account_ids=["111122223333", "444455556666"]
)
aws_kms_master_key_provider = DiscoveryAwsKmsMasterKeyProvider(
        discovery_filter=accounts
)
```

------
#### [ AWS Encryption CLI ]

This example shows how to encrypt and decrypt using the AWS Encryption CLI version 1\.7\.*x* or later\. Beginning in version 1\.7\.*x*, the `--wrapping-keys` parameter is required when encrypting and decrypting\. The `--wrapping-keys` parameter supports strict mode and discovery mode\. For complete examples, see [Examples of the AWS Encryption CLI](crypto-cli-examples.md)\.

When encrypting, this example specifies a wrapping key, which is required\. When decrypting, it explicitly chooses *discovery mode* by using the `discovery` attribute of the `--wrapping-keys` parameter with a value of `true`\. 

To limit the wrapping keys that the AWS Encryption SDK can use in discovery mode to those in particular AWS accounts, this example uses the `discovery-partition` and `discovery-account` attributes of the `--wrapping-keys` parameter\. These optional attributes are valid only when the `discovery` attribute is set to `true`\. You must use the `discovery-partition` and `discovery-account` attributes together; neither is valid alone\.

```
\\ Replace the example key ARN with a valid value
$ cmkAlias=arn:aws:kms:us-west-2:111122223333:alias/ExampleAlias

\\ Encrypt your plaintext data
$ aws-encryption-cli --encrypt \
                     --input hello.txt \
                     --wrapping-keys key=$cmkAlias \
                     --metadata-output ~/metadata \
                     --encryption-context purpose=test \
                     --output .

\\ Decrypt your ciphertext
\\ Replace the example account IDs with valid values           
$ aws-encryption-cli --decrypt \
                     --input hello.txt.encrypted \
                     --wrapping-keys discovery=true \
                                     discovery-partition=aws \
                                     discovery-account=111122223333 \
                                     discovery-account=444455556666 \
                     --encryption-context purpose=test \
                     --metadata-output ~/metadata \
                     --output .
```

------