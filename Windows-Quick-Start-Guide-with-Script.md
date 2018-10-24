This guide provides step-by-step instructions to set up the Alexa Voice Service (AVS) Device SDK on 64-bit Windows. When finished, you'll have a working sample app to test interactions with Alexa.

**WARNING**: This guide doesn't include instructions to enable wake word.

## Install Dependencies

### Download and install MSYS2

1. Download and run the MSYS2 (64-bit) installer. MSYS2 is a software distribution and building platform for Windows. This will install three different shells: MSYS2, MinGW32, and MinGW64. You will use the MinGW64 shell in the steps below.
[Install MSYS2 for Windows 64-bit (x86_64)](http://www.msys2.org/)

2. Update Pacman, the package management system included with MSYS2. The latest version of Pacman is required to build the SDK. Open the MinGW64 shell, and run this command:

    ```sh
    pacman -Syu
    ```

3. Close MinGW64. Reopen MinGW64, and run this command to finish updating Pacman:

    ```sh
    pacman -Su
    ```

## Register a product

Follow these instructions to [register your product and create a security profile](https://github.com/alexa/avs-device-sdk/wiki/Create-Security-Profile).

During this process, you'll download a **config.json** file to your machine that will be used by the SDK to authorize your build of the SDK with Login With Amazon (LWA).

**Note**: If you already have a registered product that you can use for testing, you may use it, but it must be enabled for use with Code Based Linking (CBL).

Here are the instructions on [how to enable CBL for your device](https://developer.amazon.com/docs/alexa-voice-service/code-based-linking-other-platforms.html#step1).

## Setup your configuration

1. Open the MinGW64 shell, and run this command to download the installation and configuration scripts:
    ```sh
    wget https://raw.githubusercontent.com/alexa/avs-device-sdk/master/tools/Install/setup.sh \
    wget https://raw.githubusercontent.com/alexa/avs-device-sdk/master/tools/Install/genConfig.sh \
    wget https://raw.githubusercontent.com/alexa/avs-device-sdk/master/tools/Install/pi.sh
    ```
    Note: we recommend running these commands from your home directory (`C:/msys64/home/<user_name>`) or your desktop, however, you can run the script anywhere.

2. Move the **config.json** file that you downloaded when you [created your Security Profile](https://github.com/alexa/avs-device-sdk/wiki/Create-Security-Profile#create-a-security-profile) to your **home** directory.

3. Using the MinGW64 shell, run the setup script with `config.json` and the `{device serial number, ex. 123456}` as arguments.

    ```sh
    bash setup.sh config.json [-s {device serial number}]
    ```

    Note: Each instance of the SDK requires a unique **Device Serial Number** (also found in the **deviceInfo** object). This is provided by you, and in some instances may match your product's SKU. For this sample, it's pre-populated with `123456`.

## Authorize and run

When you run the sample app for the first time, you'll need to authorize your client for access to AVS.

1. Start the sample app using the `startsample.bat` file. Note: this script is a batch file, and not a bash script. You can run the script either from the Windows command line, or by using the Windows File Explorer to locate the file and then double-clicking it.

2. Wait for the sample app to display a message like this:
```sh
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
7. Wait for the sample app to report that it is authorized, and that Alexa is idle.  It will look something like this:
```sh
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
Open the MinGW64 shell.
Run the following commands:
```sh
cd <msys64_installed_path>/alexa_sdk/build/bin
./SampleApp.exe ../Integration/AlexaClientSDKConfig.json DEBUG9
```
**To run the sample app using the Windows command line**:

Add `<msys64_installed_path>/mingw64/bin` into the path.

For this option, use `mingw32-make.exe` instead of `make`.

**To build the SDK after making custom changes**:
Open the MinGW64 shell, and run `make` inside of the the `alexa_sdk/build` folder.

## Common Issues

See the [Troubleshooting Guide](https://github.com/alexa/avs-device-sdk/wiki/Troubleshooting-Guide).

## Next Steps

* [Build Options](https://github.com/alexa/avs-device-sdk/wiki/Build-Options)
* [Unit and Integration Tests](https://github.com/alexa/avs-device-sdk/wiki/Unit-and-Integration-Tests)  
