# Windows Quick Start Guide with Script

This guide provides step-by-step instructions to set up the Alexa Voice Service (AVS) Device SDK on 64-bit Windows. When finished, you'll have a working sample app to test interactions with Alexa.

**WARNING**: This guide doesn't include instructions to enable wake word.

## Install Dependencies

### Download and install MSYS2

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

## Register a product
Follow these instructions to [register your product and create a security profile](https://github.com/alexa/avs-device-sdk/wiki/Create-Security-Profile).

Make sure you save the **Product ID** from the **Product information** tab, and your **Client ID** from the **Other devices and platforms** tab from within the **Security Profile** section.

Note: If you already have a registered product that you can use for testing, you may use it but it must be enabled for use with Code Based Linking (CBL). You can find steps for enabling CBL for your device [Here](https://developer.amazon.com/docs/alexa-voice-service/code-based-linking-other-platforms.html#step1) (Step 1).

**IMPORTANT**: When you capture the **Client ID**, make sure it is from the **Other devices and platforms** tab within the **Security Profile** section, and **NOT** from the **Client ID** from the top of the **Product information**, **Security Profile**, or **Capabilities** tabs. The **Client ID** generated within your **Security Profile** is the required **Client ID**.

## Setup
1. Open the MinGW64 shell, and run this command to download the installation script and configuration file:
    ```
    wget https://raw.githubusercontent.com/alexa/avs-device-sdk/master/tools/Install/setup.sh && wget https://raw.githubusercontent.com/alexa/avs-device-sdk/master/tools/Install/mingw.sh &&wget https://raw.githubusercontent.com/alexa/avs-device-sdk/master/tools/Install/config.txt
    ```
    Note: we recommend running these commands from your home directory (`C:/msys64/home/<user_name>`) or your desktop, however, you can run the script anywhere.

2. Using your favorite text editor, update the `config.txt` file with the **Product ID** and **Client ID** (from the 'Other devices and platforms' tab of your device's security profile) for your registered product and **save**.

3. Using the MinGW64 shell, run the setup script with your configuration as an argument:
```
bash setup.sh config.txt
```

## Authorize and run

When you run the sample app for the first time, you'll need to authorize your client for access to AVS.

1. Start the sample app using the `startsample.bat` file. Note: this script is a batch file, and not a bash script. You can run the script either from the Windows command line, or by using the Windows File Explorer to locate the file and then double-clicking it.

2. Wait for the sample app to display a message like this:
```
##################################
#       NOT YET AUTHORIZED       #
##################################
################################################################################################
#       To authorize, browse to: 'https://amazon.com/us/code' and enter the code: {XXXX}       #
################################################################################################
```
3. Use a browser to navigate to the URL specified in the message from the sample app.
4. If requested to do so, authenticate using your Amazon user credentials.
5. Enter the code specified in the message from sample app.
6. Select “Allow”.
7. Wait (it may take a few seconds) for the sample app to report that it is authorized, and that Alexa is idle.  It will look something like this:
```
###########################
#       Authorized!       #
###########################
########################################
#       Alexa is currently idle!       #
########################################
```
8. You are now ready to use the sample app. The next time you start the sample app, you will not need to go through the authorization process.

Note: if you exit out of sample app via the `k` command, the `CBLAuthDelegate` database will be cleared, and you will need to reauthorize your client.

## Optional configurations
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
