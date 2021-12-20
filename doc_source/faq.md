# Frequently asked questions<a name="faq"></a>
+ [How is the AWS Encryption SDK different from the AWS SDKs?](#aws-sdks)
+ [How is the AWS Encryption SDK different from the Amazon S3 encryption client?](#s3-encryption-client)
+ [Which cryptographic algorithms are supported by the AWS Encryption SDK, and which one is the default?](#supported-algorithms-faq)
+ [How is the initialization vector \(IV\) generated and where is it stored?](#iv-generation)
+ [How is each data key generated, encrypted, and decrypted?](#key-generation)
+ [How do I keep track of the data keys that were used to encrypt my data?](#track-keys)
+ [How does the AWS Encryption SDK store encrypted data keys with their encrypted data?](#store-encrypted-keys)
+ [How much overhead does the AWS Encryption SDK message format add to my encrypted data?](#overhead)
+ [Can I use my own master key provider?](#own-provider)
+ [Can I encrypt data under more than one wrapping key?](#multiple-master-keys)
+ [Which data types can I encrypt with the AWS Encryption SDK?](#data-types)
+ [How does the AWS Encryption SDK encrypt and decrypt input/output \(I/O\) streams?](#streams)

**How is the AWS Encryption SDK different from the AWS SDKs?**  <a name="aws-sdks"></a>
The [AWS SDKs](https://aws.amazon.com/tools/) provide libraries for interacting with Amazon Web Services \(AWS\), including AWS Key Management Service \(AWS KMS\)\. You can use the AWS SDKs to interact with AWS KMS, including encrypting and decrypting small amounts of data \(up to 4,096 bytes with a symmetric encryption key\) and generating data keys for client\-side encryption\.   
However, when you generate a data key, you must manage the entire encryption and decryption process, including encrypting your data with the data key outside of AWS KMS, safely discarding the plaintext data key, storing the encrypted data key, and then decrypting the data key and decrypting your data\. The AWS Encryption SDK handles this process for you\.  
The AWS Encryption SDK provides a library that encrypts and decrypts data using industry standards and best practices\. It generates the data key, encrypts it under the wrapping keys you specify, and returns an *encrypted message*, a portable data object that includes the encrypted data and the encrypted data keys you need to decrypt it\. When it's time to decrypt, you pass in the encrypted message and at least one of the wrapping keys \(optional\), and the AWS Encryption SDK returns your plaintext data\.  
You can use AWS KMS keys as wrapping keys in the AWS Encryption SDK, but it is not required\. You can use encryption keys that you generate and those from your key manager or on\-premises hardware security module\. You can use the AWS Encryption SDK even if you don't have an AWS account\.

**How is the AWS Encryption SDK different from the Amazon S3 encryption client?**  <a name="s3-encryption-client"></a>
The Amazon S3 encryption client in the AWS SDKs provides encryption and decryption for data that you store in Amazon Simple Storage Service \(Amazon S3\)\. These clients are tightly coupled to Amazon S3 and are intended for use only with data stored there\.  
The AWS Encryption SDK provides encryption and decryption for data that you can store anywhere\. The AWS Encryption SDK and the Amazon S3 encryption client are not compatible because they produce ciphertexts with different data formats\.

**Which cryptographic algorithms are supported by the AWS Encryption SDK, and which one is the default?**  <a name="supported-algorithms-faq"></a>
The AWS Encryption SDK uses the Advanced Encryption Standard \(AES\) symmetric algorithm in Galois/Counter Mode \(GCM\), known as AES\-GCM, to encrypt your data\. It lets you choose from several symmetric and asymmetric algorithms to encrypt the data keys that encrypt your data\.   
For AES\-GCM, the default algorithm suite is AES\-GCM with a 256\-bit key, key derivation \(HKDF\), [digital signatures](concepts.md#digital-sigs), and [key commitment](concepts.md#key-commitment)\. AWS Encryption SDK also supports 192\-bit, and 128\-bit encryption keys and encryption algorithms without digital signatures and key commitment\.  
In all cases, the length of the initialization vector \(IV\) is 12 bytes; the length of the authentication tag is 16 bytes\. By default, the SDK uses the data key as an input to the HMAC\-based extract\-and\-expand key derivation function \(HKDF\) to derive the AES\-GCM encryption key, and also adds an Elliptic Curve Digital Signature Algorithm \(ECDSA\) signature\.  
For information about choosing which algorithm to use, see [Supported algorithm suites](supported-algorithms.md)\.  
For implementation details about the supported algorithms, see [Algorithms reference](algorithms-reference.md)\.

**How is the initialization vector \(IV\) generated and where is it stored?**  <a name="iv-generation"></a>
The AWS Encryption SDK uses a deterministic method to construct a different IV value for each frame\. This procedure guarantees that IVs are never repeated within a message\. \(Prior to version 1\.3\.0 of the AWS Encryption SDK for Java and the AWS Encryption SDK for Python, the AWS Encryption SDK randomly generated a unique IV value for each frame\.\)  
The IV is stored in the encrypted message that the AWS Encryption SDK returns\. For more information, see the [AWS Encryption SDK message format reference](message-format.md)\.

**How is each data key generated, encrypted, and decrypted?**  <a name="key-generation"></a>
The method depends on the keyring or master key provider you use\.   
The AWS KMS keyrings and master key providers in the AWS Encryption SDK use the AWS KMS [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) API operation to generate each data key and encrypt it under its wrapping key\. To encrypt copies of the data key under additional KMS keys, they use the AWS KMS [Encrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Encrypt.html) operation\. To decrypt the data keys, they use the AWS KMS [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operation\. For details, see [AWS KMS keyring](https://github.com/awslabs/aws-encryption-sdk-specification/blob/master/framework/kms-keyring.md) in the AWS Encryption SDK Specification in GitHub\.  
Other keyrings generate the data key, encrypt, and decrypt using best practice methods for each programming language\. For details, see the specification of the keyring or master key provider in the [Framework section](https://github.com/awslabs/aws-encryption-sdk-specification/tree/master/framework) of the AWS Encryption SDK Specification in GitHub\.

**How do I keep track of the data keys that were used to encrypt my data?**  <a name="track-keys"></a>
The AWS Encryption SDK does this for you\. When you encrypt data, the SDK encrypts the data key and stores the encrypted key along with the encrypted data in the [encrypted message](concepts.md#message) that it returns\. When you decrypt data, the AWS Encryption SDK extracts the encrypted data key from the encrypted message, decrypts it, and then uses it to decrypt the data\.

**How does the AWS Encryption SDK store encrypted data keys with their encrypted data?**  <a name="store-encrypted-keys"></a>
The encryption operations in the AWS Encryption SDK return an [encrypted message](concepts.md#message), a single data structure that contains the encrypted data and its encrypted data keys\. The message format consists of at least two parts: a *header* and a *body*\. The message header contains the encrypted data keys and information about how the message body is formed\. The message body contains the encrypted data\. If the algorithm suite includes a [digital signature](concepts.md#digital-sigs), the message format includes a *footer* that contains the signature\. For more information, see [AWS Encryption SDK message format reference](message-format.md)\.

**How much overhead does the AWS Encryption SDK message format add to my encrypted data?**  <a name="overhead"></a>
The amount of overhead added by the AWS Encryption SDK depends on several factors, including the following:  
+ The size of the plaintext data
+ Which of the supported algorithms is used
+ Whether additional authenticated data \(AAD\) is provided, and the length of that AAD
+ The number and type of wrapping keys or master keys
+ The frame size \(when [framed data](message-format.md#body-framing) is used\)
When you use the AWS Encryption SDK with its default configuration \(one AWS KMS key as the wrapping key \(or master key\), no AAD, nonframed data, and an encryption algorithm with signing\), the overhead is approximately 600 bytes\. In general, you can reasonably assume that the AWS Encryption SDK adds overhead of 1 KB or less, not including the provided AAD\. For more information, see [AWS Encryption SDK message format reference](message-format.md)\.

**Can I use my own master key provider?**  <a name="own-provider"></a>
Yes\. The implementation details vary depending on which of the [supported programming languages](programming-languages.md) you use\. However, all supported languages allow you to define custom [cryptographic materials managers \(CMMs\)](concepts.md#crypt-materials-manager), master key providers, keyrings, master keys, and wrapping keys\.

**Can I encrypt data under more than one wrapping key?**  <a name="multiple-master-keys"></a>
Yes\. You can encrypt the data key with additional wrapping keys \(or master keys\) to add redundancy when the key is in a different region or is unavailable for decryption\.  
To encrypt data under multiple wrapping keys, create a keyring or master key provider with multiple wrapping keys\. When working with keyrings, you can create a [single keyring with multiple wrapping keys](use-kms-keyring.md#kms-keyring-encrypt) or a [multi\-keyring](use-multi-keyring.md)\.  
When you encrypt data with multiple wrapping keys, the AWS Encryption SDK uses one wrapping key to generate a plaintext data key\. The data key is unique and mathematically unrelated to the wrapping key\. The operation returns the plaintext data key and a copy of the data key encrypted by the wrapping key\. Then the encryption method, encrypts the data key with the other wrapping keys\. The resulting [encrypted message](concepts.md#message) includes the encrypted data and one encrypted data key for each wrapping key\.   
The encrypted message can be decrypted by using any one of the wrapping keys used in the encryption operation\. The AWS Encryption SDK uses a wrapping key to decrypt an encrypted data key\. Then, it uses the plaintext data key to decrypt the data\.

**Which data types can I encrypt with the AWS Encryption SDK?**  <a name="data-types"></a>
The AWS Encryption SDK can encrypt raw bytes \(byte arrays\), I/O streams \(byte streams\), and strings\. We provide example code for each of the [supported programming languages](programming-languages.md)\.

**How does the AWS Encryption SDK encrypt and decrypt input/output \(I/O\) streams?**  <a name="streams"></a>
The AWS Encryption SDK creates an encrypting or decrypting stream that wraps an underlying I/O stream\. The encrypting or decrypting stream performs a cryptographic operation on a read or write call\. For example, it can read plaintext data on the underlying stream and encrypt it before returning the result\. Or it can read ciphertext from an underlying stream and decrypt it before returning the result\. We provide example code for encrypting and decrypting streams for each of the [supported programming languages](programming-languages.md)\.