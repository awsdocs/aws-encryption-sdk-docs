# Raw AES keyrings<a name="use-raw-aes-keyring"></a>

The AWS Encryption SDK lets you use an AES symmetric key that you provide as a wrapping key that protects your data key\. You need to generate, store, and protect the key material, preferably in a hardware security module \(HSM\) or key management system\. Use a Raw AES keyring when you need to provide the wrapping key and encrypt the data keys locally or offline\.

The Raw AES keyring encrypts data by using the AES\-GCM algorithm and a wrapping key that you specify as a byte array\. You can specify only one wrapping key in each Raw AES keyring, but you can include multiple Raw AES keyrings, alone or with other keyrings, in a [multi\-keyring](use-multi-keyring.md)\. 

The Raw AES keyring is equivalent to and interoperates with the [JceMasterKey](https://aws.github.io/aws-encryption-sdk-java/com/amazonaws/encryptionsdk/jce/JceMasterKey.html) class in the AWS Encryption SDK for Java and the [RawMasterKey](https://aws-encryption-sdk-python.readthedocs.io/en/latest/generated/aws_encryption_sdk.key_providers.raw.html#aws_encryption_sdk.key_providers.raw.RawMasterKey) class in the AWS Encryption SDK for Python when they are used with an AES encryption keys\. You can encrypt data with one implementation and decrypt the data with any other implementation using the same wrapping key\. For details, see [Keyring compatibility](keyring-compatibility.md)\.

**Key namespaces and names**

To identify the AES key in a keyring, the Raw AES keyring uses a *key namespace* and *key name* that you provide\. These values are not secret\. They appear in plain text in the header of the [encrypted message](concepts.md#message) that the encrypt operation returns\. We recommend using a key namespace your HSM or key management system and a key name that identifies the AES key in that system\.

**Note**  
The key namespace and key name are equivalent to the *Provider ID* \(or *Provider*\) and *Key ID* fields in the `JceMasterKey` and `RawMasterKey`\.  
The AWS Encryption SDK for C reserves the `aws-kms` key namespace value for KMS keys\. Do not use it in a Raw AES keyring or Raw RSA keyring with the AWS Encryption SDK for C\.

If you construct different keyrings to encrypt and decrypt a given message, the namespace and name values are critical\. If the key namespace and key name in the decryption keyring isn't an exact, case\-sensitive match for the key namespace and key name in the encryption keyring, the decryption keyring isn't used, even if the key material bytes are identical\.

For example, you might define a Raw AES keyring with key namespace `HSM_01` and key name `AES_256_012`\. Then, you use that keyring to encrypt some data\. To decrypt that data, construct a Raw AES keyring with the same key namespace, key name, and key material\.

The following examples show how to create a Raw AES keyring\. The `AESWrappingKey` variable represents the key material you provide\.

------
#### [ C ]

To instantiate a Raw AES keyring In the AWS Encryption SDK for C, use `aws_cryptosdk_raw_aes_keyring_new()`\. For a complete example, see [raw\_aes\_keyring\.c](https://github.com/aws/aws-encryption-sdk-c/blob/master/examples/raw_aes_keyring.c)\.

```
struct aws_allocator *alloc = aws_default_allocator();

AWS_STATIC_STRING_FROM_LITERAL(wrapping_key_namespace, "HSM_01");
AWS_STATIC_STRING_FROM_LITERAL(wrapping_key_name, "AES_256_012");

struct aws_cryptosdk_keyring *raw_aes_keyring = aws_cryptosdk_raw_aes_keyring_new(
        alloc, wrapping_key_namespace, wrapping_key_name, aes_wrapping_key, wrapping_key_len);
```

------
#### [ JavaScript Browser ]

The AWS Encryption SDK for JavaScript in the browser gets its cryptographic primitives from the [WebCrypto](https://developer.mozilla.org/en-US/docs/Web/API/Web_Crypto_API) API\. Before you construct the keyring, you must use `RawAesKeyringWebCrypto.importCryptoKey()` to import the raw key material into the WebCrypto backend\. This assures that the keyring is complete even though all calls to WebCrypto are asynchronous\.

Then, to instantiate a Raw AES keyring, use the `RawAesKeyringWebCrypto()` method\. You must specify the AES wrapping algorithm \("wrapping suite\) based on the length of your key material\. For a complete example, see [aes\_simple\.ts \(JavaScript Browser\)](https://github.com/aws/aws-encryption-sdk-javascript/blob/master/modules/example-browser/src/aes_simple.ts)\.

```
const keyNamespace = 'HSM_01'
const keyName = 'AES_256_012'

const wrappingSuite =
  RawAesWrappingSuiteIdentifier.AES256_GCM_IV12_TAG16_NO_PADDING

/* Import the plaintext AES key into the WebCrypto backend. */
const aesWrappingKey = await RawAesKeyringWebCrypto.importCryptoKey(
  rawAesKey,
  wrappingSuite
)

const rawAesKeyring = new RawAesKeyringWebCrypto({
  keyName,
  keyNamespace,
  wrappingSuite,
  aesWrappingKey
})
```

------
#### [ JavaScript Node\.js ]

To instantiate a Raw AES keyring in the AWS Encryption SDK for JavaScript for Node\.js, create an instance of the ` RawAesKeyringNode` class\. You must specify the AES wrapping algorithm \("wrapping suite"\) based on the length of your key material\. For a complete example, see [aes\_simple\.ts](https://github.com/aws/aws-encryption-sdk-javascript//blob/master/modules/example-node/src/aes_simple.ts) \(JavaScript Node\.js\)\.

```
const keyName = 'AES_256_012'
const keyNamespace = 'HSM_01'

const wrappingSuite =
  RawAesWrappingSuiteIdentifier.AES256_GCM_IV12_TAG16_NO_PADDING

const rawAesKeyring = new RawAesKeyringNode({
  keyName,
  keyNamespace,
  aesWrappingKey,
  wrappingSuite,
})
```

------