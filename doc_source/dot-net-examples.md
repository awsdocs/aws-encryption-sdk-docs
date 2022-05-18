# AWS Encryption SDK for \.NET examples<a name="dot-net-examples"></a>

The following examples show the basic coding patterns that you use when programming with the AWS Encryption SDK for \.NET\. Specifically, you instantiate the AWS Encryption SDK and the material providers library\. Then, before calling each method, you instantiate an object that defines the input for the method\. This is much like the coding pattern you use in the AWS SDK for \.NET\.

For examples showing how to configure options in the AWS Encryption SDK, such as specifying an alternate algorithm suite, limiting encrypted data keys, and using AWS KMS multi\-Region keys, see [Configuring the AWS Encryption SDK](configure.md)\.

For more examples of programming with the AWS Encryption SDK for \.NET, see the [examples](https://github.com/aws/aws-encryption-sdk-dafny/blob/mainline/aws-encryption-sdk-net/Examples) in the `aws-encryption-sdk-net` directory of the `aws-encryption-sdk-dafny` repository on GitHub\.

## Encrypting data in the AWS Encryption SDK for \.NET<a name="dot-net-example-encrypt"></a>

This example shows the basic pattern for encrypting data\. It encrypts a small file with data keys that are protected by one AWS KMS wrapping key\.

Step 1: Instantiate the AWS Encryption SDK and the material providers library\.  
Begin by instantiating the AWS Encryption SDK and the material providers library\. You'll use the methods in the AWS Encryption SDK to encrypt and decrypt data\. You'll use the methods in the material providers library to create the keyrings that specify which keys protect your data\.  

```
// Instantiate the AWS Encryption SDK and material providers
var encryptionSdk = AwsEncryptionSdkFactory.CreateDefaultAwsEncryptionSdk();
var materialProviders =
    AwsCryptographicMaterialProvidersFactory.CreateDefaultAwsCryptographicMaterialProviders();
```

Step 2: Create an input object for the keyring\.  
Each method that creates a keyring has a corresponding input object class\. For example, to create the input object for the `CreateAwsKmsKeyring()` method, create an instance of the `CreateAwsKmsKeyringInput` class\.  
Even though the input for this keyring doesn't specify a [generator key](use-kms-keyring.md#kms-keyring-encrypt), the single KMS key specified by the `KmsKeyId` parameter is the generator key\. It generates and encrypts the data key that encrypts the data\.   
This input object requires an AWS KMS client for the AWS Region of the KMS key\. To create a AWS KMS client, instantiate the `AmazonKeyManagementServiceClient` class in the AWS SDK for \.NET\. Calling the `AmazonKeyManagementServiceClient()` constructor with no parameters creates a client with the default values\.  
In an AWS KMS keyring used for encrypting with the AWS Encryption SDK for \.NET, you can [identify the KMS keys](use-kms-keyring.md#kms-keyring-id) by using the key ID, key ARN, alias name, or alias ARN\. In an AWS KMS keyring used for decrypting, you must use a key ARN to identify each KMS key\. If you plan to reuse your encryption keyring for decrypting, use a key ARN identifier for all KMS keys\.  

```
string keyArn = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";

// Instantiate the keyring input object
var kmsKeyringInput = new CreateAwsKmsKeyringInput
{    
    KmsClient = new AmazonKeyManagementServiceClient(),
    KmsKeyId = keyArn
};
```

Step 3: Create the keyring\.  
To create the keyring, call the keyring method with the keyring input object\. This example uses the `CreateAwsKmsKeyring()` method, which takes just one KMS key\.  

```
var keyring = materialProviders.CreateAwsKmsKeyring(kmsKeyringInput);
```

Step 4: Define an encryption context\.  
An [encryption context](concepts.md#encryption-context) is an optional, but strongly recommended element of cryptographic operations in the AWS Encryption SDK\. You can define one or more non\-secret key\-value pairs\.  

```
// Define the encryption context
var encryptionContext = new Dictionary<string, string>()
{
    {"purpose", "test"}
};
```

Step 5: Create the input object for encrypting\.  
Before calling the `Encrypt()` method, create an instance of the `EncryptInput` class\.  

```
string plaintext = File.ReadAllText("C:\\Documents\\CryptoTest\\TestFile.txt");
            
// Define the encrypt input
var encryptInput = new EncryptInput
{
    Plaintext = plaintext,
    Keyring = keyring,
    EncryptionContext = encryptionContext
};
```

Step 6: Encrypt the plaintext\.  
Use the `Encrypt()` method of the AWS Encryption SDK to encrypt the plaintext using the keyring you defined\.   
The `EncryptOutput` that the `Encrypt() `method returns has methods for getting the encrypted message \(`Ciphertext`\), encryption context, and algorithm suite\.   

```
var encryptOutput = encryptionSdk.Encrypt(encryptInput);
```

Step 7: Get the encrypted message\.  
The `Decrypt()` method in the AWS Encryption SDK for \.NET takes the `Ciphertext` member of the `EncryptOutput` instance\.  
The `Ciphertext` member of the `EncryptOutput` object is the [encrypted message](concepts.md#message), a portable object that includes the encrypted data, encrypted data keys, and metadata, including the encryption context\. You can safely store the encrypted message for an extended time or submit it to the `Decrypt()` method to recover the plaintext\.   

```
var encryptedMessage = encryptOutput.Ciphertext;
```

## Decrypting in strict mode in the AWS Encryption SDK for \.NET<a name="dot-net-decrypt-strict"></a>

Best practices recommend that you specify the keys that you use to decrypt data, an option known as *strict mode*\. The AWS Encryption SDK uses only the KMS keys that you specify in your keyring to decrypt the ciphertext\. The keys in your decryption keyring must include at least one of the keys that encrypted the data\.

This example shows the basic pattern of decrypting in strict mode with the AWS Encryption SDK for \.NET\.

Step 1: Instantiate the AWS Encryption SDK and material providers library\.  

```
// Instantiate the AWS Encryption SDK and material providers
var encryptionSdk = AwsEncryptionSdkFactory.CreateDefaultAwsEncryptionSdk();
var materialProviders =
    AwsCryptographicMaterialProvidersFactory.CreateDefaultAwsCryptographicMaterialProviders();
```

Step 2: Create the input object for your keyring\.  
To specify the parameters for the keyring method, create an input object\. Each keyring method in the AWS Encryption SDK for \.NET has a corresponding input object\. Because this example uses the `CreateAwsKmsKeyring()` method to create the keyring, it instantiates the `CreateAwsKmsKeyringInput` class for the input\.  
In a decryption keyring, you must use a key ARN to identify KMS keys\.  

```
string keyArn = "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab";

// Instantiate the keyring input object
var kmsKeyringInput = new CreateAwsKmsKeyringInput
{
    KmsClient = new AmazonKeyManagementServiceClient(),
    KmsKeyId = keyArn
};
```

Step 3: Create the keyring\.  
To create the decryption keyring, this example uses the `CreateAwsKmsKeyring()` method and the keyring input object\.  

```
var keyring = materialProviders.CreateAwsKmsKeyring(kmsKeyringInput);
```

Step 4: Create the input object for decrypting\.  
To create the input object for the `Decrypt()` method, instantiate the `DecryptInput` class\.  
The `Ciphertext` parameter of the `DecryptInput()` constructor takes the `Ciphertext` member of the `EncryptOutput` object that the `Encrypt()` method returned\. The `Ciphertext` property represents the [encrypted message](concepts.md#message), which includes the encrypted data, encrypted data keys, and metadata that the AWS Encryption SDK needs to decrypt the message\.  

```
var encryptedMessage = encryptOutput.Ciphertext;

var decryptInput = new DecryptInput
{
    Ciphertext = encryptedMessage,
    Keyring = keyring
};
```

Step 5: Decrypt the ciphertext\.  

```
var decryptOutput = encryptionSdk.Decrypt(decryptInput);
```

Step 6: Verify the encryption context\.  
The `Decrypt()` method of the AWS Encryption SDK does not take an encryption context\. It gets the encryption context values from the metadata in the encrypted message\. However, before returning or using the plaintext, it's a best practice to verify that encryption context that was used to decrypt the ciphertext includes the encryption context you provided when encrypting\.   
Verify that the encryption context used on encrypt *is included* in the encryption context that used to decrypt the ciphertext\. The AWS Encryption SDK adds pairs to the encryption context, including the digital signature if you're using an algorithm suite with signing, such as the default algorithm suite\.  

```
// Verify the encryption context
string contextKey = "purpose";
string contextValue = "test";

if (!decryptOutput.EncryptionContext.TryGetValue(contextKey, out var decryptContextValue)
    || !decryptContextValue.Equals(contextValue))
{
    throw new Exception("Encryption context does not match expected values");
}
```

## Decrypting with a discovery keyring in the AWS Encryption SDK for \.NET<a name="dot-net-decrypt-discovery"></a>

Rather than specifying the KMS keys for decryption, you can provide a AWS KMS *discovery keyring*, which is a keyring that doesn't specify any KMS keys\. A discovery keyring lets the AWS Encryption SDK decrypt the data by using whichever KMS key encrypted it, provided that the caller has decrypt permission on the key\. For best practices, add a discovery filter that limits the KMS keys that can be used to those in particular AWS accounts of a specified partition\. 

The AWS Encryption SDK for \.NET provides a basic discovery keyring that requires an AWS KMS client and a discovery multi\-keyring that requires you to specify one or more AWS Regions\. The client and Regions both limit the KMS keys that can be used to decrypt the encrypted message\. The input objects for both keyrings take the recommended discovery filter\.

The following example shows the pattern for decrypting data with an AWS KMS discovery keyring and discovery filter\.

Step 1: Instantiate the AWS Encryption SDK and the material providers library\.  

```
// Instantiate the AWS Encryption SDK and material providers
var encryptionSdk = AwsEncryptionSdkFactory.CreateDefaultAwsEncryptionSdk();
var materialProviders =
    AwsCryptographicMaterialProvidersFactory.CreateDefaultAwsCryptographicMaterialProviders();
```

Step 2: Create the input object for the keyring\.  
To specify the parameters for the keyring method, create an input object\. Each keyring method in the AWS Encryption SDK for \.NET has a corresponding input object\. Because this example uses the `CreateAwsKmsDiscoveryKeyring()` method to create the keyring, it instantiates the `CreateAwsKmsDiscoveryKeyringInput` class for the input\.  

```
List<string> accounts = new List<string> { "111122223333" };

var discoveryKeyringInput = new CreateAwsKmsDiscoveryKeyringInput
{
    KmsClient = new AmazonKeyManagementServiceClient(),
    DiscoveryFilter = new DiscoveryFilter()
    {
        AccountIds = accounts,
        Partition = "aws"
    }
};
```

Step 3: Create the keyring\.  
To create the decryption keyring, this example uses the `CreateAwsKmsDiscoveryKeyring()` method and the keyring input object\.  

```
var discoveryKeyring = materialProviders.CreateAwsKmsDiscoveryKeyring(discoveryKeyringInput);
```

Step 4: Create the input object for decrypting\.  
To create the input object for the `Decrypt()` method, instantiate the `DecryptInput` class\. The value of the `Ciphertext` parameter is the `Ciphertext` member of the `EncryptOutput` object that the `Encrypt()` method returns\.  

```
var ciphertext = encryptOutput.Ciphertext;

var decryptInput = new DecryptInput
{
    Ciphertext = ciphertext,
    Keyring = discoveryKeyring
};
var decryptOutput = encryptionSdk.Decrypt(decryptInput);
```

Step 5: Verify the encryption context\.  
The AWS Encryption SDK does not take an encryption context on `Decrypt()`\. It gets the encryption context values from the metadata in the encrypted message\. However, before returning or using the plaintext, it's a best practice to verify that encryption context that was used to decrypt the ciphertext includes the encryption context you provided when encrypting\.   
Verify that the encryption context used on encrypt *is included* in the encryption context that was used to decrypt the ciphertext\. The AWS Encryption SDK adds pairs to the encryption context, including the digital signature if you're using an algorithm suite with signing, such as the default algorithm suite\.   

```
// Verify the encryption context
string contextKey = "purpose";
string contextValue = "test";

if (!decryptOutput.EncryptionContext.TryGetValue(contextKey, out var decryptContextValue)
    || !decryptContextValue.Equals(contextValue))
{
    throw new Exception("Encryption context does not match expected values");
}
```