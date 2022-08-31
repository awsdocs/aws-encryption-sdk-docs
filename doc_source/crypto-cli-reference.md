# AWS Encryption SDK CLI syntax and parameter reference<a name="crypto-cli-reference"></a>

This topic provides syntax diagrams and brief parameter descriptions to help you use the AWS Encryption SDK Command Line Interface \(CLI\)\. For help with wrapping keys and other parameters, see [How to use the AWS Encryption CLI](crypto-cli-how-to.md)\. For examples, see [Examples of the AWS Encryption CLI](crypto-cli-examples.md)\. For complete documentation, see [Read the Docs](https://aws-encryption-sdk-cli.readthedocs.io/en/latest/)\.

**Topics**
+ [AWS Encryption CLI syntax](#crypto-cli-syntax)
+ [AWS Encryption CLI command line parameters](#crypto-cli-parameters)
+ [Advanced parameters](#cli-advanced-parameters)

## AWS Encryption CLI syntax<a name="crypto-cli-syntax"></a>

These AWS Encryption CLI syntax diagrams show the syntax for each task that you perform with the AWS Encryption CLI\. They represent recommended syntax in AWS Encryption CLI version 2\.1\.*x* and later\.

New security features were originally released in AWS Encryption CLI versions 1\.7\.*x* and 2\.0\.*x*\. However, AWS Encryption CLI version 1\.8\.*x* replaces version 1\.7\.*x* and AWS Encryption CLI 2\.1\.*x* replaces 2\.0\.*x*\. For details, see the relevant [security advisory](https://github.com/aws/aws-encryption-sdk-cli/security/advisories/GHSA-2xwp-m7mq-7q3r) in the [aws\-encryption\-sdk\-cli](https://github.com/aws/aws-encryption-sdk-cli/) repository on GitHub\.

**Note**  
Unless noted in the parameter description, each parameter or attribute can be used only once in each command\.  
If you use an attribute that a parameter does not support, the AWS Encryption CLI ignores that unsupported attribute without a warning or error\.

**Get help**  
To get the full AWS Encryption CLI syntax with parameter descriptions, use `--help` or `-h`\.  

```
aws-encryption-cli (--help | -h)
```

**Get the version**  
To get the version number of your AWS Encryption CLI installation, use `--version`\. Be sure to include the version when you ask questions, report problems, or share tips about using the AWS Encryption CLI\.  

```
aws-encryption-cli --version
```

**Encrypt data**  
The following syntax diagram shows the parameters that an encrypt command uses\.   

```
aws-encryption-cli --encrypt
                   --input <input> [--recursive] [--decode]
                   --output <output> [--interactive] [--no-overwrite] [--suffix [<suffix>]] [--encode]
                   --wrapping-keys  [--wrapping-keys] ...
                       key=<keyID> [key=<keyID>] ...
                       [provider=<provider-name>] [region=<aws-region>] [profile=<aws-profile>]
                   --metadata-output <location> [--overwrite-metadata] | --suppress-metadata]
                   [--commitment-policy <commitment-policy>]
                   [--encryption-context <encryption_context> [<encryption_context> ...]]
                   [--max-encrypted-data-keys <integer>]
                   [--algorithm <algorithm_suite>]
                   [--caching <attributes>] 
                   [--frame-length <length>]
                   [-v | -vv | -vvv | -vvvv]
                   [--quiet]
```

**Decrypt data**  
The following syntax diagram shows the parameters that a decrypt command uses\.   
In version 1\.8\.*x*, the `--wrapping-keys` parameter is optional when decrypting, but recommended\. Beginning in version 2\.1\.*x*, the `--wrapping-keys` parameter is required when encrypting and decrypting\. For AWS KMS keys, you can use the **key** attribute to specify wrapping keys \(best practice\) or set the **discovery** attribute to `true`, which doesn't limit the wrapping keys that the AWS Encryption CLI can use\.  

```
aws-encryption-cli --decrypt (or [--decrypt-unsigned]) 
                   --input <input> [--recursive] [--decode]
                   --output <output> [--interactive] [--no-overwrite]  [--suffix [<suffix>]] [--encode]           
                   --wrapping-keys  [--wrapping-keys] ...
                       [key=<keyID>] [key=<keyID>] ...
                       [discovery={true|false}] [discovery-partition=<aws-partition-name> discovery-account=<aws-account-ID> [discovery-account=<aws-account-ID>] ...] 
                       [provider=<provider-name>] [region=<aws-region>] [profile=<aws-profile>]
                   --metadata-output <location> [--overwrite-metadata] | --suppress-metadata]
                   [--commitment-policy <commitment-policy>]
                   [--encryption-context <encryption_context> [<encryption_context> ...]]
                   [--buffer]
                   [--max-encrypted-data-keys <integer>]
                   [--caching <attributes>]
                   [--max-length <length>]
                   [-v | -vv | -vvv | -vvvv]
                   [--quiet]
```

**Use configuration files**  
You can refer to configuration files that contain parameters and their values\. This is equivalent to typing the parameters and values in the command\. For an example, see [How to store parameters in a configuration file](crypto-cli-how-to.md#crypto-cli-config-file)\.  

```
aws-encryption-cli @<configuration_file>

# In a PowerShell console, use a backtick to escape the @.
aws-encryption-cli `@<configuration_file>
```

## AWS Encryption CLI command line parameters<a name="crypto-cli-parameters"></a>

This list provides a basic description of the AWS Encryption CLI command parameters\. For a complete description, see the [aws\-encryption\-sdk\-cli documentation](http://aws-encryption-sdk-cli.readthedocs.io/en/latest/)\.

**\-\-encrypt \(\-e\)**  
Encrypts the input data\. Every command must have an `--encrypt`, or `--decrypt`, or `--decrypt-unsigned` parameter\.

**\-\-decrypt \(\-d\)**  
Decrypts the input data\. Every command must have an `--encrypt`, `--decrypt`, or `--decrypt-unsigned` parameter\.

**\-\-decrypt\-unsigned \[Introduced in versions 1\.9\.*x* and 2\.2\.*x*\]**  
The `--decrypt-unsigned` parameter decrypts ciphertext and ensures that messages are unsigned before decryption\. Use this parameter if you used the `--algorithm` parameter and selected an algorithm suite without digital signing to encrypt data\. If the ciphertext is signed, decryption fails\.  
You can use `--decrypt` or `--decrypt-unsigned` for decryption but not both\.

**\-\-wrapping\-keys \(\-w\) \[Introduced in version 1\.8\.*x*\]**  <a name="wrapping-keys"></a>
Specifies the [wrapping keys](concepts.md#master-key) \(or *master keys*\) used in encryption and decryption operations\. You can use [multiple `--wrapping-keys` parameters](crypto-cli-how-to.md#cli-many-cmks) in each command\.   
Beginning in version 2\.1\.*x*, the `--wrapping-keys` parameter is required in encrypt and decrypt commands\. In version 1\.8\.*x*, encrypt commands require either a `--wrapping-keys` or `--master-keys` parameter\. In version 1\.8\.*x* decrypt commands, a `--wrapping-keys` parameter is optional but recommended\.   
When using a custom master key provider, encrypt and decrypt commands require **key** and **provider** attributes\. When using AWS KMS keys, encrypt commands require a **key** attribute\. Decrypt commands require a **key** attribute or a **discovery** attribute with a value of `true` \(but not both\)\. Using the **key** attribute when decrypting is an [AWS Encryption SDK best practice](best-practices.md)\. It is particularly important if you're decrypting batches of unfamiliar messages, such as those in an Amazon S3 bucket or an Amazon SQS queue\.  
For an example showing how to use AWS KMS multi\-Region keys as wrapping keys, see [Use multi\-Region AWS KMS keys](configure.md#config-mrks)\.  
**Attributes**: The value of the `--wrapping-keys` parameter consists of the following attributes\. The format is `attribute_name=value`\.     
**key**  
Identifies the wrapping key used in the operation\. The format is a **key**=ID pair\. You can specify multiple **key** attributes in each `--wrapping-keys` parameter value\.  
+ **Encrypt commands**: All encrypt commands require the **key** attribute \. When you use an AWS KMS key in an encrypt command, the value of the **key** attribute can be a key ID, key ARN, an alias name, or an alias ARN\. For descriptions of the AWS KMS key identifiers, see [Key identifiers](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#key-id) in the *AWS Key Management Service Developer Guide*\. 
+ **Decrypt commands**: When decrypting with AWS KMS keys, the `--wrapping-keys` parameter requires a **key** attribute with a [key ARN](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#key-id-key-ARN) value or a **discovery** attribute with a value of `true` \(but not both\)\. Using the **key** attribute is an [AWS Encryption SDK best practice](best-practices.md)\. When decrypting with a custom master key provider, the **key** attribute is required\.
**Note**  
To specify an AWS KMS wrapping key in a decrypt command, the value of the **key** attribute must be a key ARN\. If you use a key ID, alias name, or alias ARN, the AWS Encryption CLI does not recognize the wrapping key\.
You can specify multiple **key** attributes in each `--wrapping-keys` parameter value\. However, any **provider**, **region**, and **profile** attributes in a `--wrapping-keys` parameter apply to all wrapping keys in that parameter value\. To specify wrapping keys with different attribute values, use multiple `--wrapping-keys` parameters in the command\.  
**discovery**  
Allows the AWS Encryption CLI to use any AWS KMS key to decrypt the message\. The **discovery** value can be `true` or `false`\. The default value is `false`\. The **discovery** attribute is valid only in decrypt commands and only when the master key provider is AWS KMS\.   
When decrypting with AWS KMS keys, the `--wrapping-keys` parameter requires a **key** attribute or a **discovery** attribute with a value of `true` \(but not both\)\. If you use the **key** attribute, you can use a **discovery** attribute with a value of `false` to explicitly reject discovery\.   
+ `False` \(default\) — When the **discovery** attribute isn't specified or its value is `false`, the AWS Encryption CLI decrypts the message using only the AWS KMS keys specified by the **key** attribute of the `--wrapping-keys` parameter\. If you don't specify a **key** attribute when discovery is `false`, the decrypt command fails\. This value supports an AWS Encryption CLI [best practice](best-practices.md)\.
+ `True` — When the value of the **discovery** attribute is `true`, the AWS Encryption CLI gets the AWS KMS keys from metadata in the encrypted message, and uses those AWS KMS keys to decrypt the message\. The **discovery** attribute with a value of `true` behaves like versions of the AWS Encryption CLI before version 1\.8\.*x* that didn't permit you to specify a wrapping key when decrypting\. However, your intent to use any AWS KMS key is explicit\. If you specify a **key** attribute when discovery is `true`, the decrypt command fails\. 

  The `true` value might cause the AWS Encryption CLI to use AWS KMS keys in different AWS accounts and Regions, or attempt to use AWS KMS keys that the user isn't authorized to use\. 
When **discovery** is `true`, it's a best practice to use the **discovery\-partition** and **discovery\-account** attributes to limit the AWS KMS keys used to those in the AWS accounts you specify\.   
**discovery\-account**  
Limits the AWS KMS keys used for decrypting to those in the specified AWS account\. The only valid value for this attribute is an [AWS account ID](https://docs.aws.amazon.com/general/latest/gr/acct-identifiers.html)\.  
This attribute is optional and valid only in decrypt commands with AWS KMS keys where the **discovery** attribute is set to `true` and the **discovery\-partition** attribute is specified\.  
Each **discovery\-account** attribute takes just one AWS account ID, but you can specify multiple **discovery\-account** attributes in the same `--wrapping-keys` parameter\. All accounts specified in a given `--wrapping-keys` parameter must be in the specified AWS partition\.  
**discovery\-partition**  
Specifies the AWS partition for the accounts in the **discovery\-account** attribute\. Its value must be an AWS partition, such as `aws`, `aws-cn`, or `aws-gov-cloud`\. For information, see [Amazon Resource Names](https://docs.aws.amazon.com/general/latest/gr/aws-arns-and-namespaces.html#arns-syntax) in the *AWS General Reference*\.  
This attribute is required when you use the **discovery\-account** attribute\. You can specify only one **discovery\-partition** attribute in each `--wrapping keys` parameter\. To specify AWS accounts in multiple partitions, use an additional `--wrapping-keys` parameter\.  
**provider**  
Identifies the [master key provider](concepts.md#master-key-provider)\. The format is a **provider**=ID pair\. The default value, **aws\-kms**, represents AWS KMS\. This attribute is required only when the master key provider is not AWS KMS\.  
**region**  
Identifies the AWS Region of an AWS KMS key\. This attribute is valid only for AWS KMS keys\. It is used only when the **key** identifier does not specify a Region; otherwise, it is ignored\. When it is used, it overrides the default Region in the AWS CLI named profile\.   
**profile**  
Identifies an AWS CLI [named profile](https://docs.aws.amazon.com/cli/latest/userguide/cli-multiple-profiles.html)\. This attribute is valid only for AWS KMS keys\. The Region in the profile is used only when the key identifier does not specify a Region and there is no **region** attribute in the command\. 

**\-\-input \(\-i\)**  
Specifies the location of the data to encrypt or decrypt\. This parameter is required\. The value can be a path to a file or directory, or a file name pattern\. If you are piping input to the command \(stdin\), use `-`\.  
If the input does not exist, the command completes successfully without error or warning\.    
**\-\-recursive \(\-r, \-R\)**  
Performs the operation on files in the input directory and its subdirectories\. This parameter is required when the value of `--input` is a directory\.  
**\-\-decode**  
Decodes Base64\-encoded input\.   
If you are decrypting a message that was encrypted and then encoded, you must decode the message before decrypting it\. This parameter does that for you\.   
For example, if you used the `--encode` parameter in an encrypt command, use the `--decode` parameter in the corresponding decrypt command\. You can also use this parameter to decode Base64\-encoded input before you encrypt it\.

**\-\-output \(\-o\)**  
Specifies a destination for the output\. This parameter is required\. The value can be a file name, an existing directory, or `-`, which writes output to the command line \(stdout\)\.   
If the specified output directory does not exist, the command fails\. If the input contains subdirectories, the AWS Encryption CLI reproduces the subdirectories under the output directory that you specify\.   
By default, the AWS Encryption CLI overwrites files with the same name\. To change that behavior, use the `--interactive` or `--no-overwrite` parameters\. To suppress the overwrite warning, use the `--quiet` parameter\.  
If a command that would overwrite an output file fails, the output file is deleted\.  
**\-\-interactive**  
Prompts before overwriting the file\.  
**\-\-no\-overwrite**  
Does not overwrite files\. Instead, if the output file exists, the AWS Encryption CLI skips the corresponding input\.  
**\-\-suffix**  
Specifies a custom file name suffix for files that the AWS Encryption CLI creates\. To indicate no suffix, use the parameter with no value \(`--suffix`\)\.  
By default, when the `--output` parameter does not specify a file name, the output file name has the same name as the input file name plus the suffix\. The suffix for encrypt commands is `.encrypted`\. The suffix for decrypt commands is `.decrypted`\.   
**\-\-encode**  
Applies Base64 \(binary to text\) encoding to the output\. Encoding prevents the shell host program from misinterpreting non\-ASCII characters in output text\.  
Use this parameter when writing encrypted output to stdout \(`--output -`\), especially in a PowerShell console, even when you are piping the output to another command or saving it in a variable\.

**\-\-metadata\-output**  
Specifies a location for metadata about the cryptographic operations\. Enter a path and file name\. If the directory does not exist, the command fails\. To write the metadata to the command line \(stdout\), use `-`\.   
You cannot write command output \(`--output`\) and metadata output \(`--metadata-output`\) to stdout in the same command\. Also, when the value of `--input` or `--output` is a directory \(without file names\), you cannot write the metadata output to the same directory or to any subdirectory of that directory\.  
If you specify an existing file, by default, the AWS Encryption CLI appends new metadata records to any content in the file\. This feature lets you create a single file that contains the metadata for all of your cryptographic operations\. To overwrite the content in an existing file, use the `--overwrite-metadata` parameter\.  
The AWS Encryption CLI returns a JSON\-formatted metadata record for each encryption or decryption operation that the command performs\. Each metadata record includes the full paths to the input and output file, the encryption context, the algorithm suite, and other valuable information that you can use to review the operation and verify that it meets your security standards\.    
**\-\-overwrite\-metadata**  
Overwrites the content in the metadata output file\. By default, the `--metadata-output` parameter appends metadata to any existing content in the file\.

**\-\-suppress\-metadata \(\-S\)**  
Suppresses the metadata about the encryption or decryption operation\. 

**\-\-commitment\-policy**  <a name="syntax-commitment-policy"></a>
Specifies the [commitment policy](concepts.md#commitment-policy) for encrypt and decrypt commands\. The commitment policy determines whether your message is encrypted and decrypted with the [key commitment](concepts.md#key-commitment) security feature\.  
The `--commitment-policy` parameter is introduced in version 1\.8\.*x*\. It is valid in encrypt and decrypt commands\.  
**In version 1\.8\.*x***, the AWS Encryption CLI uses the `forbid-encrypt-allow-decrypt` commitment policy for all encrypt and decrypt operations\. When you use the `--wrapping-keys` parameter in an encrypt or decrypt command, a `--commitment-policy` parameter with the `forbid-encrypt-allow-decrypt` value is required\. If you don't use the `--wrapping-keys` parameter, the `--commitment-policy` parameter is invalid\. Setting a commitment policy explicitly prevents your commitment policy from changing automatically to `require-encrypt-require-decrypt` when you upgrade to version 2\.1\.*x*  
Beginning in **version 2\.1\.*x***, all commitment policy values are supported\. The `--commitment-policy` parameter is optional and the default value is `require-encrypt-require-decrypt`\.   
This parameter has the following values:  
+ `forbid-encrypt-allow-decrypt` — Cannot encrypt with key commitment\. It can decrypt ciphertexts encrypted with or without key commitment\. 

  In version 1\.8\.*x*, this is the only valid value\. The AWS Encryption CLI uses the `forbid-encrypt-allow-decrypt` commitment policy for all encrypt and decrypt operations\. 
+ `require-encrypt-allow-decrypt` — Encrypts only with key commitment\. Decrypts with and without key commitment\. This value is introduced in version 2\.1\.*x*\.
+ `require-encrypt-require-decrypt` \(default\) — Encrypts and decrypts only with key commitment\. This value is introduced in version 2\.1\.*x*\. It is the default value in versions 2\.1\.*x* and later\. With this value, the AWS Encryption CLI will not decrypt any ciphertext that was encrypted with earlier versions of the AWS Encryption SDK\.
For detailed information about setting your commitment policy, see [Migrating your AWS Encryption SDK](migration.md)\.

**\-\-encryption\-context \(\-c\)**  
Specifies an [encryption context](crypto-cli-how-to.md#crypto-cli-encryption-context) for the operation\. This parameter is not required, but it is recommended\.   
+ In an `--encrypt` command, enter one or more `name=value` pairs\. Use spaces to separate the pairs\.
+ In a `--decrypt` command, enter `name=value` pairs, `name` elements with no values, or both\.
If the `name` or `value` in a `name=value` pair includes spaces or special characters, enclose the entire pair in quotation marks\. For example, `--encryption-context "department=software development"`\.

**\-\-buffer \(\-b\) \[Introduced in versions 1\.9\.*x* and 2\.2\.*x*\]**  
Returns plaintext only after all input is processed, including verifying the digital signature if one is present\.

**\-\-max\-encrypted\-data\-keys \[Introduced in versions 1\.9\.*x* and 2\.2\.*x*\]**  
Specifies the maximum number of encrypted data keys in an encrypted message\. This parameter is optional\.   
Valid values are 1 – 65,535\. If you omit this parameter, the AWS Encryption CLI does not enforce any maximum\. An encrypted message can hold up to 65,535 \(2^16 \- 1\) encrypted data keys\.  
You can use this parameter in encrypt commands to prevent a malformed message\. You can use it in decrypt commands to detect malicious messages and avoid decrypting messages with numerous encrypted data keys that you can't decrypt\. For details and an example, see [Limit encrypted data keys](configure.md#config-limit-keys)\.

**\-\-help \(\-h\)**  
Prints usage and syntax at the command line\.

**\-\-version**  
Gets the version of the AWS Encryption CLI\.

**\-v \| \-vv \| \-vvv \| \-vvvv**  
Displays verbose information, warning, and debugging messages\. The detail in the output increases with the number of `v`s in the parameter\. The most detailed setting \(`-vvvv`\) returns debugging\-level data from the AWS Encryption CLI and all of the components that it uses\.

**\-\-quiet \(\-q\)**  
Suppresses warning messages, such as the message that appears when you overwrite an output file\.

**\-\-master\-keys \(\-m\) \[Deprecated\]**  
The \-\-master\-keys parameter is deprecated in 1\.8\.*x* and removed in version 2\.1\.*x*\. Instead, use the [\-\-wrapping\-keys](#wrapping-keys) parameter\.
Specifies the [master keys](concepts.md#master-key) used in encryption and decryption operations\. You can use multiple master keys parameters in each command\.  
The `--master-keys` parameter is required in encrypt commands\. It is required in decrypt commands only when you are using a custom \(non\-AWS KMS\) master key provider\.  
**Attributes**: The value of the `--master-keys` parameter consists of the following attributes\. The format is `attribute_name=value`\.     
**key**  
Identifies the [wrapping key](concepts.md#master-key) used in the operation\. The format is a **key**=ID pair\. The **key** attribute is required in all encrypt commands\.   
When you use an AWS KMS key in an encrypt command, the value of the **key** attribute can be a key ID, key ARN, an alias name, or an alias ARN\. For details about AWS KMS key identifiers, see [Key identifiers](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#key-id) in the *AWS Key Management Service Developer Guide*\.  
The **key** attribute is required in decrypt commands when the master key provider is not AWS KMS\. The **key** attribute is not permitted in commands that decrypt data that was encrypted under an AWS KMS key\.   
You can specify multiple **key** attributes in each `--master-keys` parameter value\. However, any **provider**, **region**, and **profile** attributes apply to all master keys in the parameter value\. To specify master keys with different attribute values, use multiple `--master-keys` parameters in the command\.   
**provider**  
Identifies the [master key provider](concepts.md#master-key-provider)\. The format is a **provider**=ID pair\. The default value, **aws\-kms**, represents AWS KMS\. This attribute is required only when the master key provider is not AWS KMS\.  
**region**  
Identifies the AWS Region of an AWS KMS key\. This attribute is valid only for AWS KMS keys\. It is used only when the **key** identifier does not specify a Region; otherwise, it is ignored\. When it is used, it overrides the default Region in the AWS CLI named profile\.   
**profile**  
Identifies an AWS CLI [named profile](https://docs.aws.amazon.com/cli/latest/userguide/cli-multiple-profiles.html)\. This attribute is valid only for AWS KMS keys\. The Region in the profile is used only when the key identifier does not specify a Region and there is no **region** attribute in the command\. 

## Advanced parameters<a name="cli-advanced-parameters"></a>

**\-\-algorithm**  
Specifies an alternate [algorithm suite](concepts.md#crypto-algorithm)\. This parameter is optional and valid only in encrypt commands\.   
If you omit this parameter, the AWS Encryption CLI uses one of the default algorithm suites for the AWS Encryption SDK introduced in version 1\.8\.*x*\. Both default algorithms use AES\-GCM with an [HKDF](https://en.wikipedia.org/wiki/HKDF), an ECDSA signature, and a 256\-bit encryption key\. One uses key commitment; one does not\. The choice of default algorithm suite is determined by the [commitment policy](concepts.md#commitment-policy) for the command\.  
The default algorithm suites are recommended for most encryption operations\. For a list of valid values, see the values for the `algorithm` parameter in [Read the Docs](https://aws-encryption-sdk-cli.readthedocs.io/en/latest/index.html#execution)\.

**\-\-frame\-length**  
Creates output with specified frame length\. This parameter is optional and valid only in encrypt commands\.   
Enter a value in bytes\. Valid values are 0 and 1 – 2^31 \- 1\. A value of 0 indicates nonframed data\. The default is 4096 \(bytes\)\.   
Whenever possible, use framed data\. The AWS Encryption SDK supports nonframed data only for legacy use\. Some language implementations of the AWS Encryption SDK can still generate nonframed ciphertext\. All supported language implementations can decrypt framed and nonframed ciphertext\.

**\-\-max\-length**  
Indicates the maximum frame size \(or maximum content length for nonframed messages\) in bytes to read from encrypted messages\. This parameter is optional and valid only in decrypt commands\. It is designed to protect you from decrypting extremely large malicious ciphertext\.   
Enter a value in bytes\. If you omit this parameter, the AWS Encryption SDK does not limit the frame size when decrypting\.

**\-\-caching**  
Enables the [data key caching](data-key-caching.md) feature, which reuses data keys, instead of generating a new data key for each input file\. This parameter supports an advanced scenario\. Be sure to read the [Data Key Caching](data-key-caching.md) documentation before using this feature\.   
The `--caching` parameter has the following attributes\.    
**capacity \(required\)**  
Determines the maximum number of entries in the cache\.   
The minimum value is 1\. There is no maximum value\.  
**max\_age \(required\)**  
Determine how long cache entries are used, in seconds, beginning when they are added to the cache\.  
Enter a value greater than 0\. There is no maximum value\.  
**max\_messages\_encrypted \(optional\)**  
Determines the maximum number of messages that a cached entry can encrypt\.   
Valid values are 1 – 2^32\. The default value is 2^32 \(messages\)\.  
**max\_bytes\_encrypted \(optional\)**  
Determines the maximum number of bytes that a cached entry can encrypt\.  
Valid values are 0 and 1 – 2^63 \- 1\. The default value is 2^63 \- 1 \(messages\)\. A value of 0 lets you use data key caching only when you are encrypting empty message strings\.