# AWS KMS keyrings<a name="use-kms-keyring"></a>

An AWS KMS keyring uses symmetric encryption [AWS KMS keys](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#master_keys) to generate, encrypt, and decrypt data keys\. AWS Key Management Service \(AWS KMS\) protects your KMS keys and performs cryptographic operations within the FIPS boundary\. We recommend that you use a AWS KMS keyring, or a keyring with similar security properties, whenever possible\.

You can use an AWS KMS multi\-Region key in an AWS KMS keyring or master key provider beginning in [version 2\.3\.*x*](about-versions.md#version2.3) of the AWS Encryption SDK and [version 3\.0\.*x*](about-versions.md#version3.0) of the AWS Encryption CLI\. For details and examples of using the new multi\-Region\-aware symbol, see [Use multi\-Region AWS KMS keys](configure.md#config-mrks)\. For information about multi\-Region keys, see [Using multi\-Region keys](https://docs.aws.amazon.com/kms/latest/developerguide/multi-region-keys-overview.html) in the *AWS Key Management Service Developer Guide*\.

**Note**  
The AWS Encryption SDK does not support asymmetric KMS keys\. If you include an asymmetric KMS key in an encryption keyring, the encrypt call fails\. If you include it in a decryption keyring, it is ignored\.  
All mentions of *KMS keyrings* in the AWS Encryption SDK refer to AWS KMS keyrings\.

AWS KMS keyrings can include two types of wrapping keys:
+ **Generator key**: Generates a plaintext data key and encrypts it\. A keyring that encrypts data must have one generator key\.
+ **Additional keys**: Encrypts the plaintext data key that the generator key generated\. AWS KMS keyrings can have zero or more additional keys\.

When encrypting, the AWS KMS keyring that you use must have a generator key\. When decrypting, the generator key is optional, and the distinction between generators keys and additional keys is ignored\. 

When an AWS KMS encryption keyring has just one AWS KMS key, that key is used to generate and encrypt the data key\. 

Like all keyrings, AWS KMS keyrings can be used independently or in a [multi\-keyring](use-multi-keyring.md) with other keyrings of the same or a different type\.

**Topics**
+ [Required permissions for AWS KMS keyrings](#kms-keyring-permissions)
+ [Identifying AWS KMS keys in an AWS KMS keyring](#kms-keyring-id)
+ [Creating an AWS KMS keyring for encryption](#kms-keyring-encrypt)
+ [Creating an AWS KMS keyring for decryption](#kms-keyring-decrypt)
+ [Using an AWS KMS discovery keyring](#kms-keyring-discovery)
+ [Using an AWS KMS regional discovery keyring](#kms-keyring-regional)

## Required permissions for AWS KMS keyrings<a name="kms-keyring-permissions"></a>

The AWS Encryption SDK doesn't require an AWS account and it doesn't depend on any AWS service\. However, to use an AWS KMS keyring, you need an AWS account and the following minimum permissions on the AWS KMS keys in your keyring\. 
+ To encrypt with an AWS KMS keyring, you need [kms:GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) permission on the generator key\. You need [kms:Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html) permission on all additional keys in the AWS KMS keyring\.
+ To decrypt with an AWS KMS keyring, you need [kms:Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) permission on at least one key in the AWS KMS keyring\.
+ To encrypt with a multi\-keyring comprised of AWS KMS keyrings, you need [kms:GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) permission on the generator key in the generator keyring\. You need [kms:Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html) permission on all other keys in all other AWS KMS keyrings\. 

For detailed information about permissions for AWS KMS keys, see [Authentication and access control](https://docs.aws.amazon.com/kms/latest/developerguide/control-access.html) in the *AWS Key Management Service Developer Guide*\.

## Identifying AWS KMS keys in an AWS KMS keyring<a name="kms-keyring-id"></a>

An AWS KMS keyring can include one or more AWS KMS keys\. To specify an AWS KMS key in an AWS KMS keyring, use a supported AWS KMS key identifier\. The key identifiers you can use to identify an AWS KMS key in a keyring vary with the operation and the language implementation\. For details about the key identifiers for an AWS KMS key, see [Key Identifiers](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#key-id) in the *AWS Key Management Service Developer Guide*\.

As a best practice, use the most specific key identifier that is practical for your task\.
+ In an encryption keyring for the AWS Encryption SDK for C, you can use a [key ARN](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#key-id-key-ARN) or [alias ARN](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#key-id-alias-ARN) to identify KMS keys\. In all other language implementations, you can use a [key ID](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#key-id-key-id), [key ARN](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#key-id-key-ARN), [alias name](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#key-id-alias-name), or [alias ARN](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#key-id-alias-ARN) to encrypt data\.
+ In a decryption keyring, you must use a key ARN to identify AWS KMS keys\. This requirement applies to all language implementations of the AWS Encryption SDK\. For details, see [Select wrapping keys](configure.md#config-keys)\.
+ In a keyring used for encryption and decryption, you must use a key ARN to identify AWS KMS keys\. This requirement applies to all language implementations of the AWS Encryption SDK\.

If you specify an alias name or alias ARN for a KMS key in an encryption keyring, the encrypt operation saves the key ARN currently associated with the alias in the metadata of the encrypted data key\. It does not save the alias\. Changes to the alias don't affect the KMS key used to decrypt your encrypted data keys\.

## Creating an AWS KMS keyring for encryption<a name="kms-keyring-encrypt"></a>

You can configure each AWS KMS keyring with a single AWS KMS key or multiple AWS KMS keys in the same or different AWS accounts and AWS Regions\. The AWS KMS keys must be symmetric encryption keys \(SYMMETRIC\_DEFAULT\)\. You can also use a symmetric encryption [multi\-Region KMS key](configure.md#config-mrks)\. As with all keyrings, you can use one or more AWS KMS keyrings in a [multi\-keyring](use-multi-keyring.md)\. 

When you create an AWS KMS keyring to encrypt data, you must specify a *generator key*, which is an AWS KMS key that is used to generate a plaintext data key and encrypt it\. The data key is mathematically unrelated to the KMS key\. Then, if you choose, you can specify additional AWS KMS keys that encrypt the same plaintext data key\. 

To decrypt the encrypted message protected by this keyring, the keyring that you use must include at least one of the AWS KMS keys defined in the keyring, or no AWS KMS keys\. \(An AWS KMS keyring with no AWS KMS keys is known as an [AWS KMS discovery keyring](#kms-keyring-discovery)\.\)

In AWS Encryption SDK language implementations other than the AWS Encryption SDK for C, all wrapping keys in an encryption keyring or multi\-keyring must be able to encrypt the data key\. If any wrapping key fails to encrypt, the encrypt method fails\. As a result, the caller must have the [required permissions](#kms-keyring-permissions) for all keys in the keyring\. If you use a discovery keyring to encrypt data, alone or in a multi\-keyring, the encrypt operation fails\. The exception is the AWS Encryption SDK for C, where the encrypt operation ignores a standard discovery keyring, but fails if you specify a multi\-Region discovery keyring, alone or in a multi\-keyring\.

The following examples create an AWS KMS keyring with one generator key and one additional key\. These examples use [key ARNs](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#key-id-key-ARN) to identify the KMS keys\. This is a best practice for AWS KMS keyrings used for encryption, and a requirement for AWS KMS keyrings used for decryption\. For details, see [Identifying AWS KMS keys in an AWS KMS keyring](#kms-keyring-id)\.

------
#### [ C ]

To identify an AWS KMS key in an encryption keyring in the AWS Encryption SDK for C, specify a [key ARN](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#key-id-key-ARN) or [alias ARN](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#key-id-alias-arn)\. In a decryption keyring, you must use a key ARN\. For details, see [Identifying AWS KMS keys in an AWS KMS keyring](#kms-keyring-id)\.

For a complete example, see [string\.cpp](https://github.com/aws/aws-encryption-sdk-c/blob/master/examples/string.cpp)\.

```
const char * generator_key = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab"

const char * additional_key = "arn:aws:kms:us-west-2:111122223333:key/0987dcba-09fe-87dc-65ba-ab0987654321"    

struct aws_cryptosdk_keyring *kms_encrypt_keyring = 
       Aws::Cryptosdk::KmsKeyring::Builder().Build(generator_key,{additional_key});
```

------
#### [ C\# / \.NET ]

To create an AWS KMS keyring with one or multiple AWS KMS keys in the AWS Encryption SDK for \.NET, create a multi\-keyring\. The AWS Encryption SDK for \.NET includes a multi\-keyring just for AWS KMS keys\.

When you specify an AWS KMS key for an encryption keyring in the AWS Encryption SDK for \.NET, you can use any valid key identifier: a [key ID](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#key-id-key-id), [key ARN](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#key-id-key-ARN), [alias name](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#key-id-alias-name), or [alias ARN](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#key-id-alias-arn)\. For help identifying the AWS KMS keys in an AWS KMS keyring, see [Identifying AWS KMS keys in an AWS KMS keyring](#kms-keyring-id)\.

The following example creates an AWS KMS keyring with a generator key and additional keys\. For a complete example, see [AwsKmsMultiKeyringExample\.cs](https://github.com/aws/aws-encryption-sdk-dafny/blob/mainline/aws-encryption-sdk-net/Examples/Keyring/AwsKmsMultiKeyringExample.cs)\.

```
// Instantiate the AWS Encryption SDK and material provider
var encryptionSdk = AwsEncryptionSdkFactory.CreateDefaultAwsEncryptionSdk();
var materialProviders =
    AwsCryptographicMaterialProvidersFactory.CreateDefaultAwsCryptographicMaterialProviders();

string generatorKey = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";
List<string> additionalKey = new List<string> { "alias/exampleAlias" };

// Instantiate the keyring input object
var kmsEncryptKeyringInput = new CreateAwsKmsMultiKeyringInput
{
    Generator = generatorKey,
    KmsKeyIds = additionalKey
};

var kmsEncryptKeyring = materialProviders.CreateAwsKmsMultiKeyring(kmsEncryptKeyringInput);
```

------
#### [ JavaScript Browser ]

When you specify an AWS KMS key for an encryption keyring in the AWS Encryption SDK for JavaScript, you can use any valid key identifier: a [key ID](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#key-id-key-id), [key ARN](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#key-id-key-ARN), [alias name](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#key-id-alias-name), or [alias ARN](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#key-id-alias-arn)\. For help identifying the AWS KMS keys in an AWS KMS keyring, see [Identifying AWS KMS keys in an AWS KMS keyring](#kms-keyring-id)\.

For a complete example, see [kms\_simple\.ts](https://github.com/aws/aws-encryption-sdk-javascript/blob/master/modules/example-browser/src/kms_simple.ts) in the AWS Encryption SDK for JavaScript repository in GitHub\.

```
const clientProvider = getClient(KMS, { credentials })
const generatorKeyId = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'
const additionalKey = 'alias/exampleAlias'

const keyring = new KmsKeyringBrowser({
  clientProvider, 
  generatorKeyId, 
  keyIds: [additionalKey] 
})
```

------
#### [ JavaScript Node\.js ]

When you specify an AWS KMS key for an encryption keyring in the AWS Encryption SDK for JavaScript, you can use any valid key identifier: a [key ID](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#key-id-key-id), [key ARN](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#key-id-key-ARN), [alias name](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#key-id-alias-name), or [alias ARN](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#key-id-alias-arn)\. For help identifying the AWS KMS keys in an AWS KMS keyring, see [Identifying AWS KMS keys in an AWS KMS keyring](#kms-keyring-id)\.

For a complete example, see [kms\_simple\.ts](https://github.com/aws/aws-encryption-sdk-javascript/blob/master/modules/example-node/src/kms_simple.ts) in the AWS Encryption SDK for JavaScript repository in GitHub\.

```
const generatorKeyId = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'
                            
const additionalKey = 'alias/exampleAlias'

const keyring = new KmsKeyringNode({
  generatorKeyId,
  keyIds: [additionalKey]
})
```

------

## Creating an AWS KMS keyring for decryption<a name="kms-keyring-decrypt"></a>

You also specify an AWS KMS keyring when decrypting the [encrypted message](concepts.md#message) that the AWS Encryption SDK returns\. If the decryption keyring specifies AWS KMS keys, the AWS Encryption SDK will use only those wrapping keys to decrypt the encrypted data keys in the encrypted message\. \(You can also use an [AWS KMS discovery keyring](#kms-keyring-discovery), which doesn't specify any AWS KMS keys\.\)

When decrypting, the AWS Encryption SDK searches the AWS KMS keyring for an AWS KMS key that can decrypt one of the encrypted data keys\. Specifically, the AWS Encryption SDK uses the following pattern for each encrypted data key in an encrypted message\.
+ The AWS Encryption SDK gets the key ARN of the AWS KMS key that encrypted the data key from the metadata of the encrypted message\.
+ The AWS Encryption SDK searches the decryption keyring for an AWS KMS key with a matching key ARN\.
+ If it finds an AWS KMS key with a matching key ARN in the keyring, the AWS Encryption SDK asks AWS KMS to use the KMS key to decrypt the encrypted data key\.
+ Otherwise, it skips to the next encrypted data key, if any\. 

The AWS Encryption SDK never attempts to decrypt an encrypted data key unless the key ARN of the AWS KMS key that encrypted that data key is included in the decryption keyring\. If the decryption keyring doesn't include the ARNs of any of the AWS KMS keys that encrypted any of the data keys, the AWS Encryption SDK fails the decrypt call without ever calling AWS KMS\.

Beginning in [version 1\.7\.*x*](about-versions.md#version-1.7), when decrypting an encrypted data key, the AWS Encryption SDK always passes the key ARN of the AWS KMS key to the `KeyId` parameter of the AWS KMS [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operation\. Identifying the AWS KMS key when decrypting is an AWS KMS best practice that assures that you decrypt the encrypted data key with the wrapping key you intend to use\. 

A decrypt call with an AWS KMS keyring succeeds when at least one AWS KMS key in the decryption keyring can decrypt one of the encrypted data keys in the encrypted message\. Also, the caller must have `kms:Decrypt` permission on that AWS KMS key\. This behavior enables you to encrypt data under multiple AWS KMS keys in different AWS Regions and accounts, but provide a more limited decryption keyring tailored to a particular account, Region, user, group, or role\. 

When you specify an AWS KMS key in a decryption keyring, you must use its key ARN\. Otherwise, the AWS KMS key is not recognized\. For help finding the key ARN, see [Finding the Key ID and ARN](https://docs.aws.amazon.com/kms/latest/developerguide/viewing-keys.html#find-cmk-id-arn) in the *AWS Key Management Service Developer Guide*\. 

**Note**  
If you reuse an encryption keyring for decrypting, be sure that the AWS KMS keys in the keyring are identified by their key ARNs\.

For example, the following AWS KMS keyring includes only the additional key that was used in the encryption keyring\. However, instead of referring to the additional key by its alias, `alias/exampleAlias`, the example uses the additional key's key ARN as required by decrypt calls\.

You can use this keyring to decrypt a message that was encrypted under both the generator key and the additional key, provided that you have permission to use the additional key to decrypt data\.

------
#### [ C ]

```
const char * additional_key = "arn:aws:kms:us-west-2:111122223333:key/0987dcba-09fe-87dc-65ba-ab0987654321"    

struct aws_cryptosdk_keyring *kms_decrypt_keyring = 
       Aws::Cryptosdk::KmsKeyring::Builder().Build(additional_key);
```

------
#### [ C\# / \.NET ]

Because this decrypt keyring includes only one AWS KMS key, the example uses the `CreateAwsKmsKeyring()` method with an instance of its `CreateAwsKmsKeyringInput` object\. To create a AWS KMS keyring with one AWS KMS key, you can use a single\-key or multi\-key keyring\. For details, see [Encrypting data in the AWS Encryption SDK for \.NET](dot-net-examples.md#dot-net-example-encrypt)\.

```
// Instantiate the AWS Encryption SDK and material providers
var encryptionSdk = AwsEncryptionSdkFactory.CreateDefaultAwsEncryptionSdk();
var materialProviders =
    AwsCryptographicMaterialProvidersFactory.CreateDefaultAwsCryptographicMaterialProviders();

string additionalKey = "arn:aws:kms:us-west-2:111122223333:key/0987dcba-09fe-87dc-65ba-ab0987654321";

// Instantiate a KMS keyring for one AWS KMS key.
var kmsDecryptKeyringInput = new CreateAwsKmsKeyringInput
{
    KmsClient = new AmazonKeyManagementServiceClient(),
    KmsKeyId = additionalKey
};

var kmsDecryptKeyring = materialProviders.CreateAwsKmsKeyring(kmsDecryptKeyringInput);
```

------
#### [ JavaScript Browser ]

```
const clientProvider = getClient(KMS, { credentials })
const additionalKey = 'arn:aws:kms:us-west-2:111122223333:key/0987dcba-09fe-87dc-65ba-ab0987654321'

const keyring = new KmsKeyringBrowser({ clientProvider, keyIds: [additionalKey] })
```

------
#### [ JavaScript Node\.js ]

```
const additionalKey = 'arn:aws:kms:us-west-2:111122223333:key/0987dcba-09fe-87dc-65ba-ab0987654321'

const keyring = new KmsKeyringNode({ keyIds: [additionalKey] })
```

------

You can also use an AWS KMS keyring that specifies a generator key for decrypting, such as the following one\. When decrypting, the AWS Encryption SDK ignores the distinction between generator keys and additional keys\. It can use any of the specified AWS KMS keys to decrypt an encrypted data key\. The call to AWS KMS succeeds only when the caller has permission to use that AWS KMS key to decrypt data\.

------
#### [ C ]

```
struct aws_cryptosdk_keyring *kms_decrypt_keyring = 
       Aws::Cryptosdk::KmsKeyring::Builder().Build(generator_key, {additional_key, other_key});
```

------
#### [ C\# / \.NET ]

```
// Instantiate the AWS Encryption SDK and material providers
var encryptionSdk = AwsEncryptionSdkFactory.CreateDefaultAwsEncryptionSdk();
var materialProviders =
    AwsCryptographicMaterialProvidersFactory.CreateDefaultAwsCryptographicMaterialProviders();

string generatorKey = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";

// Instantiate a KMS keyring for one AWS KMS key.
var kmsDecryptKeyringInput = new CreateAwsKmsKeyringInput
{
    KmsClient = new AmazonKeyManagementServiceClient(),
    KmsKeyId = generatorKey
};

var kmsDecryptKeyring = materialProviders.CreateAwsKmsKeyring(kmsDecryptKeyringInput);
```

------
#### [ JavaScript Browser ]

```
const clientProvider = getClient(KMS, { credentials })

const keyring = new KmsKeyringBrowser({
  clientProvider, 
  generatorKeyId, 
  keyIds: [additionalKey, otherKey]
})
```

------
#### [ JavaScript Node\.js ]

```
const keyring = new KmsKeyringNode({
  generatorKeyId,
  keyIds: [additionalKey, otherKey]
})
```

------

Unlike an encryption keyring that uses all of the specified AWS KMS keys, you can decrypt an encrypted message using a decryption keyring that includes AWS KMS keys that are unrelated to the encrypted message, and AWS KMS keys that the caller doesn't have permission to use\. If a decrypt call to AWS KMS fails, such as when the caller doesn't have the required permission, the AWS Encryption SDK just skips to the next encrypted data key\. 

## Using an AWS KMS discovery keyring<a name="kms-keyring-discovery"></a>

When decrypting, it's a [best practice](best-practices.md) to specify the wrapping keys that the AWS Encryption SDK can use\. To follow this best practice, use an AWS KMS decryption keyring that limits the AWS KMS wrapping keys to those that you specify\. However, you can also create an *AWS KMS discovery keyring*, that is, an AWS KMS keyring that doesn't specify any wrapping keys\. 

The AWS Encryption SDK provides a standard AWS KMS discovery keyring and a discovery keyring for AWS KMS multi\-Region keys\. For information about using multi\-Region keys with the AWS Encryption SDK, see [Use multi\-Region AWS KMS keys](configure.md#config-mrks)\.

Because it doesn't specify any wrapping keys, a discovery keyring can't encrypt data\. If you use a discovery keyring to encrypt data, alone or in a multi\-keyring, the encrypt operation fails\. The exception is the AWS Encryption SDK for C, where the encrypt operation ignores a standard discovery keyring, but fails if you specify a multi\-Region discovery keyring, alone or in a multi\-keyring\.

When decrypting, a discovery keyring allows the AWS Encryption SDK to ask AWS KMS to decrypt any encrypted data key by using the AWS KMS key that encrypted it, regardless of who owns or has access to that AWS KMS key\. The call succeeds only when the caller has `kms:Decrypt` permission on the AWS KMS key\.

**Important**  
If you include an AWS KMS discovery keyring in a decryption [multi\-keyring](use-multi-keyring.md), the discovery keyring overrides all KMS key restrictions specified by other keyrings in the multi\-keyring\. The multi\-keyring behaves like its least restrictive keyring\. An AWS KMS discovery keyring has no effect on encryption when used by itself or in a multi\-keyring\.

If you use a discovery keyring, we recommend that you use a *discovery filter* to limit the KMS keys that can be used to those in specified AWS accounts and partitions\. Discovery filters are supported in versions 1\.7\.*x* and later of the AWS Encryption SDK\. For help finding your account ID and partition, see [Your AWS account identifiers](https://docs.aws.amazon.com/general/latest/gr/acct-identifiers.html) and [ARN format](https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html#arns-syntax) in the *AWS General Reference*\.

The AWS Encryption SDK provides an AWS KMS discovery keyring for convenience\. However, we recommend that you use a more limited keyring whenever possible for the following reasons\.
+ **Authenticity** – An AWS KMS discovery keyring can use any AWS KMS key that was used to encrypt a data key in the encrypted message, just so the caller has permission to use that AWS KMS key to decrypt\. This might not be the AWS KMS key that the caller intends to use\. For example, one of the encrypted data keys might have been encrypted under a less secure AWS KMS key that anyone can use\. 
+ **Latency and performance** – An AWS KMS discovery keyring might be perceptibly slower than other keyrings because the AWS Encryption SDK tries to decrypt all of the encrypted data keys, including those encrypted by AWS KMS keys in other AWS accounts and Regions, and AWS KMS keys that the caller doesn't have permission to use for decryption\. 

The following code instantiates an AWS KMS discovery keyring with a discovery filter that limits the KMS keys the AWS Encryption SDK can use to those in the 111122223333 example account\.

------
#### [ C ]

For a complete example, see: [ kms\_discovery\.cpp](https://github.com/aws/aws-encryption-sdk-c/blob/master/examples/kms_discovery.cpp)\.

```
std::shared_ptr<KmsKeyring::> discovery_filter(
    KmsKeyring::DiscoveryFilter::Builder("aws")
        .AddAccount("111122223333")
        .Build());

struct aws_cryptosdk_keyring *kms_discovery_keyring = Aws::Cryptosdk::KmsKeyring::Builder()
       .BuildDiscovery(discovery_filter));
```

------
#### [ C\# / \.NET ]

```
// Instantiate the AWS Encryption SDK and material providers
var encryptionSdk = AwsEncryptionSdkFactory.CreateDefaultAwsEncryptionSdk();
var materialProviders =
    AwsCryptographicMaterialProvidersFactory.CreateDefaultAwsCryptographicMaterialProviders();

List<string> account = new List<string> { "111122223333" };
        
// In a discovery keyring, you specify an AWS KMS client and a discovery filter,
// but not a AWS KMS key
var kmsDiscoveryKeyringInput = new CreateAwsKmsDiscoveryKeyringInput
{
    KmsClient = new AmazonKeyManagementServiceClient(),
    DiscoveryFilter = new DiscoveryFilter()
    {
        AccountIds = account,
        Partition = "aws"
    }
};
        
var kmsDiscoveryKeyring = materialProviders.CreateAwsKmsDiscoveryKeyring(kmsDiscoveryKeyringInput);
```

------
#### [ JavaScript Browser ]

In JavaScript, you must explicitly specify the discovery property\.

```
const clientProvider = getClient(KMS, { credentials })

const discovery = true
const keyring = new KmsKeyringBrowser(clientProvider, {
    discovery,
    discoveryFilter: { accountIDs: [111122223333], partition: 'aws' }
})
```

------
#### [ JavaScript Node\.js ]

In JavaScript, you must explicitly specify the discovery property\.

```
const discovery = true

const keyring = new KmsKeyringNode({
    discovery,
    discoveryFilter: { accountIDs: ['111122223333'], partition: 'aws' }
})
```

------

## Using an AWS KMS regional discovery keyring<a name="kms-keyring-regional"></a>

An *AWS KMS regional discovery keyring* is a keyring that doesn't specify the ARNs of KMS keys\. Instead, it allows the AWS Encryption SDK to decrypt using only the KMS keys in particular AWS Regions\. 

When decrypting with an AWS KMS regional discovery keyring, the AWS Encryption SDK decrypts any encrypted data key that was encrypted under an AWS KMS key in the specified AWS Region\. To succeed, the caller must have `kms:Decrypt` permission on at least one of the AWS KMS keys in the specified AWS Region that encrypted a data key\.

Like other discovery keyrings, the regional discovery keyring has no effect on encryption\. It works only when decrypting encrypted messages\. If you use a regional discovery keyring in a multi\-keyring that is used for encrypting and decrypting, it is effective only when decrypting\. If you use a multi\-Region discovery keyring to encrypt data, alone or in a multi\-keyring, the encrypt operation fails\.

**Important**  
If you include an AWS KMS regional discovery keyring in a decryption [multi\-keyring](use-multi-keyring.md), the regional discovery keyring overrides all KMS key restrictions specified by other keyrings in the multi\-keyring\. The multi\-keyring behaves like its least restrictive keyring\. An AWS KMS discovery keyring has no effect on encryption when used by itself or in a multi\-keyring\.

The regional discovery keyring in the AWS Encryption SDK for C attempts to decrypt only with KMS keys in the specified Region\. When you use a discovery keyring in the AWS Encryption SDK for JavaScript and AWS Encryption SDK for \.NET, you configure the Region on the AWS KMS client\. These AWS Encryption SDK implementations don't filter KMS keys by Region, but AWS KMS will fail a decrypt request for KMS keys outside of the specified Region\.

If you use a discovery keyring, we recommend that you use a *discovery filter* to limit the KMS keys used in decryption to those in specified AWS accounts and partitions\. Discovery filters are supported in versions 1\.7\.*x* and later of the AWS Encryption SDK\.

For example, the following code creates an AWS KMS regional discovery keyring with a discovery filter\. This keyring limits the AWS Encryption SDK to KMS keys in account 111122223333 in the US West \(Oregon\) Region \(us\-west\-2\)\.

------
#### [ C ]

To view this keyring, and the `create_kms_client` method, in a working example, see [kms\_discovery\.cpp](https://github.com/aws/aws-encryption-sdk-c/blob/master/examples/kms_discovery.cpp)\.

```
std::shared_ptr<KmsKeyring::DiscoveryFilter> discovery_filter(
    KmsKeyring::DiscoveryFilter::Builder("aws")
        .AddAccount("111122223333")
        .Build());

struct aws_cryptosdk_keyring *kms_regional_keyring = Aws::Cryptosdk::KmsKeyring::Builder()
       .WithKmsClient(create_kms_client(Aws::Region::US_WEST_2)).BuildDiscovery(discovery_filter));
```

------
#### [ C\# / \.NET ]

The AWS Encryption SDK for \.NET does not have a dedicated regional discovery keyring\. However, you can use several techniques to limit the KMS keys used when decrypting to a particular Region\.

The most efficient way to limit the Regions in a discovery keyring is to use a multi\-Region\-aware discovery keyring, even if you encrypted the data using only single\-Region keys\. When it encounters single\-Region keys, the multi\-Region\-aware keyring does not use any multi\-Region features\. 

The keyring returned by the `CreateAwsKmsMrkDiscoveryKeyring()` method filters KMS keys by Region before calling AWS KMS\. It sends a decrypt request to AWS KMS only when the encrypted data key was encrypted by a KMS key in the Region specified by the `Region` parameter in the `CreateAwsKmsMrkDiscoveryKeyringInput` object\.

```
// Instantiate the AWS Encryption SDK and material providers
var encryptionSdk = AwsEncryptionSdkFactory.CreateDefaultAwsEncryptionSdk();
var materialProviders =
    AwsCryptographicMaterialProvidersFactory.CreateDefaultAwsCryptographicMaterialProviders();

List<string> account = new List<string> { "111122223333" };

// Create the discovery filter
var filter = DiscoveryFilter = new DiscoveryFilter
{
    AccountIds = account,
    Partition = "aws"
};
                                
var regionalDiscoveryKeyringInput = new CreateAwsKmsMrkDiscoveryKeyringInput
{
    KmsClient = new AmazonKeyManagementServiceClient(RegionEndpoint.USWest2),
    Region = RegionEndpoint.USWest2,
    DiscoveryFilter = filter
};                                

var kmsRegionalDiscoveryKeyring = materialProviders.CreateAwsKmsMrkDiscoveryKeyring(regionalDiscoveryKeyringInput);
```

You can also limit KMS keys to a particular AWS Region by specifying a Region in your instance of the AWS KMS client \([AmazonKeyManagementServiceClient](https://docs.aws.amazon.com/sdkfornet/v3/apidocs/items/KeyManagementService/TKeyManagementServiceClient.html)\)\. However, this configuration is less efficient and potentially more costly than using a multi\-Region\-aware discovery keyring\. Instead of filtering KMS keys by Region before calling AWS KMS, the AWS Encryption SDK for \.NET calls AWS KMS for each encrypted data key \(until it decrypts one\) and relies on AWS KMS to limit the KMS keys it uses to the specified Region\.

```
// Instantiate the AWS Encryption SDK and material providers
var encryptionSdk = AwsEncryptionSdkFactory.CreateDefaultAwsEncryptionSdk();
var materialProviders =
    AwsCryptographicMaterialProvidersFactory.CreateDefaultAwsCryptographicMaterialProviders();

List<string> account = new List<string> { "111122223333" };
        
// Create the discovery filter,
// but not a AWS KMS key
var createRegionalDiscoveryKeyringInput = new CreateAwsKmsDiscoveryKeyringInput
{
    KmsClient = new AmazonKeyManagementServiceClient(RegionEndpoint.USWest2),
    DiscoveryFilter = new DiscoveryFilter()
    {
        AccountIds = account,
        Partition = "aws"
    }
};
        
var kmsRegionalDiscoveryKeyring = materialProviders.CreateAwsKmsDiscoveryKeyring(createRegionalDiscoveryKeyringInput);
```

------
#### [ JavaScript Browser ]

```
const clientProvider = getClient(KMS, { credentials })

const discovery = true
const clientProvider = limitRegions(['us-west-2'], getKmsClient)
const keyring = new KmsKeyringBrowser(clientProvider, {
    discovery,
    discoveryFilter: { accountIDs: ['111122223333'], partition: 'aws' }
})
```

------
#### [ JavaScript Node\.js ]

To view this keyring, and the `limitRegions` and function, in a working example, see [kms\_regional\_discovery\.ts](https://github.com/aws/aws-encryption-sdk-javascript/blob/master/modules/example-node/src/kms_regional_discovery.ts)\.

```
const discovery = true
const clientProvider = limitRegions(['us-west-2'], getKmsClient)
const keyring = new KmsKeyringNode({
    clientProvider,
    discovery,
    discoveryFilter: { accountIDs: ['111122223333'], partition: 'aws' }
})
```

------

The AWS Encryption SDK for JavaScript also exports an `excludeRegions` function for Node\.js and the browser\. This function creates an AWS KMS regional discovery keyring that omits AWS KMS keys in particular regions\. The following example creates an AWS KMS regional discovery keyring that can use AWS KMS keys in account 111122223333 in every AWS Region except for US East \(N\. Virginia\) \(us\-east\-1\)\. 

The AWS Encryption SDK for C does not have an analogous method, but you can implement one by creating a custom [ClientSupplier](https://github.com/aws/aws-encryption-sdk-c/blob/master/aws-encryption-sdk-cpp/include/aws/cryptosdk/cpp/kms_keyring.h#L157)\.

This example shows the code for Node\.js\.

```
const discovery = true
const clientProvider = excludeRegions(['us-east-1'], getKmsClient)
const keyring = new KmsKeyringNode({
    clientProvider,
    discovery,
    discoveryFilter: { accountIDs: [111122223333], partition: 'aws' }
})
```