# CMake Parameters

[CMake](https://cmake.org/) is a build s a build tool that can manage your application's dependencies and create native makefiles suitable for the platform you're building on. It's an easy way to create and build projects using the AVS Device SDK for C++.

You can use the CMake variables and options listed on this page to customize how your SDK builds.

**Build types**:

* [Build types](https://github.com/alexa/avs-device-sdk/wiki/cmake-options#build-types)
  * [Specify log level](https://github.com/alexa/avs-device-sdk/wiki/cmake-options#Specify-log-level)
  * [Log sensitive data](https://github.com/alexa/avs-device-sdk/wiki/cmake-options#Log-sensitive-data)

**Build Options**

* [Bluetooth](https://github.com/alexa/avs-device-sdk/wiki/cmake-options#Bluetooth)
* [MediaPlayer](https://github.com/alexa/avs-device-sdk/wiki/cmake-options#MediaPlayer)
* [Wake word detector](https://github.com/alexa/avs-device-sdk/wiki/cmake-options#Wake-word-detector )
* [Additional options](https://github.com/alexa/avs-device-sdk/wiki/cmake-options#Additional-options)
* [Android CMake parameters and variables](https://github.com/alexa/avs-device-sdk/wiki/cmake-options#Android-CMake-variables-and-options)

## Build types

The following [CMake build types](https://cmake.org/cmake/help/latest/variable/CMAKE_BUILD_TYPE.html) are supported:

* `Debug` - Enables logging. Shows debug logs with `-g` compiler flag.
* `Release` - Adds `-O2` flag and removes `-g` flag.
* `MinSizeRel` - Compiles with `RELEASE` flags and optimizations (`-O`s) for a smaller build size.

**CMAKE_BUILD_TYPE**

Specifies the build type. If no build type is specified, the default is `Release`.

*Values*

`DEBUG` | `RELEASE` | `MINISIZEREL`

*Default*

`OFF`

*Example*:

You can set options with the CMake GUI tools or the command line by using `-D.` In this example, the build type Debug is specified:

```sh
cmake <absolute-path-to-source> -DCMAKE_BUILD_TYPE=DEBUG
```

### Specify log level

You can specify the debug log level by including a `DEBUG<x>` variable when you invoke the sample app. The allowed values are `DEBUG0`, `DEBUG1`, ... `DEBUG9`, where `DEBUG9` provides the most information.

 For example:

```shell
./SampleApp/src/SampleApp ./Integration/AlexaClientSDKConfig.json DEBUG9
```  

### Log sensitive data

**ACSDK_EMIT_SENSITIVE_LOGS**

Enables potentially sensitive data logging in Debug builds.

*Values*

`OFF` | `ON`

*Default*

`OFF`

*Example*:

`-DACSDK_EMIT_SENSITIVE_LOGS=ON`

**IMPORTANT**: If you want to post logs generated with this setting to a public forum, be sure to review them and redact any data you consider sensitive before doing so.

## Bluetooth

Currently, Bluetooth is only available for Linux and Raspberry Pi. There are [dependencies](https://github.com/alexa/avs-device-sdk/wiki/Dependencies#bluetooth) required in order to build with Bluetooth.

**BLUETOOTH_BLUEZ**

Enables Bluetooth, based on [BlueZ](http://www.bluez.org/about/).

*Values*

`OFF` | `ON`

*Default*

`ON`

*Example*:

`-DBLUETOOTH_BLUEZ=ON`

**BLUETOOTH_BLUEZ_PULSEAUDIO_OVERRIDE_ENDPOINTS**

Automates loading and unloading of PulseAudio Bluetooth modules. To use this option you must install [libpulse-dev](https://packages.debian.org/sid/libpulse-dev). See [Bluetooth dependencies](https://github.com/alexa/avs-device-sdk/wiki/Dependencies#bluetooth-dependencies) for more information.

**IMPORTANT** By enabling this option, the AVS Device SDK for C++ will make modifications to the PulseAudio application.

*Values*

`OFF` | `ON`

*Default*

`OFF`

*Example*:

`-DBLUETOOTH_BLUEZ_PULSEAUDIO_OVERRIDE_ENDPOINTS=ON`

## MediaPlayer

MediaPlayer is a reference implementation of the [GStreamer](https://gstreamer.freedesktop.org/)`MediaPlayerInterface`.

**GSTREAMER_MEDIA_PLAYER**

*Values*

`OFF` | `ON`

*Default*

`OFF`

*Example*:

`-DGSTREAMER_MEDIA_PLAYER=ON`

If GStreamer was [installed from source](https://gstreamer.freedesktop.org/documentation/frequently-asked-questions/getting.html#how-can-i-install-gstreamer-from-source), you must specify the prefix path:

```
-DCMAKE_PREFIX_PATH==<absolute-path-to-GStreamer-build>
```

## PortAudio

**Required** - [PortAudio](http://www.portaudio.com/) is a cross-platform audio input/output library that's required to build and run the sample app.

**PORTAUDIO**

Enables PortAudio.

*Values*

`ON` | `OFF`

*Default*

`ON`

**PORTAUDIO_LIB_PATH**

Specifies the path to the PortAudio library.

**PORTAUDIO_INCLUDE_DIR**

Specifies the path to the PortAudio directory.

*Example*:

```
cmake <absolute-path-to-source> -DPORTAUDIO=ON -DPORTAUDIO_LIB_PATH=<absolute-path>/portaudio/lib/.libs/libportaudio.a -DPORTAUDIO_INCLUDE_DIR=<absolute-path>/portaudio/include
```

## Wake word detector  

**Note**: Wake word detector (WWD) and key word detector (KWD) are used interchangeably.

The AVS Device SDK supports wake word detectors from [Sensory](https://github.com/Sensory/alexa-rpi) and [KITT.ai](https://github.com/Kitt-AI/snowboy/). The following options are required to build with a wake word detector.

Replace `<wake-word-name>` with `SENSORY` for Sensory, and `KITTAI` for KITT.ai.

**<wake-word-name>_KEY_WORD_DETECTOR**

Specifies if the WWD is enabled or disabled during build.

*Values*

`OFF` | `ON`

*Default*

`OFF`

Example:

Replace `<wake-word-name>` with `SENSORY` for Sensory, and `KITTAI` for KITT.ai.

`-DSENSORY_KEY_WORD_DETECTOR=ON`

**<wake-word-name>_KEY_WORD_DETECTOR_LIB_PATH=<absolute-path-to-lib>**
The path to the wake word detector library.

**<wake-word-name>_KEY_WORD_DETECTOR_INCLUDE_DIR=<absolute-path-to-include-dir>**
The path to the wake word detector include directory.

### Sensory

If using the Sensory wake word detector, version [5.0.0-beta.10.2](https://github.com/Sensory/alexa-rpi) or later is required.

This is an example CMake command to build with Sensory:

```
cmake <absolute-path-to-source> -DSENSORY_KEY_WORD_DETECTOR=ON -DSENSORY_KEY_WORD_DETECTOR_LIB_PATH=<absolute-path>/alexa-rpi/lib/libsnsr.a -DSENSORY_KEY_WORD_DETECTOR_INCLUDE_DIR=<absolute-path>/alexa-rpi/include
```

Note that you may need to license the Sensory library for use prior to enabling it as a CMake options. A script to license the Sensory library can be found on the [Sensory Github](https://github.com/Sensory/alexa-rpi/blob/master/bin/license.sh).

### KITT.ai

A matrix calculation library, known as BLAS, is required to use KITT.ai. To install this library, run:  
* Generic Linux - `apt-get install libatlas-base-dev`
* macOS -  `brew install homebrew/science/openblas`

This is an example CMake command to build with KITT.ai:

```
cmake <absolute-path-to-source> -DKITTAI_KEY_WORD_DETECTOR=ON -DKITTAI_KEY_WORD_DETECTOR_LIB_PATH=<absolute-path>/snowboy-1.2.0/lib/libsnowboy-detect.a -DKITTAI_KEY_WORD_DETECTOR_INCLUDE_DIR=<absolute-path>/snowboy-1.2.0/include
```

## Additional options

You can build the SDK with the following options. To enable these features, you can include these variables in your CMake build:

| Feature | Variable | Description |
|--|--|--|
| **Alexa for Business (A4B)** | `-DA4B=ON` | By default, this option is `OFF`. Turning this option `ON` will enable A4B, including support for handling [`RevokeAuthorization`](https://developer.amazon.com/docs/alexa-voice-service/system.html#revokeauth) directives. |
| **Opus** | `-DOPUS=ON` | Opus is an interactive audio codec that is supported by v1.11 or greater of the SDK. In order to enable Opus, this flag must be on, and you must have [libopus](http://opus-codec.org/release/stable/2018/10/18/libopus-1_3.html) installed. For information about Opus, see [http://opus-codec.org/](http://opus-codec.org/) |
| **SAMPLE-AES decryption** | `-DENABLE_SAMPLE_AES=ON -DFFMPEG_INCLUDE_DIR=<absolute-path-to-ffmpeg-include-dir> -DFFMPEG_LIB_PATH=<absolute-path-to-ffmpeg-lib>` | Note: to enable SAMPLE-AES decryption for HLS playlists, the FFMPEG dependency is required, and the build options `FFMPEG_INCLUDE_DIR` & `FFMPEG_LIB_PATH` must be provided. |

## Android CMake variables and options

Use the following parameters when you are creating an Android build of the SDK.

The following `ANDROID_*` variables are only relevant if the `ANDROID` variable is `ON` (`-DANDROID=ON`).

**ANDROID** - Enables the android build.

*Values*

`OFF` | `ON`

*Default*

`OFF`

**ANDROID_LOGGER** - Publishes logs using the Android logging facility.

*Values*

`OFF` | `ON`

*Default*

`ON`

**ANDROID_MICROPHONE** - Uses the Android microphone implementation, based on the Native Development Kit (NDK).

*Values*

`OFF` | `ON`

*Default*

`ON`

**ANDROID_MEDIA_PLAYER** - Uses the Android media player implementation, based on the NDK and FFmpeg.

*Values*

`OFF` | `ON`

*Default*

`ON`

**ANDROID_DEVICE_INSTALL_PREFIX** - The SDK installation path on the Android device or emulator.
