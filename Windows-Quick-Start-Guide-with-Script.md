# Windows Quick Start Guide with Script

This guide provides step-by-step instructions to set up the Alexa Voice Service (AVS) Device SDK on 64-bit Windows. When finished, you'll have a working sample app to test interactions with Alexa.

**Table of Contents**
* [1. Install and configure dependencies for the SDK](#1-install-and-configure-dependencies-for-the-sdk)
* [2. Build the SDK](#2-build-the-sdk)
* [3. Obtain credentials and set up your local auth server](#3-obtain-credentials-and-set-up-your-local-auth-server)
* [4. Run the sample app](#4-run-the-sample-app)
* [5. Setup shortcuts](#5-setup-shortcuts)
* [Common Issues](#common-issues)
* [Next Steps](#next-steps)

**WARNING**: This guide doesn't include instructions to enable wake word.

## 1. Install and configure dependencies for the SDK

### 1.1 Download and install MSYS2

1. Download and run the MSYS2 (64-bit) installer. MSYS2 is a software distribution and building platform for Windows. This will install three different shells: MSYS2, MinGW32, and MinGW64. You will use the MinGW64 shell in the steps below.
[Install MSYS2 for Windows 64-bit (x86_64)](http://www.msys2.org/)

2. Update Pacman, the package management system included with MSYS2. The latest version of Pacman is required to build the SDK. Open the MinGW64 shell, and run this command:

    ```
    pacman -Syu
    ```

3. Close MinGW64. Reopen MinGW64, and run this command to finish updating Pacman:

    ```
    pacman -Su
    ```

## 2. Obtain credentials and set up your local auth server
Before we get started, you'll need to register a device and create a security profile at developer.amazon.com. Click [here](https://github.com/alexa/alexa-avs-sample-app/wiki/Create-Security-Profile) for step-by-step instructions.

**IMPORTANT**: The allowed origin under web settings should be `http://localhost:3000` and `https://localhost:3000`. The return URL under web settings should be `http://localhost:3000/authresponse` and `https://localhost:3000/authresponse`.

If you already have a registered product that you can use for testing, feel free to skip ahead.

## 3. Setup and Run
1. Open the MinGW64 shell, and run this command to download the installation script and configuration file:
    ```
    wget https://raw.githubusercontent.com/alexa/avs-device-sdk/master/tools/Install/setup.sh && wget https://raw.githubusercontent.com/alexa/avs-device-sdk/master/tools/Install/mingw.sh &&wget https://raw.githubusercontent.com/alexa/avs-device-sdk/master/tools/Install/config.txt
    ```
    Note: we recommend running these commands from your home directory (`C:/msys64/home/<user_name>`) or your desktop, however, you can run the script anywhere.

2. Using your favorite text editor, update the `config.txt` file with the **Client ID**, **Client Secret**, and **Product ID** for your registered product and **save**.

3. Using the MinGW64 shell, run the setup script with your configuration as an argument:
    ```
    bash setup.sh config.txt
    ```

4. After the setup script has finished running, you'll need to generate an authorization token. Run this command:
    ```
    bash startauth.sh
    ```
    
5. Navigate to [http://localhost:3000](http://localhost:3000). Log in with your Amazon credentials, and follow the instructions provided.

6. Last and most importantly, let's run the sample app using the `startsample.bat` file. Note: this script is a batch file, and not a bash script. You can run the script either from the Windows command line, or by using the Windows File Explorer to locate the file and then double-clicking it.

7. You can also run integration and unit tests:
    `bash test.sh`

## 4. Optional configurations
**To run the sample app manually**:
Open the MinGW64 shell. Run the following commands:
`cd <msys64_installed_path>/alexa_sdk/build/bin`
`./SampleApp.exe ../Integration/AlexaClientSDKConfig.json DEBUG9`

**To run the sample app using the Windows command line**:
Add `<msys64_installed_path>/mingw64/bin` into the path. Note: For this option, use `mingw32-make.exe` instead of `make`.

**To build the SDK after making custom changes**:
Open the MinGW64 shell, and run `make` inside of the the `alexa_sdk/build` folder.

## Common Issues

See the [Troubleshooting Guide](https://github.com/alexa/avs-device-sdk/wiki/Troubleshooting-Guide).

## Next Steps

* [Build Options](https://github.com/alexa/avs-device-sdk/wiki/Build-Options)
* [Unit and Integration Tests](https://github.com/alexa/avs-device-sdk/wiki/Unit-and-Integration-Tests)  
