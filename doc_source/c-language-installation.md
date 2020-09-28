# Installing the AWS Encryption SDK for C<a name="c-language-installation"></a>

This topic explains how to install the AWS Encryption SDK for C on all supported platforms\. If you're having trouble with your installation, file an issue on the [aws\-encryption\-sdk\-c](https://github.com/aws/aws-encryption-sdk-c/issues) repository or use the** Feedback** link in the lower\-right corner of this page\.

Before you begin, decide whether you want to use [AWS Key Management Service](https://docs.aws.amazon.com/kms/latest/developerguide/) \(AWS KMS\) to generate and protect the encryption keys that protect your data\. Otherwise, you need to provide master keys that you generate and protect\. If you use AWS KMS, you need to install the AWS SDK for C\+\+\. 

If you don't use hardware security modules \(HSMs\) or another encryption key management service, we recommend AWS KMS\. You need to create at least one AWS KMS [customer master key](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#master_keys)\. For instructions, see [Creating Keys](https://docs.aws.amazon.com/kms/latest/developerguide/create-keys.html)\.

**Topics**
+ [Required libraries](#c-build-dependencies)
+ [Install the AWS Encryption SDK for C](#c-language-how-to-install)
+ [Compile and build](#c-language-compile)

## Required libraries<a name="c-build-dependencies"></a>

The AWS Encryption SDK for C requires the following libraries:
+ [OpenSSL](https://www.openssl.org/) 1\.0\.2 or greater, including 1\.1\.0 or later – The AWS Encryption SDK uses the encryption algorithms and other cryptographic primitives in the OpenSSL cryptography library\. The SDK doesn't define or create any cryptographic primitives\.
+ [aws\-c\-common](https://github.com/awslabs/aws-c-common/) 0\.4\.42 or later – This library defines general\-purpose tools and structures that the AWS Encryption SDK for C uses\. It's installed automatically when you install the AWS SDK for C\+\+\.
**Note**  
The preferred version of the `aws-c-common` library is installed automatically when you install the AWS SDK for C\+\+\. If you're installing the SDK for C\+\+, don't install the `aws-c-common` library separately\. 

To build the SDK, you need:
+ A C compiler for your platform\.
+ [CMake](https://cmake.org/) 3\.9 or later\.

To use the AWS Encryption SDK for C with AWS KMS, you need the following:
+ [AWS SDK for C\+\+](https://docs.aws.amazon.com/sdk-for-cpp/latest/developer-guide/) 1\.8\.32 or later – The AWS Encryption SDK for C uses the AWS SDK for C\+\+ to interact with AWS KMS\. To use AWS KMS to protect your encryption keys, and to run many of the examples in this guide and in the repository, install the SDK for C\+\+\. This C\+\+ SDK requires a C\+\+ compiler and the `curl` tool\. 

## Install the AWS Encryption SDK for C<a name="c-language-how-to-install"></a>

Use the following procedure to install the required components and build the AWS Encryption SDK for C on your preferred platform\.

------
#### [ Amazon Linux ]

These instructions install the AWS Encryption SDK for C on an Amazon Elastic Compute Cloud \(Amazon EC2\) instance with Amazon Linux 1\. 

1. 

**Install CMake 3\.9 or later\.**

   [Download and install CMake](https://cmake.org/download/) 3\.9 or later\. Then, confirm that `CMake` is in your path\. Don't use the earlier version of CMake that `yum` installs\. 

1. 

**Install the dependent libraries\.**

   These instructions install the tools that the AWS Encryption SDK requires, including the `aws-c-common` library, which includes many of the basic functions that the AWS Encryption SDK for C uses\.

    

   Run option **A** or **B**, but not both\.

    

   If you plan to use a [KMS keyring](choose-keyring.md#use-kms-keyring), or run the [examples](c-examples.md) in the AWS Encryption SDK for C that use the AWS KMS keyring, use installation option **A**\. This option installs the AWS SDK for C\+\+, which is required for the AWS Encryption SDK to interact with AWS Key Management Service \(AWS KMS\)\. Installing the SDK for C\+\+ automatically installs the `aws-c-common` library for you\.

    

   **Option A: Install with AWS SDK for C\+\+ \(for AWS KMS\)**\.

   The AWS SDK for C\+\+ is required to use the AWS Encryption SDK for C with AWS KMS\. It is also required for many of the [examples](c-examples.md) in this guide and the [examples](https://github.com/aws/aws-encryption-sdk-c/tree/master/examples) in the `aws-encryption-sdk-c` repository\.

    

   Run the following commands to download and install the dependent libraries for the AWS Encryption SDK and SDK for C\+\+\.

   ```
   sudo yum update -y
   sudo yum install -y openssl-devel git libcurl-devel gcc-c++
   ```

   Next, to download and install the SDK for C\+\+, change to your preferred build directory and run the following commands\. When you install the SDK for C\+\+, the `aws-c-common` is installed automatically\. *Do not install it again*\. 

    

   These commands install only the AWS KMS modules of the SDK for C\+\+\. If you are using other libraries in the SDK for C\+\+, you can omit the `-DBUILD_ONLY="kms"` parameter, but it might take an extended amount of time to install\.

   ```
   git clone https://github.com/aws/aws-sdk-cpp.git
   mkdir build-aws-sdk-cpp && cd build-aws-sdk-cpp
   cmake -DBUILD_SHARED_LIBS=ON -DBUILD_ONLY="kms" -DENABLE_UNITY_BUILD=ON ../aws-sdk-cpp
   make && sudo make install ; cd ..
   ```

   **Option B: Install without SDK for C\+\+**\.

   Use these installation instructions only if you are not using the AWS Encryption SDK for C with AWS KMS or running the examples that use AWS KMS\. These components do not allow you to use the AWS Encryption SDK for C with AWS KMS\.

    

   Run the following commands to download and install the dependent libraries for the AWS Encryption SDK\.

   ```
   sudo yum update -y
   sudo yum install -y openssl-devel git gcc
   ```

   Next, to download and install the [aws\-c\-common](https://github.com/awslabs/aws-c-common/) library, change to your preferred build directory, and run the following commands\.

   ```
   git clone https://github.com/awslabs/aws-c-common.git
   mkdir build-aws-c-common && cd build-aws-c-common
   cmake -DBUILD_SHARED_LIBS=ON ../aws-c-common
   make && sudo make install ; cd ..
   ```

1. 

**Install the AWS Encryption SDK\.**

   Change to your preferred directory\. Then, run the following commands to install and build the AWS Encryption SDK for C\. 

   ```
   git clone https://github.com/aws/aws-encryption-sdk-c.git
   mkdir build-aws-encryption-sdk-c && cd build-aws-encryption-sdk-c
   cmake -DBUILD_SHARED_LIBS=ON ../aws-encryption-sdk-c
   make && sudo make install ; cd ..
   ```

------
#### [ macOS ]

These instructions install the AWS Encryption SDK in standard directories in `/usr/local` using the [Homebrew](https://brew.sh/) package management system\. If you don't have it yet, install `Homebrew`, then follow these instructions\.

1. 

**Install the dependent libraries\.**

   The following command installs the required versions of OpenSSL and CMake\. 

   ```
   brew install openssl@1.1 cmake
   ```

1. 

**Install the aws\-c\-common library and \(optionally\) the AWS SDK for C\+\+\.**

   These instructions install the tools that the AWS Encryption SDK requires, including the `aws-c-common` library, which includes many of the basic functions that the AWS Encryption SDK for C uses\.

    

   Run option **A** or **B**, but not both\.

    

   If you plan to use a [KMS keyring](choose-keyring.md#use-kms-keyring), or run the [examples](c-examples.md) in the AWS Encryption SDK for C that use the AWS KMS keyring, use installation option **A**\. This option installs the AWS SDK for C\+\+, which is required for the AWS Encryption SDK to interact with AWS Key Management Service \(AWS KMS\)\. Installing the SDK for C\+\+ automatically installs the `aws-c-common` library for you\.

    

   **Option A: Install with AWS SDK for C\+\+** \(for AWS KMS\)\.

   The AWS SDK for C\+\+ is required to use the AWS Encryption SDK for C with AWS KMS\. It is also required for many of the [examples](c-examples.md) in this guide and the [examples](https://github.com/aws/aws-encryption-sdk-c/tree/master/examples) in the `aws-encryption-sdk-c` repository\.

    

   Run the following commands to download and install the dependent libraries for the AWS Encryption SDK and SDK for C\+\+\.

   When you install the SDK for C\+\+, the `aws-c-common` is installed automatically\. *Do not install it again*\.

   ```
   git clone https://github.com/aws/aws-sdk-cpp.git
   mkdir build-aws-sdk-cpp && cd build-aws-sdk-cpp
   cmake -DBUILD_SHARED_LIBS=ON -DBUILD_ONLY="kms" -DENABLE_UNITY_BUILD=ON ../aws-sdk-cpp
   make && sudo make install ; cd ..
   ```

   **Option B: Install without SDK for C\+\+**\.

   Use these installation instructions only if you aren't using the AWS Encryption SDK for C with AWS KMS or running the examples that use AWS KMS\. These components don't allow you to use the AWS Encryption SDK for C with AWS KMS\.

    

   Run the following commands to install the `aws-c-common` library that the AWS Encryption SDK for C requires\.

   ```
   git clone https://github.com/awslabs/aws-c-common.git
   mkdir build-aws-c-common && cd build-aws-c-common
   cmake -DBUILD_SHARED_LIBS=ON ../aws-c-common
   make && sudo make install ; cd ..
   ```

1. 

**Install the AWS Encryption SDK\.**

   Change to your preferred directory\. Then, run the following commands to install and build the AWS Encryption SDK for C\. These commands specify the location of the OpenSSL library, because HomeBrew doesn't install it in the default location\.

   ```
   git clone https://github.com/aws/aws-encryption-sdk-c.git
   mkdir build-aws-encryption-sdk-c && cd build-aws-encryption-sdk-c
   cmake -DBUILD_SHARED_LIBS=ON -DOPENSSL_ROOT_DIR=/usr/local/opt/openssl\@1.1 ../aws-encryption-sdk-c
   make && sudo make install ; cd ..
   ```

------
#### [ Ubuntu ]

These instructions install the AWS Encryption SDK for C on Ubuntu\.

1. 

**Install dependent libraries\.**

   These instructions install the tools that the AWS Encryption SDK requires, including the `aws-c-common` library, which includes many of the basic functions that the AWS Encryption SDK for C uses\.

    

   Run option **A** or **B**, but not both\.

    

   If you plan to use a [KMS keyring](choose-keyring.md#use-kms-keyring), or run the [examples](c-examples.md) in the AWS Encryption SDK for C that use the AWS KMS keyring, use installation option **A**\. This option installs the AWS SDK for C\+\+, which is required for the AWS Encryption SDK to interact with AWS Key Management Service \(AWS KMS\)\. Installing the SDK for C\+\+ automatically installs the `aws-c-common` library for you\.

    

   **Option A: Install with AWS SDK for C\+\+** \(for AWS KMS\)\.

   The AWS SDK for C\+\+ is required to use the AWS Encryption SDK for C with AWS KMS\. It is also required for many of the [examples](c-examples.md) in this guide and the [examples](https://github.com/aws/aws-encryption-sdk-c/tree/master/examples) in the `aws-encryption-sdk-c` repository\.

    

   Run the following commands to download and install the dependent libraries for the AWS Encryption SDK and SDK for C\+\+\.

   ```
   sudo apt-get update
   sudo apt-get install -y libssl-dev cmake g++ libcurl4-openssl-dev zlib1g-dev
   ```

   Next, to download and install the SDK for C\+\+, change to your preferred build directory and run the following commands\. When you install the SDK for C\+\+, the `aws-c-common` is installed automatically\. *Do not install it again*\.

    

   These commands install only the AWS KMS modules of the SDK for C\+\+\. If you are using other libraries in the SDK for C\+\+, you can omit the `-DBUILD_ONLY="kms"` parameter, but it might take an extended amount of time to install\.

   ```
   git clone https://github.com/aws/aws-sdk-cpp.git
   mkdir build-aws-sdk-cpp && cd build-aws-sdk-cpp
   cmake -DBUILD_SHARED_LIBS=ON -DBUILD_ONLY="kms" -DENABLE_UNITY_BUILD=ON ../aws-sdk-cpp
   make && sudo make install ; cd ..
   ```

   **Option B: Install without SDK for C\+\+**\.

   Use these installation instructions only if you are not using the AWS Encryption SDK for C with AWS KMS or running the examples that use AWS KMS\. These components do not allow you to use the AWS Encryption SDK for C with AWS KMS\.

    

   Run the following commands to download and install the dependent libraries for the AWS Encryption SDK\.

   ```
   sudo apt-get update
   sudo apt-get install -y libssl-dev cmake gcc
   ```

    Next, to download and install the [aws\-c\-common](https://github.com/awslabs/aws-c-common) library, change to your preferred build directory, and run the following commands\. 

   ```
   git clone https://github.com/awslabs/aws-c-common.git
   mkdir build-aws-c-common && cd build-aws-c-common
   cmake -DBUILD_SHARED_LIBS=ON ../aws-c-common
   make && sudo make install ; cd ..
   ```

1. 

**Install the AWS Encryption SDK\.**

   Run the following commands to install and build the AWS Encryption SDK for C\. 

   ```
   git clone https://github.com/aws/aws-encryption-sdk-c.git
   mkdir build-aws-encryption-sdk-c && cd build-aws-encryption-sdk-c
   cmake -DBUILD_SHARED_LIBS=ON ../aws-encryption-sdk-c
   make && sudo make install ; cd ..
   ```

------
#### [ Windows ]

The following instructions install the AWS Encryption SDK and its dependencies\. They require [Visual Studio 15 or later](https://docs.microsoft.com/en-us/visualstudio/install/install-visual-studio), [Git for Windows](https://git-scm.com/download/win), and the [Universal C Runtime Library](https://docs.microsoft.com/en-us/cpp/porting/upgrade-your-code-to-the-universal-crt)\. Run the following commands in the **x64 Native Tools Command Prompt for Visual Studio** in your preferred build and installation directory\.

1. 

**Install the dependent libraries\.**

   These commands install the [vcpkg](https://github.com/Microsoft/vcpkg) library manager\. Then, use `vcpkg` to install `curl` and OpenSSL\.

   ```
   mkdir install && mkdir build && cd build
   git clone https://github.com/Microsoft/vcpkg.git
   cd vcpkg && .\bootstrap-vcpkg.bat
   .\vcpkg install curl:x64-windows openssl:x64-windows && cd ..
   ```

1. 

**Install the aws\-c\-common library and \(optionally\) the AWS SDK for C\+\+\.**

   These instructions install the tools that the AWS Encryption SDK requires, including the `aws-c-common` library, which includes many of the basic functions that the AWS Encryption SDK for C uses\.

    

   Run option **A** or **B**, but not both\.

    

   If you plan to use a [KMS keyring](choose-keyring.md#use-kms-keyring), or run the [examples](c-examples.md) in the AWS Encryption SDK for C that use the AWS KMS keyring, use installation option **A**\. This option installs the AWS SDK for C\+\+, which is required for the AWS Encryption SDK to interact with AWS Key Management Service \(AWS KMS\)\. Installing the SDK for C\+\+ automatically installs the `aws-c-common` library for you\.

    

   **Option A: With AWS SDK for C\+\+** for AWS KMS

   The AWS SDK for C\+\+ is required to use the AWS Encryption SDK for C with AWS KMS\. It is also required for many of the [examples](c-examples.md) in this guide and the [examples](https://github.com/aws/aws-encryption-sdk-c/tree/master/examples) in the `aws-encryption-sdk-c` repository\.

    

   The following commands install only the AWS KMS modules of the SDK for C\+\+\. If you are using other libraries in the SDK for C\+\+, you can omit the `-DBUILD_ONLY="kms"` parameter, but it might take an extended amount of time to install\.

    

   Run the following commands in your preferred build directory\. When you install the SDK for C\+\+, the `aws-c-common` is installed automatically\. *Do not install it again*\. 

   ```
   git clone https://github.com/aws/aws-sdk-cpp.git
   mkdir build-aws-sdk-cpp && cd build-aws-sdk-cpp
   cmake -DCMAKE_INSTALL_PREFIX=%cd%\..\..\install -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=ON -DENABLE_UNITY_BUILD=ON -DBUILD_ONLY=kms -DCMAKE_TOOLCHAIN_FILE=%cd%\..\vcpkg\scripts\buildsystems\vcpkg.cmake -G Ninja ..\aws-sdk-cpp
   cmake --build . && cmake --build . --target install && cd ..
   ```

   **Option B: Without SDK for C\+\+**\.

   Use these installation instructions only if you aren't using the AWS Encryption SDK for C with AWS KMS or running the examples that use AWS KMS\.

    

   Run the following commands in your preferred build directory\. These commands install and build the `aws-c-common` library\. These components don't allow you to use the AWS Encryption SDK for C with AWS KMS\.

   ```
   git clone https://github.com/awslabs/aws-c-common.git
   mkdir build-aws-c-common && cd build-aws-c-common
   cmake -DCMAKE_INSTALL_PREFIX=%cd%\..\..\install -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=ON -DCMAKE_TOOLCHAIN_FILE=%cd%\..\vcpkg\scripts\buildsystems\vcpkg.cmake -G Ninja ..\aws-c-common
   cmake --build . && cmake --build . --target install && cd ..
   ```

1. 

**Install the AWS Encryption SDK\.**

   Change to your preferred directory\. Then, run the following commands to install and build the AWS Encryption SDK for C\. 

   ```
   git clone https://github.com/aws/aws-encryption-sdk-c.git
   mkdir build-aws-encryption-sdk-c && cd build-aws-encryption-sdk-c
   cmake -DCMAKE_INSTALL_PREFIX=%cd%\..\..\install -DCMAKE_BUILD_TYPE=Release -DBUILD_SHARED_LIBS=ON -DCMAKE_TOOLCHAIN_FILE=%cd%\..\vcpkg\scripts\buildsystems\vcpkg.cmake -G Ninja ..\aws-encryption-sdk-c
   cmake --build . && cmake --build . --target install && cd ..
   ```

------

## Compile and build<a name="c-language-compile"></a>

After you install the SDK, you can start building and using it\. This guide includes topics that help you to [understand keyrings](choose-keyring.md#using-keyrings), [choose a keyring](choose-keyring.md), learn the basic [programming patterns](c-language-using.md#c-language-using-pattern), and work through a [simple example](c-examples.md#c-example-strings)\. 

For help compiling and building your applications, see the [README file](https://github.com/aws/aws-encryption-sdk-c/blob/master/README.md) in the `aws-encryption-sdk-c` repository\. 