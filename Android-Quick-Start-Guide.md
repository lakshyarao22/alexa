This guide provides step-by-step instructions to set up the Alexa Voice Service (AVS) Device SDK for Android. When finished, you'll have a working sample app to test interactions with Alexa.

## Requirements

**Hardware**: You'll need a Linux or MacOS machine that meets [ these system requirements](https://developer.android.com/studio/#system-requirements-a-namerequirementsa).

**Software**: [These dependencies](https://github.com/alexa/avs-device-sdk/wiki/Dependencies#android-dependencies) must be installed on your machine.

## Storage recommendation:

We recommend installing the Android sample app onto the **internal storage** of your Android device or emulator.

Installing onto external storage may result in installation failures, depending on the storage format. For example, the sample app will not work on FAT32.

## Download installation script and configuration file

You can specify any directory you prefer to run the scripts and download these files to. The following commands will download all necessary files.
```sh
wget https://raw.githubusercontent.com/alexa/avs-device-sdk/master/tools/Install/androidConfig.txt
wget https://raw.githubusercontent.com/alexa/avs-device-sdk/master/tools/Install/setup.sh
wget https://raw.githubusercontent.com/alexa/avs-device-sdk/master/tools/Install/genConfig.sh
wget https://raw.githubusercontent.com/alexa/avs-device-sdk/master/tools/Install/android.sh
```
## Install Android tools

You can install Android tools either by downloading [Android Studio](https://developer.android.com/studio/#downloads) (which includes the tools by default) or [via the command line](https://developer.android.com/studio/#command-tools).

### Install `platformtools`

After you've installed Android tools, you'll need to install `platformtools` using the Android `sdkmanager`, which is part of the Android tools package.

During this process, you'll need to specify the CPU type and Android version number.

For example, this is how we would install Android API 23 on arm:

```sh
# install cmake, ndk-bundle, and android abi sdkmanager "ndk-bundle"
sdkmanager "cmake;3.6.4111459"
sdkmanager "system-images;android-23;google_apis;armeabi-v7a"
sdkmanager "platform-tools"
```

## Set your PATH environment

Once the tools are installed, set your `PATH` environment variable to include the Android tools.

For example, if you've installed the Android tools on your home directory `~/Android/sdk`, run this command:

```sh
export PATH=$PATH:~/Android/sdk/platform-tools:~/Android/sdk/tools:~/Android/sdk/tools/bin
```
## Install dependencies

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

## Configure your test platform

You can use any of these platform types to test and debug your build:

| Device Type                                                           | Minimum Requirements                                                     |
|-----------------------------------------------------------------------|--------------------------------------------------------------------------|
| [Android Emulator](https://developer.android.com/studio/run/emulator) | 500 MB minimum of available disk space.                                  |
| Android Device, such as a phone                                       | 500 MB minimum of available disk space.                                  |
| Android Things-supported Hardware                                     | See requirements [here](https://developer.android.com/things/hardware/). |

### Connect to the Android Debug Bridge (adb)

You'll need to connect your target device to the [adb](https://developer.android.com/studio/command-line/adb) in order to load the SDK to your device.

For Android devices and Android Things-enabled hardware, you'll need to **enable USB debugging** and **connect your device to the adb** before you run the `startsample.sh` build script.

The **Android Emulator** is connected to the adb by default. No configuration is required.

To connect your device to the adb, follow these steps:

1. Physically connect your device to your Mac or Linux machine via USB.

2. [Enable USB debugging](https://developer.android.com/studio/command-line/adb#Enabling) for your device.

3. Connect your device to the adb:
```sh
adb connect <device_ip>
```

## Register your product and create security profile

Follow these instructions to [register your product and create a security profile](https://github.com/alexa/avs-device-sdk/wiki/Create-Security-Profile).

During this process, you'll download a **config.json** file to your machine that will be used by the SDK to authorize your build of the SDK with Login With Amazon (LWA).

**Note**: If you already have a registered product that you can use for testing, you may use it, but it must be enabled for use with Code Based Linking (CBL).

Here are the instructions on [how to enable CBL for your device](https://developer.amazon.com/docs/alexa-voice-service/code-based-linking-other-platforms.html#step1).

## Setup your configuration

In order to build and run the sample app, you'll need to set the configuration values for your device.

Follow these steps:

1. Navigate to the directory where you downloaded the setup script.


2. Update `androidConfig.txt` with the following variables:

| Parameter              | Value                                                                                                                                                                                                                                                             |
|------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `PLATFORM`             | This variable should be set to `"android"`.                                                                                                                                                                                                                       |
| `API_LEVEL`            | The Android API level. See [this table](https://source.android.com/setup/start/build-numbers) to identify the target level. The API level will depend on the device or emulator configuration.  <br> <br> **Supported version**: API 23+ or greater is supported. |
| `TARGET_SYSTEM`        | Choose this value according to the CPU architecture of your target device. For example, choose `arm` for the Raspberry Pi. <br> <br> **Accepted values**: `arm` and `x86`.                                                                                        |
| `DEVICE_INSTALL_PATH`  | The file path on the device where the script should install the sample app and dependencies.  <br> <br> **Note:** Make sure this path exists on the device, and that you have write permissions.                                                                  |
| `BUILD_TYPE`           | The type of build that will be used.  <br> <br> **Accepted values**: `debug`, `release`, `MinSizeRel`, and `RelWithDebInfo`.                                                                                                                                      |

3. Run the setup script with `config.json` and the `{device serial number, ex. 123456}` as arguments.:

```sh
bash setup.sh config.json -a androidConfig.txt [-s {device serial number}]
```

Note: Each instance of the SDK requires a unique **Device Serial Number** (also found in the **deviceInfo** object). This is provided by you, and in some instances may match your product's SKU. For this sample, it's pre-populated with `123456`.

## Run and authorize

When you run the sample app for the first time, you'll need to authorize your client for access to AVS.

1. Start the sample app using the `startsample.sh` script.

 Note: this script is a batch file, and not a bash script. You can run the script either from the command line, or by locating the file and then double-clicking it.

`bash startsample.sh`

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
7. Wait (it may take a few seconds) for the sample app to report that it is authorized, and that Alexa is idle.  It will look something like this:
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

## Next Steps

* [Build Options](https://github.com/alexa/avs-device-sdk/wiki/Build-Options)
* [Unit and Integration Tests](https://github.com/alexa/avs-device-sdk/wiki/Unit-and-Integration-Tests)  