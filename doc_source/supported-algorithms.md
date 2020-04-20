# Supported algorithm suites in the AWS Encryption SDK<a name="supported-algorithms"></a>

An *algorithm suite* is a collection of cryptographic algorithms and related values\. Cryptographic systems use the algorithm implementation to generate the ciphertext message\.

The AWS Encryption SDK algorithm suite uses the Advanced Encryption Standard \(AES\) algorithm in Galois/Counter Mode \(GCM\), known as AES\-GCM, to encrypt raw data\. The SDK supports 256\-bit, 192\-bit, and 128\-bit encryption keys\. The length of the initialization vector \(IV\) is always 12 bytes\. The length of the authentication tag is always 16 bytes\.

The SDK supports several different implementations of AES\-GCM\. By default, the SDK uses AES\-GCM with an HMAC\-based extract\-and\-expand key derivation function \([HKDF](https://en.wikipedia.org/wiki/HKDF)\), signing, and a 256\-bit encryption key\.

## Recommended: AES\-GCM with key derivation and signing<a name="recommended-algorithms"></a>

In the recommended algorithm suite, the SDK uses the data encryption key as an input to the HMAC\-based extract\-and\-expand key derivation function \(HKDF\) to derive the AES\-GCM encryption key\. The SDK also adds an Elliptic Curve Digital Signature Algorithm \(ECDSA\) signature\. By default, the SDK uses this algorithm suite with a 256\-bit encryption key\.

The HKDF helps you avoid accidental reuse of a data encryption key\. 

This algorithm suite uses ECDSA and a message signing algorithm \(SHA\-384 or SHA\-256\)\. ECDSA is used by default, even when it is not specified by the policy for the underlying master key\. Message signing verifies the identity of the message sender and adds message authenticity to the envelope encrypted data\. It is particularly useful when the authorization policy for a master key allows one set of users to encrypt data and a different set of users to decrypt data\. 

The following table lists the variations of the recommended algorithm suites\.


**AWS Encryption SDK Algorithm Suites**  

| Algorithm name | Data encryption key length \(in bits\) | Algorithm mode | Key derivation algorithm | Signature algorithm | 
| --- | --- | --- | --- | --- | 
| AES | 256 | GCM | HKDF with SHA\-384 | ECDSA with P\-384 and SHA\-384 | 
| AES | 192 | GCM | HKDF with SHA\-384 | ECDSA with P\-384 and SHA\-384 | 
| AES | 128 | GCM | HKDF with SHA\-256 | ECDSA with P\-256 and SHA\-256 | 

## Other supported algorithm suites<a name="other-algorithms"></a>

The AWS Encryption SDK supports the following alternate algorithm suites for backward compatibility\. In general, we do not recommend these algorithm suites\. However, we recognize that using an algorithm suite with key derivation, but without signing, is appropriate in some cases\. We discourage the use of any algorithm suite that lacks both key derivation and signing\.

**AES\-GCM with key derivation only**  
This algorithm suite uses a key derivation function, but lacks the ECDSA signature that provides authenticity and non\-repudiation\. Use this suite when the users who encrypt data and those who decrypt it are equally trusted\.

**AES\-GCM without key derivation or signing**  
This algorithm suite uses the data encryption key as the AES\-GCM encryption key, instead of using a key derivation function to derive a unique key\. We discourage using this suite to generate ciphertext, but the SDK supports it for compatibility reasons\.

For more information about how these suites are represented and used in the library, see [AWS Encryption SDK algorithms reference](algorithms-reference.md)\.