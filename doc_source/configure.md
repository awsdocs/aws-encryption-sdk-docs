# Configuring the AWS Encryption SDK<a name="configure"></a>

The AWS Encryption SDK is designed to be easy to use\. Although the AWS Encryption SDK has several configuration options, the default values are carefully chosen to be practical and secure for most applications\. However, you might need to adjust your configuration to improve performance or include a custom feature in your design\. 

When configuring your implementation, review the AWS Encryption SDK [best practices](best-practices.md) and implement as many as you can\.

**Topics**
+ [Select a programming language](#config-language)
+ [Select wrapping keys](#config-keys)
+ [Examine the security features](#config-security)
+ [Review other options](#config-other)

## Select a programming language<a name="config-language"></a>

The AWS Encryption SDK is available in multiple [programming languages](programming-languages.md)\. The language implementations are designed to be fully interoperable and to offer the same features, although they might be implemented in different ways\. Typically, you use the library that is compatible with your application\. However, you might select a programming language for a particular implementation\. For example, if you prefer to work with [keyrings](choose-keyring.md), you might choose the AWS Encryption SDK for C or the AWS Encryption SDK for JavaScript\. 

## Select wrapping keys<a name="config-keys"></a>

The AWS Encryption SDK generates a unique symmetric data key to encrypt each message\. Unless you are using [data key caching](data-key-caching.md), you don't need to configure, manage, or use the data keys\. The AWS Encryption SDK does it for you\.

However, you must select one or more wrapping keys to encrypt each data key\. The AWS Encryption SDK supports AES symmetric keys and RSA asymmetric keys in different sizes\. It also supports [AWS Key Management Service](https://docs.aws.amazon.com/kms/latest/developerguide/) \(AWS KMS\) symmetric customer master keys\. You are responsible for the safety and durability of your wrapping keys, so we recommend that you use an encryption key in a hardware security module or a key infrastructure service, such as AWS KMS\. 

To specify your wrapping keys for encryption and decryption, you use a keyring \(C and JavaScript\) or a master key provider \(Java and Python\)\. You can specify one wrapping key or multiple wrapping keys of the same or different types\. If you use multiple wrapping keys to wrap a data key, each wrapping key will encrypt a copy of the same data key\. The encrypted data keys \(one per wrapping key\) are stored with the encrypted data in the encrypted message that the AWS Encryption SDK returns\. To decrypt the data, the AWS Encryption SDK must first use one of your wrapping keys to decrypt an encrypted data key\. 

### Using multi\-Region KMS keys<a name="config-mrks"></a>

You can use AWS Key Management Service \(AWS KMS\) multi\-Region keys as wrapping keys in the AWS Encryption SDK\. If you encrypt with a multi\-Region key in one AWS Region, you can decrypt using a related multi\-Region key in a different AWS Region\. Support for multi\-Region keys is introduced in version 2\.3\.*x* of the AWS Encryption SDK and version 3\.0\.*x* of the AWS Encryption CLI\.

AWS KMS multi\-Region keys are a set of AWS KMS customer master keys \(CMKs\) in different AWS Regions that have the same key material and key ID\. You can use these *related* keys as though they were the same key in different Regions\. Multi\-Region keys support common disaster recovery and backup scenarios that require encrypting in one Region and decrypting in a different Region without making a cross\-Region call to AWS KMS\. For information about multi\-Region keys, see [Using multi\-Region keys](https://docs.aws.amazon.com/kms/latest/developerguide/multi-region-keys-overview.html) in the *AWS Key Management Service Developer Guide*\.

To support multi\-Region keys, the AWS Encryption SDK includes AWS KMS multi\-Region\-aware keyrings and master key providers\.

The new multi\-Region\-aware symbol in each programming language supports both single\-Region and multi\-Region keys\. 
+ For single\-Region keys, the multi\-Region\-aware symbol behaves just like the single\-Region AWS KMS keyring and master key provider\. It attempts to decrypt ciphertext only with the single\-Region key that encrypted the data\.
+ For multi\-Region keys, the multi\-Region\-aware symbol attempts to decrypt ciphertext with the same multi\-Region key that encrypted the data or any related multi\-Region key\.

  For example, suppose you encrypt with a multi\-Region key in the Europe \(Frankfurt\) Region\. Then you move your data to Asia Pacific \(Singapore\) and construct a multi\-Region\-aware symbol that specifies the related multi\-Region key in the Asia Pacific \(Singapore\) Region\. The multi\-Region\-aware symbol recognizes that the keys in Europe \(Frankfurt\) and Asia Pacific \(Singapore\) are related multi\-Region keys\. It uses the key in Asia Pacific \(Singapore\) to decrypt\.

You can specify multiple wrapping keys in the multi\-Region\-aware keyrings and master key providers\. However, you can specify only one key from each set of related multi\-Region keys\. If you specify more than one key identifier with the same key ID, the constructor call fails\.

You can also use a multi\-Region key with the standard, single\-Region AWS KMS keyrings and master key providers\. However, you must use the same multi\-Region key in the same Region to encrypt and decrypt\. The single\-Region keyrings and master key providers attempt to decrypt ciphertext only with the keys that encrypted the data\.

The following examples show how to encrypt and decrypt data using multi\-Region keys and the new multi\-Region\-aware keyrings and master key providers\. For example, to encrypt data in the `us-east-1` Region with a multi\-Region key, instantiate the multi\-Region\-aware symbol and specify a multi\-Region key in `us-east-1`\.

Before running these examples, replace the example multi\-Region key ARN with a valid value from your AWS account\.

------
#### [ C ]

To encrypt with a multi\-Region key, use the `Aws::Cryptosdk::KmsMrkAwareSymmetricKeyring::Builder()` method to instantiate the keyring\. Specify a multi\-Region key\.

This simple example does not include an [encryption context](concepts.md#encryption-context)\. For an example that uses an encryption context in C, see [Encrypting and decrypting strings](c-examples.md#c-example-strings)\.

For a complete example, see [kms\_multi\_region\_keys\.cpp](https://github.com/aws/aws-encryption-sdk-c/tree/master/examples/kms_multi_region_keys.cpp) in the AWS Encryption SDK for C repository on GitHub\.

```
/* Initialize a multi-Region keyring */
const char *mrk_us_east_1 = "arn:aws:kms:us-east-1:111122223333:key/mrk-1234abcd12ab34cd56ef1234567890ab";    

struct aws_cryptosdk_keyring *mrk_keyring = 
    Aws::Cryptosdk::KmsMrkAwareSymmetricKeyring::Builder().Build(mrk_us_east_1);

/* Create a session; release the keyring */
struct aws_cryptosdk_session *session =
    aws_cryptosdk_session_new_from_keyring_2(aws_default_allocator(), AWS_CRYPTOSDK_ENCRYPT, mrk_keyring);

aws_cryptosdk_session_set_commitment_policy(session,
    COMMITMENT_POLICY_REQUIRE_ENCRYPT_REQUIRE_DECRYPT);

aws_cryptosdk_keyring_release(mrk_keyring);

/* Encrypt the data
 *   aws_cryptosdk_session_process_full is designed for non-streaming data
 */
aws_cryptosdk_session_process_full(
    session, ciphertext, ciphertext_buf_sz, &ciphertext_len, plaintext, plaintext_len));

/* Clean up the session */
aws_cryptosdk_session_destroy(session);
```

------
#### [ AWS Encryption CLI ]

This example encrypts the `hello.txt` file under a multi\-Region key in the us\-east\-1 Region\. Because the example specifies a key ARN with a Region element, this example doesn't use the **region** attribute of the `--wrapping-keys` parameter\. 

When the key ID of the wrapping key doesn't specify a Region, you can use the **region** attribute of the `--wrapping-keys` to specify the region, such as `--wrapping-keys key=$keyID region=us-east-1`\. 

```
# To run this example, replace the fictitious key ARN with a valid value.
$ mrkUSEast1=arn:aws:kms:us-east-1:111122223333:key/mrk-1234abcd12ab34cd56ef1234567890ab

$ aws-encryption-cli --encrypt \
                     --input hello.txt \
                     --wrapping-keys key=$mrkUSEast1 \
                     --metadata-output ~/metadata \
                     --encryption-context purpose=test \
                     --commitment-policy require-encrypt-require-decrypt \
                     --output .
```

------
#### [ Java ]

To encrypt with a multi\-Region key, instantiate an `AwsKmsMrkAwareMasterKeyProvider` and specify a multi\-Region key\. 

For a complete example, see [BasicMultiRegionKeyEncryptionExample\.java](https://github.com/aws/aws-encryption-sdk-java/blob/master/src/examples/java/com/amazonaws/crypto/examples/BasicMultiRegionKeyEncryptionExample.java) in the AWS Encryption SDK for Java repository on GitHub\.

```
//Encrypt in us-east-1 Region

// Instantiate the client
final AwsCrypto crypto = AwsCrypto.builder()
    .withCommitmentPolicy(CommitmentPolicy.RequireEncryptRequireDecrypt)
    .build();

// Multi-Region keys have a distinctive key ID that begins with 'mrk'
// Specify a multi-Region key in us-east-1
final String mrkUSEast1 = "arn:aws:kms:us-east-1:111122223333:key/mrk-1234abcd12ab34cd56ef1234567890ab";

// Instantiate an AWS KMS master key provider in strict mode for multi-Region keys
// Configure it to encrypt with the multi-Region key in us-east-1
final AwsKmsMrkAwareMasterKeyProvider kmsMrkProvider = AwsKmsMrkAwareMasterKeyProvider
    .builder()
    .buildStrict(mrkUSEast1);

// Create an encryption context
final Map<String, String> encryptionContext = Collections.singletonMap("Purpose", "Test");

// Encrypt your plaintext data
final CryptoResult<byte[], AwsKmsMrkAwareMasterKey> encryptResult = crypto.encryptData(
    kmsMrkProvider,
    encryptionContext,
    sourcePlaintext);
byte[] ciphertext = encryptResult.getResult();
```

------
#### [ JavaScript Browser ]

To encrypt with a multi\-Region key, use the `buildAwsKmsMrkAwareStrictMultiKeyringBrowser()` method to create the keyring and specify a multi\-Region key\. 

For a complete example, see [kms\_multi\_region\_simple\.ts](https://github.com/aws/aws-encryption-sdk-javascript/blob/master/modules/example-browser/src/kms_multi_region_simple.ts) in the AWS Encryption SDK for JavaScript repository on GitHub\.

```
import {
  buildAwsKmsMrkAwareStrictMultiKeyringBrowser,
  buildClient,
  CommitmentPolicy,
  KMS,
} from '@aws-crypto/client-browser'


/* Instantiate an AWS Encryption SDK client */
const { encrypt } = buildClient(
  CommitmentPolicy.REQUIRE_ENCRYPT_REQUIRE_DECRYPT
)

declare const credentials: {
  accessKeyId: string
  secretAccessKey: string
  sessionToken: string
}

/* Instantiate an AWS KMS client 
 * The AWS Encryption SDK for JavaScript gets the Region from the key ARN
 */
const clientProvider = (region: string) => new KMS({ region, credentials })


/* Specify a multi-Region key in us-east-1 */
const multiRegionUsEastKey =
    'arn:aws:kms:us-east-1:111122223333:key/mrk-1234abcd12ab34cd56ef1234567890ab'

/* Instantiate the keyring */
const encryptKeyring = buildAwsKmsMrkAwareStrictMultiKeyringBrowser({
    generatorKeyId: multiRegionUsEastKey,
    clientProvider,
  })


/* Set the encryption context */
const context = {
    purpose: 'test',
  }

/* Test data to encrypt */
const cleartext = new Uint8Array([1, 2, 3, 4, 5])

/* Encrypt the data */
const { result } = await encrypt(encryptKeyring, cleartext, {
    encryptionContext: context,
  })
```

------
#### [ JavaScript Node\.js ]

To encrypt with a multi\-Region key, use the `buildAwsKmsMrkAwareStrictMultiKeyringNode()` method to create the keyring and specify a multi\-Region key\. 

For a complete example, see [kms\_multi\_region\_simple\.ts](https://github.com/aws/aws-encryption-sdk-javascript/blob/master/modules/example-node/src/kms_multi_region_simple.ts) in the AWS Encryption SDK for JavaScript repository on GitHub\.

```
import { buildClient } from '@aws-crypto/client-node'

/* Instantiate the AWS Encryption SDK client
const { encrypt } = buildClient(
  CommitmentPolicy.REQUIRE_ENCRYPT_REQUIRE_DECRYPT
)

/* Test string to encrypt */
const cleartext = 'asdf'

/* Multi-Region keys have a distinctive key ID that begins with 'mrk'
 * Specify a multi-Region key in us-east-1
 */
const multiRegionUsEastKey =
    'arn:aws:kms:us-east-1:111122223333:key/mrk-1234abcd12ab34cd56ef1234567890ab'

/* Create an AWS KMS keyring */
const mrkEncryptKeyring = buildAwsKmsMrkAwareStrictMultiKeyringNode({
    generatorKeyId: multiRegionUsEastKey,
  })

/* Specify an encryption context */
const context = {
    purpose: 'test',
  }

/* Create an encryption keyring */
const { result } = await encrypt(mrkEncryptKeyring, cleartext, {
    encryptionContext: context,
  })
```

------
#### [ Python ]

To encrypt with an AWS KMS multi\-Region key, use the `MRKAwareStrictAwsKmsMasterKeyProvider()` method and specify a multi\-Region key\. 

For a complete example, see [mrk\_aware\_kms\_provider\.py](https://github.com/aws/aws-encryption-sdk-python/blob/master/examples/src/mrk_aware_kms_provider.py) in the AWS Encryption SDK for Python repository on GitHub\.

```
# Instantiate the client
client = aws_encryption_sdk.EncryptionSDKClient(commitment_policy=CommitmentPolicy.REQUIRE_ENCRYPT_REQUIRE_DECRYPT)

# Specify a multi-Region key in us-east-1
mrk_us_east_1 = "arn:aws:kms:us-east-1:111122223333:key/mrk-1234abcd12ab34cd56ef1234567890ab"

# Use the multi-Region method to create the master key provider
# in strict mode
strict_mrk_key_provider = MRKAwareStrictAwsKmsMasterKeyProvider(
        key_ids=[mrk_us_east_1]
)

# Set the encryption context
encryption_context = {
        "purpose": "test"
    }

# Encrypt your plaintext data
ciphertext, encrypt_header = client.encrypt(
        source=source_plaintext,
        encryption_context=encryption_context,
        key_provider=strict_mrk_key_provider
)
```

------

Next, move your ciphertext to the `us-west-2` Region\. You don't need to re\-encrypt the ciphertext\.

To decrypt the ciphertext in strict mode in the `us-west-2` Region, instantiate the multi\-Region\-aware symbol with the key ARN of the related multi\-Region key in the `us-west-2` Region\. If you specify the key ARN of a related multi\-Region key in a different Region \(including `us-east-1`, where it was encrypted\), the multi\-Region\-aware symbol will make a cross\-Region call for that CMK\.

When decrypting in strict mode, the multi\-Region\-aware symbol requires a key ARN\. It accepts only one key ARN from each set of related multi\-Region keys\.

Before running these examples, replace the example multi\-Region key ARN with a valid value from your AWS account\.

------
#### [ C ]

To decrypt in strict mode with a multi\-Region key, use the `Aws::Cryptosdk::KmsMrkAwareSymmetricKeyring::Builder()` method to instantiate the keyring\. Specify the related multi\-Region key in the local \(us\-west\-2\) Region\.

For a complete example, see [kms\_multi\_region\_keys\.cpp](https://github.com/aws/aws-encryption-sdk-c/tree/master/examples/kms_multi_region_keys.cpp) in the AWS Encryption SDK for C repository on GitHub\.

```
/* Initialize a multi-Region keyring */
const char *mrk_us_west_2 = "arn:aws:kms:us-west-2:111122223333:key/mrk-1234abcd12ab34cd56ef1234567890ab";    

struct aws_cryptosdk_keyring *mrk_keyring = 
    Aws::Cryptosdk::KmsMrkAwareSymmetricKeyring::Builder().Build(mrk_us_west_2);

/* Create a session; release the keyring */
struct aws_cryptosdk_session *session =
    aws_cryptosdk_session_new_from_keyring_2(aws_default_allocator(), AWS_CRYPTOSDK_ENCRYPT, mrk_keyring);

aws_cryptosdk_session_set_commitment_policy(session,
    COMMITMENT_POLICY_REQUIRE_ENCRYPT_REQUIRE_DECRYPT);

aws_cryptosdk_keyring_release(mrk_keyring);

/* Decrypt the ciphertext 
 *   aws_cryptosdk_session_process_full is designed for non-streaming data
 */
aws_cryptosdk_session_process_full(
    session, plaintext, plaintext_buf_sz, &plaintext_len, ciphertext, ciphertext_len));

/* Clean up the session */
aws_cryptosdk_session_destroy(session);
```

------
#### [ AWS Encryption CLI ]

To decrypt with the related multi\-Region key in the us\-west\-2 Region, use the **key** attribute of the `--wrapping-keys` parameter to specify its key ARN\. 

```
# To run this example, replace the fictitious key ARN with a valid value.
$ mrkUSWest2=arn:aws:kms:us-west-2:111122223333:key/mrk-1234abcd12ab34cd56ef1234567890ab

$ aws-encryption-cli --decrypt \
                     --input hello.txt.encrypted \
                     --wrapping-keys key=$mrkUSWest2 \
                     --commitment-policy require-encrypt-require-decrypt \
                     --encryption-context purpose=test \
                     --metadata-output ~/metadata \
                     --max-encrypted-data-keys 1 \
                     --buffer \
                     --output .
```

------
#### [ Java ]

To decrypt in strict mode, instantiate an `AwsKmsMrkAwareMasterKeyProvider` and specify the related multi\-Region key in the local \(us\-west\-2\) Region\.

For a complete example, see [BasicMultiRegionKeyEncryptionExample\.java](https://github.com/aws/aws-encryption-sdk-java/blob/master/src/examples/java/com/amazonaws/crypto/examples/BasicMultiRegionKeyEncryptionExample.java) in the AWS Encryption SDK for Java repository on GitHub\.

```
// Decrypt in us-west-2 Region

// Instantiate the client
final AwsCrypto crypto = AwsCrypto.builder()
    .withCommitmentPolicy(CommitmentPolicy.RequireEncryptRequireDecrypt)
    .build();

// Related multi-Region keys have the same key ID. Their key ARNs differs only in the Region field.
String mrkUSWest2 = "arn:aws:kms:us-west-2:111122223333:key/mrk-1234abcd12ab34cd56ef1234567890ab";

// Use the multi-Region method to create the master key provider
// in strict mode
AwsKmsMrkAwareMasterKeyProvider kmsMrkProvider = AwsKmsMrkAwareMasterKeyProvider.builder()
    .buildStrict(mrkUSWest2);

// Decrypt your ciphertext
CryptoResult<byte[], AwsKmsMrkAwareMasterKey> decryptResult = crypto.decryptData(
        kmsMrkProvider,
        ciphertext);
byte[] decrypted = decryptResult.getResult();
```

------
#### [ JavaScript Browser ]

To decrypt in strict mode, use the `buildAwsKmsMrkAwareStrictMultiKeyringBrowser()` method to create the keyring and specify the related multi\-Region key in the local \(us\-west\-2\) Region\.

For a complete example, see [kms\_multi\_region\_simple\.ts](https://github.com/aws/aws-encryption-sdk-javascript/blob/master/modules/example-browser/src/kms_multi_region_simple.ts) in the AWS Encryption SDK for JavaScript repository on GitHub\.

```
import {
  buildAwsKmsMrkAwareStrictMultiKeyringBrowser,
  buildClient,
  CommitmentPolicy,
  KMS,
} from '@aws-crypto/client-browser'


/* Instantiate an AWS Encryption SDK client */
const { decrypt } = buildClient(
  CommitmentPolicy.REQUIRE_ENCRYPT_REQUIRE_DECRYPT
)

declare const credentials: {
  accessKeyId: string
  secretAccessKey: string
  sessionToken: string
}

/* Instantiate an AWS KMS client 
 * The AWS Encryption SDK for JavaScript gets the Region from the key ARN
 */
const clientProvider = (region: string) => new KMS({ region, credentials })


/* Specify a multi-Region key in us-west-2 */
const multiRegionUsWestKey =
    'arn:aws:kms:us-west-2:111122223333:key/mrk-1234abcd12ab34cd56ef1234567890ab'

/* Instantiate the keyring */
const mrkDecryptKeyring = buildAwsKmsMrkAwareStrictMultiKeyringBrowser({
    generatorKeyId: multiRegionUsWestKey,
    clientProvider,
  })


/* Decrypt the data */
const { plaintext, messageHeader } = await decrypt(mrkDecryptKeyring, result)
```

------
#### [ JavaScript Node\.js ]

To decrypt in strict mode, use the `buildAwsKmsMrkAwareStrictMultiKeyringNode()` method to create the keyring and specify the related multi\-Region key in the local \(us\-west\-2\) Region\.

For a complete example, see [kms\_multi\_region\_simple\.ts](https://github.com/aws/aws-encryption-sdk-javascript/blob/master/modules/example-node/src/kms_multi_region_simple.ts) in the AWS Encryption SDK for JavaScript repository on GitHub\.

```
import { buildClient } from '@aws-crypto/client-node'

/* Instantiate the client
const { decrypt } = buildClient(
  CommitmentPolicy.REQUIRE_ENCRYPT_REQUIRE_DECRYPT
)

/* Multi-Region keys have a distinctive key ID that begins with 'mrk'
 * Specify a multi-Region key in us-east-1
 */
const multiRegionUsWestKey =
    'arn:aws:kms:us-west-2:111122223333:key/mrk-1234abcd12ab34cd56ef1234567890ab'

/* Create an AWS KMS keyring */
const mrkDecryptKeyring = buildAwsKmsMrkAwareStrictMultiKeyringNode({
    generatorKeyId: multiRegionUsWestKey,
  })

/* Decrypt your ciphertext */
const { plaintext, messageHeader } = await decrypt(decryptKeyring, result)
```

------
#### [ Python ]

To decrypt in strict mode, use the `MRKAwareStrictAwsKmsMasterKeyProvider()` method to create the master key provider\. Specify the related multi\-Region key in the local \(us\-west\-2\) Region\.

For a complete example, see [mrk\_aware\_kms\_provider\.py](https://github.com/aws/aws-encryption-sdk-python/blob/master/examples/src/mrk_aware_kms_provider.py) in the AWS Encryption SDK for Python repository on GitHub\.

```
# Instantiate the client
client = aws_encryption_sdk.EncryptionSDKClient(commitment_policy=CommitmentPolicy.REQUIRE_ENCRYPT_REQUIRE_DECRYPT)

# Related multi-Region keys have the same key ID. Their key ARNs differs only in the Region field
mrk_us_west_2 = "arn:aws:kms:us-west-2:111122223333:key/mrk-1234abcd12ab34cd56ef1234567890ab"

# Use the multi-Region method to create the master key provider
# in strict mode
strict_mrk_key_provider = MRKAwareStrictAwsKmsMasterKeyProvider(
        key_ids=[mrk_us_west_2]
)

# Decrypt your ciphertext
plaintext, _ = client.decrypt(
        source=ciphertext, 
        key_provider=strict_mrk_key_provider
)
```

------

You can also decrypt in *discovery mode* with multi\-Region keys\. When decrypting in discovery mode, you don't specify any CMKs\.

If you encrypted with a multi\-Region key, the multi\-Region\-aware symbol in discovery mode will try to decrypt by using a related multi\-Region key in the local Region\. If none exists; the call fails\. In discovery mode, the AWS Encryption SDK will not attempt a cross\-Region call for the multi\-Region key used for encryption\. 

The following example shows how to decrypt with the multi\-Region\-aware symbol in discovery mode\. Because you don't specify a CMK, the AWS Encryption SDK must get the Region from a different source\. When possible, specify the local Region explicitly\. Otherwise, the AWS Encryption SDK gets the local Region from the Region configured in the AWS SDK for your programming language\.

Before running these examples, replace the example account ID and multi\-Region key ARN with valid values from your AWS account\.

------
#### [ C ]

To decrypt in discovery mode with a multi\-Region key, use the `Aws::Cryptosdk::KmsKeyring::DiscoveryFilter::Builder()` method\. To specify the local Region, define a `ClientConfiguration` and specify it in the AWS KMS client\.

For a complete example, see [kms\_multi\_region\_keys\.cpp](https://github.com/aws/aws-encryption-sdk-c/tree/master/examples/kms_multi_region_keys.cpp) in the AWS Encryption SDK for C repository on GitHub\.

```
/* Construct a discovery filter for the account and partition. The
 *  filter is optional, but it's a best practice that we recommend.
 */
const char *account_id = "111122223333";
const char *partition = "aws";
const std::shared_ptr<Aws::Cryptosdk::KmsKeyring::DiscoveryFilter> discovery_filter =
    Aws::Cryptosdk::KmsKeyring::DiscoveryFilter::Builder(partition).AddAccount(account_id).Build();

/* Create an AWS KMS client in the desired region. */
const char *region = "us-west-2";

Aws::Client::ClientConfiguration client_config;
client_config.region = region;
const std::shared_ptr<Aws::KMS::KMSClient> kms_client =
    Aws::MakeShared<Aws::KMS::KMSClient>("AWS_SAMPLE_CODE", client_config);

/* Initialize a multi-Region keyring */
const char *mrk_us_west_2 = "arn:aws:kms:us-west-2:111122223333:key/mrk-1234abcd12ab34cd56ef1234567890ab";   

struct aws_cryptosdk_keyring *mrk_keyring = 
    Aws::Cryptosdk::KmsMrkAwareSymmetricKeyring::Builder()
        .WithKmsClient(kms_client)
        .BuildDiscovery(region, discovery_filter);

/* Create a session; release the keyring */
struct aws_cryptosdk_session *session =
    aws_cryptosdk_session_new_from_keyring_2(aws_default_allocator(), AWS_CRYPTOSDK_DECRYPT, mrk_keyring);

aws_cryptosdk_session_set_commitment_policy(session,
    COMMITMENT_POLICY_REQUIRE_ENCRYPT_REQUIRE_DECRYPT);

aws_cryptosdk_keyring_release(mrk_keyring);

/* Decrypt the ciphertext 
 *   aws_cryptosdk_session_process_full is designed for non-streaming data
 */
aws_cryptosdk_session_process_full(
    session, plaintext, plaintext_buf_sz, &plaintext_len, ciphertext, ciphertext_len));

/* Clean up the session */
aws_cryptosdk_session_destroy(session);
```

------
#### [ AWS Encryption CLI ]

To decrypt in discovery mode, use the **discovery** attribute of the `--wrapping-keys` parameter\. The **discovery\-account** and **discovery\-partition** attributes create a discovery filter that is optional, but recommended\.

To specify the Region, this command includes the **region** attribute of the `--wrapping-keys` parameter\.

```
$ aws-encryption-cli --decrypt \
                     --input hello.txt.encrypted \
                     --wrapping-keys discovery=true \ 
                                     discovery-account=111122223333 \
                                     discovery-partition=aws \
                                     region=us-west-2 \
                     --commitment-policy require-encrypt-require-decrypt \
                     --encryption-context purpose=test \
                     --metadata-output ~/metadata \
                     --max-encrypted-data-keys 1 \
                     --buffer \
                     --output .
```

------
#### [ Java ]

To specify the local Region, use the `builder().withDiscoveryMrkRegion` parameter\. Otherwise, the AWS Encryption SDK gets the local Region from the Region configured in the [AWS SDK for Java](https://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/java-dg-region-selection.html)\.

For a complete example, see [DiscoveryMultiRegionKeyEncryptionExample\.java](https://github.com/aws/aws-encryption-sdk-java/blob/master/src/examples/java/com/amazonaws/crypto/examples/DiscoveryMultiRegionKeyEncryptionExample.java) in the AWS Encryption SDK for Java repository on GitHub\.

```
// Decrypt in discovery mode in us-west-2 Region

// Instantiate the client
final AwsCrypto crypto = AwsCrypto.builder()
    .withCommitmentPolicy(CommitmentPolicy.RequireEncryptRequireDecrypt)
    .build();
                
DiscoveryFilter discoveryFilter = new DiscoveryFilter("aws", 111122223333);

AwsKmsMrkAwareMasterKeyProvider mrkDiscoveryProvider = AwsKmsMrkAwareMasterKeyProvider
    .builder()
    .withDiscoveryMrkRegion("us-west-2")
    .buildDiscovery(discoveryFilter);

// Decrypt your ciphertext
final CryptoResult<byte[], AwsKmsMrkAwareMasterKey> decryptResult = crypto
    .decryptData(mrkDiscoveryProvider, ciphertext);
```

------
#### [ JavaScript Browser ]

To decrypt in discovery mode with a symmetric multi\-Region key, use the `AwsKmsMrkAwareSymmetricDiscoveryKeyringBrowser()` method\.

For a complete example, see [kms\_multi\_region\_discovery\.ts](https://github.com/aws/aws-encryption-sdk-javascript/blob/master/modules/example-browser/src/kms_multi_region_discovery.ts) in the AWS Encryption SDK for JavaScript repository on GitHub\.

```
import {
  AwsKmsMrkAwareSymmetricDiscoveryKeyringBrowser,
  buildClient,
  CommitmentPolicy,
  KMS,
} from '@aws-crypto/client-browser'


/* Instantiate an AWS Encryption SDK client */
const { decrypt } = buildClient(
  CommitmentPolicy.REQUIRE_ENCRYPT_REQUIRE_DECRYPT
)

declare const credentials: {
  accessKeyId: string
  secretAccessKey: string
  sessionToken: string
}

/* Instantiate the KMS client with an explicit Region */
const client = new KMS({ region: 'us-west-2', credentials })


/* Create a discovery filter */
const discoveryFilter = { partition: 'aws', accountIDs: ['111122223333'] }


/* Create an AWS KMS discovery keyring */
const mrkDiscoveryKeyring = new AwsKmsMrkAwareSymmetricDiscoveryKeyringBrowser({
    client,
    discoveryFilter,
  })


/* Decrypt the data */
const { plaintext, messageHeader } = await decrypt(mrkDiscoveryKeyring, ciphertext)
```

------
#### [ JavaScript Node\.js ]

To decrypt in discovery mode with a symmetric multi\-Region key, use the `AwsKmsMrkAwareSymmetricDiscoveryKeyringNode()` method\.

For a complete example, see [kms\_multi\_region\_discovery\.ts](https://github.com/aws/aws-encryption-sdk-javascript/blob/master/modules/example-node/src/kms_multi_region_discovery.ts) in the AWS Encryption SDK for JavaScript repository on GitHub\.

```
import {
  AwsKmsMrkAwareSymmetricDiscoveryKeyringNode,
  buildClient,
  CommitmentPolicy,
  KMS,
} from '@aws-crypto/client-node'

/* Instantiate the Encryption SDK client
const { decrypt } = buildClient(
  CommitmentPolicy.REQUIRE_ENCRYPT_REQUIRE_DECRYPT
)

/* Instantiate the KMS client with an explicit Region */
const client = new KMS({ region: 'us-west-2' })

/* Create a discovery filter */
const discoveryFilter = { partition: 'aws', accountIDs: ['111122223333'] }

/* Create an AWS KMS discovery keyring */
const mrkDiscoveryKeyring = new AwsKmsMrkAwareSymmetricDiscoveryKeyringNode({
    client,
    discoveryFilter,
  })

/* Decrypt your ciphertext */
const { plaintext, messageHeader } = await decrypt(mrkDiscoveryKeyring, result)
```

------
#### [ Python ]

To decrypt in discovery mode with a multi\-Region key, use the `MRKAwareDiscoveryAwsKmsMasterKeyProvider()` method\.

For a complete example, see [mrk\_aware\_kms\_provider\.py](https://github.com/aws/aws-encryption-sdk-python/blob/master/examples/src/mrk_aware_kms_provider.py) in the AWS Encryption SDK for Python repository on GitHub\.

```
# Instantiate the client
client = aws_encryption_sdk.EncryptionSDKClient(commitment_policy=CommitmentPolicy.REQUIRE_ENCRYPT_REQUIRE_DECRYPT)

# Create the discovery filter and specify the region
decrypt_kwargs = dict(
        discovery_filter=DiscoveryFilter(account_ids="111122223333", partition="aws"),
        discovery_region="us-west-2",
    )

# Use the multi-Region method to create the master key provider
# in discovery mode
mrk_discovery_key_provider = MRKAwareDiscoveryAwsKmsMasterKeyProvider(**decrypt_kwargs)

# Decrypt your ciphertext
plaintext, _ = client.decrypt(
        source=ciphertext, 
        key_provider=mrk_discovery_key_provider
)
```

------

## Examine the security features<a name="config-security"></a>

The AWS Encryption SDK supports the following security features\. Although they aren't required, we recommend that you use them whenever it's practical\.

### Choosing an algorithm suite<a name="config-algorithm"></a>

The AWS Encryption SDK supports several [symmetric and asymmetric encryption algorithms](concepts.md#symmetric-key-encryption) for encrypting your data keys under the wrapping keys you specify\. However, when it uses those data keys to encrypt your data, the AWS Encryption SDK defaults to a [recommended algorithm suite](supported-algorithms.md#recommended-algorithms) that uses the AES\-GCM algorithm with key derivation, digital signatures, and key commitment\. Although this algorithm suite is likely to be suitable for most applications, you can choose an alternate algorithm suite\. For example, some trust models would be satisfied by an algorithm suite without [digital signatures](concepts.md#digital-sigs)\. 

 For information about the algorithm suites that the AWS Encryption SDK supports, see [Supported algorithm suites in the AWS Encryption SDK](supported-algorithms.md)\.

### Limiting encrypted data keys<a name="config-limit-keys"></a>

You can limit the number of encrypted data keys in an encrypted message\. This best practice feature can help you detect a misconfigured keyring when encrypting or a malicious ciphertext when decrypting\. It also prevents unnecessary, expensive, and potentially exhaustive calls to your key infrastructure\. Limiting encrypted data keys is most valuable when you are decrypting messages from an untrusted source\. 

Although most encrypted messages have one encrypted data key for each wrapping key used in the encryption, an encrypted message can contain up to 65,535 encrypted data keys\. A malicious actor might construct an encrypted message with thousands of encrypted data keys, none of which can be decrypted\. As a result, the AWS Encryption SDK would attempt to decrypt each encrypted data key until it exhausted the encrypted data keys in the message\.

To limit encrypted data keys, use the `MaxEncryptedDataKeys` parameter\. This parameter is available for all supported programming languages beginning in versions 1\.9\.*x* and 2\.2\.*x* of the AWS Encryption SDK\. It is optional and valid when encrypting and decrypting\. The following examples decrypt data that was encrypted under three different wrapping keys\. The `MaxEncryptedDataKeys` value is set to 3\.

------
#### [ C ]

```
/* Construct an AWS KMS keyring */
struct aws_cryptosdk_keyring *kms_keyring = 
      Aws::Cryptosdk::KmsKeyring::Builder().Build(key_arn1, { key_arn2, key_arn3 });

/* Create a session */
struct aws_cryptosdk_session *session = 
    aws_cryptosdk_session_new_from_keyring_2(alloc, AWS_CRYPTOSDK_DECRYPT, kms_keyring);
aws_cryptosdk_keyring_release(kms_keyring);  

/* Limit encrypted data keys */
aws_cryptosdk_session_set_max_encrypted_data_keys(session, 3);
  
/* Decrypt */
size_t ciphertext_consumed_output;
aws_cryptosdk_session_process(session,
    plaintext_output,
    plaintext_buf_sz_output,
    &plaintext_len_output,
    ciphertext_input,
    ciphertext_len_input,
    &ciphertext_consumed_output);
assert(aws_cryptosdk_session_is_done(session));
assert(ciphertext_consumed == ciphertext_len);
```

------
#### [ AWS Encryption CLI ]

```
# Decrypt with limited encrypted data keys

$ aws-encryption-cli --decrypt \
    --input hello.txt.encrypted \
    --wrapping-keys key=$key_arn1 key=$key_arn2 key=$key_arn3 \
    --buffer \
    --max-encrypted-data-keys 3 \
    --encryption-context purpose=test \
    --metadata-output ~/metadata \
    --output .
```

------
#### [ Java ]

```
// Construct a client with limited encrypted data keys
final AwsCrypto crypto = AwsCrypto.builder()
  .withMaxEncryptedDataKeys(3)
  .build();

// Create an AWS KMS master key provider
final KmsMasterKeyProvider keyProvider = KmsMasterKeyProvider.builder()
      .buildStrict(keyArn1, keyArn2, keyArn3);
   
// Decrypt
final CryptoResult<byte[], KmsMasterKey> decryptResult = crypto.decryptData(keyProvider, ciphertext)
```

------
#### [ JavaScript Browser ]

```
// Construct a client with limited encrypted data keys
const { encrypt, decrypt } = buildClient({ maxEncryptedDataKeys: 3 })

declare const credentials: {
  accessKeyId: string
  secretAccessKey: string
  sessionToken: string
}
const clientProvider = getClient(KMS, {
  credentials: { accessKeyId, secretAccessKey, sessionToken }
})

// Create an AWS KMS keyring
const keyring = new KmsKeyringBrowser({
  clientProvider,
  keyIds: [keyArn1, keyArn2, keyArn3],
})

// Decrypt
const { plaintext, messageHeader } = await decrypt(keyring, ciphertext)
```

------
#### [ JavaScript Node\.js ]

```
// Construct a client with limited encrypted data keys
const { encrypt, decrypt } = buildClient({ maxEncryptedDataKeys: 3 })

// Create an AWS KMS keyring
const keyring = new KmsKeyringBrowser({
  keyIds: [keyArn1, keyArn2, keyArn3],
})
  
// Decrypt
const { plaintext, messageHeader } = await decrypt(keyring, ciphertext)
```

------
#### [ Python ]

```
# Instantiate a client with limited encrypted data keys
client = aws_encryption_sdk.EncryptionSDKClient(max_encrypted_data_keys=3)

# Create an AWS KMS master key provider
master_key_provider = aws_encryption_sdk.StrictAwsKmsMasterKeyProvider(
    key_ids=[key_arn1, key_arn2, key_arn3])

# Decrypt
plaintext, header = client.decrypt(source=ciphertext, key_provider=master_key_provider)
```

------

### Working with streaming data<a name="config-stream"></a>

When you stream data for decryption, be aware that the AWS Encryption SDK returns decrypted plaintext after the integrity checks are complete, but before the digital signature is verified\. To ensure that you don't return or use plaintext until the signature is verified, we recommend that you buffer the streamed plaintext until the entire decryption process is complete\. 

This issue arises only when you are streaming ciphertext for decryption, and only when you are using an algorithm suite, such as the [default algorithm suite](supported-algorithms.md), that includes [digital signatures](concepts.md#digital-sigs)\. 

To make the buffering easier, some AWS Encryption SDK language implementations, such as AWS Encryption SDK for JavaScript in Node\.js, include a buffering feature as part of the decrypt method\. The AWS Encryption CLI, which always streams input and output introduced a `--buffer` parameter in versions 1\.9\.*x* and 2\.2\.*x*\. In other language implementations, you can use existing buffering features\.

If you are using an algorithm suite without digital signatures, be sure to use the `decrypt-unsigned` feature in each language implementation\. This feature decrypts ciphertext but fails if it encounters signed ciphertext\. For details, see [Choosing an algorithm suite](#config-algorithm)\. 

## Review other options<a name="config-other"></a>

The AWS Encryption SDK supports following configuration options\. Before using these options, consider the risks and benefits and determine whether the option is right for your application\.

### Data key caching<a name="config-caching"></a>

In general, reusing data keys is discouraged, but the AWS Encryption SDK offers a [data key caching](data-key-caching.md) option that provides limited reuse of data keys\. Data key caching can improve the performance of some applications and reduce calls to your key infrastructure\. Before using data key caching in production, adjust the [security thresholds](thresholds.md), and test to make sure that the benefits outweigh the disadvantages of reusing data keys\. 