# Body Additional Authenticated Data \(AAD\) Reference for the AWS Encryption SDK<a name="body-aad-reference"></a>


|  | 
| --- |
|  The information on this page is a reference for building your own encryption library that is compatible with the AWS Encryption SDK\. If you are not building your own compatible encryption library, you likely do not need this information\. To use the AWS Encryption SDK in one of the supported programming languages, see [Programming Languages](programming-languages.md)\.  | 

Regardless of which type of body data is used to form the message body \(non\-framed or framed\), you must provide additional authenticated data \(AAD\) to the AES\-GCM algorithm for each cryptographic operation\. For more information about AAD, see the definition section in [the Galois/Counter Mode of Operation \(GCM\) specification](http://csrc.nist.gov/groups/ST/toolkit/BCM/documents/proposedmodes/gcm/gcm-spec.pdf)\.

The following table describes the fields that form the body AAD\. The bytes are appended in the order shown\.


**Body AAD Structure**  

| Field | Length, in bytes | 
| --- | --- | 
| [Message ID](#body-aad-message-id) | 16 | 
| [Body AAD Content](#body-aad-content) | Variable\. See Body AAD Content in the following list\. | 
| [Sequence Number](#body-aad-sequence-number) | 4 | 
| [Content Length](#body-aad-content-length) | 8 | 

**Message ID**  
The same [Message ID](message-format.md#header-message-id) value set in the message header\.

**Body AAD Content**  
A UTF\-8 encoded value determined by the type of body data used\.  
For non\-framed data, use the value `AWSKMSEncryptionClient Single Block`\.  
For regular frames in framed data, use the value `AWSKMSEncryptionClient Frame`\.  
For the final frame in framed data, use the value `AWSKMSEncryptionClient Final Frame`\.

**Sequence Number**  
A 4\-byte value interpreted as a 32\-bit unsigned integer\.  
For framed data, this is the frame sequence number\.  
For non\-framed data, use the value 1, encoded as the 4 bytes `00 00 00 01` in hexadecimal notation\.

**Content Length**  
The length, in bytes, of the plaintext data provided to the algorithm for encryption\. It is an 8\-byte value interpreted as a 64\-bit unsigned integer\.