# Examples of the AWS Encryption CLI<a name="crypto-cli-examples"></a>

Use the following examples to try the AWS Encryption CLI on the platform you prefer\. For help with master keys and other parameters, see [How to use the AWS Encryption CLI](crypto-cli-how-to.md)\. For a quick reference, see [AWS Encryption SDK CLI syntax and parameter reference](crypto-cli-reference.md)\.

**Note**  
The following examples use the syntax for versions 2\.0\.*x* of the AWS Encryption CLI\. 

**Topics**
+ [Encrypting a file](#cli-example-encrypt-file)
+ [Decrypting a file](#cli-example-decrypt-file)
+ [Encrypting all files in a directory](#cli-example-encrypt-directory)
+ [Decrypting all files in a directory](#cli-example-decrypt-directory)
+ [Encrypting and decrypting on the command line](#cli-example-stdin)
+ [Using multiple master keys](#cli-example-multimaster)
+ [Encrypting and decrypting in scripts](#cli-example-script)
+ [Using data key caching](#cli-example-caching)

## Encrypting a file<a name="cli-example-encrypt-file"></a>

This example uses the AWS Encryption CLI to encrypt the contents of the `hello.txt` file, which contains a "Hello World" string\. 

When you run an encrypt command on a file, the AWS Encryption CLI gets the contents of the file, generates a unique [data key](concepts.md#DEK), encrypts the file contents under the data key, and then writes the [encrypted message](concepts.md#message) to a new file\. 

The first command saves the [Amazon Resource Name \(ARN\)](https://docs.aws.amazon.com/kms/latest/developerguide/viewing-keys.html#find-cmk-id-arn) of an AWS KMS customer master key \(CMK\) in the `$cmkArn` variable\. For details about the key identifiers for an AWS KMS customer master key, see [Key Identifiers](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#key-id) in the *AWS Key Management Service Developer Guide*\.

The second command encrypts the file contents\. The command uses the `--encrypt` parameter to specify the operation and the `--input` parameter to indicate the file to encrypt\. The [`--wrapping-keys` parameter](crypto-cli-how-to.md#crypto-cli-master-key), and its required **key** attribute, tell the command to use the CMK represented by the key ARN\. 

The command uses the `--metadata-output` parameter to specify a text file for the metadata about the encryption operation\. As a best practice, the command uses the `--encryption-context` parameter to specify an [encryption context](crypto-cli-how-to.md#crypto-cli-encryption-context)\. 

This command also uses the `--commitment-policy` parameter to set the commitment policy explicitly\. This parameter is required in version 1\.7\.*x*\. Beginning in version 2\.0\.*x*, it is optional, but recommended\.

The value of the `--output` parameter, a dot \(\.\), tells the command to write the output file to the current directory\. 

------
#### [ Bash ]

```
\\ To run this example, replace the fictitious key ARN with a valid value.
$ cmkArn=arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab

$ aws-encryption-cli --encrypt \
                     --input hello.txt \
                     --wrapping-keys key=$cmkArn \
                     --metadata-output ~/metadata \
                     --encryption-context purpose=test \
                     --commitment-policy require-encrypt-require-decrypt \
                     --output .
```

------
#### [ PowerShell ]

```
# To run this example, replace the fictitious key ARN with a valid value.
PS C:\> $CmkArn = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'

PS C:\> aws-encryption-cli --encrypt `
                           --input Hello.txt `
                           --wrapping-keys key=$CmkArn `
                           --metadata-output $home\Metadata.txt `
                           --commitment-policy require-encrypt-require-decrypt `
                           --encryption-context purpose=test `
                           --output .
```

------

When the encrypt command succeeds, it does not return any output\. To determine whether the command succeeded, check the Boolean value in the `$?` variable\. When the command succeeds, the value of `$?` is `0` \(Bash\) or `True` \(PowerShell\)\. When the command fails, the value of `$?` is non\-zero \(Bash\) or `False` \(PowerShell\)\.

------
#### [ Bash ]

```
$ echo $?
0
```

------
#### [ PowerShell ]

```
PS C:\> $?
True
```

------

You can also use a directory listing command to see that the encrypt command created a new file, `hello.txt.encrypted`\. Because the encrypt command did not specify a file name for the output, the AWS Encryption CLI wrote the output to a file with the same name as the input file plus a `.encrypted` suffix\. To use a different suffix, or suppress the suffix, use the `--suffix` parameter\.

The `hello.txt.encrypted` file contains an [encrypted message](concepts.md#message) that includes the ciphertext of the `hello.txt` file, an encrypted copy of the data key, and additional metadata, including the encryption context\.

------
#### [ Bash ]

```
$  ls
hello.txt  hello.txt.encrypted
```

------
#### [ PowerShell ]

```
PS C:\> dir

    Directory: C:\TestCLI

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        9/15/2017   5:57 PM             11 Hello.txt
-a----        9/17/2017   1:06 PM            585 Hello.txt.encrypted
```

------

## Decrypting a file<a name="cli-example-decrypt-file"></a>

This example uses the AWS Encryption CLI to decrypt the contents of the `Hello.txt.encrypted` file that was encrypted in the previous example\.

The decrypt command uses the `--decrypt` parameter to indicate the operation and `--input` parameter to identify the file to decrypt\. The value of the `--output` parameter is a dot that represents the current directory\. 

The `--wrapping-keys` parameter with a **key** attribute specifies the wrapping key used to decrypt the encrypted message\. The `--wrapping-keys` parameter is required in a decrypt command\. If you are using AWS KMS CMKs, you can use the **key** attribute or the **discovery** attribute\. If you are using a custom master key provider, the **key** attribute is required\.

The `--encryption-context` parameter is optional in the decrypt command, even when an [encryption context](crypto-cli-how-to.md#crypto-cli-encryption-context) is provided in the encrypt command\. In this case, the decrypt command uses the same encryption context that was provided in the encrypt command\. Before decrypting, the AWS Encryption CLI verifies that the encryption context in the encrypted message includes a `purpose=test` pair\. If it does not, the decrypt command fails\.

The `--metadata-output` parameter specifies a file for metadata about the decryption operation\. The value of the `--output` parameter, a dot \(\.\), writes the output file to the current directory\. 

------
#### [ Bash ]

```
\\ To run this example, replace the fictitious key ARN with a valid value.
$ cmkArn=arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab

$ aws-encryption-cli --decrypt \
                     --input hello.txt.encrypted \
                     --wrapping-keys key=$cmkArn \
                     --commitment-policy require-encrypt-require-decrypt \
                     --encryption-context purpose=test \
                     --metadata-output ~/metadata \
                     --output .
```

------
#### [ PowerShell ]

```
\\ To run this example, replace the fictitious key ARN with a valid value.
PS C:\> $cmkArn = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'

PS C:\> aws-encryption-cli --decrypt `
                           --input Hello.txt.encrypted `
                           --wrapping-keys key=$cmkArn `
                           --commitment-policy require-encrypt-require-decrypt `
                           --encryption-context purpose=test `
                           --metadata-output $home\Metadata.txt `
                           --output .
```

------

When a decrypt command succeeds, it does not return any output\. To determine whether the command succeeded, get the value of the `$?` variable\. You can also use a directory listing command to see that the command created a new file with a `.decrypted` suffix\. To see the plaintext content, use a command to get the file content, such as `cat` or [Get\-Content](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-content)\.

------
#### [ Bash ]

```
$  ls
hello.txt  hello.txt.encrypted  hello.txt.encrypted.decrypted

$  cat hello.txt.encrypted.decrypted
Hello World
```

------
#### [ PowerShell ]

```
PS C:\> dir

    Directory: C:\TestCLI

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        9/17/2017   1:01 PM             11 Hello.txt
-a----        9/17/2017   1:06 PM            585 Hello.txt.encrypted
-a----        9/17/2017   1:08 PM             11 Hello.txt.encrypted.decrypted


PS C:\> Get-Content Hello.txt.encrypted.decrypted
Hello World
```

------

## Encrypting all files in a directory<a name="cli-example-encrypt-directory"></a>

This example uses the AWS Encryption CLI to encrypt the contents of all of the files in a directory\. 

When a command affects multiple files, the AWS Encryption CLI processes each file individually\. It gets the file contents, gets a unique [data key](concepts.md#DEK) for the file from the master key, encrypts the file contents under the data key, and writes the results to a new file in the output directory\. As a result, you can decrypt the output files independently\. 

This listing of the `TestDir` directory shows the plaintext files that we want to encrypt\. 

------
#### [ Bash ]

```
$  ls testdir
cool-new-thing.py  hello.txt  employees.csv
```

------
#### [ PowerShell ]

```
PS C:\> dir C:\TestDir

    Directory: C:\TestDir

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        9/12/2017   3:11 PM           2139 cool-new-thing.py
-a----        9/15/2017   5:57 PM             11 Hello.txt
-a----        9/17/2017   1:44 PM             46 Employees.csv
```

------

The first command saves the [Amazon Resource Name \(ARN\)](https://docs.aws.amazon.com/kms/latest/developerguide/viewing-keys.html#find-cmk-id-arn) of an AWS KMS customer master key \(CMK\) in the `$cmkArn` variable\.

The second command encrypts the content of files in the `TestDir` directory and writes the files of encrypted content to the `TestEnc` directory\. If the `TestEnc` directory doesn't exist, the command fails\. Because the input location is a directory, the `--recursive` parameter is required\. 

The [`--wrapping-keys` parameter](crypto-cli-how-to.md#crypto-cli-master-key), and its required **key** attribute, specify the wrapping key to use\. The encrypt command includes an [encryption context](crypto-cli-how-to.md#crypto-cli-encryption-context), `dept=IT`\. When you specify an encryption context in a command that encrypts multiple files, the same encryption context is used for all of the files\. 

The command also has a `--metadata-output` parameter to tell the AWS Encryption CLI where to write the metadata about the encryption operations\. The AWS Encryption CLI writes one metadata record for each file that was encrypted\.

The [`--commitment-policy parameter`](crypto-cli-how-to.md#crypto-cli-commitment-policy) is optional beginning in version 2\.0\.*x*, but it is recommended\. If the command or script fails because it cannot decrypt a ciphertext, the explicit commitment\-policy setting might help you to detect the problem quickly\.

When the command completes, the AWS Encryption CLI writes the encrypted files to the `TestEnc` directory, but it does not return any output\. 

The final command lists the files in the `TestEnc` directory\. There is one output file of encrypted content for each input file of plaintext content\. Because the command did not specify an alternate suffix, the encrypt command appended `.encrypted` to each of the input file names\.

------
#### [ Bash ]

```
# To run this example, replace the fictitious key ARN with a valid master key identifier.
$  cmkArn=arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab

$ aws-encryption-cli --encrypt \
                     --input testdir --recursive\
                     --wrapping-keys key=$cmkArn \
                     --encryption-context dept=IT \
                     --commitment-policy \
                     --metadata-output ~/metadata \
                     --output testenc

$ ls testenc
cool-new-thing.py.encrypted  employees.csv.encrypted  hello.txt.encrypted
```

------
#### [ PowerShell ]

```
# To run this example, replace the fictitious key ARN with a valid master key identifier.
PS C:\> $cmkArn = arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab

PS C:\> aws-encryption-cli --encrypt `
                           --input .\TestDir --recursive `
                           --wrapping-keys key=$cmkArn `
                           --encryption-context dept=IT `
                           --commitment-policy `
                           --metadata-output .\Metadata\Metadata.txt `
                           --output .\TestEnc

PS C:\> dir .\TestEnc

    Directory: C:\TestEnc

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        9/17/2017   2:32 PM           2713 cool-new-thing.py.encrypted
-a----        9/17/2017   2:32 PM            620 Hello.txt.encrypted
-a----        9/17/2017   2:32 PM            585 Employees.csv.encrypted
```

------

## Decrypting all files in a directory<a name="cli-example-decrypt-directory"></a>

This example decrypts all files in a directory\. It starts with the files in the `TestEnc` directory that were encrypted in the previous example\.

------
#### [ Bash ]

```
$  ls testenc
cool-new-thing.py.encrypted  hello.txt.encrypted  employees.csv.encrypted
```

------
#### [ PowerShell ]

```
PS C:\> dir C:\TestEnc

    Directory: C:\TestEnc

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        9/17/2017   2:32 PM           2713 cool-new-thing.py.encrypted
-a----        9/17/2017   2:32 PM            620 Hello.txt.encrypted
-a----        9/17/2017   2:32 PM            585 Employees.csv.encrypted
```

------

This decrypt command decrypts all of the files in the TestEnc directory and writes the plaintext files to the TestDec directory\. The `--wrapping-keys` parameter with a **key** attribute tells the AWS Encryption CLI which CMKs to use to decrypt the files\. The command uses the `--interactive` parameter to tell the AWS Encryption CLI to prompt you before overwriting a file with the same name\.

This command also uses the encryption context that was provided when the files were encrypted\. When decrypting multiple files, the AWS Encryption CLI checks the encryption context of every file\. If the encryption context check on any file fails, the AWS Encryption CLI rejects the file, writes a warning, records the failure in the metadata, and then continues checking the remaining files\. If the AWS Encryption CLI fails to decrypt a file for any other reason, the entire decrypt command fails immediately\. 

In this example, the encrypted messages in all of the input files contain the `dept=IT` encryption context element\. However, if you were decrypting messages with different encryption contexts, you might still be able to verify part of the encryption context\. For example, if some messages had an encryption context of `dept=finance` and others had `dept=IT`, you could verify that the encryption context always contains a `dept` name without specifying the value\. If you wanted to be more specific, you could decrypt the files in separate commands\. 

The decrypt command does not return any output, but you can use a directory listing command to see that it created new files with the `.decrypted` suffix\. To see the plaintext content, use a command to get the file content\.

------
#### [ Bash ]

```
# To run this example, replace the fictitious key ARN with a valid master key identifier.
$ cmkArn=arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab

$ aws-encryption-cli --decrypt \
                     --input testenc --recursive \
                     --wrapping-keys key=$cmkArn \
                     --encryption-context dept=IT \
                     --commitment-policy require-encrypt-require-decrypt \
                     --metadata-output ~/metadata \
                     --output testdec --interactive

$ ls testdec
cool-new-thing.py.encrypted.decrypted  hello.txt.encrypted.decrypted  employees.csv.encrypted.decrypted
```

------
#### [ PowerShell ]

```
# To run this example, replace the fictitious key ARN with a valid master key identifier.
PS C:\> $cmkArn = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'

PS C:\> aws-encryption-cli --decrypt `
                           --input C:\TestEnc --recursive `
                           --wrapping-keys key=$cmkArn `
                           --encryption-context dept=IT `
                           --commitment-policy require-encrypt-require-decrypt `
                           --metadata-output $home\Metadata.txt `
                           --output C:\TestDec --interactive

PS C:\> dir .\TestDec


    Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        10/8/2017   4:57 PM           2139 cool-new-thing.py.encrypted.decrypted
-a----        10/8/2017   4:57 PM             46 Employees.csv.encrypted.decrypted
-a----        10/8/2017   4:57 PM             11 Hello.txt.encrypted.decrypted
```

------

## Encrypting and decrypting on the command line<a name="cli-example-stdin"></a>

These examples show you how to pipe input to commands \(stdin\) and write output to the command line \(stdout\)\. They explain how to represent stdin and stdout in a command and how to use the built\-in Base64 encoding tools to prevent the shell from misinterpreting non\-ASCII characters\.

This example pipes a plaintext string to an encrypt command and saves the encrypted message in a variable\. Then, it pipes the encrypted message in the variable to a decrypt command, which writes its output to the pipeline \(stdout\)\. 

The example consists of three commands:
+ The first command saves the [Amazon Resource Name \(ARN\)](https://docs.aws.amazon.com/kms/latest/developerguide/viewing-keys.html#find-cmk-id-arn) of an AWS KMS customer master key \(CMK\) in the `$cmkArn` variable\.

------
#### [ Bash ]

  ```
  $  cmkArn=arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab
  ```

------
#### [ PowerShell ]

  ```
  PS C:\> $cmkArn = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'
  ```

------

   
+ The second command pipes the `Hello World` string to the encrypt command and saves the result in the `$encrypted` variable\. 

  The `--input` and `--output` parameters are required in all AWS Encryption CLI commands\. To indicate that input is being piped to the command \(stdin\), use a hyphen \(`-`\) for the value of the `--input` parameter\. To send the output to the command line \(stdout\), use a hyphen for the value of the `--output` parameter\. 

  The `--encode` parameter Base64\-encodes the output before returning it\. This prevents the shell from misinterpreting the non\-ASCII characters in the encrypted message\. 

  Because this command is just a proof of concept, we omit the encryption context and suppress the metadata \(`-S`\)\. 

------
#### [ Bash ]

  ```
  $ encrypted=$(echo 'Hello World' | aws-encryption-cli --encrypt -S \
                                                        --input - --output - --encode \
                                                        --wrapping-keys key=$cmkArn )
  ```

------
#### [ PowerShell ]

  ```
  PS C:\> $encrypted = 'Hello World' | aws-encryption-cli --encrypt -S `
                                                          --input - --output - --encode `
                                                          --wrapping-keys key=$cmkArn
  ```

------

   
+ The third command pipes the encrypted message in the `$encrypted` variable to the decrypt command\. 

  This decrypt command uses `--input -` to indicate that input is coming from the pipeline \(stdin\) and `--output -` to send the output to the pipeline \(stdout\)\. \(The input parameter takes the location of the input, not the actual input bytes, so you cannot use the `$encrypted` variable as the value of the `--input` parameter\.\) 

  This example uses the **discovery** attribute of the `--wrapping-keys` parameter to allow the AWS Encryption CLI to use any CMK to decrypt the data\. It doesn't specify a commitment policy, so it uses the default value for version 2\.0\.*x* and later, `require-encrypt-require-decrypt`\.

  Because the output was encrypted and then encoded, the decrypt command uses the `--decode` parameter to decode Base64\-encoded input before decrypting it\. You can also use the `--decode` parameter to decode Base64\-encoded input before encrypting it\.

  Again, the command omits the encryption context and suppresses the metadata \(\-`S`\)\. 

------
#### [ Bash ]

  ```
  $  echo $encrypted | aws-encryption-cli --decrypt --wrapping-keys discovery=true --input - --output - --decode -S
  Hello World
  ```

------
#### [ PowerShell ]

  ```
  PS C:\> $encrypted | aws-encryption-cli --decrypt --wrapping-keys discovery=$true --input - --output - --decode -S
  Hello World
  ```

------

You can also perform the encrypt and decrypt operations in a single command without the intervening variable\. 

As in the previous example, the `--input` and `--output` parameters have a `-` value and the command uses the `--encode` parameter to encode the output and the `--decode` parameter to decode the input\.

------
#### [ Bash ]

```
$  cmkArn=arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab

$  echo 'Hello World' |
          aws-encryption-cli --encrypt --wrapping-keys key=$cmkArn --input - --output - --encode -S |
          aws-encryption-cli --decrypt --wrapping-keys key=discovery=true --input - --output - --decode -S
Hello World
```

------
#### [ PowerShell ]

```
PS C:\> $cmkArn = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'

PS C:\> 'Hello World' |
               aws-encryption-cli --encrypt --wrapping-keys key=$cmkArn --input - --output - --encode -S |
               aws-encryption-cli --decrypt --wrapping-keys discovery=$true --input - --output - --decode -S
Hello World
```

------

## Using multiple master keys<a name="cli-example-multimaster"></a>

This example shows how to use multiple master keys when encrypting and decrypting data in the AWS Encryption CLI\. 

When you use multiple master keys to encrypt data, any one of the master keys can be used to decrypt the data\. This strategy assures that you can decrypt the data even if one of the master keys is unavailable\. If you are storing the encrypted data in multiple AWS Regions, this strategy lets you use a master key in the same Region to decrypt the data\. 

When you encrypt with multiple master keys, the first master key plays a special role\. It generates the data key that is used to encrypt the data\. The remaining master keys encrypt the plaintext data key\. The resulting [encrypted message](concepts.md#message) includes the encrypted data and a collection of encrypted data keys, one for each master key\. Although the first master key generated the data key, any of the master keys can decrypt one of the data keys, which can be used to decrypt the data\. 

**Encrypting with three master keys**

This example command uses three wrapping keys to encrypt the `Finance.log` file, one in each of three AWS Regions\. 

It writes the encrypted message to the `Archive` directory\. The command uses the `--suffix` parameter with no value to suppress the suffix, so the input and output files names will be the same\. 

The command uses the `--wrapping-keys` parameter with three **key** attributes\. You can also use multiple `--wrapping-keys` parameters in the same command\. 

To encrypt the log file, the AWS Encryption CLI asks the first wrapping key in the list, `$cmk1`, to generate the data key that it uses to encrypt the data\. Then, it uses each of the other wrapping keys to encrypt a plaintext copy of the same data key\. The encrypted message in the output file includes all three of the encrypted data keys\. 

------
#### [ Bash ]

```
$ cmk1=arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab
$ cmk2=arn:aws:kms:us-east-2:111122223333:key/0987ab65-43cd-21ef-09ab-87654321cdef
$ cmk3=arn:aws:kms:ap-southeast-1:111122223333:key/1a2b3c4d-5e6f-1a2b-3c4d-5e6f1a2b3c4d

$ aws-encryption-cli --encrypt --input /logs/finance.log \
                               --output /archive --suffix \
                               --encryption-context class=log \
                               --metadata-output ~/metadata \
                               --wrapping-keys key=$cmk1 key=$cmk2 key=$cmk3
```

------
#### [ PowerShell ]

```
PS C:\> $cmk1 = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'
PS C:\> $cmk2 = 'arn:aws:kms:us-east-2:111122223333:key/0987ab65-43cd-21ef-09ab-87654321cdef'
PS C:\> $cmk3 = 'arn:aws:kms:ap-southeast-1:111122223333:key/1a2b3c4d-5e6f-1a2b-3c4d-5e6f1a2b3c4d'

PS C:\> aws-encryption-cli --encrypt --input D:\Logs\Finance.log `
                           --output D:\Archive --suffix `
                           --encryption-context class=log `
                           --metadata-output $home\Metadata.txt `
                           --wrapping-keys key=$cmk1 key=$cmk2 key=$cmk3
```

------

This command decrypts the encrypted copy of the `Finance.log` file and writes it to a `Finance.log.clear` file in the `Finance` directory\. 

To tell the AWS Encryption CLI which AWS KMS CMKs use to decrypt your data, use the **key** attribute of the `--wrapping-keys` parameter\. You must permission to call the [Decrypt API](https://docs.aws.amazon.com/kms/latest/APIReference/API_Decrypt.html) on the CMKs you specify\. For more information, see [ Authentication and Access Control for AWS KMS](https://docs.aws.amazon.com/kms/latest/developerguide/control-access.html)\. 

For example, to decrypt data encrypted under three CMKs, you can specify the same three CMKs or any subset of them\. This example specifies only one of the CMKs\.

------
#### [ Bash ]

```
$ aws-encryption-cli --decrypt --input /archive/finance.log \
                     --wrapping-keys key=$cmk1 \
                     --output /finance --suffix '.clear' \
                     --metadata-output ~/metadata \
                     --encryption-context class=log
```

------
#### [ PowerShell ]

```
PS C:\> aws-encryption-cli --decrypt `
                           --input D:\Archive\Finance.log `
                           --wrapping-keys key=$cmk1 `
                           --output D:\Finance --suffix '.clear' `
                           --metadata-output .\Metadata\Metadata.txt `
                           --encryption-context class=log
```

------

## Encrypting and decrypting in scripts<a name="cli-example-script"></a>

This example shows how to use the AWS Encryption CLI in scripts\. You can write scripts that just encrypt and decrypt data, or scripts that encrypt or decrypt as part of a data management process\.

In this example, the script gets a collection of log files, compresses them, encrypts them, and then copies the encrypted files to an Amazon S3 bucket\. This script processes each file separately, so that you can decrypt and expand them independently\.

When you compress and encrypt files, be sure to compress before you encrypt\. Properly encrypted data is not compressible\.

**Warning**  
Be careful when compressing data that includes both secrets and data that might be controlled by a malicious actor\. The final size of the compressed data might inadvertently reveal sensitive information about its contents\.

------
#### [ PowerShell ]

```
#Requires -Modules AWSPowerShell, Microsoft.PowerShell.Archive
Param
(
    [Parameter(Mandatory)]
    [ValidateScript({Test-Path $_})]
    [String[]]
    $FilePath,

    [Parameter()]
    [Switch]
    $Recurse,

    [Parameter(Mandatory=$true)]
    [String]
    $wrappingKeyID,

    [Parameter()]
    [String]
    $masterKeyProvider = 'aws-kms',

    [Parameter(Mandatory)]
    [ValidateScript({Test-Path $_})]
    [String]
    $ZipDirectory,

    [Parameter(Mandatory)]
    [ValidateScript({Test-Path $_})]
    [String]
    $EncryptDirectory,

    [Parameter()]
    [String]
    $EncryptionContext,

    [Parameter(Mandatory)]
    [ValidateScript({Test-Path $_})]
    [String]
    $MetadataDirectory,

    [Parameter(Mandatory)]
    [ValidateScript({Test-S3Bucket -BucketName $_})]
    [String]
    $S3Bucket,

    [Parameter()]
    [String]
    $S3BucketFolder
)

BEGIN {}
PROCESS {
    if ($files = dir $FilePath -Recurse:$Recurse)
    {

        # Step 1: Compress
        foreach ($file in $files)
        {
            $fileName = $file.Name
            try
            {
                Microsoft.PowerShell.Archive\Compress-Archive -Path $file.FullName -DestinationPath $ZipDirectory\$filename.zip
            }
            catch
            {
                Write-Error "Zip failed on $file.FullName"
            }

            # Step 2: Encrypt
            if (-not (Test-Path "$ZipDirectory\$filename.zip"))
            {
                Write-Error "Cannot find zipped file: $ZipDirectory\$filename.zip"
            }
            else
            {
                # 2>&1 captures command output
                $err = (aws-encryption-cli -e -i "$ZipDirectory\$filename.zip" `
                                           -o $EncryptDirectory `
                                           -m key=$wrappingKeyID provider=$masterKeyProvider `
                                           -c $EncryptionContext `
                                           --metadata-output $MetadataDirectory `
                                           -v) 2>&1

                # Check error status
                if ($? -eq $false)
                {
                    # Write the error
                    $err
                }
                elseif (Test-Path "$EncryptDirectory\$fileName.zip.encrypted")
                {
                    # Step 3: Write to S3 bucket
                    if ($S3BucketFolder)
                    {
                        Write-S3Object -BucketName $S3Bucket -File "$EncryptDirectory\$fileName.zip.encrypted" -Key "$S3BucketFolder/$fileName.zip.encrypted"

                    }
                    else
                    {
                        Write-S3Object -BucketName $S3Bucket -File "$EncryptDirectory\$fileName.zip.encrypted"
                    }
                }
            }
        }
    }
}
```

------
#### [ Bash ]

```
# Continue running even if an operation fails.
set +e

dir=$1
encryptionContext=$2
s3bucket=$3
s3folder=$4
masterKeyProvider="aws-kms"
metadataOutput="/tmp/metadata-$(date +%s)"

compress(){
    gzip -qf $1
}

encrypt(){
    # -e encrypt
    # -i input
    # -o output
    # --metadata-output unique file for metadata
    # -m masterKey read from environment variable
    # -c encryption context read from the second argument.
    # -v be verbose
    aws-encryption-cli -e -i ${1} -o $(dirname ${1}) --metadata-output ${metadataOutput} -m key="${masterKey}" provider="${masterKeyProvider}" -c "${encryptionContext}" -v
}


s3put (){
    # copy file argument 1 to s3 location passed into the script.
    aws s3 cp ${1} ${s3bucket}/${s3folder}
}

# Validate all required arguments are present.
if [ "${dir}" ] && [ "${encryptionContext}" ] && [ "${s3bucket}" ] && [ "${s3folder}" ] && [ "${masterKey}" ]; then

# Is $dir a valid directory?
test -d "${dir}"
if [ $? -ne 0 ]; then
    echo "Input is not a directory; exiting"
    exit 1
fi

# Iterate over all the files in the directory, except *gz and *encrypted (in case of a re-run).
for f in $(find ${dir} -type f \( -name "*" ! -name \*.gz ! -name \*encrypted \) ); do
    echo "Working on $f"
    compress ${f}
    encrypt ${f}.gz
    rm -f ${f}.gz
    s3put ${f}.gz.encrypted
done;
else
    echo "Arguments: <Directory> <encryption context> <s3://bucketname> <s3 folder>"
    echo " and ENV var \$masterKey must be set"
    exit 255
fi
```

------

## Using data key caching<a name="cli-example-caching"></a>

This example uses [data key caching](data-key-caching.md) in a command that encrypts a large number of files\. 

By default, the AWS Encryption CLI \(and other versions of the AWS Encryption SDK\) generates a unique data key for each file that it encrypts\. Although using a unique data key for each operation is a cryptographic best practice, limited reuse of data keys is acceptable for some situations\. If you are considering data key caching, consult with a security engineer to understand the security requirements of your application and determine security thresholds that are right for you\. 

In this example, data key caching speeds up the encryption operation by reducing the frequency of requests to the master key provider\.

The command in this example encrypts a large directory with multiple subdirectories that contain a total of approximately 800 small log files\. The first command saves the ARN of the CMK in a `cmkARN` variable\. The second command encrypts all of the files in the input directory \(recursively\) and writes them to an archive directory\. The command uses the `--suffix` parameter to specify the `.archive` suffix\. 

The `--caching` parameter enables data key caching\. The **capacity** attribute, which limits the number of data keys in the cache, is set to 1, because serial file processing never uses more than one data key at a time\. The **max\_age** attribute, which determines how long the cached data key can used, is set to 10 seconds\. 

The optional **max\_messages\_encrypted** attribute is set to 10 messages, so a single data key is never used to encrypt more than 10 files\. Limiting the number of files encrypted by each data key reduces the number of files that would be affected in the unlikely event that a data key was compromised\.

To run this command on log files that your operating system generates, you might need administrator permissions \(`sudo` in Linux; **Run as Administrator** in Windows\)\.

------
#### [ Bash ]

```
$  cmkArn=arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab

$  aws-encryption-cli --encrypt \
                      --input /var/log/httpd --recursive \
                      --output ~/archive --suffix .archive \
                      --wrapping-keys key=$cmkArn \
                      --encryption-context class=log \
                      --suppress-metadata \
                      --caching capacity=1 max_age=10 max_messages_encrypted=10
```

------
#### [ PowerShell ]

```
PS C:\> $cmkARN = 'arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab'

PS C:\> aws-encryption-cli --encrypt `
                           --input C:\Windows\Logs --recursive `
                           --output $home\Archive --suffix '.archive' `
                           --wrapping-keys key=$cmkARN `
                           --encryption-context class=log `
                           --suppress-metadata `
                           --caching capacity=1 max_age=10 max_messages_encrypted=10
```

------

To test the effect of data key caching, this example uses the [Measure\-Command](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/measure-command) cmdlet in PowerShell\. When you run this example without data key caching, it takes about 25 seconds to complete\. This process generates a new data key for each file in the directory\.

```
PS C:\> Measure-Command {aws-encryption-cli --encrypt `
                                            --input C:\Windows\Logs --recursive `
                                            --output $home\Archive  --suffix '.archive' `
                                            --wrapping-keys key=$cmkARN `
                                            --encryption-context class=log `
                                            --suppress-metadata }


Days              : 0
Hours             : 0
Minutes           : 0
Seconds           : 25
Milliseconds      : 453
Ticks             : 254531202
TotalDays         : 0.000294596298611111
TotalHours        : 0.00707031116666667
TotalMinutes      : 0.42421867
TotalSeconds      : 25.4531202
TotalMilliseconds : 25453.1202
```

Data key caching makes the process quicker, even when you limit each data key to a maximum of 10 files\. The command now takes less than 12 seconds to complete and reduces the number of calls to the master key provider to 1/10 of the original value\.

```
PS C:\> Measure-Command {aws-encryption-cli --encrypt `
                                            --input C:\Windows\Logs --recursive `
                                            --output $home\Archive  --suffix '.archive' `
                                            --wrapping-keys key=$cmkARN `
                                            --encryption-context class=log `
                                            --suppress-metadata `
                                            --caching capacity=1 max_age=10 max_messages_encrypted=10}


Days              : 0
Hours             : 0
Minutes           : 0
Seconds           : 11
Milliseconds      : 813
Ticks             : 118132640
TotalDays         : 0.000136727592592593
TotalHours        : 0.00328146222222222
TotalMinutes      : 0.196887733333333
TotalSeconds      : 11.813264
TotalMilliseconds : 11813.264
```

If you eliminate the `max_messages_encrypted` restriction, all files are encrypted under the same data key\. This change increases the risk of reusing data keys without making the process much faster\. However, it reduces the number of calls to the master key provider to 1\.

```
PS C:\> Measure-Command {aws-encryption-cli --encrypt `
                                            --input C:\Windows\Logs --recursive `
                                            --output $home\Archive  --suffix '.archive' `
                                            --wrapping-keys key=$cmkARN `
                                            --encryption-context class=log `
                                            --suppress-metadata `
                                            --caching capacity=1 max_age=10}


Days              : 0
Hours             : 0
Minutes           : 0
Seconds           : 10
Milliseconds      : 252
Ticks             : 102523367
TotalDays         : 0.000118661304398148
TotalHours        : 0.00284787130555556
TotalMinutes      : 0.170872278333333
TotalSeconds      : 10.2523367
TotalMilliseconds : 10252.3367
```