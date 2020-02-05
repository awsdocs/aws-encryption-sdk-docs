# Installing the AWS Encryption SDK for JavaScript<a name="javascript-installation"></a>

The AWS Encryption SDK for JavaScript consists of a collection of interdependent modules\. Several of the modules are just collections of modules that are designed to work together\. Some modules are designed to work independently\. A few modules are required for all implementations; a few others are required only for special cases\. For information about the modules in the AWS Encryption SDK for JavaScript, see [Modules in the AWS Encryption SDK for JavaScript](javascript-modules.md) and the `Readme.md` file in each of the modules in the [aws\-encryption\-sdk\-javascript](https://github.com/aws/aws-encryption-sdk-javascript/tree/master/modules) repository on GitHub\.

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