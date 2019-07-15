# AWS Encryption SDK for Java<a name="java"></a>

This topic explains how to install and use the AWS Encryption SDK for Java\. For details about programming with the AWS Encryption SDK for Java, see the [aws\-encryption\-sdk\-java](https://github.com/aws/aws-encryption-sdk-java/) repository on GitHub\. For API documentation, see the [Javadoc](https://aws.github.io/aws-encryption-sdk-java/javadoc/) for the AWS Encryption SDK for Java\.

**Topics**
+ [Prerequisites](#java-prerequisites)
+ [Installation](#java-installation)
+ [Example Code](java-example-code.md)

## Prerequisites<a name="java-prerequisites"></a>

Before you install the AWS Encryption SDK for Java, be sure you have the following prerequisites\.

**A Java development environment**  
You will need Java 8 or later\. On the Oracle website, go to [Java SE Downloads](https://www.oracle.com/technetwork/java/javase/downloads/index.html), and then download and install the Java SE Development Kit \(JDK\)\.  
If you use the Oracle JDK, you must also download and install the [Java Cryptography Extension \(JCE\) Unlimited Strength Jurisdiction Policy Files](http://www.oracle.com/technetwork/java/javase/downloads/jce8-download-2133166.html)\.

**Bouncy Castle**  
Bouncy Castle provides a cryptography API for Java\. If you don't have Bouncy Castle, go to [Bouncy Castle latest releases](https://bouncycastle.org/latest_releases.html) to download the provider file that corresponds to your JDK\.  
If you use [Apache Maven](https://maven.apache.org/), Bouncy Castle is available with the following dependency definition\.  

```
<dependency>
  <groupId>org.bouncycastle</groupId>
  <artifactId>bcprov-ext-jdk15on</artifactId>
  <version>1.58</version>
</dependency>
```

**AWS SDK for Java \(Optional\)**  
Although you don't need the AWS SDK for Java to use the AWS Encryption SDK for Java, you do need it to use [AWS Key Management Service \(AWS KMS\)](https://aws.amazon.com/kms/) as a master key provider, and to use some of the [example Java code](java-example-code.md) in this guide\. For more information about installing and configuring the AWS SDK for Java, see [AWS SDK for Java](https://aws.amazon.com/sdk-for-java/)\.

## Installation<a name="java-installation"></a>

You can install the AWS Encryption SDK for Java in the following ways\.

**Manually**  
To install the AWS Encryption SDK for Java, clone or download the [aws\-encryption\-sdk\-java GitHub repository](https://github.com/aws/aws-encryption-sdk-java/)\.

**Using Apache Maven**  
The AWS Encryption SDK for Java is available through [Apache Maven](https://maven.apache.org/) with the following dependency definition\.  

```
<dependency>
  <groupId>com.amazonaws</groupId>
  <artifactId>aws-encryption-sdk-java</artifactId>
  <version>1.6.0</version>
</dependency>
```

After you install the SDK, get started by looking at the [example Java code](java-example-code.md) in this guide and the [Javadoc on GitHub](https://aws.github.io/aws-encryption-sdk-java/javadoc/)\.