# AWS Encryption SDK for JavaScript<a name="javascript"></a>

The AWS Encryption SDK for JavaScript is designed to provide a client\-side encryption library for developers who are writing web browser applications in JavaScript or web server applications in Node\.js\.

Like all implementations of the AWS Encryption SDK, the AWS Encryption SDK for JavaScript offers advanced data protection features\. These include [envelope encryption](https://docs.aws.amazon.com/crypto/latest/userguide/cryptography-concepts.html#define-envelope-encryption), [additional authenticated data](https://docs.aws.amazon.com/crypto/latest/userguide/cryptography-concepts.html#term-aad) \(AAD\), and secure, authenticated, symmetric key [algorithm suites](concepts.md#crypto-algorithm), such as 256\-bit AES\-GCM with key derivation and signing\.

All language\-specific implementations of the AWS Encryption SDK are designed to be interoperable, subject to the constraints of the language\. For details about language constraints for JavaScript, see [Compatibility of the AWS Encryption SDK for JavaScript](#javascript-compatibility)\.

**Learn More**
+ For details about programming with the AWS Encryption SDK for JavaScript, see the [aws\-encryption\-sdk\-javascript](https://github.com/aws/aws-encryption-sdk-javascript/) repository on GitHub\.

   
+ For programming examples, see [AWS Encryption SDK for JavaScript Examples](js-examples.md) and the [example\-browser](https://github.com/aws/aws-encryption-sdk-javascript/tree/master/modules/example-browser) and [example\-node](https://github.com/aws/aws-encryption-sdk-javascript/tree/master/modules/example-node) modules in the [aws\-encryption\-sdk\-javascript](https://github.com/aws/aws-encryption-sdk-javascript/) repository\. 

   
+  For a real\-world example of using the AWS Encryption SDK for JavaScript to encrypt data in a web application, see [How to enable encryption in a browser with the AWS Encryption SDK for JavaScript and Node\.js](http://aws.amazon.com/blogs/security/how-to-enable-encryption-browser-aws-encryption-sdk-javascript-node-js/) in the AWS Security Blog\.

**Topics**
+ [Compatibility](#javascript-compatibility)
+ [Installation](#javascript-installation)
+ [Modules](#javascript-modules)
+ [Examples](js-examples.md)

## Compatibility of the AWS Encryption SDK for JavaScript<a name="javascript-compatibility"></a>

The AWS Encryption SDK for JavaScript is designed to be interoperable with other language implementations of the AWS Encryption SDK\. In most cases, you can encrypt data with the AWS Encryption SDK for JavaScript and decrypt it with any other language implementation, including the [AWS Encryption SDK Command Line Interface](crypto-cli.md)\. And you can use the AWS Encryption SDK for JavaScript to decrypt [encrypted messages](concepts.md#message) produced by other language implementations of the AWS Encryption SDK\.

However, when you use the AWS Encryption SDK for JavaScript, you need to be aware of some compatibility issues in the JavaScript language implementation and in web browsers\.

Also, when using different language implementations, be sure to configure compatible master key providers, master keys, and keyrings\. For details, see [Keyring Compatibility](choose-keyring.md#keyring-compatibility)\.

### AWS Encryption SDK for JavaScript Compatibility<a name="javascript-language-compatibility"></a>

The JavaScript implementation of the AWS Encryption SDK differs from other language implementations in the following ways:
+ The encrypt operation of the AWS Encryption SDK for JavaScript doesn't return nonframed ciphertext\. However, the AWS Encryption SDK for JavaScript will decrypt framed and nonframed ciphertext returned by other language implementations of the AWS Encryption SDK\.

   
+ Node\.js supports only the following RSA key wrapping options:
  + OAEP with SHA1 and MGF1 with SHA1
  + PKCS1v15

### Browser Compatibility<a name="javascript-browser-compatibility"></a>

Some web browsers don't support basic cryptographic operations that the AWS Encryption SDK for JavaScript requires\. You can compensate for some of the missing operations by configuring a fallback for the WebCrypto API that the browser implements\.

**Web Browser Limitations**

The following limitations are common to all web browsers:
+ The WebCrypto API doesn't support PKCS1v15 key wrapping\.
+ Browsers don't support 192\-bit keys\.

**Required Cryptographic Operations**

The AWS Encryption SDK for JavaScript requires the following operations in web browsers\. If a browser doesn't support these operations, it's incompatible with the AWS Encryption SDK for JavaScript\.
+ The browser must include `crypto.getRandomValues()`, which is a method for generating cryptographically random values\. For information about the web browser versions that support `crypto.getRandomValues()`, see [Can I Use crypto\.getRandomValues\(\)?](https://caniuse.com/#feat=getrandomvalues)\.

**Required Fallback**

The AWS Encryption SDK for JavaScript requires the following libraries and operations in web browsers\. If you support a web browser that doesn't fulfill these requirements, you must configure a fallback\. Otherwise, attempts to use the AWS Encryption SDK for JavaScript with the browser will fail\.
+ The WebCrypto API, which performs basic cryptographic operations in web applications, isn't available for all browsers\. For information about the web browser versions that support web cryptography, see [Can I Use Web Cryptography?](https://caniuse.com/#feat=cryptography)\.

   
+ Modern versions of the Safari web browser don't support AES\-GCM encryption of zero bytes, which the AWS Encryption SDK requires\. If the browser implements the WebCrypto API, but can't use AES\-GCM to encrypt zero bytes, the AWS Encryption SDK for JavaScript uses the fallback library only for zero\-byte encryption\. It uses the WebCrypto API for all other operations\.

To configure a fallback for either limitation, add the following statements to your code\. In the [configureFallback](https://github.com/aws/aws-encryption-sdk-javascript//blob/master/modules/web-crypto-backend/src/backend-factory.ts#L78) function, specify a library that supports the missing features\. The following example uses the Microsoft Research JavaScript Cryptography Library \(`msrcrypto`\), but you can replace it with a compatible library\.

```
import { configureFallback } from '@aws-crypto/client-browser'
configureFallback(msrCrypto)
```

## Installing the AWS Encryption SDK for JavaScript<a name="javascript-installation"></a>

The AWS Encryption SDK for JavaScript consists of a collection of interdependent modules\. Several of the modules are just collections of modules that are designed to work together\. Some modules are designed to work independently\. A few modules are required for all implementations; a few others are required only for special cases\. For information about the modules in the AWS Encryption SDK for JavaScript, see [Modules in the AWS Encryption SDK for JavaScript](#javascript-modules) and the `Readme.md` file in each of the modules in the [aws\-encryption\-sdk\-javascript](https://github.com/aws/aws-encryption-sdk-javascript/tree/master/modules) repository on GitHub\.

To install the modules, use the [npm package manager](https://www.npmjs.com/get-npm)\. 

For example, to install the `client-node` module, which includes all of the modules you need to program with the AWS Encryption SDK for JavaScript in Node\.js, use the following command\. 

```
npm install @aws-crypto/client-node
```

To install the `client-browser` module, which includes all of the modules you need to program with the AWS Encryption SDK for JavaScript in the browser, use the following command\. 

```
npm install @aws-crypto/client-browser
```

For working examples of how to use the AWS Encryption SDK for JavaScript, see the examples in the `example-node` and `example-browser` modules in the [aws\-encryption\-sdk\-javascript](https://github.com/aws/aws-encryption-sdk-javascript/) repository on GitHub\.

## Modules in the AWS Encryption SDK for JavaScript<a name="javascript-modules"></a>

The modules in the AWS Encryption SDK for JavaScript make it easy to install the code that you need for your projects\.

### Modules for JavaScript Node\.js<a name="jsn-modules-node"></a>

[client\-node](https://github.com/aws/aws-encryption-sdk-javascript/tree/master/modules/client-node)  
Includes all of the modules you need to program with the AWS Encryption SDK for JavaScript in Node\.js\.

[caching\-materials\-manager\-node](https://github.com/aws/aws-encryption-sdk-javascript/tree/master/modules/caching-materials-manager-node)  
Exports functions that support the [data key caching](data-key-caching.md) feature in the AWS Encryption SDK for JavaScript in Node\.js\. 

[decrypt\-node](https://github.com/aws/aws-encryption-sdk-javascript/tree/master/modules/decrypt-node)  
Exports functions that decrypt and verify encrypted messages representing data and data streams\. Included in the `client-node` module\.

[encrypt\-node](https://github.com/aws/aws-encryption-sdk-javascript/tree/master/modules/encrypt-node)  
Exports functions that encrypt and sign different types of data\. Included in the `client-node` module\.

[example\-node](https://github.com/aws/aws-encryption-sdk-javascript/tree/master/modules/example-node)  
Exports working examples of programming with the AWS Encryption SDK for JavaScript in Node\.js\. Includes example of different types of keyrings and different types of data\.

[hkdf\-node](https://github.com/aws/aws-encryption-sdk-javascript/tree/master/modules/hkdf-node)  
Exports an [HMAC\-based Key Derivation Function](https://en.wikipedia.org/wiki/HKDF) \(HKDF\) that the AWS Encryption SDK for JavaScript in Node\.js uses in particular algorithm suites\. The AWS Encryption SDK for JavaScript in the browser uses the native HKDF function in the WebCrypto API\.

[integration\-node](https://github.com/aws/aws-encryption-sdk-javascript/tree/master/modules/integration-node)  
Defines tests that verify that the AWS Encryption SDK for JavaScript in Node\.js is compatible with other language implementations of the AWS Encryption SDK\.

[kms\-keyring\-node](https://github.com/aws/aws-encryption-sdk-javascript/tree/master/modules/kms-keyring-node)  
Exports functions that support KMS keyrings in Node\.js\.

[raw\-aes\-keyring\-node](https://github.com/aws/aws-encryption-sdk-javascript/tree/master/modules/raw-aes-keyring-node)  
Exports functions that support [Raw AES keyrings](choose-keyring.md#use-raw-aes-keyring) in Node\.js\.

[raw\-rsa\-keyring\-node](https://github.com/aws/aws-encryption-sdk-javascript/tree/master/modules/raw-rsa-keyring-node)  
Exports functions that support [Raw RSA keyrings](choose-keyring.md#use-raw-rsa-keyring) in Node\.js\.

### Modules for JavaScript Browser<a name="jsn-modules-browser"></a>

[client\-browser](https://github.com/aws/aws-encryption-sdk-javascript/tree/master/modules/client-browser)  
Includes all of the modules you need to program with the AWS Encryption SDK for JavaScript in the browser\.

[caching\-materials\-manager\-browser](https://github.com/aws/aws-encryption-sdk-javascript/tree/master/modules/caching-materials-manager-browser)  
Exports functions that support the [data key caching](data-key-caching.md) feature for JavaScript in the browser\.

[decrypt\-browser](https://github.com/aws/aws-encryption-sdk-javascript/tree/master/modules/decrypt-browser)  
Exports functions that decrypt and verify encrypted messages representing data and data streams\.

[encrypt\-browser](https://github.com/aws/aws-encryption-sdk-javascript/tree/master/modules/encrypt-browser)  
Exports functions that encrypt and sign different types of data\. 

[example\-browser](https://github.com/aws/aws-encryption-sdk-javascript/tree/master/modules/example-browser)  
Working examples of programming with the AWS Encryption SDK for JavaScript in the browser\. Includes examples of different types of keyrings and different types of data\.

[integration\-browser](https://github.com/aws/aws-encryption-sdk-javascript/tree/master/modules/integration-browser)  
Defines tests that verify that the AWS Encryption SDK for JavaScript in the browser is compatible with other language implementations of the AWS Encryption SDK\.

[kms\-keyring\-browser](https://github.com/aws/aws-encryption-sdk-javascript/tree/master/modules/kms-keyring-browser)  
Exports functions that support [KMS keyrings](choose-keyring.md#use-kms-keyring) in the browser\.

[raw\-aes\-keyring\-browser](https://github.com/aws/aws-encryption-sdk-javascript/tree/master/modules/raw-aes-keyring-browser)  
Exports functions that support [Raw AES keyrings](choose-keyring.md#use-raw-aes-keyring) in the browser\.

[raw\-rsa\-keyring\-browser](https://github.com/aws/aws-encryption-sdk-javascript/tree/master/modules/raw-rsa-keyring-browser)  
Exports functions that support [Raw RSA keyrings](choose-keyring.md#use-raw-rsa-keyring) in the browser\.

### Modules for All Implementations<a name="jsn-modules-all"></a>

[cache\-material](https://github.com/aws/aws-encryption-sdk-javascript/tree/master/modules/cache-material)  
Supports the [data key caching](data-key-caching.md) feature\. Provides code for assembling the cryptographic materials that are cached with each data key\.

[kms\-keyring](https://github.com/aws/aws-encryption-sdk-javascript/tree/master/modules/kms-keyring)  
Exports functions that support [KMS keyrings](choose-keyring.md#use-kms-keyring)\.

[material\-management](https://github.com/aws/aws-encryption-sdk-javascript/tree/master/modules/material-management)  
Implements the [cryptographic materials manager](concepts.md#crypt-materials-manager) \(CMM\)\.

[raw\-keyring](https://github.com/aws/aws-encryption-sdk-javascript/tree/master/modules/raw-keyring)  
Exports functions required for raw AES and RSA keyrings\.

[serialize](https://github.com/aws/aws-encryption-sdk-javascript/tree/master/modules/serialize)  
Exports functions that the SDK uses to serialize its output\.

[web\-crypto\-backend](https://github.com/aws/aws-encryption-sdk-javascript/tree/master/modules/web-crypto-backend)  
Exports functions that use the WebCrypto API in the AWS Encryption SDK for JavaScript in the browser\.