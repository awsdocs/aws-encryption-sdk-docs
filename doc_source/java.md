# AWS Encryption SDK for Java<a name="java"></a>

This topic explains how to install and use the AWS Encryption SDK for Java\. For details about programming with the AWS Encryption SDK for Java, see the [aws\-encryption\-sdk\-java](https://github.com/aws/aws-encryption-sdk-java/) repository on GitHub\. For API documentation, see the [Javadoc](https://aws.github.io/aws-encryption-sdk-java/) for the AWS Encryption SDK for Java\.

**Topics**
+ [Prerequisites](#java-prerequisites)
+ [Installation](#java-installation)
+ [Example code](java-example-code.md)

## Prerequisites<a name="java-prerequisites"></a>

Before you install the AWS Encryption SDK for Java, be sure you have the following prerequisites\.

**A Java development environment**  
You will need Java 8 or later\. On the Oracle website, go to [Java SE Downloads](https://www.oracle.com/technetwork/java/javase/downloads/index.html), and then download and install the Java SE Development Kit \(JDK\)\.  
If you use the Oracle JDK, you must also download and install the [Java Cryptography Extension \(JCE\) Unlimited Strength Jurisdiction Policy Files](http://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html)\.

**Bouncy Castle**  
The AWS Encryption SDK for Java requires [Bouncy Castle](https://www.bouncycastle.org/java.html)\.   
+ AWS Encryption SDK for Java versions 1\.6\.1 and later use Bouncy Castle to serialize and deserialize cryptographic objects\. You can use Bouncy Castle or [Bouncy Castle FIPS](https://www.bouncycastle.org/fips_faq.html) to satisfy this requirement\. For help installing and configuring Bouncy Castle FIPS, see [BC FIPS Documentation](https://www.bouncycastle.org/documentation.html), especially the **User Guides** and **Security Policy** PDFs\.
+ Earlier versions of the AWS Encryption SDK for Java use Bouncy Castle's cryptography API for Java\. This requirement is satisfied only by non\-FIPS Bouncy Castle\.
If you don't have Bouncy Castle, go to [Bouncy Castle latest releases](https://bouncycastle.org/latest_releases.html) to download the provider file that corresponds to your JDK\. You can also use [Apache Maven](https://maven.apache.org/) to get the artifact for the standard Bouncy Castle provider \([bcprov\-ext\-jdk15on](https://mvnrepository.com/artifact/org.bouncycastle/bcprov-ext-jdk15on)\) or the artifact for Bouncy Castle FIPS \([bc\-fips](https://mvnrepository.com/artifact/org.bouncycastle/bc-fips)\)\.

**AWS SDK for Java \(Optional\)**  
The AWS Encryption SDK for Java does not require the AWS SDK for Java\. However, the AWS SDK for Java is required to use [AWS Key Management Service](https://aws.amazon.com/kms/) \(AWS KMS\) as a master key provider\. It's also required for some of the Java code examples in this guide\.  
Beginning in the AWS Encryption SDK for Java version 2\.4\.0, the AWS Encryption SDK for Java supports both version 1\.x and 2\.x of the AWS SDK for Java\. AWS Encryption SDK code for the AWS SDK for Java 1\.x and 2\.x are interoperable\. For example, you can encrypt data with AWS Encryption SDK code that supports AWS SDK for Java 1\.x and decrypt it using code that supports AWS SDK for Java 2\.x \(or vice versa\)\. Versions of the AWS Encryption SDK for Java earlier than 2\.4\.0 support only AWS SDK for Java 1\.x\. For information about updating your version of the AWS Encryption SDK, see [Migrating to AWS Encryption SDK Versions 2\.0\.*x* and later](migration.md)\.  
When updating your AWS Encryption SDK for Java code from the AWS SDK for Java 1\.x to AWS SDK for Java 2\.x, replace references to the [`AWSKMS` interface](https://docs.aws.amazon.com/AWSJavaSDK/latest/javadoc/com/amazonaws/services/kms/package-summary.html) in AWS SDK for Java 1\.x with references to the [`KmsClient` interface](https://sdk.amazonaws.com/java/api/latest/software/amazon/awssdk/services/kms/package-summary.html) in AWS SDK for Java 2\.x\. Also, update your code to use the AWS KMS\-related objects in the `kmssdkv2` namespace, instead of the `kms` namespace\.   
To install the AWS SDK for Java, use Apache Maven\.   
+ To [import the entire AWS SDK for Java](https://docs.aws.amazon.com/sdk-for-java/latest/developer-guide/setup-project-maven.html#build-the-entire-sdk-into-your-project) as a dependency, declare it in your `pom.xml` file\.
+ To create a dependency only for the AWS KMS module in AWS SDK for Java 1\.x, follow the instructions for [specifying particular modules](https://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/setup-project-maven.html#modules-dependencies), and set the `artifactId` to `aws-java-sdk-kms`\.
+ To create a dependency only for the AWS KMS module in AWS SDK for Java 2\.x, follow the instructions for [specifying particular modules](https://docs.aws.amazon.com/sdk-for-java/latest/developer-guide/setup-project-maven.html#modules-dependencies)\. Set the `groupId` to `software.amazon.awssdk` and the `artifactId` to `kms`\.
For more changes, see [What's different between the AWS SDK for Java 1\.x and 2\.x](https://docs.aws.amazon.com/sdk-for-java/latest/developer-guide/migration-whats-different.html) in the AWS SDK for Java 2\.x Developer Guide\.  
Java examples in the AWS Encryption SDK Developer Guide use the AWS SDK for Java 2\.x\.

## Installation<a name="java-installation"></a>

You can install the AWS Encryption SDK for Java in the following ways\.

**Manually**  
To install the AWS Encryption SDK for Java, clone or download the [aws\-encryption\-sdk\-java](https://github.com/aws/aws-encryption-sdk-java/) GitHub repository\.

**Using Apache Maven**  
The AWS Encryption SDK for Java is available through [Apache Maven](https://maven.apache.org/) with the following dependency definition\.  

```
<dependency>
  <groupId>com.amazonaws</groupId>
  <artifactId>aws-encryption-sdk-java</artifactId>
  <version>2.0.0</version>
</dependency>
```

After you install the SDK, get started by looking at the [example Java code](java-example-code.md) in this guide and the [Javadoc on GitHub](https://aws.github.io/aws-encryption-sdk-java/)\.