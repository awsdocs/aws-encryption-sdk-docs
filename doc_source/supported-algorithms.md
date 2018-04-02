# Supported Algorithm Suites in the AWS Encryption SDK<a name="supported-algorithms"></a>

An *algorithm suite* is a collection of cryptographic algorithms and related values\. Cryptographic systems use the algorithm implemenation to generate the ciphertext message\.

The AWS Encryption SDK algorithm suite uses the Advanced Encryption Standard \(AES\) algorithm in Galois/Counter Mode \(GCM\), known as AES\-GCM, to encrypt raw data\. The SDK supports 256\-bit, 192\-bit, and 128\-bit encryption keys\. The length of the initialization vector \(IV\) is always 12 bytes; the length of the authentication tag is always 16 bytes\.

The SDK implements AES\-GCM in one of three ways\. By default, the SDK uses AES\-GCM with an HMAC\-based extract\-and\-expand key derivation function \([HKDF](https://en.wikipedia.org/wiki/HKDF)\), signing, and a 256\-bit encryption key\.

## Recommended: AES\-GCM with Key Derivation and Signing<a name="recommended-algorithms"></a>

In the recommended algorithm suite, the SDK uses the data encryption key as an input to the HMAC\-based extract\-and\-expand key derivation function \(HKDF\) to derive the AES\-GCM encryption key\. The SDK also adds an Elliptic Curve Digital Signature Algorithm \(ECDSA\) signature\. By default, the SDK uses this algorithm suite with a 256\-bit encryption key\.

The HKDF helps you avoid accidental reuse of a data encryption key\. 

This algorithm suite uses ECDSA and a message signing algorithm \(SHA\-384 or SHA\-256\)\. ECDSA is used by default, even when it is not specified by the policy for the underlying master key\. Message signing verifies the identity of the message sender and adds message authenticity to the envelope encrypted data\. It is particularly useful when the authorization policy for a master key allows one set of users to encrypt data and a different set of users to decrypt data\. 

The following table lists the variations of the recommended algorithm suites\.


**AWS Encryption SDK Algorithm Suites**  

| Algorithm Name | Data Encryption Key Length \(in bits\) | Algorithm Mode | Key Derivation Algorithm | Signature Algorithm | 
| --- | --- | --- | --- | --- | 
| AES | 256 | GCM | HKDF with SHA\-384 | ECDSA with P\-384 and SHA\-384 | 
| AES | 192 | GCM | HKDF with SHA\-384 | ECDSA with P\-384 and SHA\-384 | 
| AES | 128 | GCM | HKDF with SHA\-256 | ECDSA with P\-256 and SHA\-256 | 

## Other Supported Algorithm Suites<a name="other-algorithms"></a>

The AWS Encryption SDK supports the alternate algorithm suites for backward compatibility, although we do not recommend them\. If you cannot use an algorithm suite with HKDF and signing, we recommend an algorithm suite with HKDF over one that lacks both elements\.

**AES\-GCM with Key Derivation Only**  
This algorithm suite uses a key derivation function, but lacks the ECDSA signature that provides authenticity and nonrepudiation\. Use this suite when the users who encrypt data and those who decrypt it are equally trusted\.

**AES\-GCM without Key Derivation or Signing**  
This algorithm suite uses the data encryption key as the AES\-GCM encryption key, instead of using a key derivation function to derive a unique key\. We discourage using this suite to generate ciphertext, but the SDK supports it for compatibility reasons\.

For more information about how these suites are represented and used in the library, see [AWS Encryption SDK Algorithms Reference](algorithms-reference.md)\.