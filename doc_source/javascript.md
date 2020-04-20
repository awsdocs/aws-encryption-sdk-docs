# AWS Encryption SDK for JavaScript<a name="javascript"></a>

The AWS Encryption SDK for JavaScript is designed to provide a client\-side encryption library for developers who are writing web browser applications in JavaScript or web server applications in Node\.js\.

Like all implementations of the AWS Encryption SDK, the AWS Encryption SDK for JavaScript offers advanced data protection features\. These include [envelope encryption](https://docs.aws.amazon.com/crypto/latest/userguide/cryptography-concepts.html#define-envelope-encryption), [additional authenticated data](https://docs.aws.amazon.com/crypto/latest/userguide/cryptography-concepts.html#term-aad) \(AAD\), and secure, authenticated, symmetric key [algorithm suites](concepts.md#crypto-algorithm), such as 256\-bit AES\-GCM with key derivation and signing\.

All language\-specific implementations of the AWS Encryption SDK are designed to be interoperable, subject to the constraints of the language\. For details about language constraints for JavaScript, see [Compatibility of the AWS Encryption SDK for JavaScript](javascript-compatibility.md)\.

**Learn More**
+ For details about programming with the AWS Encryption SDK for JavaScript, see the [aws\-encryption\-sdk\-javascript](https://github.com/aws/aws-encryption-sdk-javascript/) repository on GitHub\.
+ For programming examples, see [AWS Encryption SDK for JavaScript examples](js-examples.md) and the [example\-browser](https://github.com/aws/aws-encryption-sdk-javascript/tree/master/modules/example-browser) and [example\-node](https://github.com/aws/aws-encryption-sdk-javascript/tree/master/modules/example-node) modules in the [aws\-encryption\-sdk\-javascript](https://github.com/aws/aws-encryption-sdk-javascript/) repository\. 
+  For a real\-world example of using the AWS Encryption SDK for JavaScript to encrypt data in a web application, see [How to enable encryption in a browser with the AWS Encryption SDK for JavaScript and Node\.js](http://aws.amazon.com/blogs/security/how-to-enable-encryption-browser-aws-encryption-sdk-javascript-node-js/) in the AWS Security Blog\.

**Topics**
+ [Compatibility](javascript-compatibility.md)
+ [Installation](javascript-installation.md)
+ [Modules](javascript-modules.md)
+ [Examples](js-examples.md)