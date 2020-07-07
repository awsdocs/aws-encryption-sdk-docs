# Compatibility of the AWS Encryption SDK for JavaScript<a name="javascript-compatibility"></a>

The AWS Encryption SDK for JavaScript is designed to be interoperable with other language implementations of the AWS Encryption SDK\. In most cases, you can encrypt data with the AWS Encryption SDK for JavaScript and decrypt it with any other language implementation, including the [AWS Encryption SDK Command Line Interface](crypto-cli.md)\. And you can use the AWS Encryption SDK for JavaScript to decrypt [encrypted messages](concepts.md#message) produced by other language implementations of the AWS Encryption SDK\.

However, when you use the AWS Encryption SDK for JavaScript, you need to be aware of some compatibility issues in the JavaScript language implementation and in web browsers\.

Also, when using different language implementations, be sure to configure compatible master key providers, master keys, and keyrings\. For details, see [Keyring compatibility](choose-keyring.md#keyring-compatibility)\.

## AWS Encryption SDK for JavaScript compatibility<a name="javascript-language-compatibility"></a>

The JavaScript implementation of the AWS Encryption SDK differs from other language implementations in the following ways:
+ The encrypt operation of the AWS Encryption SDK for JavaScript doesn't return nonframed ciphertext\. However, the AWS Encryption SDK for JavaScript will decrypt framed and nonframed ciphertext returned by other language implementations of the AWS Encryption SDK\.
+ Beginning in Node\.js version 12\.9\.0, Node\.js supports the following RSA key wrapping options:
  + OAEP with SHA1, SHA256, SHA384, or SHA512
  + OAEP with SHA1 and MGF1 with SHA1
  + PKCS1v15
+ Before version 12\.9\.0, Node\.js supports only the following RSA key wrapping options:
  + OAEP with SHA1 and MGF1 with SHA1
  + PKCS1v15

## Browser compatibility<a name="javascript-browser-compatibility"></a>

Some web browsers don't support basic cryptographic operations that the AWS Encryption SDK for JavaScript requires\. You can compensate for some of the missing operations by configuring a fallback for the WebCrypto API that the browser implements\.

**Web browser limitations**

The following limitations are common to all web browsers:
+ The WebCrypto API doesn't support PKCS1v15 key wrapping\.
+ Browsers don't support 192\-bit keys\.

**Required cryptographic operations**

The AWS Encryption SDK for JavaScript requires the following operations in web browsers\. If a browser doesn't support these operations, it's incompatible with the AWS Encryption SDK for JavaScript\.
+ The browser must include `crypto.getRandomValues()`, which is a method for generating cryptographically random values\. For information about the web browser versions that support `crypto.getRandomValues()`, see [Can I Use crypto\.getRandomValues\(\)?](https://caniuse.com/#feat=getrandomvalues)\.

**Required fallback**

The AWS Encryption SDK for JavaScript requires the following libraries and operations in web browsers\. If you support a web browser that doesn't fulfill these requirements, you must configure a fallback\. Otherwise, attempts to use the AWS Encryption SDK for JavaScript with the browser will fail\.
+ The WebCrypto API, which performs basic cryptographic operations in web applications, isn't available for all browsers\. For information about the web browser versions that support web cryptography, see [Can I Use Web Cryptography?](https://caniuse.com/#feat=cryptography)\.
+ Modern versions of the Safari web browser don't support AES\-GCM encryption of zero bytes, which the AWS Encryption SDK requires\. If the browser implements the WebCrypto API, but can't use AES\-GCM to encrypt zero bytes, the AWS Encryption SDK for JavaScript uses the fallback library only for zero\-byte encryption\. It uses the WebCrypto API for all other operations\.

To configure a fallback for either limitation, add the following statements to your code\. In the [configureFallback](https://github.com/aws/aws-encryption-sdk-javascript/blob/master/modules/web-crypto-backend/src/backend-factory.ts#L78) function, specify a library that supports the missing features\. The following example uses the Microsoft Research JavaScript Cryptography Library \(`msrcrypto`\), but you can replace it with a compatible library\. For a complete example, see [fallback\.ts](https://github.com/aws/aws-encryption-sdk-javascript/blob/master/modules/example-browser/src/fallback.ts)\.

```
import { configureFallback } from '@aws-crypto/client-browser'
configureFallback(msrCrypto)
```