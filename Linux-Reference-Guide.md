This guide assumes familiarity with Linux and requires you to build dependencies from source.

## Get started

Follow these steps:

1. [Register an AVS Product and Create a Security Profile](https://github.com/alexa/avs-device-sdk/wiki/Create-Security-Profile), if you haven't already. It must be enabled for [Code-Based Linking](https://developer.amazon.com/docs/alexa-voice-service/code-based-linking-other-platforms.html#step1).

2. Install the [Dependencies](https://github.com/alexa/avs-device-sdk/wiki/Dependencies).

3. Now, set up your build environment. The SDK must be built out-of-source. Create these folders: **build**, **source**, **third-party**, **application-necessities** > **sound-files**.

4. Clone the AVS Device SDK into your **source** folder:  

```shell
git clone https://github.com/alexa/avs-device-sdk.git
```

## Build the AVS Device SDK

Navigate to your **build** folder. This sample command does a few things:  
* It declares that PortAudio is used to capture microphone data and points to its lib path and includes directory
   * It declares that gstreamer is installed and will be used when you build the SampleApp
   * It declares that the wake word detector is **OFF**  

*Linux supports wake word detectors from Sensory and Kitt.ai. Each requires a license from the provider. For instructions to build with a wake word detector, please see [Cmake parameters](https://github.com/alexa/avs-device-sdk/wiki/cmake-options).*  

```shell
cmake /{source}/avs-device-sdk \
-DSENSORY_KEY_WORD_DETECTOR=OFF \
-DGSTREAMER_MEDIA_PLAYER=ON \
-DPORTAUDIO=ON \
-DPORTAUDIO_LIB_PATH=/{PATH TO PORTAUDIO}/lib/.libs/libportaudio.a \
-DPORTAUDIO_INCLUDE_DIR=/{PATH TO PORTAUDIO}/include

make
```

**IMPORTANT**: See [Cmake parameters](https://github.com/alexa/avs-device-sdk/wiki/cmake-options) for a additional configuration options.

### Include Bluetooth (optional)

Building with Bluetooth is optional, and is currently limited to Linux and Raspberry Pi. The `A2DP-SINK`, `A2DP-SOURCE`, `AVRCPTarget`, and `AVRCPController` profiles are supported for Linux/Ubuntu.

To enable Bluetooth on Linux, follow these steps:

1. Install dependencies: In order to use Bluetooth, you must install these dependencies:

* Core [Bluetooth dependencies](https://github.com/alexa/avs-device-sdk/wiki/Dependencies#bluetooth-dependencies)
* [libpulse-dev](https://packages.debian.org/sid/libpulse-dev)
* PulseAudio
* PulseAudio bluetooth modules:
```
sudo apt-get install pulseaudio-module-bluetooth
```

2. Include the following CMake option in your build:

`BLUETOOTH_BLUEZ_PULSEAUDIO_OVERRIDE_ENDPOINTS`

3. AVRCPTarget (*if applicable*)

If you are using the `AVRCPTarget` profile, you'll need to enable permissions for BlueZ to interact with `"org.mpris.MediaPlayer2.Player"`.

To do this, open **/etc/dbus-1/system.d/bluetooth.conf**  and add `<allow send_interface="org.mpris.MediaPlayer2.Player"/>` to your root policy.

For example:

```sh
<!DOCTYPE busconfig PUBLIC "-//freedesktop//DTD D-BUS Bus Configuration 1.0//EN"
 "http://www.freedesktop.org/standards/dbus/1.0/busconfig.dtd">
<busconfig>

  <!-- ../system.conf have denied everything, so we just punch some holes -->

  <policy user="root">
	...
   	<allow send_interface="org.mpris.MediaPlayer2.Player"/>
  </policy>

...

</busconfig>
```

### Enable debug logs (optional)

Debug logs are available in Debug builds.  You can enable them by including Cmake variable:
```shell
cmake < other cmake parameters > -DCMAKE_BUILD_TYPE=DEBUG
```
An additional sample app command line parameter allows you to select the level of debug logging to be output.  The allowed values are DEBUG0, DEBUG1, ... DEBUG9, where DEBUG9 provides the most information.  For example:
```shell
./SampleApp/src/SampleApp ./Integration/Integration/AlexaClientSDKConfig.json DEBUG9
```  

## Authorization

After you have built the SDK, you'll need to configure and authorize it to access AVS using Login with Amazon (LWA). To do this, follow [these authorization instructions](https://github.com/alexa/avs-device-sdk/wiki/Authorization#Generic-Linux).

## Run integration and unit tests
After you've authorized your sample app, run integration and unit tests to ensure that the AVS Device SDK is functioning as designed.

* Use this command to run integration tests:
   ```shell
   make all integration
   ```
* Use this command to run unit tests:  
   ```shell
   make all test
   ```

For additional details, see [Unit and Integration Tests](https://github.com/alexa/avs-device-sdk/wiki/Unit-and-Integration-Tests).

## Set up launch shortcut

You can designate aliases to launch the sample app. To do this:  

1. Open `~/.bash_profile`, and add these aliases and **save**:  
   **IMPORTANT**: Make sure you update the paths to match your folder structure.  
   ```shell
   alias alexac="{path_to}/{build}/SampleApp/src/SampleApp {path_to}/{build}/Integration/AlexaClientSDKConfig.json"
   alias alexacdebug="{path_to}/{build}/SampleApp/src/SampleApp {path_to}/{build}/Integration/AlexaClientSDKConfig.json DEBUG9"
   ```
2. After you've added these aliases, make sure to activate your `~/.bash_profile`:  
   ```shell
   source ~/.bash_profile
   ```
3. Now to launch the sample app, you can run this command:  
   ```
   alexac  
   ```