# Supported algorithm suites in the AWS Encryption SDK<a name="supported-algorithms"></a>

An *algorithm suite* is a collection of cryptographic algorithms and related values\. Cryptographic systems use the algorithm implementation to generate the ciphertext message\.

The AWS Encryption SDK algorithm suite uses the Advanced Encryption Standard \(AES\) algorithm in Galois/Counter Mode \(GCM\), known as AES\-GCM, to encrypt raw data\. The AWS Encryption SDK supports 256\-bit, 192\-bit, and 128\-bit encryption keys\. The length of the initialization vector \(IV\) is always 12 bytes\. The length of the authentication tag is always 16 bytes\.

By default, the AWS Encryption SDK uses an algorithm suite with AES\-GCM with an HMAC\-based extract\-and\-expand key derivation function \([HKDF](https://en.wikipedia.org/wiki/HKDF)\), signing, and a 256\-bit encryption key\. If the [commitment policy](concepts.md#commitment-policy) requires [key commitment](concepts.md#key-commitment), the AWS Encryption SDK selects an algorithm suite that also supports key commitment; otherwise, it selects an algorithm suite with key derivation and signing, but not key commitment\.

## Recommended: AES\-GCM with key derivation, signing, and key commitment<a name="recommended-algorithms"></a>

The AWS Encryption SDK recommends an algorithm suite that derives an AES\-GCM encryption key by supplying a 256\-bit data encryption key to the HMAC\-based extract\-and\-expand key derivation function \(HKDF\)\. The AWS Encryption SDK adds an Elliptic Curve Digital Signature Algorithm \(ECDSA\) signature\. To support [key commitment](concepts.md#key-commitment), this algorithm suite also derives a *key commitment string* – a non\-secret data\-key identifier – that is stored in the metadata of the encrypted message\. This key commitment string is also derived through HKDF using a procedure similar to deriving the data encryption key\.


**AWS Encryption SDK Algorithm Suite**  

| Encryption algorithm | Data encryption key length \(in bits\) | Key derivation algorithm | Signature algorithm | Key commitment | 
| --- | --- | --- | --- | --- | 
| AES\-GCM | 256 | HKDF with SHA\-384 | ECDSA with P\-384 and SHA\-384 | HKDF with SHA\-512 | 

The HKDF helps you avoid accidental reuse of a data encryption key and reduces the risk of overusing a data key\. 

For signing, this algorithm suite uses ECDSA with a cryptographic hash function algorithm \(SHA\-384\)\. ECDSA is used by default, even when it is not specified by the policy for the underlying master key\. Message signing verifies the identity of the message sender and adds message authenticity to the envelope encrypted data\. It is particularly useful when the authorization policy for a master key allows one set of users to encrypt data and a different set of users to decrypt data\. 

Algorithm suites with key commitment ensure that each ciphertext decrypts to only one plaintext\. They do this by validating the identity of the data key used as input to the encryption algorithm\. When encrypting, these algorithm suites derive a key commitment string\. Before decrypting, they validate that the data key matches the key commitment string\. If it does not, the decrypt call fails\.

## Other supported algorithm suites<a name="other-algorithms"></a>

The AWS Encryption SDK supports the following alternate algorithm suites for backward compatibility\. In general, we do not recommend these algorithm suites\. However, we recognize that signing can hinder performance significantly, so we offer a key committing suite with key derivation for those cases\. For applications that must make more significant performance tradeoffs, we continue to offer suites that lack signing, key commitment, and key derivation\.

**AES\-GCM without key commitment**  
Algorithm suites without key commitment do not validate the data key before decrypting\. As a result, these algorithm suites might decrypt a single ciphertext into different plaintext messages\. However, because algorithm suites with key commitment produce a [slightly larger \(\+30 bytes\) encrypted message](message-format.md) and take longer to process, they might not be the best choice for every application\.   
The AWS Encryption SDK supports an algorithm suite with key derivation, key commitment, signing, and one with key derivation and key commitment, but not signing\. We do not recommend using an algorithm suite without key commitment\. If you must, we recommend an algorithm suite with key derivation and key commitment, but not signing\. However, if your application performance profile supports using an algorithm suite, using an algorithm suite with key commitment, key derivation, and signing is a best practice\.

**AES\-GCM without signing**  
Algorithm suites without signing lack the ECDSA signature that provides authenticity and non\-repudiation\. Use these suites only when the users who encrypt data and those who decrypt data are equally trusted\.   
When using an algorithm suite without signing, we recommend that you choose one with key derivation and key commitment\. 

**AES\-GCM without key derivation**  
Algorithm suites without key derivation use the data encryption key as the AES\-GCM encryption key, instead of using a key derivation function to derive a unique key\. We discourage using this suite to generate ciphertext, but the AWS Encryption SDK supports it for compatibility reasons\.

For more information about how these suites are represented and used in the library, see [AWS Encryption SDK algorithms reference](algorithms-reference.md)\.