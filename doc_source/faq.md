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
The [AWS SDKs](https://aws.amazon.com/tools/) provide libraries for interacting with Amazon Web Services \(AWS\)\. They integrate with AWS Key Management Service \(AWS KMS\) to generate, encrypt, and decrypt data keys\. However, in most cases you can't use them to directly encrypt or decrypt raw data\.  
The AWS Encryption SDK provides an encryption library that optionally integrates with AWS KMS to provide wrapping keys\. The AWS Encryption SDK builds on AWS KMS operations to do the following things:  
+ Generate, encrypt, and decrypt data keys
+ Use those data keys to encrypt and decrypt your raw data
+ Store the encrypted data keys with the corresponding encrypted data in a single object
You can also use the AWS Encryption SDK with no AWS integration by defining a custom wrapping key, master key, or master key provider\.

**How is the AWS Encryption SDK different from the Amazon S3 encryption client?**  <a name="s3-encryption-client"></a>
The Amazon S3 encryption client in the AWS SDKs provides encryption and decryption for data that you store in Amazon Simple Storage Service \(Amazon S3\)\. These clients are tightly coupled to Amazon S3 and are intended for use only with data stored there\.  
The AWS Encryption SDK provides encryption and decryption for data that you can store anywhere\. The AWS Encryption SDK and the Amazon S3 encryption client are not compatible because they produce ciphertexts with different data formats\.

**Which cryptographic algorithms are supported by the AWS Encryption SDK, and which one is the default?**  <a name="supported-algorithms-faq"></a>
The AWS Encryption SDK uses the Advanced Encryption Standard \(AES\) algorithm in Galois/Counter Mode \(GCM\), known as AES\-GCM\. The SDK supports 256\-bit, 192\-bit, and 128\-bit encryption keys\. In all cases, the length of the initialization vector \(IV\) is 12 bytes; the length of the authentication tag is 16 bytes\. By default, the SDK uses the data key as an input to the HMAC\-based extract\-and\-expand key derivation function \(HKDF\) to derive the AES\-GCM encryption key, and also adds an Elliptic Curve Digital Signature Algorithm \(ECDSA\) signature\.  
For information about choosing which algorithm to use, see [Supported algorithm suites](supported-algorithms.md)\.  
For implementation details about the supported algorithms, see [Algorithms reference](algorithms-reference.md)\.

**How is the initialization vector \(IV\) generated and where is it stored?**  <a name="iv-generation"></a>
In previous releases, the AWS Encryption SDK randomly generated a unique IV value for each encryption operation\. The SDK now uses a deterministic method to construct a different IV value for each frame so that every IV is unique within its message\. The SDK stores the IV in the encrypted message that it returns\. For more information, see [AWS Encryption SDK message format reference](message-format.md)\.

**How is each data key generated, encrypted, and decrypted?**  <a name="key-generation"></a>
The method depends on the master key provider or keyring and its implementation of its master keys or wrapping keys\. When AWS KMS is the master key provider, the AWS Encryption SDK uses the AWS KMS [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) API operation to generate each data key in both plaintext and encrypted forms\. It uses the [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operation to decrypt the data key\. AWS KMS encrypts and decrypts the data key by using the customer master key \(CMK\) that you specified when configuring the master key provider or keyring\.

**How do I keep track of the data keys that were used to encrypt my data?**  <a name="track-keys"></a>
The AWS Encryption SDK does this for you\. When you encrypt data, the SDK encrypts the data key and stores the encrypted key along with the encrypted data in the [encrypted message](concepts.md#message) that it returns\. When you decrypt data, the AWS Encryption SDK extracts the encrypted data key from the encrypted message, decrypts it, and then uses it to decrypt the data\.

**How does the AWS Encryption SDK store encrypted data keys with their encrypted data?**  <a name="store-encrypted-keys"></a>
The encryption operations in the AWS Encryption SDK return an [encrypted message](concepts.md#message), a single data structure that contains the encrypted data and its encrypted data keys\. The message format consists of at least two parts: a *header* and a *body*\. In some cases, the message format consists of a third part known as a *footer*\. The message header contains the encrypted data keys and information about how the message body is formed\. The message body contains the encrypted data\. The message footer contains a signature that authenticates the message header and message body\. For more information, see [AWS Encryption SDK message format reference](message-format.md)\.

**How much overhead does the AWS Encryption SDK message format add to my encrypted data?**  <a name="overhead"></a>
The amount of overhead added by the AWS Encryption SDK depends on several factors, including the following:  
+ The size of the plaintext data
+ Which of the supported algorithms is used
+ Whether additional authenticated data \(AAD\) is provided, and the length of that AAD
+ The number and type of wrapping keys or master keys
+ The frame size \(when [framed data](message-format.md#body-framing) is used\)
When you use the AWS Encryption SDK with its default configuration \(one AWS KMS CMK as the wrapping key \(or master key\), no AAD, nonframed data, and an encryption algorithm with signing\), the overhead is approximately 600 bytes\. In general, you can reasonably assume that the AWS Encryption SDK adds overhead of 1 KB or less, not including the provided AAD\. For more information, see [AWS Encryption SDK message format reference](message-format.md)\.

**Can I use my own master key provider?**  <a name="own-provider"></a>
Yes\. The implementation details vary depending on which of the [supported programming languages](programming-languages.md) you use\. However, all supported languages allow you to define custom [cryptographic materials managers \(CMMs\)](concepts.md#crypt-materials-manager), master key providers, keyrings, master keys, and wrapping keys\.

**Can I encrypt data under more than one wrapping key?**  <a name="multiple-master-keys"></a>
Yes\. You can encrypt the data key with additional wrapping keys \(or master keys\) to add redundancy when the key is in a different region or is unavailable for decryption\.  
To encrypt data under multiple master keys, create a master key provider with multiple master keys or a keyring with multiple wrapping keys\. You can see examples of this pattern in the example code for [Java](java-example-code.md#java-example-multiple-providers) and [Python](python-example-code.md#python-example-multiple-providers)\. When working with keyrings, you can create a [single keyring with multiple wrapping keys](choose-keyring.md#kms-keyring-encrypt) or a [multi\-keyring](choose-keyring.md#use-multi-keyring)\.  
When you encrypt data by using a master key provider that returns multiple master keys or a keyring with multiple wrapping keys, the AWS Encryption SDK uses one wrapping key to generate a plaintext data key\. It encrypts the data that you pass to the encryption methods with the data key and encrypts that data key with the same wrapping key\. Then, it encrypts the data key with the other wrapping keys\. The resulting [encrypted message](concepts.md#message) includes the encrypted data and one encrypted data key for each wrapping key\. The resulting message can be decrypted by using any one of the wrapping keys used in the encryption operation\.

**Which data types can I encrypt with the AWS Encryption SDK?**  <a name="data-types"></a>
The AWS Encryption SDK can encrypt raw bytes \(byte arrays\), I/O streams \(byte streams\), and strings\. We provide example code for each of the [supported programming languages](programming-languages.md)\.

**How does the AWS Encryption SDK encrypt and decrypt input/output \(I/O\) streams?**  <a name="streams"></a>
The AWS Encryption SDK creates an encrypting or decrypting stream that wraps an underlying I/O stream\. The encrypting or decrypting stream performs a cryptographic operation on a read or write call\. For example, it can read plaintext data on the underlying stream and encrypt it before returning the result\. Or it can read ciphertext from an underlying stream and decrypt it before returning the result\. We provide example code for encrypting and decrypting streams for each of the [supported programming languages](programming-languages.md)\.