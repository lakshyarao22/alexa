# CMake build types and options

The following CMake build types are supported:

* `DEBUG` - Shows debug logs with `-g` compiler flag.
* `RELEASE` - Adds `-O2` flag and removes `-g` flag.
* `MINSIZEREL` - Compiles with `RELEASE` flags and optimizations (`-O`s) for a smaller build size.

To specify a build type, you can use this command:  

```sh
cmake <absolute-path-to-source> -DCMAKE_BUILD_TYPE=<build-type>
```

## Debug builds

Debug logs are available in Debug builds. You can enable them by adding the following to the end of your CMake command:

```shell
cmake <absolute-path-to-source> <other_cmake_parameters> -DCMAKE_BUILD_TYPE=DEBUG
```

### Specify log level

You can specify the level of debug logging, by including a `DEBUG<x>` variable when you invoke the sample app. The allowed values are `DEBUG0`, `DEBUG1`, ... `DEBUG9`, where `DEBUG9` provides the most information.

 For example:

```shell
./SampleApp/src/SampleApp ./Integration/AlexaClientSDKConfig.json DEBUG9
```  

### Log sensitive data

Logging of potentially sensitive data is disabled by default, but can be enabled in Debug builds by including this CMake command:

`-DACSDK_EMIT_SENSITIVE_LOGS=ON`

Note: If you want to post logs generated with this setting to a public forum, please be sure to review them and redact any data you consider sensitive before doing so.


## Build with PortAudio  

