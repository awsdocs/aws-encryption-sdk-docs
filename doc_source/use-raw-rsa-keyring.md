# Raw RSA keyrings<a name="use-raw-rsa-keyring"></a>

The Raw RSA keyring performs asymmetric encryption and decryption of data keys in local memory with an RSA public and private keys that you provide\. You need to generate, store, and protect the private key, preferably in a hardware security module \(HSM\) or key management system\. The encryption function encrypts the data key under the RSA public key\. The decryption function decrypts the data key using the private key\. You can select from among the several [RSA padding modes](https://github.com/aws/aws-encryption-sdk-c/blob/master/include/aws/cryptosdk/cipher.h)\.

A Raw RSA keyring that encrypts and decrypts must include an asymmetric public key and private key pair\. However, you can encrypt data with a Raw RSA keyring that has only a public key, and you can decrypt data with a Raw RSA keyring that has only a private key\. You can include any Raw RSA keyring in a [multi\-keyring](use-multi-keyring.md)\. If you configure a Raw RSA keyring with a public and private key, be sure that they are part of the same key pair\. Some language implementations of the AWS Encryption SDK will not construct a Raw RSA keyring with keys from different pairs\. Others rely on you to verify that your keys are from the same key pair\.

 The Raw RSA keyring is equivalent to and interoperates with the [JceMasterKey](https://aws.github.io/aws-encryption-sdk-java/com/amazonaws/encryptionsdk/jce/JceMasterKey.html) in the AWS Encryption SDK for Java and the [RawMasterKey](https://aws-encryption-sdk-python.readthedocs.io/en/latest/generated/aws_encryption_sdk.key_providers.raw.html#aws_encryption_sdk.key_providers.raw.RawMasterKey) in the AWS Encryption SDK for Python when they are used with RSA asymmetric encryption keys\. You can encrypt data with one implementation and decrypt the data with any other implementation using the same wrapping key\. For details, see [Keyring compatibility](keyring-compatibility.md)\.

**Note**  
Do not use an [asymmetric KMS key](https://docs.aws.amazon.com/kms/latest/developerguide/symm-asymm-concepts.html#asymmetric-cmks) in a Raw RSA keyring\. The Raw RSA keyring does not support KMS keys\.  
If you encrypt data with a Raw RSA keyring that includes the public key of an RSA KMS key, neither the AWS Encryption SDK nor AWS KMS can decrypt it\. You cannot export the private key of an AWS KMS asymmetric KMS key into a Raw RSA keyring\. The AWS KMS Decrypt operation cannot decrypt the [encrypted message](concepts.md#message) that the AWS Encryption SDK returns\.

When constructing a Raw RSA keyring in the AWS Encryption SDK for C, be sure to provide the *contents* of the PEM file that includes each key as a null\-terminated C\-string, not as a path or file name\. When constructing a Raw RSA keyring in JavaScript, be aware of [potential incompatibility](javascript-compatibility.md) with other language implementations\.

For an example of how to use a Raw RSA keyring, see:
+ C: [raw\_rsa\_keyring\.c](https://github.com/aws/aws-encryption-sdk-c/blob/master/examples/raw_rsa_keyring.c)
+ JavaScript Node\.js: [rsa\_simple\.ts](https://github.com/aws/aws-encryption-sdk-javascript/blob/master/modules/example-node/src/rsa_simple.ts)
+ JavaScript Browser: [rsa\_simple\.ts](https://github.com/aws/aws-encryption-sdk-javascript/blob/master/modules/example-browser/src/rsa_simple.ts)

**Namespaces and names**

To identify the RSA key material in a keyring, the Raw RSA keyring uses a *key namespace* and *key name* that you provide\. These values are not secret\. They appear in plain text in the header of the [encrypted message](concepts.md#message) that the encrypt operation returns\. We recommend using the key namespace and key name that identifies the RSA key pair \(or its private key\) in your HSM or key management system\.

**Note**  
The key namespace and key name are equivalent to the *Provider ID* \(or *Provider*\) and *Key ID* fields in the `JceMasterKey` and `RawMasterKey`\.   
The AWS Encryption SDK for C reserves the `aws-kms` key namespace value for KMS keys\. Do not use it in a Raw AES keyring or Raw RSA keyring with the AWS Encryption SDK for C\.

If you construct different keyrings to encrypt and decrypt a given message, the namespace and name values are critical\. If the key namespace and key name in the decryption keyring isn't an exact, case\-sensitive match for the key namespace and key name in the encryption keyring, the decryption keyring isn't used, even if the keys are from the same key pair\.

The key namespace and key name of the key material in the encryption and decryption keyrings must be same whether the keyring contains the RSA public key, the RSA private key, or both keys in the key pair\. For example, suppose you encrypt data with a Raw RSA keyring for an RSA public key with key namespace `HSM_01` and key name `RSA_2048_06`\. To decrypt that data, construct a Raw RSA keyring with the private key \(or key pair\), and the same key namespace and name\.

**Padding mode**

You must specify a padding mode for Raw RSA keyrings used for encryption and decryption, or use features of your language implementation that specify it for you\.

The AWS Encryption SDK supports the following padding modes, subjects to the constraints of each language\. We recommend an [OAEP](https://tools.ietf.org/html/rfc8017#section-7.1) padding mode, particularly OAEP with SHA\-256 and MGF1 with SHA\-256 Padding\. The [PKCS1](https://tools.ietf.org/html/rfc8017#section-7.2) padding mode is supported only for backward compatibility\.
+ OAEP with SHA\-1 and MGF1 with SHA\-1 Padding
+ OAEP with SHA\-256 and MGF1 with SHA\-256 Padding
+ OAEP with SHA\-384 and MGF1 with SHA\-384 Padding
+ OAEP with SHA\-512 and MGF1 with SHA\-512 Padding
+ PKCS1 v1\.5 Padding 

The following examples show how to create a Raw RSA keyring with the public and private key of an RSA key pair and the OAEP with SHA\-256 and MGF1 with SHA\-256 padding mode\. The `RSAPublicKey` and `RSAPrivateKey` variables represent the key material you provide\.

------
#### [ C ]

In the AWS Encryption SDK for C, use `aws_cryptosdk_raw_rsa_keyring_new` to create a Raw RSA keyring\. 

When constructing a Raw RSA keyring in the AWS Encryption SDK for C, be sure to provide the *contents* of the PEM file that includes each key as a null\-terminated C\-string, not as a path or file name\. For a complete example, see [raw\_rsa\_keyring\.c](https://github.com/aws/aws-encryption-sdk-c/blob/master/examples/raw_rsa_keyring.c)\.

```
struct aws_allocator *alloc = aws_default_allocator();

AWS_STATIC_STRING_FROM_LITERAL(key_namespace, "HSM_01");
AWS_STATIC_STRING_FROM_LITERAL(key_name, "RSA_2048_06");

struct aws_cryptosdk_keyring *rawRsaKeyring = aws_cryptosdk_raw_rsa_keyring_new(
    alloc,
    key_namespace,
    key_name,
    private_key_from_pem,
    public_key_from_pem,
    AWS_CRYPTOSDK_RSA_OAEP_SHA256_MGF1);
```

------
#### [ JavaScript Browser ]

The AWS Encryption SDK for JavaScript in the browser gets its cryptographic primitives from the [WebCrypto](https://developer.mozilla.org/en-US/docs/Web/API/Web_Crypto_API) library\. Before you construct the keyring, you must use `importPublicKey()` and/or `importPrivateKey()` to import the raw key material into the WebCrypto backend\. This assures that the keyring is complete even though all calls to WebCrypto are asynchronous\. The object that the import methods take includes the wrapping algorithm and its padding mode\.

After importing the key material, use the `RawRsaKeyringWebCrypto()` method to instantiate the keyring\. When constructing a Raw RSA keyring in JavaScript, be aware of [potential incompatibility](javascript-compatibility.md) with other language implementations\.

For a complete example, see [rsa\_simple\.ts](https://github.com/aws/aws-encryption-sdk-javascript/blob/master/modules/example-browser/src/rsa_simple.ts) \(JavaScript Browser\)\.

```
const privateKey = await RawRsaKeyringWebCrypto.importPrivateKey(
  privateRsaJwKKey
)

const publicKey = await RawRsaKeyringWebCrypto.importPublicKey(
  publicRsaJwKKey
)

const keyNamespace = 'HSM_01'
const keyName = 'RSA_2048_06'

const keyring = new RawRsaKeyringWebCrypto({
  keyName,
  keyNamespace,
  publicKey,
  privateKey,
})
```

------
#### [ JavaScript Node\.js ]

To instantiate a Raw RSA keyring in AWS Encryption SDK for JavaScript for Node\.js, create a new instance of the `RawRsaKeyringNode` class\. The `wrapKey` parameter holds the public key\. The `unwrapKey` parameter holds the private key\. The `RawRsaKeyringNode` constructor calculates a default padding mode for you, although you can specify a preferred padding mode\.

When constructing a raw RSA keyring in JavaScript, be aware of [potential incompatibility](javascript-compatibility.md) with other language implementations\.

For a complete example, see [rsa\_simple\.ts](https://github.com/aws/aws-encryption-sdk-javascript/blob/master/modules/example-node/src/rsa_simple.ts) \(JavaScript Node\.js\)\. 

```
const keyNamespace = 'HSM_01'
const keyName = 'RSA_2048_06'

const keyring = new RawRsaKeyringNode({ keyName, keyNamespace, rsaPublicKey, rsaPrivateKey})
```

------