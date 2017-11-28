# How to Use the AWS Encryption SDK Command Line Interface<a name="crypto-cli-how-to"></a>

This topic explains how to use the parameters in the AWS Encryption CLI\. For examples, see [Examples of the AWS Encryption SDK Command Line Interface](crypto-cli-examples.md)\. For complete documentation, see [Read the Docs](http://aws-encryption-sdk-cli.readthedocs.io/en/latest/)\. 


+ [How to Encrypt and Decrypt Data](#crypto-cli-e-d-intro)
+ [How to Specify a Master Key](#crypto-cli-master-key)
+ [How to Provide Input](#crypto-cli-input)
+ [How to Specify the Output Location](#crypto-cli-output)
+ [How to Use an Encryption Context](#crypto-cli-encryption-context)
+ [How to Store Parameters in a Configuration File](#crypto-cli-config-file)

## How to Encrypt and Decrypt Data<a name="crypto-cli-e-d-intro"></a>

The AWS Encryption CLI uses the features of the AWS Encryption SDK to make it easy to encrypt and decrypt data securely\.

+ When you encrypt data in the AWS Encryption CLI, you must specify your plaintext data \(as input\), a master key provider \(the default is 'aws\-kms'\), and a master key, such as an AWS Key Management Service \(AWS KMS\) customer master key \(CMK\)\. You also need to specify the output location for the encrypted data and an output location for metadata about the encryption operation\. An encryption context is optional, but recommended\.

  The AWS Encryption CLI gets a unique data key from the master key, encrypts your data, and returns an encrypted message\. The encrypted message contains your encrypted data \(*ciphertext*\) and an encrypted copy of the data key\. You don't have to worry about storing, managing, or losing the data key\.

  ```
  aws-encryption-cli --encrypt --input myData \
                     --master-keys key=1234abcd-12ab-34cd-56ef-1234567890ab \
                     --output myEncryptedMessage \
                     --encryption-context purpose=test \
                     --metadata-output ~/metadata
  ```

+ When you decrypt data, you pass in your encrypted message, the same master key, the optional encryption context, an output location, and a location for the metadata\. If you are using an AWS KMS CMK, you do not supply the master key, because AWS KMS derives it from the encrypted message\. 

  The AWS Encryption CLI uses the master key to decrypt the data key in the encrypted message\. Then, it uses the data key to decrypt your data, and it returns your plaintext data\.

  ```
  aws-encryption-cli --decrypt --input myEncryptedMessage \
                     --output myPlaintextData \
                     --encryption-context purpose=test \
                     --metadata-output ~/metadata
  ```

## How to Specify a Master Key<a name="crypto-cli-master-key"></a>

When you encrypt data in the AWS Encryption CLI, you need to specify a master key\. You can use an AWS KMS customer master key \(CMK\) or a master key from any compatible Python master key provider\. To specify a master key and master key provider, use the `--master-keys` parameter\. 

To specify a master key, use the `--master-keys` parameter \(`-m`\)\. Its value is a collection of space\-separated attributes\. Each attribute has an `attribute=value` format\. You can include multiple `--master-keys` parameters in the same command\. 

If you are using a custom master key provider \(not AWS KMS\), you must specify a `--master-keys` parameter with **key** and **provider** attributes in all commands\.

When you are using AWS KMS CMKs, a `--master-keys` parameter is required in all encrypt commands\. In decrypt commands, the `--master-keys` parameter is optional\. You can use it only to specify an AWS CLI named profile\.

### Master Key Parameter Attributes<a name="cli-master-key-attributes"></a>

The value of the `--master-keys` parameter consists of the following attributes and their values\. 

If an attribute name or value includes spaces or special characters, enclose both the name and value in quotation marks\. For example, `--master-keys key=12345 "provider=my cool provider".`

**Key: Specify a Master Key**  
Use the **key** attribute to identify a master key\. The value can be any key identifier that the master key provider recognizes\.   

```
--master-keys key=1234abcd-12ab-34cd-56ef-1234567890ab
```
In an encrypt command, each `--master-keys` parameter value must include at least one **key** attribute and value\. You can use multiple **key** attributes in each `--master-keys` parameter value\.  

```
aws-encryption-cli --encrypt --master-keys key=1234abcd-12ab-34cd-56ef-1234567890ab key=1a2b3c4d-5e6f-1a2b-3c4d-5e6f1a2b3c4d
```
In encrypt commands that use AWS KMS CMKs, the value of **key** can be the CMK ID, its [Amazon Resource Name \(ARN\)](http://docs.aws.amazon.com/kms/latest/developerguide/viewing-keys.html#find-cmk-id-arn), an alias name, or alias ARN\. If you use a CMK identifier that does not specify a region, the AWS Encryption CLI uses the default region specified in your AWS CLI **named profile**\. You can also use the **region** attribute to specify a region\.  
For example, this encrypt command uses a CMK ID in the value of the **key** attribute\.   

```
aws-encryption-cli --encrypt --master-keys key=0987dcba-09fe-87dc-65ba-ab0987654321
```
In decrypt commands that use a custom master key provider \(not AWS KMS\), the **key** attribute is required\.   
The **key** attribute is not permitted in commands that decrypt data that was encrypted under an AWS KMS CMK\. The AWS Encryption CLI can use any CMK that was used to encrypt a data key, provided that the AWS credentials you are using have permission to call the [Decrypt API](http://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) on the master key\. For more information, see [Authentication and Access Control for AWS KMS](http://docs.aws.amazon.com/kms/latest/developerguide/control-access.html)\.

**Provider: Specify the Master Key Provider**  
The **provider** attribute identifies the master key provider\. The default value is `aws-kms`, which represents AWS KMS\. If you are using a different master key provider, the **provider** attribute is required\.   
For example, the **provider** attribute in this command identifies the master key provider that supplies the master key in the **key** attribute\.  

```
--master-keys key=12345 provider=my_custom_provider
--master-keys key=1234abcd-12ab-34cd-56ef-1234567890ab provider=aws-kms
```
For more information about using custom \(non\-AWS KMS\) master key providers, see the **Advanced Configuration** topic in the [README](https://github.com/awslabs/aws-encryption-sdk-cli/blob/master/README.rst) file for the [AWS Encryption SDK CLI](https://github.com/awslabs/aws-encryption-sdk-cli/) repository\.

**Region: Specify an AWS Region**  
Use the **region** attribute to specify the AWS Region of an AWS KMS CMK\.   
By default, AWS Encryption CLI commands use the AWS Region that is specified in the **key** attribute value \(if it includes a region\), or the default region specified in your AWS CLI [named profile](http://docs.aws.amazon.com/cli/latest/userguide/cli-multiple-profiles.html), if any\. You can use the **region** attribute to override the default AWS Region in your AWS CLI named profile, but if the **key** value specifies a AWS Region, the **region** attribute is ignored\.  
When you decrypt data that was encrypted with a CMK, the AWS Region of the CMK is used, and the **region** attribute is ignored\.  
The following example uses the **region** attribute in an encrypt command\. In this case, the CMK alias name does not specify an AWS Region and there is no default AWS Region in the user's profile\.  

```
--encrypt --master-keys key=alias/primary-key region=us-east-2
```

**Profile: Specify a Named Profile**  
If you are using an AWS KMS CMK, you can use the **profile** attribute to specify an AWS CLI [named profile](http://docs.aws.amazon.com/cli/latest/userguide/cli-multiple-profiles.html)\. Named profiles can include credentials and an AWS Region\.   
You can use the **profile** attribute to specify alternate credentials in encrypt and decrypt commands\. However, in an encrypt command, the AWS Encryption CLI uses the AWS Region in the named profile only when the **key** value does not include a region and there is no **region** attribute\. In a decrypt command, the **region** attribute is ignored\.  
The following example uses a **profile** attribute to specify the "admin1" profile\.  

```
--master-keys key=alias/primary-key profile=admin-1
```

### How to Specify Multiple Master Keys<a name="cli-many-cmks"></a>

You can specify multiple master keys in each command\. 

If you specify more than one master key, the first master key generates \(and encrypts\) the data key that is used to encrypt your data\. The other master keys only encrypt the data key\. The resulting encrypted message contains the encrypted data \("ciphertext"\) and a collection of encrypted data keys, one encrypted by each master key\. Any of the master keys can decrypt one data key and then decrypt the data\.

There are two ways to specify multiple master keys: 

+ Include multiple **key** attributes in a `--master-keys` parameter value\.

  ```
  $cmk_oregon=arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab
  $cmk_ohio=arn:aws:kms:us-east-2:111122223333:key/0987ab65-43cd-21ef-09ab-87654321cdef
  
  --master-keys key=$cmk_oregon key=$cmk_ohio
  ```

+ Include multiple `--master-keys` parameters in the same command\. Use this syntax when the attribute values that you specify do not apply to all of the master keys in the command\.

  ```
  --master-keys region=us-east-2 key=alias/primary_CMK \
  --master-keys region=us-west-1 key=alias/primary_CMK
  ```

## How to Provide Input<a name="crypto-cli-input"></a>

The encrypt operation in the AWS Encryption CLI takes plaintext data as input and returns encrypted data\. The decrypt operation takes encrypted data as input and returns plaintext data\. 

The `--input` parameter \(`-i`\) , which tells the AWS Encryption CLI where to find the input, is required in all AWS Encryption CLI commands\. 

You can provide input in any of the following ways:

+ Use a file\.

  ```
  --input myData.txt
  ```

+ Use a file name pattern\. 

  ```
  --input testdir/*.xml
  ```

+ Use a directory or directory name pattern\. When the input is a directory, the `--recursive` parameter \(`-r`, `-R`\) is required\.

  ```
  --input testdir --recursive
  ```

+ Pipe input to the command \(stdin\)\. Use a value of `-` for the `--input` parameter\. \(The `--input` parameter is always required\.\)

  ```
  echo 'Hello World' | aws-encryption-cli --encrypt --input -
  ```

## How to Specify the Output Location<a name="crypto-cli-output"></a>

The `--output` parameter tells the AWS Encryption CLI where to write the results of the encryption or decryption operation\. It is required in every AWS Encryption CLI command\. The AWS Encryption CLI creates a new output file for every input file in the operation\. 

If an output file already exists, by default, the AWS Encryption CLI prints a warning, then overwrites the file\. To prevent overwriting, use the `--interactive` parameter, which prompts you for confirmation before overwriting, or `--no-overwrite`, which skips the input if the output would cause an overwrite\. To suppress the overwrite warning, use `--quiet`\. To capture errors and warnings from the AWS Encryption CLI, use the `2>&1` redirection operator to write them to the output stream\.

**Note**  
If a command that would overwrite an output file fails, the output file is deleted\.

You have the following options for the output location:

+ Specify a file name\. If you specify a path to the file, all directories in the path must exist before the command runs\. 

  ```
  --output myEncryptedData.txt
  ```

+ Specify a directory\. The output directory must exist before the command runs\. 

  If the input contains subdirectories, the command reproduces the subdirectories under the specified directory\.

  ```
  --output Test
  ```

  When the output location is a directory \(without file names\), the AWS Encryption CLI creates output file names based on the input file names plus a suffix\. Encrypt operations append `.encrypted` to the input file name and the decrypt operations append `.decrypted`\. To change the suffix, use the `--suffix` parameter\.

  For example, if you encrypt `file.txt`, the encrypt command creates `file.txt.encrypted`\. If you decrypt `file.txt.encrypted`, the decrypt command creates `file.txt.encrypted.decrypted`\.

   

+ Write to the command line \(stdout\)\. Enter a value of `-` for the `--output` parameter\. You can use `--output -` to pipe output to another command or program\.

  ```
  --output -
  ```

## How to Use an Encryption Context<a name="crypto-cli-encryption-context"></a>

The AWS Encryption CLI lets you provide an encryption context in encrypt and decrypt commands\. It is not required, but it is a cryptographic best practice that we recommend\.

An *encryption context* is a type of arbitrary, non\-secret *additional authenticated data*\. In the AWS Encryption CLI, the encryption context consists of a collection of `name=value` pairs\. You can use any content in the pairs, including information about the files, data that helps you to find the encryption operation in logs, or data that your grants or policies require\. 

**In an Encrypt Command**

The encryption context that you specify in an encrypt command, along with any additional encryption context that the encryption components add, is cryptographically bound to the encrypted data\. It is also included \(in plaintext\) in the encrypted message that the command returns\. If you are using an AWS KMS customer master key \(CMK\), the encryption context also might appear in plaintext in audit records and logs, such as AWS CloudTrail\. 

The following example shows a valid encryption context for an encrypt command\.

```
--encryption-context purpose=test dept=IT class=confidential 
```

**In a Decrypt Command**

The encryption context that you specify in a decrypt command helps you to confirm that you are decrypting the encrypted message that you expect\. You are not required to provide an encryption context when you decrypt an encrypted message, even if an encryption context was used to encrypt it\. However, if you do, the AWS Encryption CLI verifies that every element in the encryption context of the decrypt command matches an element in the encryption context of the encrypted message\. If any element does not match, the decrypt command fails\. 

For example, you might want to decrypt only the messages for a particular department or only messages that have been classified, regardless of the classification value\. 

The following example shows a valid encryption context for a decrypt command\. It includes `dept=IT`, which is a `name=value` pair, and `class`, which is a name element without a value\.

```
--encryption-context dept=IT class
```

An encryption context is an important part of your security strategy\. However, when choosing an encryption context, remember that its values are not secret\. Do not include any confidential data in the encryption context\.

**To specify an encryption context:**

+ In an **encrypt** command, use the `--encryption-context` parameter with one or more `name=value` pairs\. Use a space to separate each pair\. 

  ```
  --encryption-context name=value [name=value] ...
  ```

+ In a **decrypt** command, the `--encryption-context` parameter value can include `name=value` pairs, `name` elements \(with no values\), or a combination of both\.

  ```
  --encryption-context name[=value] [name] [name=value] ...
  ```

If the `name` or `value` in a `name=value` pair includes spaces or special characters, enclose the entire pair in quotation marks\.

```
--encryption-context "department=software engineering" "AWS Region=us-west-2"
```

For example, this encrypt command includes an encryption context with two pairs, `purpose=test` and `dept=23`\.

```
aws-encryption-cli --encrypt --encryption-context purpose=test dept=23 ...
```

This command decrypts the content of the `data.encrypted` file successfully\. The encryption context in each of these commands is a subset of the original encryption context\.

```
\\ Any one or both of the encryption context pairs
aws-encryption-cli --decrypt --encryption-context dept=23 ...

\\ Any one or both of the encryption context names
aws-encryption-cli --decrypt --encryption-context purpose ...

\\ Any combination of names and pairs
aws-encryption-cli --decrypt --encryption-context dept purpose=test ...
```

However, this decrypt command would fail because the encryption context in the encrypted message does not contain the specified elements\.

```
aws-encryption-cli --decrypt --encryption-context dept=Finance ...
aws-encryption-cli --decrypt --encryption-context scope ...
```

When a command that would overwrite an output file fails, including a decrypt command with an invalid encryption context, the output file is deleted\.

## How to Store Parameters in a Configuration File<a name="crypto-cli-config-file"></a>

You can save time and avoid typing errors by saving frequently used AWS Encryption CLI parameters and values in configuration files\. 

A *configuration file* is a text file that contains parameters and values for an AWS Encryption CLI command\. When you refer to a configuration file in a AWS Encryption CLI command, the reference is replaced by the parameters and values in the configuration file\. The effect is the same is if you typed the file content at the command line\. A configuration file can have any name and it can be located in any directory that the current user can access\. 

The following example configuration file, `cmk.conf`, specifies two AWS KMS CMKs in different regions\.

```
--master-keys key=arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab
--master-keys key=arn:aws:kms:us-east-2:111122223333:key/0987ab65-43cd-21ef-09ab-87654321cdef
```

To use the configuration file in a command, prefix the file name with an at sign \(`@`\)\. In a PowerShell console, use a backtick character to escape the at sign \(``@`\)\.

This example command uses the `cmk.conf` file in an encrypt command\.

------
#### [ Bash ]

```
$ aws-encryption-cli -e @cmk.conf -i hello.txt -o testdir  
```

------
#### [ PowerShell ]

```
PS C:\> aws-encryption-cli -e `@cmk.conf -i .\Hello.txt -o .\TestDir
```

------

**Configuration File Rules**

The rules for using configuration files are as follows:

+ You can include multiple parameters in each configuration file and list them in any order\. List each parameter with its values \(if any\) on a separate line\. 

+ Use `#` to add a comment to all or part of a line\.

+ You can include references to other configuration files\. Do not use a backtick to escape the `@` sign, even in PowerShell\.

+ If you use quotes in a configuration file, the quoted text cannot span multiple lines\.

For example, this is the contents of an example `encrypt.conf` file\.

```
# Archive Files
--encrypt
--output /archive/logs
--recursive
--interactive
--encryption-context class=unclassified dept=IT
--suffix  # No suffix
--metadata-output ~/metadata
@caching.conf  # Use limited caching
```

You can also include multiple configuration files in a command\. This example command uses both the `encrypt.conf` and `master-keys.conf` configurations files\.

------
#### [ Bash ]

```
$  aws-encryption-cli -i /usr/logs @encrypt.conf @master-keys.conf
```

------
#### [ PowerShell ]

```
PS C:\> aws-encryption-cli -i $home\Test\*.log `@encrypt.conf `@master-keys.conf
```

------

**Next: **Try the AWS Encryption CLI examples