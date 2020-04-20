# Data key caching details<a name="data-caching-details"></a>

Most applications can use the default implementation of data key caching without writing custom code\. This section describes the default implementation and some details about options\. 

**Topics**
+ [How data key caching works](#how-caching-works)
+ [Creating a cryptographic materials cache](#simplecache)
+ [Creating a caching cryptographic materials manager](#caching-cmm)
+ [What is in a data key cache entry?](#cache-entries)
+ [Encryption context: How to select cache entries](#caching-encryption-context)
+ [Is my application using cached data keys?](#caching-effect)

## How data key caching works<a name="how-caching-works"></a>

When you use data key caching in a request to encrypt or decrypt data, the AWS Encryption SDK first searches the cache for a data key that matches the request\. If it finds a valid match, it uses the cached data key to encrypt the data\. Otherwise, it generates a new data key, just as it would without the cache\. 

Data key caching is not used for data of unknown size, such as streamed data\. This allows the caching CMM to properly enforce the [maximum bytes threshold](thresholds.md)\. To avoid this behavior, add the message size to the encryption request\. 

In addition to a cache, data key caching uses a [caching cryptographic materials manager](#caching-cmm) \(caching CMM\)\. The caching CMM is a specialized [cryptographic materials manager \(CMM\)](concepts.md#crypt-materials-manager) that interacts with a [cache](#simplecache) and an underlying [CMM](concepts.md#crypt-materials-manager)\. \(When you specify a [master key provider](concepts.md#master-key-provider) or keyring, the AWS Encryption SDK creates a default CMM for you\.\) The caching CMM caches the data keys that its underlying CMM returns\. The caching CMM also enforces cache security thresholds that you set\. 

To prevent the wrong data key from being selected from the cache, all compatible caching CMMs require that the following properties of the cached cryptographic materials match the materials request\.
+ [Algorithm suite](concepts.md#crypto-algorithm)
+ [Encryption context](#caching-encryption-context) \(even when empty\)
+ Partition name \(a string that identifies the caching CMM\)
+ \(Decryption only\) Encrypted data keys

**Note**  
The AWS Encryption SDK caches data keys only when the [algorithm suite](concepts.md#crypto-algorithm) uses a [key derivation function](https://en.wikipedia.org/wiki/Key_derivation_function)\.

The following workflows show how a request to encrypt data is processed with and without data key caching\. They show how the caching components that you create, including the cache and the caching CMM, are used in the process\.

### Encrypt data without caching<a name="workflow-wo-cache"></a>

To get encryption materials without caching:

1. An application asks the AWS Encryption SDK to encrypt data\. 

   The request specifies a master key provider or keyring\. The AWS Encryption SDK creates a default CMM that interacts with your master key provider or keyring\.

1. The AWS Encryption SDK asks the CMM for encryption materials \(get cryptographic materials\)\.

1. The CMM asks its [keyring](concepts.md#keyring) \(C and JavaScript\) or [master key provider](concepts.md#master-key-provider) \(Java and Python\) for cryptographic materials\. This might involve a call to a cryptographic service, such as AWS Key Management Service \(AWS KMS\)\. The CMM returns the encryption materials to the AWS Encryption SDK\.

1. The AWS Encryption SDK uses the plaintext data key to encrypt the data\. It stores the encrypted data and encrypted data keys in an [encrypted message](concepts.md#message), which it returns to the user\.

![\[Encrypt data without caching\]](http://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/images/encrypt-workflow-no-cache.png)

### Encrypt data with caching<a name="workflow-with-cache"></a>

To get encryption materials with data key caching:

1. An application asks the AWS Encryption SDK to encrypt data\. 

   The request specifies a [caching cryptographic materials manager \(caching CMM\)](#caching-cmm) that is associated with a underlying cryptographic materials manager \(CMM\)\. When you specify a master key provider or keyring, the AWS Encryption SDK creates a default CMM for you\.

1. The SDK asks the specified caching CMM for encryption materials\.

1. The caching CMM requests encryption materials from the cache\.

   1. If the cache finds a match, it updates the age and use values of the matched cache entry, and returns the cached encryption materials to the caching CMM\. 

      If the cache entry conforms to its [security thresholds](thresholds.md), the caching CMM returns it to the SDK\. Otherwise, it tells the cache to evict the entry and proceeds as though there was no match\.

   1. If the cache cannot find a valid match, the caching CMM asks its underlying CMM to generate a new data key\. 

      The underlying CMM gets the cryptographic materials from its keyring \(C and JavaScript\) or master key provider \(Java and Python\)\. This might involve a call to a service, such as AWS Key Management Service\. The underlying CMM returns the plaintext and encrypted copies of the data key to the caching CMM\. 

      The caching CMM saves the new encryption materials in the cache\.

1. The caching CMM returns the encryption materials to the AWS Encryption SDK\.

1. The AWS Encryption SDK uses the plaintext data key to encrypt the data\. It stores the encrypted data and encrypted data keys in an [encrypted message](concepts.md#message), which it returns to the user\.

![\[Encrypt data with data key caching\]](http://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/images/encrypt-workflow-with-cache.png)

## Creating a cryptographic materials cache<a name="simplecache"></a>

The AWS Encryption SDK defines the requirements for a cryptographic materials cache used in data key caching\. It also provides a local cache, which is a configurable, in\-memory, [least recently used \(LRU\) cache](https://en.wikipedia.org/wiki/Cache_replacement_policies#Least_Recently_Used_.28LRU.29)\. To create an instance of the local cache, use the `LocalCryptoMaterialsCache` constructor in Java and Python, the getLocalCryptographicMaterialsCache function in JavaScript, or the `aws_cryptosdk_materials_cache_local_new` constructor in C\.

The local cache includes logic for basic cache management, including adding, evicting, and matching cached entries, and maintaining the cache\. You don't need to write any custom cache management logic\. You can use the local cache as is, customize it, or substitute any compatible cache\. 

When you create a local cache, you set its *capacity*, that is, the maximum number of entries that the cache can hold\. This setting helps you to design an efficient cache with limited data key reuse\.

The AWS Encryption SDK for Java and the AWS Encryption SDK for Python also provide a *null cryptographic materials cache* \(NullCryptoMaterialsCache\)\. The NullCryptoMaterialsCache returns a miss for all `GET` operations and does not respond to `PUT` operations\. You can use the NullCryptoMaterialsCache in testing or to temporarily disable caching in an application that includes caching code\. 

In the AWS Encryption SDK, each cryptographic materials cache is associated with a [caching cryptographic materials manager](#caching-cmm) \(caching CMM\)\. The caching CMM gets data keys from the cache, puts data keys in the cache, and enforces [security thresholds](thresholds.md) that you set\. When you create a caching CMM, you specify the cache that it uses and the underlying CMM or master key provider that generates the data keys that it caches\.

## Creating a caching cryptographic materials manager<a name="caching-cmm"></a>

To enable data key caching, you create a [cache](#simplecache) and a *caching cryptographic materials manager* \(caching CMM\)\. Then, in your requests to encrypt or decrypt data, you specify a caching CMM, instead of a standard [cryptographic materials manager \(CMM\)](concepts.md#crypt-materials-manager), or [master key provider](concepts.md#master-key-provider) or [keyring](concepts.md#keyring)\.

There are two types of CMMs\. Both get data keys \(and related cryptographic material\), but in different ways, as follows:
+ A CMM is associated with a keyring \(C or JavaScript\) or a master key provider \(Java and Python\)\. When the SDK asks the CMM for encryption or decryption materials, the CMM gets the materials from its keyring or master key provider\. In Java and Python, the CMM uses the master keys to generate, encrypt, or decrypt the data keys\. In C and JavaScript, the keyring generates, encrypts, and returns the cryptographic materials\.
+ A caching CMM is associated with one cache, such as a [local cache](#simplecache), and an underlying CMM\. When the SDK asks the caching CMM for cryptographic materials, the caching CMM tries to get them from the cache\. If it cannot find a match, the caching CMM asks its underlying CMM for the materials\. Then, it caches the new cryptographic materials before returning them to the caller\. 

The caching CMM also enforces [security thresholds](thresholds.md) that you set for each cache entry\. Because the security thresholds are set in and enforced by the caching CMM, you can use any compatible cache, even if the cache is not designed for sensitive material\.

## What is in a data key cache entry?<a name="cache-entries"></a>

Data key caching stores data keys and related cryptographic materials in a cache\. Each entry includes the elements listed below\. You might find this information useful when you're deciding whether to use the data key caching feature, and when you're setting security thresholds on a caching cryptographic materials manager \(caching CMM\)\.

**Cached Entries for Encryption Requests**  
The entries that are added to a data key cache as a result of a encryption operation include the following elements:
+ Plaintext data key
+ Encrypted data keys \(one or more\)
+ [Encryption context](#caching-encryption-context) 
+ Message signing key \(if one is used\)
+ [Algorithm suite](concepts.md#crypto-algorithm)
+ Metadata, including usage counters for enforcing security thresholds

**Cached Entries for Decryption Requests**  
The entries that are added to a data key cache as a result of a decryption operation include the following elements:
+ Plaintext data key
+ Signature verification key \(if one is used\)
+ Metadata, including usage counters for enforcing security thresholds

## Encryption context: How to select cache entries<a name="caching-encryption-context"></a>

You can specify an encryption context in any request to encrypt data\. However, the encryption context plays a special role in data key caching\. It lets you create subgroups of data keys in your cache, even when the data keys originate from the same caching CMM\.

An [encryption context](concepts.md#encryption-context) is a set of key\-value pairs that contain arbitrary nonsecret data\. During encryption, the encryption context is cryptographically bound to the encrypted data so that the same encryption context is required to decrypt the data\. In the AWS Encryption SDK, the encryption context is stored in the [encrypted message](concepts.md#message) with the encrypted data and data keys\. 

When you use a data key cache, you can also use the encryption context to select particular cached data keys for your encryption operations\. The encryption context is saved in the cache entry with the data key \(it's part of the cache entry ID\)\. Cached data keys are reused only when their encryption contexts match\. If you want to reuse certain data keys for an encryption request, specify the same encryption context\. If you want to avoid those data keys, specify a different encryption context\. 

The encryption context is always optional, but recommended\. If you don't specify an encryption context in your request, an empty encryption context is included in the cache entry identifier and matched to each request\.

## Is my application using cached data keys?<a name="caching-effect"></a>

Data key caching is an optimization strategy that is very effective for certain applications and workloads\. However, because it entails some risk, it's important to determine how effective it is likely to be for your situation, and then decide whether the benefits outweigh the risks\.

Because data key caching reuses data keys, the most obvious effect is reducing the number of calls to generate new data keys\. When data key caching is implemented, the AWS Encryption SDK calls the AWS KMS `GenerateDataKey` operation only to create the initial data key and when the cache misses\. But, caching improves performance perceptibly only in applications that generate numerous data keys with the same characteristics, including the same encryption context and algorithm suite\.

To determine whether your implementation of the AWS Encryption SDK is actually using data keys from the cache, try the following techniques\.
+ In the logs of your master key infrastructure, check the frequency of calls to create new data keys\. When data key caching is effective, the number of calls to create new keys should drop perceptibly\. For example, if you are using a AWS KMS master key provider or keyring, search the CloudTrail logs for [GenerateDataKey](https://docs.aws.amazon.com/kms/latest/APIReference/API_GenerateDataKey.html) calls\. 
+ Compare the [encrypted messages](concepts.md#message) that the AWS Encryption SDK returns in response to different encrypt requests\. For example, if you are using the AWS Encryption SDK for Java, compare the [ParsedCiphertext](https://aws.github.io/aws-encryption-sdk-java/javadoc/com/amazonaws/encryptionsdk/ParsedCiphertext.html) object from different encrypt calls\. In the AWS Encryption SDK for JavaScript, compare the contents of the `encryptedDataKeys` property of the [MessageHeader](https://github.com/aws/aws-encryption-sdk-javascript/blob/master/modules/serialize/src/types.ts#L21)\. When data keys are reused, the encrypted data keys in the encrypted message are identical\.