**Required** - PortAudio is required to build and run the sample app. This is covered in the quickstart guides for [Ubuntu Linux](https://github.com/alexa/avs-device-sdk/wiki/Ubuntu-Linux-Quick-Start-Guide), [macOS](https://github.com/alexa/avs-device-sdk/wiki/macOS-Quick-Start-Guide), and [Raspberry Pi](https://github.com/alexa/avs-device-sdk/wiki/Raspberry-Pi-Quick-Start-Guide-with-Script).

These options must be declared in your CMake command to build with PortAudio. Please note, GStreamer is also included, as it is required to run the sample app:  

* `-DGSTREAMER_MEDIA_PLAYER=ON`
* `-DPORTAUDIO=ON`
* `-DPORTAUDIO_LIB_PATH=<absolute-path-to-portaudio-lib>`
* `-DPORTAUDIO_INCLUDE_DIR=<absolute-path-to-portaudio-include-dir`

This is a sample CMake command:
```
cmake <absolute-path-to-source> -DGSTREAMER_MEDIA_PLAYER=ON -DPORTAUDIO=ON -DPORTAUDIO_LIB_PATH=<absolute-path>/portaudio/lib/.libs/libportaudio.a -DPORTAUDIO_INCLUDE_DIR=<absolute-path>/portaudio/include
```

## Build with wake word detector  

**Note**: Wake word detector and key word detector (KWD) are used interchangeably.

The AVS Device SDK supports wake word detectors from [Sensory](https://github.com/Sensory/alexa-rpi) and [KITT.ai](https://github.com/Kitt-AI/snowboy/). The following options are required to build with a wake word detector, please replace `<wake-word-name>` with `SENSORY` for Sensory, and `KITTAI` for KITT.ai:

* `-D<wake-word-name>_KEY_WORD_DETECTOR=<ON or OFF>` - Specifies if the wake word detector is enabled or disabled during build.
* `-D<wake-word-name>_KEY_WORD_DETECTOR_LIB_PATH=<absolute-path-to-lib>` - The path to the wake word detector library.
* `-D<wake-word-name>_KEY_WORD_DETECTOR_INCLUDE_DIR=<absolute-path-to-include-dir>` - The path to the wake word detector include directory.

**Note**: To list all available CMake options, use the following command: `-LH`.

### Sensory

If using the Sensory wake word detector, version [5.0.0-beta.10.2](https://github.com/Sensory/alexa-rpi) or later is required.

This is an example CMake command to build with Sensory:

```
cmake <absolute-path-to-source> -DSENSORY_KEY_WORD_DETECTOR=ON -DSENSORY_KEY_WORD_DETECTOR_LIB_PATH=<absolute-path>/alexa-rpi/lib/libsnsr.a -DSENSORY_KEY_WORD_DETECTOR_INCLUDE_DIR=<absolute-path>/alexa-rpi/include
```

Note that you may need to license the Sensory library for use prior to running cmake and building it into the SDK. A script to license the Sensory library can be found on the Sensory [Github](https://github.com/Sensory/alexa-rpi) page under `bin/license.sh`.

### KITT.ai

A matrix calculation library, known as BLAS, is required to use KITT.ai. To install this library, run:  
* Generic Linux - `apt-get install libatlas-base-dev`
* macOS -  `brew install homebrew/science/openblas`

This is an example CMake command to build with KITT.ai:

```
cmake <absolute-path-to-source> -DKITTAI_KEY_WORD_DETECTOR=ON -DKITTAI_KEY_WORD_DETECTOR_LIB_PATH=<absolute-path>/snowboy-1.2.0/lib/libsnowboy-detect.a -DKITTAI_KEY_WORD_DETECTOR_INCLUDE_DIR=<absolute-path>/snowboy-1.2.0/include
```

## Build with a MediaPlayer implementation

**Optional** - `MediaPlayer`, the reference implementation of the `MediaPlayerInterface`, is based upon [GStreamer](https://gstreamer.freedesktop.org/). By default, it **is not** included. To build 'MediaPlayer' use this option:

```
-DGSTREAMER_MEDIA_PLAYER=ON
```

If GStreamer was [installed from source](https://gstreamer.freedesktop.org/documentation/frequently-asked-questions/getting.html), you must specify the prefix path:

```
-DCMAKE_PREFIX_PATH==<absolute-path-to-GStreamer-build>
```

For this example, This is a sample CMake command:

```
cmake <absolute-path-to-source> -DGSTREAMER_MEDIA_PLAYER=ON -DCMAKE_PREFIX_PATH=<absolute-path-to-GStreamer-build>
```

## Build for Android

The following variables can be declared in your CMake command to build for Android.

Note: All of the `ANDROID_*` variables are only relevant if the `ANDROID` variable is `ON` (`-DANDROID=ON`)

* `ANDROID`: Enables the android build (`OFF` by default).

* `ANDROID_LOGGER`: Publishes logs using the Android logging facility (`ON` by default)

* `ANDROID_MICROPHONE`: Uses the Android microphone implementation, based on the Native Development Kit (NDK) (`ON` by default).

* `ANDROID_MEDIA_PLAYER`: Uses the Android media player implementation, based on the NDK and FFmpeg (`ON` by default).

* `ANDROID_DEVICE_INSTALL_PREFIX`: The SDK installation path on the Android device or emulator.

## Additional options

You can build the SDK with the following options. To enable these features, you can include in your CMake build command.

| Feature | Command | Description |
|--|--|--|
| **Alexa for Business (A4B)** | `-DA4B=ON` | By default, this option is `OFF`. Turning this option `ON` will enable A4B, including support for handling [`RevokeAuthorization`](https://developer.amazon.com/docs/alexa-voice-service/system.html#revokeauth) directives. |
| **Bluetooth** | `-DBLUETOOTH_BLUEZ=ON` | Currently, Bluetooth is only available for Linux and Raspberry Pi.|
| **Opus** | `-DOPUS=ON` | Opus is an interactive audio codec that is supported by v1.11 or greater of the SDK. In order to enable Opus, this flag must be on, and you must have [libopus](http://opus-codec.org/release/stable/2018/10/18/libopus-1_3.html) installed. For information about Opus, see [http://opus-codec.org/](http://opus-codec.org/) |
| **SAMPLE-AES decryption** | `-DENABLE_SAMPLE_AES=ON -DFFMPEG_INCLUDE_DIR=<absolute-path-to-ffmpeg-include-dir> -DFFMPEG_LIB_PATH=<absolute-path-to-ffmpeg-lib>` | Note: to enable SAMPLE-AES decryption for HLS playlists, the [FFMPEG dependency]() is required, and the build options `FFMPEG_INCLUDE_DIR` & `FFMPEG_LIB_PATH` must be provided. |
