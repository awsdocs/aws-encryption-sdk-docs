# AWS Encryption SDK for JavaScript examples<a name="js-examples"></a>

The following examples show you how to use the AWS Encryption SDK for JavaScript to encrypt and decrypt data\. 

You can find more examples of using the AWS Encryption SDK for JavaScript in the [example\-node](https://github.com/aws/aws-encryption-sdk-javascript/tree/master/modules/example-node) and [example\-browser](https://github.com/aws/aws-encryption-sdk-javascript/tree/master/modules/example-browser) modules in the [aws\-encryption\-sdk\-javascript](https://github.com/aws/aws-encryption-sdk-javascript/) repository on GitHub\. These example modules are not installed when you install the `client-browser` or `client-node` modules\.

**See the complete code samples**: Node: [kms\_simple\.ts](https://github.com/aws/aws-encryption-sdk-javascript/blob/master/modules/example-node/src/kms_simple.ts), Browser: [kms\_simple\.ts](https://github.com/aws/aws-encryption-sdk-javascript/blob/master/modules/example-browser/src/kms_simple.ts)

**Topics**
+ [Encrypting data with an AWS KMS keyring](#javascript-example-encrypt)
+ [Decrypting data with an AWS KMS keyring](#javascript-example-decrypt)

## Encrypting data with an AWS KMS keyring<a name="javascript-example-encrypt"></a>

The following example shows you how to use the AWS Encryption SDK for JavaScript to encrypt and decrypt a short string or byte array\. 

This example features an [AWS KMS keyring](choose-keyring.md#use-kms-keyring), a type of keyring that uses an [AWS Key Management Service \(AWS KMS\)](https://aws.amazon.com/kms/) customer master key \(CMK\) to generate and encrypt data keys\. For help creating a CMK, see [Creating Keys](https://docs.aws.amazon.com/kms/latest/developerguide/create-keys.html) in the *AWS Key Management Service Developer Guide*\. For help identifying CMKs in an AWS KMS keyring, see [Identifying CMKs in an AWS KMS keyring](choose-keyring.md#kms-keyring-id)

Step 1: Construct the keyring\.  
Create an AWS KMS keyring for encryption\.   
When encrypting with an AWS KMS keyring, you must specify a *generator key*, that is, an AWS KMS CMK that is used to generate the plaintext data key and encrypt it\. You can also specify zero or more *additional keys* that encrypt the same plaintext data key\. The keyring returns the plaintext data key and one encrypted copy of that data key for each CMK in the keyring, including the generator key\. To decrypt the data, you need to decrypt any one of the encrypted data keys\.  
To specify the CMKs for an encryption keyring in the AWS Encryption SDK for JavaScript, you can use [any supported AWS KMS key identifier](choose-keyring.md#kms-keyring-id)\. This example uses a generator key, which is identified by its [alias ARN](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#key-id-alias-ARN), and one additional key, which is identified by a [key ARN](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#key-id-key-ARN)\.  
If you plan to reuse your AWS KMS keyring for decrypting, you must use key ARNs to identify the CMKs in the keyring\.
Before running this code, replace the example CMK identifiers with valid identifiers\. You must have the [permissions required to use the CMKs](choose-keyring.md#kms-keyring-permissions) in the keyring\.  
Begin by providing your credentials to the browser\. The AWS Encryption SDK for JavaScript examples use the [webpack\.DefinePlugin](https://webpack.js.org/plugins/define-plugin/), which replaces the credential constants with your actual credentials\. But you can use any method to provide your credentials\. Then, use the credentials to create an AWS KMS client\.  

```
declare const credentials: {accessKeyId: string, secretAccessKey:string, sessionToken:string }

const clientProvider = getClient(KMS, {
  credentials: {
    accessKeyId,
    secretAccessKey,
    sessionToken
  }
})
```
Next, specify the AWS KMS CMKs for the generator key and additional key\. Then, create an AWS KMS keyring using the AWS KMS client and the CMKs\.  

```
const generatorKeyId = 'arn:aws:kms:us-west-2:111122223333:alias/EncryptDecrypt'
const keyIds = ['arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab']

const keyring = new KmsKeyringBrowser({ clientProvider, generatorKeyId, keyIds })
```

```
const generatorKeyId = 'arn:aws:kms:us-west-2:111122223333:alias/EncryptDecrypt'
const keyIds = ['arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab']

const keyring = new KmsKeyringNode({ generatorKeyId, keyIds })
```

Step 2: Set the encryption context\.  
An [encryption context](concepts.md#encryption-context) is arbitrary, non\-secret additional authenticated data\. When you provide an encryption context on encrypt, the AWS Encryption SDK cryptographically binds the encryption context to the ciphertext so that the same encryption context is required to decrypt the data\. Using an encryption context is optional, but we recommend it as a best practice\.  
Create a simple object that includes the encryption context pairs\. The key and value in each pair must be a string\.  

```
const context = {
  stage: 'demo',
  purpose: 'simple demonstration app',
  origin: 'us-west-2'
}
```

```
const context = {
  stage: 'demo',
  purpose: 'simple demonstration app',
  origin: 'us-west-2'
}
```

Step 3: Encrypt the data\.  
To encrypt the plaintext data, call the `encrypt` function\. Pass in the AWS KMS keyring, the plaintext data, and the encryption context\.  
The `encrypt` function returns an [encrypted message](concepts.md#message) \(`result`\) that contains the encrypted data, the encrypted data keys, and important metadata, including the encryption context and signature\.  
You can [decrypt this encrypted message](#javascript-example-decrypt) by using the AWS Encryption SDK for any supported programming language\.  

```
const plaintext = new Uint8Array([1, 2, 3, 4, 5])

const { result } = await encrypt(keyring, plaintext, { encryptionContext: context })
```

```
const plaintext = 'asdf'

const { result } = await encrypt(keyring, plaintext, { encryptionContext: context })
```

## Decrypting data with an AWS KMS keyring<a name="javascript-example-decrypt"></a>

You can use the AWS Encryption SDK for JavaScript to decrypt the encrypted message and recover the original data\.

In this example, we decrypt the data that we encrypted in the [Encrypting data with an AWS KMS keyring](#javascript-example-encrypt) example\.

Step 1: Construct the keyring\.  
To decrypt the data, pass in the [encrypted message](concepts.md#message) \(`result`\) that the `encrypt` function returned\. The encrypted message includes the encrypted data, the encrypted data keys, and important metadata, including the encryption context and signature\.  
You must also specify an [AWS KMS keyring](choose-keyring.md#use-kms-keyring) when decrypting\. You can use the same keyring that was used to encrypt the data or a different keyring\. To succeed, at least one CMK in the decryption keyring must be able to decrypt one of the encrypted data keys in the encrypted message\. Because no data keys are generated, you do not need to specify a generator key in a decryption keyring\. If you do, the generator key and additional keys are treated the same way\.  
To specify a CMK for a decryption keyring in the AWS Encryption SDK for JavaScript, you must use the [key ARN](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#key-id-key-ARN)\. Otherwise, the CMK is not recognized\. For help identifying CMKs in an AWS KMS keyring, see [Identifying CMKs in an AWS KMS keyring](choose-keyring.md#kms-keyring-id)  
If you use the same keyring for encrypting and decrypting, use key ARNs to identify the CMKs in the keyring\.
In this example, we create a keyring that includes only one of the CMKs in the encryption keyring\. Before running this code, replace the example key ARN with a valid one\. You must have `kms:Decrypt` permission on the CMK\.  
Begin by providing your credentials to the browser\. The AWS Encryption SDK for JavaScript examples use the [webpack\.DefinePlugin](https://webpack.js.org/plugins/define-plugin/), which replaces the credential constants with your actual credentials\. But you can use any method to provide your credentials\. Then, use the credentials to create an AWS KMS client\.  

```
declare const credentials: {accessKeyId: string, secretAccessKey:string, sessionToken:string }

const clientProvider = getClient(KMS, {
  credentials: {
    accessKeyId,
    secretAccessKey,
    sessionToken
  }
})
```
Next, create an AWS KMS keyring using the AWS KMS client\. This example uses just one of the CMKs from the encryption keyring\.  

```
const keyIds = ['arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab']

const keyring = new KmsKeyringBrowser({ clientProvider, keyIds })
```

```
const keyIds = ['arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab']

const keyring = new KmsKeyringNode({ keyIds })
```

Step 2: Decrypt the data\.  
Next, call the `decrypt` function\. Pass in the decryption keyring that you just created \(`keyring`\) and the [encrypted message](concepts.md#message) that the `encrypt` function returned \(`result`\)\. The AWS Encryption SDK uses the keyring to decrypt one of the encrypted data keys\. Then it uses the plaintext data key to decrypt the data\.  
If the call succeeds, the `plaintext` field contains the plaintext \(decrypted\) data\. The `messageHeader` field contains metadata about the decryption process, including the encryption context that was used to decrypt the data\.  

```
const { plaintext, messageHeader } = await decrypt(keyring, result)
```

```
const { plaintext, messageHeader } = await decrypt(keyring, result)
```

Step 3: Verify the encryption context\.  
The [encryption context](concepts.md#encryption-context) that was used to decrypt the data is included in the message header \(`messageHeader`\) that the `decrypt` function returns\. Before your application returns the plaintext data, verify that the encryption context that you provided when encrypting is included in the encryption context that was used when decrypting\. A mismatch might indicate that the data was tampered with, or that you didn't decrypt the right ciphertext\.  
When verifying the encryption context, do not require an exact match\. When you use an encryption algorithm with signing, the [cryptographic materials manager](concepts.md#crypt-materials-manager) \(CMM\) adds the public signing key to the encryption context before encrypting the message\. But all of the encryption context pairs that you submitted should be included in the encryption context that was returned\.  
First, get the encryption context from the message header\. Then, verify that each key\-value pair in the original encryption context \(`context`\) matches a key\-value pair in the returned encryption context \(`encryptionContext`\)\.  

```
const { encryptionContext } = messageHeader

Object
  .entries(context)
  .forEach(([key, value]) => {
    if (encryptionContext[key] !== value) throw new Error('Encryption Context does not match expected values')
})
```

```
const { encryptionContext } = messageHeader

Object
  .entries(context)
  .forEach(([key, value]) => {
    if (encryptionContext[key] !== value) throw new Error('Encryption Context does not match expected values')
})
```
If the encryption context check succeeds, you can return the plaintext data\.