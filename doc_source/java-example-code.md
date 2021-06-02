# AWS Encryption SDK for Java example code<a name="java-example-code"></a>

The following examples show you how to use the AWS Encryption SDK for Java to encrypt and decrypt data\. These examples show how to use [version 2\.0\.*x*](about-versions.md) and later of the AWS Encryption SDK for Java\. For examples that use earlier versions, find your release in the [Releases](https://github.com/aws/aws-encryption-sdk-java/releases) list of the [aws\-encryption\-sdk\-java](https://github.com/aws/aws-encryption-sdk-java/) repository on GitHub\.



**Topics**
+ [Strings](#java-example-strings)
+ [Byte streams](#java-example-streams)
+ [Byte streams with multiple master key providers](#java-example-multiple-providers)

## Encrypting and decrypting strings<a name="java-example-strings"></a>

The following example shows you how to use the AWS Encryption SDK for Java to encrypt and decrypt strings\. Before using the string, convert it into a byte array\.

This example uses an [AWS Key Management Service \(AWS KMS\)](https://aws.amazon.com/kms/) customer master key \(CMK\) as the wrapping key\. When encrypting, the `KmsMasterKeyProvider` `buildStrict()` method takes a key ID, key ARN, alias name, or alias ARN\. When decrypting, [it requires a key ARN](concepts.md#specifying-keys)\. In this case, because the `keyArn` parameter is used for encrypting and decrypting, its value must be a key ARN\. For information about IDs for AWS KMS keys, see [Key identifiers](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#key-id) in the *AWS Key Management Service Developer Guide*\.

When you call the `encryptData()` method, it returns an [encrypted message](concepts.md#message) \(`CryptoResult`\) that includes the ciphertext, the encrypted data keys, and the encryption context\. When you call `getResult` on the `CryptoResult` object, it returns a base\-64\-encoded string version of the [encrypted message](message-format.md) that you can pass to the `decryptData()` method\.

Similarly, when you call `decryptData()`, the `CryptoResult` object it returns contains the plaintext message and a CMK ID\. Before your application returns the plaintext, verify that the CMK ID and the encryption context in the encrypted message are the ones that you expect\.

```
// Copyright Amazon.com Inc. or its affiliates. All Rights Reserved.
// SPDX-License-Identifier: Apache-2.0

package com.amazonaws.crypto.examples;

import java.nio.charset.StandardCharsets;
import java.util.Arrays;
import java.util.Collections;
import java.util.Map;

import com.amazonaws.encryptionsdk.AwsCrypto;
import com.amazonaws.encryptionsdk.CryptoResult;
import com.amazonaws.encryptionsdk.kms.KmsMasterKey;
import com.amazonaws.encryptionsdk.kms.KmsMasterKeyProvider;
import com.amazonaws.encryptionsdk.CommitmentPolicy;

/**
 * <p>
 * Encrypts and then decrypts data using an AWS KMS customer master key.
 *
 * <p>
 * Arguments:
 * <ol>
 * <li>Key ARN: For help finding the Amazon Resource Name (ARN) of your AWS KMS customer master
 *    key (CMK), see 'Viewing Keys' at http://docs.aws.amazon.com/kms/latest/developerguide/viewing-keys.html
 * </ol>
 */
public class BasicEncryptionExample {

    private static final byte[] EXAMPLE_DATA = "Hello World".getBytes(StandardCharsets.UTF_8);

    public static void main(final String[] args) {
        final String keyArn = args[0];

        encryptAndDecrypt(keyArn);
    }

    static void encryptAndDecrypt(final String keyArn) {
        // 1. Instantiate the SDK
        // This builds the AwsCrypto client with the RequireEncryptRequireDecrypt commitment policy,
        // which enforces that this client only encrypts using committing algorithm suites and enforces
        // that this client will only decrypt encrypted messages that were created with a committing algorithm suite.
        // This is the default commitment policy if you build the client with `AwsCrypto.builder().build()`
        // or `AwsCrypto.standard()`.
        final AwsCrypto crypto = AwsCrypto.builder()
                .withCommitmentPolicy(CommitmentPolicy.RequireEncryptRequireDecrypt)
                .build();

        // 2. Instantiate an AWS KMS master key provider in strict mode using buildStrict().
        // In strict mode, the AWS KMS master key provider encrypts and decrypts only by using the key
        // indicated by keyArn.
        // To encrypt and decrypt with this master key provider, use an AWS KMS key ARN to identify the CMKs.
        // In strict mode, the decrypt operation requires a key ARN.
        final KmsMasterKeyProvider keyProvider = KmsMasterKeyProvider.builder().buildStrict(keyArn);

        // 3. Create an encryption context
        // Most encrypted data should have an associated encryption context
        // to protect integrity. This sample uses placeholder values.
        // For more information see:
        // blogs.aws.amazon.com/security/post/Tx2LZ6WBJJANTNW/How-to-Protect-the-Integrity-of-Your-Encrypted-Data-by-Using-AWS-Key-Management
        final Map<String, String> encryptionContext = Collections.singletonMap("ExampleContextKey", "ExampleContextValue");

        // 4. Encrypt the data
        final CryptoResult<byte[], KmsMasterKey> encryptResult = crypto.encryptData(keyProvider, EXAMPLE_DATA, encryptionContext);
        final byte[] ciphertext = encryptResult.getResult();

        // 5. Decrypt the data
        final CryptoResult<byte[], KmsMasterKey> decryptResult = crypto.decryptData(keyProvider, ciphertext);

        // 6. Verify that the encryption context in the result contains the
        // encryption context supplied to the encryptData method. Because the
        // SDK can add values to the encryption context, don't require that
        // the entire context matches.
        if (!encryptionContext.entrySet().stream()
                .allMatch(e -> e.getValue().equals(decryptResult.getEncryptionContext().get(e.getKey())))) {
            throw new IllegalStateException("Wrong Encryption Context!");
        }

        // 7. Verify that the decrypted plaintext matches the original plaintext
        assert Arrays.equals(decryptResult.getResult(), EXAMPLE_DATA);
    }
}
```

## Encrypting and decrypting byte streams<a name="java-example-streams"></a>

The following example shows you how to use the AWS Encryption SDK to encrypt and decrypt byte streams\. This example does not use AWS\. It uses the Java Cryptography Extension \(JCE\) to protect the master key\.

When encrypting, this example uses the `AwsCrypto.builder() .withEncryptionAlgorithm()` method to specify an algorithm suite without [digital signatures](concepts.md#digital-sigs)\. When decrypting, to ensure that the ciphertext is unsigned, this example uses the `createUnsignedMessageDecryptingStream()` method\. The `createUnsignedMessageDecryptingStream()` method, introduced in AWS Encryption SDK 1\.9\.*x* and 2\.2\.*x*, fails if it encounters a ciphertext with a digital signature\. 

If you're encrypting with the default algorithm suite, which includes digital signatures, use the `createDecryptingStream()` method instead, as shown in the next example\.

```
// Copyright Amazon.com Inc. or its affiliates. All Rights Reserved.
// SPDX-License-Identifier: Apache-2.0

package com.amazonaws.crypto.examples;

import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.security.SecureRandom;
import java.util.Collections;
import java.util.Map;

import javax.crypto.SecretKey;
import javax.crypto.spec.SecretKeySpec;

import com.amazonaws.encryptionsdk.AwsCrypto;
import com.amazonaws.encryptionsdk.CryptoAlgorithm;
import com.amazonaws.encryptionsdk.CryptoInputStream;
import com.amazonaws.encryptionsdk.MasterKey;
import com.amazonaws.encryptionsdk.jce.JceMasterKey;
import com.amazonaws.encryptionsdk.CommitmentPolicy;
import com.amazonaws.util.IOUtils;

/**
 * <p>
 * Encrypts and then decrypts a file under a random key.
 *
 * <p>
 * Arguments:
 * <ol>
 * <li>Name of file containing plaintext data to encrypt
 * </ol>
 *
 * <p>
 * This program demonstrates using a standard Java {@link SecretKey} object as a {@link MasterKey} to
 * encrypt and decrypt streaming data.
 */
public class FileStreamingExample {
    private static String srcFile;

    public static void main(String[] args) throws IOException {
        srcFile = args[0];

        // In this example, we generate a random key. In practice, 
        // you would get a key from an existing store
        SecretKey cryptoKey = retrieveEncryptionKey();

        // Create a JCE master key provider using the random key and an AES-GCM encryption algorithm
        JceMasterKey masterKey = JceMasterKey.getInstance(cryptoKey, "Example", "RandomKey", "AES/GCM/NoPadding");

        // Instantiate the SDK.
        // This builds the AwsCrypto client with the RequireEncryptRequireDecrypt commitment policy,
        // which enforces that this client only encrypts using committing algorithm suites and enforces
        // that this client will only decrypt encrypted messages that were created with a committing algorithm suite.
        // This is the default commitment policy if you build the client with `AwsCrypto.builder().build()`
        // or `AwsCrypto.standard()`.
        // This also chooses to encrypt with an algorithm suite that doesn't include signing for faster decryption.
        // The use case assumes that the contexts that encrypt and decrypt are equally trusted.
        final AwsCrypto crypto = AwsCrypto.builder()
                .withCommitmentPolicy(CommitmentPolicy.RequireEncryptRequireDecrypt)
                .withEncryptionAlgorithm(CryptoAlgorithm.ALG_AES_256_GCM_HKDF_SHA512_COMMIT_KEY)
                .build();

        // Create an encryption context to identify this ciphertext
        Map<String, String> context = Collections.singletonMap("Example", "FileStreaming");

        // Because the file might be to large to load into memory, we stream the data, instead of 
        //loading it all at once.
        FileInputStream in = new FileInputStream(srcFile);
        CryptoInputStream<JceMasterKey> encryptingStream = crypto.createEncryptingStream(masterKey, in, context);
        FileOutputStream out = new FileOutputStream(srcFile + ".encrypted");
        IOUtils.copy(encryptingStream, out);
        encryptingStream.close();
        out.close();

        // Decrypt the file. Verify the encryption context before returning the plaintext.
        in = new FileInputStream(srcFile + ".encrypted");
        // Since we encrypted using an unsigned algorithm suite, we can use the recommended
        // createUnsignedMessageDecryptingStream method that only accepts unsigned messages.
        CryptoInputStream<JceMasterKey> decryptingStream = crypto.createUnsignedMessageDecryptingStream(masterKey, in);
        // Does it contain the expected encryption context?
        if (!"FileStreaming".equals(decryptingStream.getCryptoResult().getEncryptionContext().get("Example"))) {
            throw new IllegalStateException("Bad encryption context");
        }

        // Write the plaintext data to disk.
        out = new FileOutputStream(srcFile + ".decrypted");
        IOUtils.copy(decryptingStream, out);
        decryptingStream.close();
        out.close();
    }

    /**
     * In practice, this key would be saved in a secure location. 
     * For this demo, we generate a new random key for each operation.
     */
    private static SecretKey retrieveEncryptionKey() {
        SecureRandom rnd = new SecureRandom();
        byte[] rawKey = new byte[16]; // 128 bits
        rnd.nextBytes(rawKey);
        return new SecretKeySpec(rawKey, "AES");
    }
}
```

## Encrypting and decrypting byte streams with multiple master key providers<a name="java-example-multiple-providers"></a>

The following example shows you how to use the AWS Encryption SDK with more than one master key provider\. Using more than one master key provider creates redundancy if one master key provider is unavailable for decryption\. This example uses a CMK in [AWS KMS](https://aws.amazon.com/kms/) and an RSA key pair as the master keys\.

This example encrypts with the [default algorithm suite](supported-algorithms.md), which includes a [digital signature](concepts.md#digital-sigs)\. When streaming, the AWS Encryption SDK releases plaintext after integrity checks, but before it has verified the digital signature\. To avoid using the plaintext until the signature is verified, this example buffers the plaintext, and writes it to disk only when decryption and verification are complete\. 

```
// Copyright Amazon.com Inc. or its affiliates. All Rights Reserved.
// SPDX-License-Identifier: Apache-2.0

package com.amazonaws.crypto.examples;

import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.security.GeneralSecurityException;
import java.security.KeyPair;
import java.security.KeyPairGenerator;
import java.security.PrivateKey;
import java.security.PublicKey;

import com.amazonaws.encryptionsdk.AwsCrypto;
import com.amazonaws.encryptionsdk.CryptoOutputStream;
import com.amazonaws.encryptionsdk.MasterKeyProvider;
import com.amazonaws.encryptionsdk.jce.JceMasterKey;
import com.amazonaws.encryptionsdk.kms.KmsMasterKeyProvider;
import com.amazonaws.encryptionsdk.multi.MultipleProviderFactory;
import com.amazonaws.encryptionsdk.CommitmentPolicy;
import com.amazonaws.util.IOUtils;

/**
 * <p>
 * Encrypts a file using both AWS KMS and an asymmetric key pair.
 *
 * <p>
 * Arguments:
 * <ol>
 * <li>Key ARN: For help finding the Amazon Resource Name (ARN) of your AWS KMS customer master
 *    key (CMK), see 'Viewing Keys' at http://docs.aws.amazon.com/kms/latest/developerguide/viewing-keys.html
 *
  * <li>Name of file containing plaintext data to encrypt
 * </ol>
 *
 * You might use AWS Key Management Service (AWS KMS) for most encryption and decryption operations, but
 * still want the option of decrypting your data offline independently of AWS KMS. This sample
 * demonstrates one way to do this.
 * 
 * The sample encrypts data under both an AWS KMS customer master key (CMK) and an "escrowed" RSA key pair
 * so that either key alone can decrypt it. You might commonly use the AWS KMS CMK for decryption. However,
 * at any time, you can use the private RSA key to decrypt the ciphertext independent of AWS KMS.
 *
 * This sample uses the JCEMasterKey class to generate a RSA public-private key pair
 * and saves the key pair in memory. In practice, you would store the private key in a secure offline 
 * location, such as an offline HSM, and distribute the public key to your development team.
 *
 */
public class EscrowedEncryptExample {
    private static PublicKey publicEscrowKey;
    private static PrivateKey privateEscrowKey;

    public static void main(final String[] args) throws Exception {
        // This sample generates a new random key for each operation.
        // In practice, you would distribute the public key and save the private key in secure
        // storage.
        generateEscrowKeyPair();

        final String kmsArn = args[0];
        final String fileName = args[1];

        standardEncrypt(kmsArn, fileName);
        standardDecrypt(kmsArn, fileName);

        escrowDecrypt(fileName);
    }

    private static void standardEncrypt(final String kmsArn, final String fileName) throws Exception {
        // Encrypt with the KMS CMK and the escrowed public key
        // 1. Instantiate the SDK
        // This builds the AwsCrypto client with the RequireEncryptRequireDecrypt commitment policy,
        // which enforces that this client only encrypts using committing algorithm suites and enforces
        // that this client will only decrypt encrypted messages that were created with a committing algorithm suite.
        // This is the default commitment policy if you build the client with `AwsCrypto.builder().build()`
        // or `AwsCrypto.standard()`.
        final AwsCrypto crypto = AwsCrypto.builder()
                .withCommitmentPolicy(CommitmentPolicy.RequireEncryptRequireDecrypt)
                .build();

        // 2. Instantiate an AWS KMS master key provider in strict mode using buildStrict()
        //
        // In strict mode, the AWS KMS master key provider encrypts and decrypts only by using the key
        // indicated by kmsArn.
        final KmsMasterKeyProvider keyProvider = KmsMasterKeyProvider.builder().buildStrict(kmsArn);
        
        // 3. Instantiate a JCE master key provider
        // Because the user does not have access to the private escrow key,
        // they pass in "null" for the private key parameter.
        final JceMasterKey escrowPub = JceMasterKey.getInstance(publicEscrowKey, null, "Escrow", "Escrow",
                "RSA/ECB/OAEPWithSHA-512AndMGF1Padding");

        // 4. Combine the providers into a single master key provider
        final MasterKeyProvider<?> provider = MultipleProviderFactory.buildMultiProvider(keyProvider, escrowPub);

        // 5. Encrypt the file
        // To simplify the code, we omit the encryption context. Production code should always 
        // use an encryption context. For an example, see the other SDK samples.
        final FileInputStream in = new FileInputStream(fileName);
        final FileOutputStream out = new FileOutputStream(fileName + ".encrypted");
        final CryptoOutputStream<?> encryptingStream = crypto.createEncryptingStream(provider, out);
        // Buffer the data in memory before writing to disk. This ensures verfication of the digital signature before returning plaintext.
        IOUtils.copy(in, encryptingStream);
        in.close();
        encryptingStream.close();
    }

    private static void standardDecrypt(final String kmsArn, final String fileName) throws Exception {
        // Decrypt with the AWS KMS CMK and the escrow public key. You can use a combined provider,
        // as shown here, or just the AWS KMS master key provider.

        // 1. Instantiate the SDK.
        // This builds the AwsCrypto client with the RequireEncryptRequireDecrypt commitment policy,
        // which enforces that this client only encrypts using committing algorithm suites and enforces
        // that this client will only decrypt encrypted messages that were created with a committing algorithm suite.
        // This is the default commitment policy if you build the client with `AwsCrypto.builder().build()`
        // or `AwsCrypto.standard()`.
        final AwsCrypto crypto = AwsCrypto.builder()
                .withCommitmentPolicy(CommitmentPolicy.RequireEncryptRequireDecrypt)
                .build();

        // 2. Instantiate an AWS KMS master key provider in strict mode using buildStrict()
        //
        // In strict mode, the AWS KMS master key provider encrypts and decrypts only by using the key
        // indicated by kmsArn.
        final KmsMasterKeyProvider keyProvider = KmsMasterKeyProvider.builder().buildStrict(kmsArn);
        
        // 3. Instantiate a JCE master key provider
        // Because the user does not have access to the private 
        // escrow key, they pass in "null" for the private key parameter.
        final JceMasterKey escrowPub = JceMasterKey.getInstance(publicEscrowKey, null, "Escrow", "Escrow",
                "RSA/ECB/OAEPWithSHA-512AndMGF1Padding");

        // 4. Combine the providers into a single master key provider
        final MasterKeyProvider<?> provider = MultipleProviderFactory.buildMultiProvider(keyProvider, escrowPub);

        // 5. Decrypt the file
        // To simplify the code, we omit the encryption context. Production code should always 
        // use an encryption context. For an example, see the other SDK samples.
        final FileInputStream in = new FileInputStream(fileName + ".encrypted");
        final FileOutputStream out = new FileOutputStream(fileName + ".decrypted");
        // Since we are using a signing algorithm suite, we avoid streaming plaintext directly to the output file.
        // This ensures that the trailing signature is verified before writing any untrusted plaintext to disk.
        final ByteArrayOutputStream plaintextBuffer = new ByteArrayOutputStream();
        final CryptoOutputStream<?> decryptingStream = crypto.createDecryptingStream(provider, plaintextBuffer);
        IOUtils.copy(in, decryptingStream);
        in.close();
        decryptingStream.close();
        final ByteArrayInputStream plaintextReader = new ByteArrayInputStream(plaintextBuffer.toByteArray());
        IOUtils.copy(plaintextReader, out);
        out.close();
    }

    private static void escrowDecrypt(final String fileName) throws Exception {
        // You can decrypt the stream using only the private key.
        // This method does not call AWS KMS.

        // 1. Instantiate the SDK
        final AwsCrypto crypto = AwsCrypto.standard();

        // 2. Instantiate a JCE master key provider
        // This method call uses the escrowed private key, not null 
        final JceMasterKey escrowPriv = JceMasterKey.getInstance(publicEscrowKey, privateEscrowKey, "Escrow", "Escrow",
                "RSA/ECB/OAEPWithSHA-512AndMGF1Padding");

        // 3. Decrypt the file
        // To simplify the code, we omit the encryption context. Production code should always 
        // use an encryption context. For an example, see the other SDK samples.
        final FileInputStream in = new FileInputStream(fileName + ".encrypted");
        final FileOutputStream out = new FileOutputStream(fileName + ".deescrowed");
        final CryptoOutputStream<?> decryptingStream = crypto.createDecryptingStream(escrowPriv, out);
        IOUtils.copy(in, decryptingStream);
        in.close();
        decryptingStream.close();

    }

    private static void generateEscrowKeyPair() throws GeneralSecurityException {
        final KeyPairGenerator kg = KeyPairGenerator.getInstance("RSA");
        kg.initialize(4096); // Escrow keys should be very strong
        final KeyPair keyPair = kg.generateKeyPair();
        publicEscrowKey = keyPair.getPublic();
        privateEscrowKey = keyPair.getPrivate();

    }
}
```