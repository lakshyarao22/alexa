This guide provides step-by-step instructions to set up the Alexa Voice Service (AVS) Device SDK for Android. When finished, you'll have a working sample app to test interactions with Alexa.

## Requirements

**Hardware**: You'll need a Linux or MacOS machine that meets [ these system requirements](https://developer.android.com/studio/#system-requirements-a-namerequirementsa).

**Software**: [These dependencies](https://github.com/alexa/avs-device-sdk/wiki/Dependencies#android-dependencies) must be installed on your machine.

## Storage recommendations:

We recommend installing the Android sample app onto the **internal storage** of your Android device or emulator.

Installing onto external storage may result in installation failures, depending on the storage format. For example, the sample app will not work on FAT32.

## Test platform configuration

You can use any of these platform types to test and debug your build:

| Device Type                                                           | Minimum Requirements                                                     |
|-----------------------------------------------------------------------|--------------------------------------------------------------------------|
| [Android Emulator](https://developer.android.com/studio/run/emulator) | 500 MB minimum of available disk space.                                  |
| Android Device, such as a phone                                       | 500 MB minimum of available disk space.                                  |
| Android Things-supported Hardware                                     | See requirements [here](https://developer.android.com/things/hardware/). |

## Get Started

1. [Register an AVS Product and Create a Security Profile](https://github.com/alexa/avs-device-sdk/wiki/Create-Security-Profile), if you haven't already. It must be enabled for [Code-Based Linking](https://developer.amazon.com/docs/alexa-voice-service/code-based-linking-other-platforms.html#step1).

2. Install Android tools

You can install Android tools either by downloading [Android Studio](https://developer.android.com/studio/#downloads) (which includes the tools by default) or [via the command line](https://developer.android.com/studio/#command-tools).

3. Install platformtools

After you've installed Android tools, you'll need to install `platformtools` using the Android `sdkmanager`, which is part of the Android tools package.

During this process, you'll need to specify the CPU type and Android version number.

For example, this is how we would install Android API 23 on arm:

```sh
# install cmake, ndk-bundle, and android abi sdkmanager "ndk-bundle"
sdkmanager "cmake;3.6.4111459"
sdkmanager "system-images;android-23;google_apis;armeabi-v7a"
sdkmanager "platform-tools"
```

4. Set your PATH environment

Once the tools are installed, set your `PATH` environment variable to include the Android tools.

For example, if you've installed the Android tools on your home directory `~/Android/sdk`, run this command:

```sh
export PATH=$PATH:~/Android/sdk/platform-tools:~/Android/sdk/tools:~/Android/sdk/tools/bin
```
5. Install dependencies

Once the tools are installed, you'll need to install these packages: `autoconf`, `automake`, `libtool`, `pkg-config`.

MacOS:

```sh
brew install autoconf automake libtool pkg-config
```
Linux:

```sh
sudo apt-get update
sudo apt-get install autoconf automake libtool pkg-config
```

6. Connect to the Android Debug Bridge (adb)

You'll need to connect your target device to the [adb](https://developer.android.com/studio/command-line/adb) in order to load the SDK to your device.

For Android devices and Android Things-enabled hardware, you'll need to **enable USB debugging** and **connect your device to the adb** before you run the `startsample.sh` build script.

The **Android Emulator** is connected to the adb by default. No configuration is required.

To connect your device to the adb, follow these steps:

    1. Physically connect your device to your Mac or Linux machine via USB.

    2. [Enable USB debugging](https://developer.android.com/studio/command-line/adb#Enabling) for your device.

    3. Connect your device to the adb:
    ```sh
    adb root adb connect <device_ip>
```

7. Download installation script and configuration file:

You can specify any directory you prefer to run the scripts and download these files to. The following commands will download all the necessary files:
```sh
wget https://raw.githubusercontent.com/alexa/avs-device-sdk/master/tools/Install/androidConfig.txt
wget https://raw.githubusercontent.com/alexa/avs-device-sdk/master/tools/Install/setup.sh
wget https://raw.githubusercontent.com/alexa/avs-device-sdk/master/tools/Install/genConfig.sh
wget https://raw.githubusercontent.com/alexa/avs-device-sdk/master/tools/Install/android.sh
```

8. Finally, you'll need to configure and authorize it to access AVS using Login with Amazon (LWA). To do this, follow [these authorization instructions](https://github.com/alexa/avs-device-sdk/wiki/Authorization#Android).

## Additional resources

* [Cmake parameters](https://github.com/alexa/avs-device-sdk/wiki/cmake-options)
* [Unit and Integration Tests](https://github.com/alexa/avs-device-sdk/wiki/Unit-and-Integration-Tests)  
