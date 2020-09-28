# Versions of the AWS Encryption SDK<a name="about-versions"></a>

Upgrading from version 1\.7\.*x* to version 2\.0\.*x* is designed to help you implement AWS Encryption SDK [best practices](best-practices.md) in your application\. 

**Note**  
Version 1\.7\.*x* indicates any version that begins with `1.7`\. To get 1\.7\.*x* features, use the latest 1\.7\.*x* version of the AWS Encryption SDK available for your programming language\.  
Version 2\.0\.*x* indicates any version that begins with `2.0`\. To get features introduced in version 2\.0\.*x*, use the latest version of the AWS Encryption SDK available for your programming language\.

The AWS Encryption SDK language implementations use [semantic versioning](https://semver.org/) to make it easier for you to identify the magnitude of changes in each release\. A change in the major version number, such as 1\.*x*\.*x* to 2\.*x*\.*x*, indicates a breaking change that is likely to require code changes and a planned deployment\. A change in a minor version, such as *x*\.1\.*x* to *x*\.2\.*x*, is always backward compatible, but might include deprecated elements\. 

For a detailed description of the changes for your programming language, see the Changelog for each language\. 
+ C — [CHANGELOG\.md](https://github.com/aws/aws-encryption-sdk-c/blob/master/CHANGELOG.md)
+ Java — [CHANGELOG\.md](https://github.com/aws/aws-encryption-sdk-java/blob/master/CHANGELOG.md)
+ Python — [CHANGELOG\.rst](https://github.com/aws/aws-encryption-sdk-python/blob/master/CHANGELOG.rst)
+ AWS Encryption CLI — [Versions of the AWS Encryption CLI](crypto-cli-versions.md) and [CHANGELOG\.rst](https://github.com/aws/aws-encryption-sdk-cli/blob/master/CHANGELOG.rst)

The following list describes the major differences between the versions\. 

**Topics**
+ [Versions earlier than 1\.7\.0](#versions-earlier)
+ [Version 1\.7\.*x*](#version-1.7)
+ [Version 2\.0\.*x*](#version-2)

## Versions earlier than 1\.7\.0<a name="versions-earlier"></a>

Versions of the AWS Encryption SDK earlier than 1\.7\.0 provide important security features, including encryption with the Advanced Encryption Standard algorithm in Galois/Counter Mode \(AES\-GCM\), an HMAC\-based extract\-and\-expand key derivation function \(HKDF\), signing, and a 256\-bit encryption key\. However, these versions don't support [best practices](best-practices.md) that we recommend, including [key commitment](concepts.md#key-commitment)\. 

## Version 1\.7\.*x*<a name="version-1.7"></a>

Version 1\.7\.*x* is designed to help users of earlier versions of the AWS Encryption SDK to upgrade to version 2\.0\.*x* and later\. If you are new to the AWS Encryption SDK, you can skip this version and begin with the latest available version in your programming language\.

Version 1\.7\.*x* is fully backward compatible; it does not introduce any breaking changes or change the behavior of the AWS Encryption SDK\. It's also forwards compatible; it allows you to update your code so it's compatible with version 2\.0\.*x*\. It includes new features, but doesn't fully enable them\. And it requires configuration values that prevent you from immediately adopting all new features until you are ready\.

Version 1\.7\.*x* includes the following changes:

**AWS KMS master key provider updates \(required\)**  <a name="changes-to-mkps"></a>
Version 1\.7\.*x* introduces new constructors to the AWS Encryption SDK for Java and AWS Encryption SDK for Python that explicitly create AWS KMS master key providers in either *strict* or *discovery* mode\. This version adds similar changes to the AWS Encryption SDK command\-line interface \(CLI\)\. For details, see [Updating AWS KMS master key providers](migrate-mkps-v2.md)\.  
+ In *strict mode*, AWS KMS master key providers require a list of wrapping keys, and they encrypt and decrypt with only the wrapping keys you specify\. This is an AWS Encryption SDK best practice that assures that you are using the wrapping keys you intend to use\. 
+ In *discovery mode*, AWS KMS master key providers do not take any wrapping keys\. You cannot use them for encrypting\. When decrypting, they can use any wrapping key to decrypt an encrypted data key\. However, you can limit the wrapping keys used for decryption to those in particular AWS accounts\. Account filtering is optional, but it's a [best practice](best-practices.md) that we recommend\.
The constructors that create earlier versions of AWS KMS master key providers are deprecated in version 1\.7\.*x* and removed in version 2\.0\.*x*\. These constructors instantiate master key providers that encrypt using the wrapping keys you specify\. However, they decrypt encrypted data keys using the wrapping key that encrypted them, without regard to the specified wrapping keys\. Users might unintentionally decrypt messages with wrapping keys they don't intend to use, including customer master keys in other AWS accounts and Regions\.  
There are no changes to constructors for AWS KMS master keys\. When encrypting and decrypting, AWS KMS master keys use only the CMK that you specify\.

**AWS KMS keyring updates \(optional\)**  
Version 1\.7\.*x* adds a new filter to the AWS Encryption SDK for C and AWS Encryption SDK for JavaScript implementations that limits [AWS KMS discovery keyrings](choose-keyring.md#kms-keyring-discovery) to particular AWS accounts\. This new account filter is optional, but it's a [best practice](best-practices.md) that we recommend\. For details, see [Updating AWS KMS keyrings](migrate-keyrings-v2.md)\.  
There are no changes to constructors for AWS KMS keyrings\. Standard AWS KMS keyrings behave like master key providers in strict mode\. AWS KMS discovery keyrings are created explicitly in discovery mode\. 

**Passing a key ID to AWS KMS Decrypt**  
Beginning in version 1\.7\.*x*, when decrypting encrypted data keys, the AWS Encryption SDK always specifies a customer master key \(CMK\) in its calls to the AWS KMS [Decrypt](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) operation\. The AWS Encryption SDK gets the key ID value for the CMK from the metadata in each encrypted data key\. This feature doesn't require any code changes\.  
Specifying the key ID of the CMK is not required to decrypt ciphertext that was encrypted under a symmetric CMK, but it is an [AWS KMS best practice](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html#KMS-Decrypt-request-KeyId)\. Like specifying wrapping keys in your key provider, this practice assures that AWS KMS only decrypts using the wrapping key you intend to use\.

**Decrypt ciphertext with key commitment**  
Version 1\.7\.*x* can decrypt ciphertext that was encrypted with or without [key commitment](concepts.md#key-commitment)\. However, it cannot encrypt ciphertext with key commitment\. This property allows you to fully deploy applications that can decrypt ciphertext encrypted with key commitment before they ever encounter any such ciphertext\. Because this version decrypts messages that are encrypted without key commitment, you don't need to re\-encrypt any ciphertext\.  
To implement this behavior, version 1\.7\.*x* includes a new [commitment policy](concepts.md#commitment-policy) configuration setting that determines whether the AWS Encryption SDK can encrypt or decrypt with key commitment\. In version 1\.7\.*x*, the only valid value for the commitment policy is `ForbidEncryptAllowDecrypt`\. This value prevents the AWS Encryption SDK from encrypting with either of the new algorithm suites that include key commitment\. It allows the AWS Encryption SDK to decrypt ciphertext with and without key commitment\.   
Although there is only one valid commitment policy value in version 1\.7\.*x*, we require that you can set this value explicitly\. This prepares you for version 2\.0\.*x* where the default commitment policy setting requires key commitment when encrypting and decrypting\. Setting this value explicitly in version 1\.7\.*x* means that you control your commitment policy\. You determine when you start encrypting messages with key commitment and stop decrypting ciphertext without key commitment\. For details, see [Setting your commitment policy](migrate-commitment-policy.md)\.

**Algorithm suites with key commitment**  
Version 1\.7\.*x* includes two new [algorithm suites](supported-algorithms.md) that support key commitment\. One includes signing; the other does not\. Like earlier supported algorithm suites, both of these new algorithm suites include encryption with AES\-GCM, a 256\-bit encryption key, and an HMAC\-based extract\-and\-expand key derivation function \(HKDF\)\.  
However, the default algorithm suite used for encryption does not change\. These algorithm suites are added to version 1\.7\.*x* to prepare your application to use them in version 2\.0\.*x* and later\. 

**CMM implementation changes**  
Version 1\.7\.*x* introduces changes to the Default cryptographic materials manager \(CMM\) interface to support key commitment\. This change affects you only if you have written a custom CMM\. For details, see the API documentation or GitHub repository for your [programming language](programming-languages.md)\.

## Version 2\.0\.*x*<a name="version-2"></a>

Version 2\.0\.*x* supports all of the security features offered in the AWS Encryption SDK, including specified wrapping keys and key commitment\. To support these features, version 2\.0\.*x* includes breaking changes for earlier versions of the AWS Encryption SDK\. You can prepare for these changes by deploying version 1\.7\.*x*\. Version 2\.0\.*x* includes all of the new features introduced in version 1\.7\.*x* with the following additions and changes\.

**AWS KMS master key providers**  
The original AWS KMS master key provider constructors that were deprecated in version 1\.7\.*x* are removed in version 2\.0\.*x*\. You must explicitly construct AWS KMS master key providers in [strict mode or discovery mode](migrate-mkps-v2.md)\.

**Encrypt and decrypt ciphertext with key commitment**  
Version 2\.0\.*x* can encrypt and decrypt ciphertext with or without [key commitment](concepts.md#key-commitment)\. Its behavior is determined by the commitment policy setting\. By default, it always encrypts with key commitment and only decrypts ciphertext encrypted with key commitment\. Unless you change the commitment policy, the AWS Encryption SDK will not decrypt ciphertexts encrypted by any earlier version of the AWS Encryption SDK, including version 1\.7\.*x*\.  
By default, version 2\.0\.*x* will not decrypt any ciphertext that was encrypted without key commitment\. If your application might encounter a ciphertext that was encrypted without key commitment, set a commitment policy value with `AllowDecrypt`\.
In version 2\.0\.*x*, the commitment policy setting has three valid values:  
+ `ForbidEncryptAllowDecrypt` — The AWS Encryption SDK cannot encrypt with key commitment\. It can decrypt ciphertexts encrypted with or without key commitment\. 
+ `RequireEncryptAllowDecrypt` — The AWS Encryption SDK must encrypt with key commitment\. It can decrypt ciphertexts encrypted with or without key commitment\. 
+ `RequireEncryptRequireDecrypt` \(default\) — The AWS Encryption SDK must encrypt with key commitment\. It only decrypts ciphertexts with key commitment\. 
If you are migrating from an earlier version of the AWS Encryption SDK to version 2\.0\.*x*, set the commitment policy to a value that assures that you can decrypt all existing ciphertexts that your application might encounter\. You are likely to adjust this setting over time\.