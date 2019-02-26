This guide provides step-by-step instructions to set up the Alexa Voice Service (AVS) Device SDK on 64-bit Windows. When finished, you'll have a working sample app to test interactions with Alexa.

**WARNING**: This guide doesn't include instructions to enable wake word.

## Get started

1. [Register an AVS Product and Create a Security Profile](https://github.com/alexa/avs-device-sdk/wiki/Create-Security-Profile), if you haven't already. It must be enabled for [Code-Based Linking](https://developer.amazon.com/docs/alexa-voice-service/code-based-linking-other-platforms.html#step1).

2. Download and run the MSYS2 (64-bit) installer. MSYS2 is a software distribution and building platform for Windows. This will install three different shells: MSYS2, MinGW32, and MinGW64. You will use the MinGW64 shell in the steps below.
[Install MSYS2 for Windows 64-bit (x86_64)](http://www.msys2.org/)

3. Update Pacman, the package management system included with MSYS2. The latest version of Pacman is required to build the SDK. Open the MinGW64 shell, and run this command:

    ```sh
    pacman -Syu
    ```

4. Close MinGW64. Reopen MinGW64, and run this command to finish updating Pacman:

    ```sh
    pacman -Su
    ```

5. Download the installation script and configuration file. Open the MinGW64 shell, and run this command to download the installation and configuration scripts:
    ```sh
    wget https://raw.githubusercontent.com/alexa/avs-device-sdk/master/tools/Install/setup.sh \
    wget https://raw.githubusercontent.com/alexa/avs-device-sdk/master/tools/Install/genConfig.sh \
    wget https://raw.githubusercontent.com/alexa/avs-device-sdk/master/tools/Install/mingw.sh
    ```
    Note: we recommend running these commands from your home directory (`C:/msys64/home/<user_name>`) or your desktop, however, you can run the script anywhere.

6. Finally, you'll need to configure and authorize the sample app to access AVS using Login with Amazon (LWA). To do this, follow [these authorization instructions](https://github.com/alexa/avs-device-sdk/wiki/Authorization#Windows)

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

* [Cmake parameters](https://github.com/alexa/avs-device-sdk/wiki/cmake-options)
* [Unit and Integration Tests](https://github.com/alexa/avs-device-sdk/wiki/Unit-and-Integration-Tests)  
