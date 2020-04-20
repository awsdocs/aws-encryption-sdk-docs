# Modules in the AWS Encryption SDK for JavaScript<a name="javascript-modules"></a>

The modules in the AWS Encryption SDK for JavaScript make it easy to install the code that you need for your projects\.

## Modules for JavaScript Node\.js<a name="jsn-modules-node"></a>

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
Exports functions that support AWS KMS keyrings in Node\.js\.

[raw\-aes\-keyring\-node](https://github.com/aws/aws-encryption-sdk-javascript/tree/master/modules/raw-aes-keyring-node)  
Exports functions that support [Raw AES keyrings](choose-keyring.md#use-raw-aes-keyring) in Node\.js\.

[raw\-rsa\-keyring\-node](https://github.com/aws/aws-encryption-sdk-javascript/tree/master/modules/raw-rsa-keyring-node)  
Exports functions that support [Raw RSA keyrings](choose-keyring.md#use-raw-rsa-keyring) in Node\.js\.

## Modules for JavaScript Browser<a name="jsn-modules-browser"></a>

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
Exports functions that support [AWS KMS keyrings](choose-keyring.md#use-kms-keyring) in the browser\.

[raw\-aes\-keyring\-browser](https://github.com/aws/aws-encryption-sdk-javascript/tree/master/modules/raw-aes-keyring-browser)  
Exports functions that support [Raw AES keyrings](choose-keyring.md#use-raw-aes-keyring) in the browser\.

[raw\-rsa\-keyring\-browser](https://github.com/aws/aws-encryption-sdk-javascript/tree/master/modules/raw-rsa-keyring-browser)  
Exports functions that support [Raw RSA keyrings](choose-keyring.md#use-raw-rsa-keyring) in the browser\.

## Modules for all implementations<a name="jsn-modules-all"></a>

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