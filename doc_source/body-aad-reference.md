# Body additional authenticated data \(AAD\) reference for the AWS Encryption SDK<a name="body-aad-reference"></a>


|  | 
| --- |
|  The information on this page is a reference for building your own encryption library that is compatible with the AWS Encryption SDK\. If you are not building your own compatible encryption library, you likely do not need this information\. To use the AWS Encryption SDK in one of the supported programming languages, see [Programming languages](programming-languages.md)\. For the specification that defines the elements of a proper AWS Encryption SDK implementation, see the *AWS Encryption SDK Specification* in the [aws\-encryption\-sdk\-specification](https://github.com/awslabs/aws-encryption-sdk-specification/) repository in GitHub\.  | 

You must provide additional authenticated data \(AAD\) to the [AES\-GCM algorithm](algorithms-reference.md) for each cryptographic operation\. This is true for both framed and nonframed [body data](message-format.md#body-structure)\. For more information about AAD and how it is used in Galois/Counter Mode \(GCM\), see [Recommendations for Block Cipher Modes of Operations: Galois/Counter Mode \(GCM\) and GMAC](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-38d.pdf)\.

The following table describes the fields that form the body AAD\. The bytes are appended in the order shown\.


**Body AAD Structure**  

| Field | Length, in bytes | 
| --- | --- | 
| [Message ID](#body-aad-message-id) | 16 | 
| [Body AAD Content](#body-aad-content) | Variable\. See Body AAD Content in the following list\. | 
| [Sequence Number](#body-aad-sequence-number) | 4 | 
| [Content Length](#body-aad-content-length) | 8 | 

**Message ID**  <a name="body-aad-message-id"></a>
The same [Message ID](message-format.md#header-message-id) value set in the message header\.

**Body AAD Content**  <a name="body-aad-content"></a>
A UTF\-8 encoded value determined by the type of body data used\.  
For [nonframed data](message-format.md#body-no-framing), use the value `AWSKMSEncryptionClient Single Block`\.  
For regular frames in [framed data](message-format.md#body-framing), use the value `AWSKMSEncryptionClient Frame`\.  
For the final frame in [framed data](message-format.md#body-framing), use the value `AWSKMSEncryptionClient Final Frame`\.

**Sequence Number**  <a name="body-aad-sequence-number"></a>
A 4\-byte value interpreted as a 32\-bit unsigned integer\.  
For [framed data](message-format.md#body-framing), this is the frame sequence number\.  
For [nonframed data](message-format.md#body-no-framing), use the value 1, encoded as the 4 bytes `00 00 00 01` in hexadecimal notation\.

**Content Length**  <a name="body-aad-content-length"></a>
The length, in bytes, of the plaintext data provided to the algorithm for encryption\. It is an 8\-byte value interpreted as a 64\-bit unsigned integer\.