# Troubleshooting migration to version 2\.0\.*x*<a name="troubleshooting-migration"></a>

Before updating your application to version 2\.0\.*x* of the AWS Encryption SDK, update to version 1\.7\.*x* and deploy it completely\. That will help you avoid most errors you might encounter when updating to version 2\.0\.*x*\. For detailed guidance, including examples, see [Migrating to version 2\.0\.*x*](migration.md)\.

This topic is designed to help you recognize and resolve the most common errors you might encounter\.

**Topics**
+ [Deprecated or removed objects](#deprecated-removed)
+ [Configuration conflict: Commitment policy and algorithm suite](#configuration-conflict_1)
+ [Configuration conflict: Commitment policy and ciphertext](#configuration-conflict_2)
+ [Key commitment validation failed](#commitment-failed)
+ [Other encryption failures](#encrypt-failed)
+ [Other decryption failures](#decrypt-failed)
+ [Rollback considerations](#migration-rollback)

## Deprecated or removed objects<a name="deprecated-removed"></a>

Version 2\.0\.*x* includes several breaking changes, including removing legacy constructors, methods, functions, and classes that were deprecated in version 1\.7\.*x*\. To avoid compiler errors, import errors, and symbol not found errors \(depending on your programming language\), upgrade first to version 1\.7\.*x* of the AWS Encryption SDK\. While using version 1\.7\.*x*, you can begin using the replacement elements before the original symbols are removed\.

If you need to upgrade to version 2\.0\.*x* immediately, [consult the changelog](about-versions.md) for your programming language, and replace the legacy symbols with the symbols the changelog recommends\.

## Configuration conflict: Commitment policy and algorithm suite<a name="configuration-conflict_1"></a>

If you specify an algorithm suite that conflicts with your [commitment policy](concepts.md#commitment-policy), the call to encrypt fails with a *Configuration conflict* error\.

To avoid this type of error, don't specify an algorithm suite\. By default, the AWS Encryption SDK chooses the most secure algorithm that is compatible with your commitment policy\. However, if you must specify an algorithm suite, such as one without signing, be sure to choose an algorithm suite that is compatible with your commitment policy\.


| Commitment policy | Compatible algorithm suites | 
| --- | --- | 
| ForbidEncryptAllowDecrypt | Any algorithm suite *without* key commitment, such as:AES\_256\_GCM\_IV12\_TAG16\_HKDF\_SHA384\_ECDSA\_P384 \([03 78](algorithms-reference.md)\) \(with signing\) `AES_256_GCM_IV12_TAG16_HKDF_SHA256` \([01 78](algorithms-reference.md)\) \(without signing\) | 
| RequireEncryptAllowDecryptRequireEncryptRequireDecrypt | Any algorithm suite *with* key commitment, such as:AES\_256\_GCM\_HKDF\_SHA512\_COMMIT\_KEY\_ECDSA\_P384 \([05 78](algorithms-reference.md)\) \(with signing\) `AES_256_GCM_HKDF_SHA512_COMMIT_KEY` \([04 78](algorithms-reference.md)\) \(without signing\) | 

If you encounter this error when you have not specified an algorithm suite, the conflicting algorithm suite might have been chosen by your [cryptographic materials manager](concepts.md#crypt-materials-manager) \(CMM\)\. The Default CMM won't select a conflicting algorithm suite, but a custom CMM might\. For help, consult the documentation for your custom CMM\.

## Configuration conflict: Commitment policy and ciphertext<a name="configuration-conflict_2"></a>

The `RequireEncryptRequireDecrypt` [commitment policy](concepts.md#commitment-policy) does not permit the AWS Encryption SDK to decrypt a message that was encrypted without [key commitment](concepts.md#key-commitment)\. If you ask the AWS Encryption SDK to decrypt a message without key commitment, it returns a *Configuration conflict* error\.

To avoid this error, before setting the `RequireEncryptRequireDecrypt` commitment policy, be sure that all ciphertexts encrypted without key commitment are decrypted and re\-encrypted with key commitment, or handled by a different application\. If you encounter this error, you can return an error for the conflicting ciphertext or change your commitment policy temporarily to `RequireEncryptAllowDecrypt`\.

If you are encountering this error because you upgraded to version 2\.0\.*x* from a version earlier than 1\.7\.0, you might consider [rolling back](#migration-rollback) to version 1\.7\.*x* and deploying that version to all hosts before upgrading to version 2\.0\.*x*\. For help, see [How to migrate and deploy the AWS Encryption SDK](migration-guide.md)\.

## Key commitment validation failed<a name="commitment-failed"></a>

When you decrypt messages that are encrypted with key commitment, you might get a *Key commitment validation failed* error message\. This indicates that the decrypt call failed because a data key in an [encrypted message](concepts.md#DEK) is not identical to the unique data key for the message\. By validating the data key during decryption, [key commitment](concepts.md#key-commitment) protects you from decrypting a message that might result in more than one plaintext\. 

This error indicates that the encrypted message that you were trying to decrypt was not returned by the AWS Encryption SDK\. It might be a manually crafted message or the result of data corruption\. If you encounter this error, your application can reject the message and continue, or stop processing new messages\.

## Other encryption failures<a name="encrypt-failed"></a>

Encryption can fail for multiple reasons\. You cannot use an [AWS KMS discovery keyring](choose-keyring.md#kms-keyring-discovery) or a [master key provider in discovery mode](migrate-mkps-v2.md) to encrypt a message\. 

Be sure that you specify a keyring or master key provider with wrapping keys that you have [permission to use](choose-keyring.md#kms-keyring-permissions) for encryption\. For help with permissions on AWS KMS customer master keys, see [Viewing a key policy](https://docs.aws.amazon.com/kms/latest/developerguide/key-policy-viewing.html) and [Determining access to an AWS KMS customer master key](https://docs.aws.amazon.com/kms/latest/developerguide/determining-access.html) in the *AWS Key Management Service Developer Guide*\.

## Other decryption failures<a name="decrypt-failed"></a>

If your attempt to decrypt an encrypted message fails, it means that the AWS Encryption SDK could not \(or would not\) decrypt any of the encrypted data keys in the message\. 

If you used a keyring or master key provider that specifies wrapping keys, the AWS Encryption SDK uses only the wrapping keys you specify\. Verify that you are using the wrapping keys that you intend and that you have `kms:Decrypt` permission on at least one of the wrapping keys\. If you are using AWS KMS customer master keys, as a fallback, you can try decrypting the message with an [AWS KMS discovery keyring](choose-keyring.md#kms-keyring-discovery) or a [master key provider in discovery mode](migrate-mkps-v2.md)\. If the operation succeeds, before returning the plaintext, verify that the key used to decrypt the message is one that you trust\. 

## Rollback considerations<a name="migration-rollback"></a>

If your application is failing to encrypt or decrypt data, you can usually resolve the problem by updating the code symbols, keyrings, master key providers, or [commitment policy](concepts.md#commitment-policy)\. However, in some cases, you might decide that it's best to roll back your application to a previous version of the AWS Encryption SDK\.

If you must roll back, do so with caution\. Versions of the AWS Encryption SDK prior to 1\.7\.*x* cannot decrypt ciphertext encrypted with [key commitment](concepts.md#key-commitment)\.
+ Rolling back from version 1\.7\.*x* to a previous version of the AWS Encryption SDK is generally safe\. You might have to undo changes you made to your code to use symbols and objects that are not supported in previous versions\. 
+ Once you have begun encrypting with key commitment \(setting your commitment policy to `RequireEncryptAllowDecrypt`\) in version 2\.0\.*x*, you can roll back to version 1\.7\.*x*, but not to any earlier version\. Versions of the AWS Encryption SDK prior to 1\.7\.*x* cannot decrypt ciphertext encrypted with [key commitment](concepts.md#key-commitment)\.

If you accidentally enable encrypting with key commitment before all hosts can decrypt with key commitment, it might be best to continue with the roll out rather than to roll back\. If messages are transient or can be safely dropped, then you might consider a rollback with loss of messages\. If a rollback is required, you might consider writing a tool that decrypts and re\-encrypts all messages\.