## AVS Device SDK

## Overview

The AVS Device SDK for C++ provides a modern C++ (11 or later) interface for the Alexa Voice Service (AVS) that allows developers to add intelligent voice control to connected products. It is modular and abstracted, providing components to handle discrete functionality such as speech capture, audio processing, and communications, with each component exposing APIs that you can use and customize for your integration. It also includes a sample app, which demonstrates interactions with the Alexa Voice Service (AVS).   

To quickly setup your Raspberry Pi development environment or to learn how to optimize libcurl for size, see the wiki or [click here](https://github.com/alexa/alexa-client-sdk/wiki).

* [Common Terms](#common-terms)  
* [Minimum Requirements and Dependencies](#minimum-requirements-and-dependencies)  
* [Prerequisites](#prerequisites)  
* [Create an Out-of-Source Build](#create-an-out-of-source-build)  
* [Run AuthServer](#run-authserver)  
* [Run Unit Tests](#run-unit-tests)  
* [Run Integration Tests](#run-integration-tests)  
* [Run the Sample App](#run-the-sample-app)  
* [Install the SDK](#install-the-sdk)  
* [AVS Device SDK for C++ API Documentation(Doxygen)](#alexa-client-sdk-api-documentation)
* [Resources and Guides](#resources-and-guides)  
* [Release Notes](#release-notes)

## Common Terms

* **Interface** - A collection of logically grouped messages called **directives** and **events**, which correspond to client functionality, such speech recognition, audio playback, and volume control.
* **Directives** - Messages sent from AVS that instruct your product to take action.
* **Events** - Messages sent from your product to AVS notifying AVS something has occurred.
* **Downchannel** - A stream you create in your HTTP/2 connection, which is used to deliver directives from AVS to your product. The downchannel remains open in a half-closed state from the device and open from AVS for the life of the connection. The downchannel is primarily used to send cloud-initiated directives to your product.
* **Cloud-initiated Directives** - Directives sent from AVS to your product. For example, when a user adjusts device volume from the Amazon Alexa App, a directive is sent to your product without a corresponding voice request.

## Minimum Requirements and Dependencies  

Instructions are available to help you quickly [setup a development environment for RaspberryPi](../wiki/Raspberry-Pi-Quick-Start-Guide) and [build libcurl with nghttp2 for macOS](../wiki/How-to-build-libcurl-with-nghttp2-for-macos).

### Core Dependencies  
* C++ 11 or later
* [GNU Compiler Collection (GCC) 4.8.5](https://gcc.gnu.org/) or later **OR** [Clang 3.3](http://clang.llvm.org/get_started.html) or later
* [CMake 3.1](https://cmake.org/download/) or later
* [libcurl 7.50.2](https://curl.haxx.se/download.html) or later
* [nghttp2 1.0](https://github.com/nghttp2/nghttp2) or later
* [OpenSSL 1.0.2](https://www.openssl.org/source/) or later
* [Doxygen 1.8.13](http://www.stack.nl/~dimitri/doxygen/download.html) or later (required to build API documentation)  
* [SQLite 3.19.3](https://www.sqlite.org/download.html) or later (required for Alerts)  
* For Alerts to work as expected:  
  * The device system clock must be set to UTC time. We recommend using `NTP` to do this   
  * A file system is required  

### MediaPlayer Reference Implementation Dependencies
Building the reference implementation of the `MediaPlayerInterface` (the class `MediaPlayer`) is optional, but requires:  
* [GStreamer 1.10.4](https://gstreamer.freedesktop.org/documentation/installing/index.html) (or later) and the following GStreamer plug-ins:  
**IMPORTANT NOTE FOR LINUX**: GStreamer 1.8 **does not** work.     
* [GStreamer Base Plugins 1.10.4](https://gstreamer.freedesktop.org/releases/gst-plugins-base/1.10.4.html) or later.
* [GStreamer Good Plugins 1.10.4](https://gstreamer.freedesktop.org/releases/gst-plugins-good/1.10.4.html) or later.
* [GStreamer Libav Plugin 1.10.4](https://gstreamer.freedesktop.org/releases/gst-libav/1.10.4.html) or later **OR**
* [GStreamer Ugly Plugins 1.10.4](https://gstreamer.freedesktop.org/releases/gst-plugins-ugly/1.10.4.html) or later, for decoding MP3 data.

**NOTE**: The plugins may depend on libraries which need to be installed for the GStreamer based `MediaPlayer` to work correctly.  

### Sample App Dependencies  
Building the sample app is optional, but requires:  
* [PortAudio v190600_20161030](http://www.portaudio.com/download.html)
* GStreamer

**NOTE**: The sample app will work with or without a wake word engine. If built without a wake word engine, hands-free mode will be disabled in the sample app.  

### Music Provider Dependencies  

The following codecs and packages are required for iHeartRadio playback:  
* [GStreamer Bad Plugins 1.10.4](https://gstreamer.freedesktop.org/releases/gst-plugins-bad/1.10.4.html) or later.  
* [Crypto Library](https://gnupg.org/software/libgcrypt/index.html) for HLS demuxers.
* [libsoup]( https://wiki.gnome.org/Projects/libsoup) an HTTP client/server library used by GStreamer.
* [libfaad-dev](https://github.com/dsvensson/faad2) - AAC and HE-AAC decoding.

## Prerequisites

Before you create your build, you'll need to install some software that is required to run `AuthServer`. `AuthServer` is a minimal authorization server built in Python using Flask. It provides an easy way to obtain your first refresh token, which will be used for integration tests and obtaining access token that are required for all interactions with AVS.

**IMPORTANT NOTE**: `AuthServer` is for testing purposed only. A commercial product is expected to obtain Login with Amazon (LWA) credentials using the instructions provided on the Amazon Developer Portal for **Remote Authorization** and **Local Authorization**. For additional information, see [AVS Authorization](https://developer.amazon.com/public/solutions/alexa/alexa-voice-service/content/avs-api-overview#authorization).

### Step 1: Install `pip`

If `pip` isn't installed on your system, follow the detailed install instructions [here](https://packaging.python.org/installing/#install-pip-setuptools-and-wheel).

### Step 2: Install `flask` and `requests`

For Windows run this command:

```
pip install flask requests
```

For Unix/Mac run this command:

```
pip install --user flask requests
```

### Step 3: Obtain Your Product ID, Cliend ID, and Client Secret

If you haven't already, follow these instructions to [register a product and create a security profile](https://github.com/alexa/alexa-avs-sample-app/wiki/Create-Security-Profile).

Make sure you note the following, you'll need these later when you configure `AuthServer`:

* Product ID
* Client ID
* Client Secret

**IMPORTANT NOTE**: Make sure that you've set your **Allowed Origins** and **Allowed Return URLs** in the **Web Settings Tab**:
* Allowed Origins: http://localhost:3000
* Allowed Return URLs: http://localhost:3000/authresponse

## Create an Out-of-Source Build

The following instructions assume that all requirements and dependencies are met and that you have cloned the repository (or saved the tarball locally).

### CMake Build Types and Options

The following build types are supported:

* `DEBUG` - Shows debug logs with `-g` compiler flag.
* `RELEASE` - Adds `-O2` flag and removes `-g` flag.
* `MINSIZEREL` - Compiles with `RELEASE` flags and optimizations (`-O`s) for a smaller build size.

To specify a build type, use this command in place of step 4 below:
`cmake <absolute-path-to-source> -DCMAKE_BUILD_TYPE=<build-type>`  

### Build with a Wake Word Detector  

**Note**: Wake word detector and key word detector (KWD) are used interchangeably.

The AVS Device SDK for C++ supports wake word detectors from [Sensory](https://github.com/Sensory/alexa-rpi) and [KITT.ai](https://github.com/Kitt-AI/snowboy/). The following options are required to build with a wake word detector, please replace `<wake-word-name>` with `SENSORY` for Sensory, and `KITTAI` for KITT.ai:

* `-D<wake-word-name>_KEY_WORD_DETECTOR=<ON or OFF>` - Specifies if the wake word detector is enabled or disabled during build.
* `-D<wake-word-name>_KEY_WORD_DETECTOR_LIB_PATH=<absolute-path-to-lib>` - The path to the wake word detector library.
* `-D<wake-word-name>_KEY_WORD_DETECTOR_INCLUDE_DIR=<absolute-path-to-include-dir>` - The path to the wake word detector include directory.

**Note**: To list all available CMake options, use the following command: `-LH`.

#### Sensory

If using the Sensory wake word detector, version [5.0.0-beta.10.2](https://github.com/Sensory/alexa-rpi) or later is required.

This is an example `cmake` command to build with Sensory:

```
cmake <absolute-path-to-source> -DSENSORY_KEY_WORD_DETECTOR=ON -DSENSORY_KEY_WORD_DETECTOR_LIB_PATH=.../alexa-rpi/lib/libsnsr.a -DSENSORY_KEY_WORD_DETECTOR_INCLUDE_DIR=.../alexa-rpi/include
```

Note that you may need to license the Sensory library for use prior to running cmake and building it into the SDK. A script to license the Sensory library can be found on the Sensory [Github](https://github.com/Sensory/alexa-rpi) page under `bin/license.sh`.

#### KITT.ai

A matrix calculation library, known as BLAS, is required to use KITT.ai. The following are sample commands to install this library:
* Generic Linux - `apt-get install libatlas-base-dev`
* macOS -  `brew install homebrew/science/openblas`

This is an example `cmake` command to build with KITT.ai:

```
cmake <absolute-path-to-source> -DKITTAI_KEY_WORD_DETECTOR=ON -DKITTAI_KEY_WORD_DETECTOR_LIB_PATH=.../snowboy-1.2.0/lib/libsnowboy-detect.a -DKITTAI_KEY_WORD_DETECTOR_INCLUDE_DIR=.../snowboy-1.2.0/include
```

### Build with an implementation of `MediaPlayer`

`MediaPlayer` (the reference implementation of the `MediaPlayerInterface`) is based upon [GStreamer](https://gstreamer.freedesktop.org/), and is not built by default. To build 'MediaPlayer' the `-DGSTREAMER_MEDIA_PLAYER=ON` option must be specified to CMake.

If GStreamer was [installed from source](https://gstreamer.freedesktop.org/documentation/frequently-asked-questions/getting.html), the prefix path provided when building must be specified to CMake with the `DCMAKE_PREFIX_PATH` option.

This is an example `cmake` command:

```
cmake <absolute-path-to-source> -DGSTREAMER_MEDIA_PLAYER=ON -DCMAKE_PREFIX_PATH=<path-to-GStreamer-build>
```

### Build with PortAudio (Required to Run the Sample App)  

PortAudio is required to build and run the AVS Device SDK for C++ Sample App. Build instructions are available for [Linux](http://portaudio.com/docs/v19-doxydocs/compile_linux.html) and [macOS](http://portaudio.com/docs/v19-doxydocs/compile_mac_coreaudio.html).  

This is sample CMake command to build the AVS Device SDK for C++ with PortAudio:

```
cmake <absolute-path-to-source> -DPORTAUDIO=ON
-DPORTAUDIO_LIB_PATH=<path-to-portaudio-lib>
-DPORTAUDIO_INCLUDE_DIR=<path-to-portaudio-include-dir>
```

For example,
```
cmake <absolute-path-to-source> -DPORTAUDIO=ON
-DPORTAUDIO_LIB_PATH=.../portaudio/lib/.libs/libportaudio.a
-DPORTAUDIO_INCLUDE_DIR=.../portaudio/include
```

## Build for Generic Linux / macOS

To create an out-of-source build:

1. Clone the repository (or download and extract the tarball).
2. Create a build directory out-of-source. **Important**: The directory cannot be a subdirectory of the source folder.
3. `cd` into your build directory.
4. From your build directory, run `cmake` on the source directory to generate make files for the SDK: `cmake <path-to-source-code>`.
5. After you've successfully run `cmake`, you should see the following message: `-- Please fill <absolute-path-to-build-directory>/Integration/AlexaClientSDKConfig.json before you execute integration tests.`. Open `Integration/AlexaClientSDKConfig.json` with your favorite text editor and fill in your product information.
6. From the build directory, run `make` to build the SDK.  

### Application Settings

The SDK will require a configuration JSON file, an example of which is located in `Integration/AlexaClientSDKConfig.json`. The contents of the JSON should be populated with your product information (which you got from the developer portal when registering a product and creating a security profile), and the location of your database and sound files. This JSON file is required for the integration tests to work properly, as well as for the Sample App.

The layout of the file is as follows:  

```json
{
    "authDelegate":{
        "clientSecret":"<Client Secret for your device from the Amazon Developer Portal>",
        "deviceSerialNumber":"<A unique value that you create, similar to a SKU or UPC. E.g. "123456">",
        "refreshToken":"${SDK_CONFIG_REFRESH_TOKEN}",
        "clientId":"<Client ID for your device from the Amazon Developer Portal>",
        "productId":"<Product ID for your device from the Amazon Developer Portal>"
     },

   "alertsCapabilityAgent":{
        "databaseFilePath":"/<absolute-path-to-db-directory>/<db-file-name>",
        "alarmSoundFilePath":"/<absolute-path-to-alarm-sound>/alarm_normal.mp3",
        "alarmShortSoundFilePath":"/<absolute-path-to-short-alarm-sound>/alarm_short.wav",
        "timerSoundFilePath":"/<absolute-path-to-timer-sound>/timer_normal.mp3",
        "timerShortSoundFilePath":"/<absolute-path-to-short-timer-sound>/timer_short.wav"
   },

   "settings":{
        "databaseFilePath":"/<absolute-path-to-db-directory>/<db-file-name>",
        "defaultAVSClientSettings":{
            "locale":"en-US"
          }
    },
   "certifiedSender":{
        "databaseFilePath":"/<absolute-path-to-db-directory>/<db-file-name>"
    }
 }
```
**NOTE**: The `deviceSerialNumber` is a unique identifier that you create. It is **not** provided by Amazon.
**NOTE**: Audio assets included in this repository are licensed as "Alexa Materials" under the [Alexa Voice
Service Agreement](https://developer.amazon.com/public/solutions/alexa/alexa-voice-service/support/terms-and-agreements).

## Run AuthServer

After you've created your out-of-source build, the next step is to run `AuthServer` to retrieve a valid refresh token from LWA.

* Run this command to start `AuthServer`:
```
python AuthServer/AuthServer.py
```
You should see a message that indicates the server is running.
* Open your favorite browser and navigate to: `http://localhost:3000`
* Follow the on-screen instructions.
* After you've entered your credentials, the server should terminate itself, and `Integration/AlexaClientSDKConfig.json` will be populated with your refresh token.
* Before you proceed, it's important that you make sure the refresh token is in `Integration/AlexaClientSDKConfig.json`.

## Run Unit Tests

Unit tests for the AVS Device SDK for C++ use the [Google Test](https://github.com/google/googletest) framework. Ensure that the [Google Test](https://github.com/google/googletest) is installed, then run the following command:
`make all test`

Ensure that all tests are passed before you begin integration testing.

### Run Unit Tests with Sensory Enabled

In order to run unit tests for the Sensory wake word detector, the following files must be downloaded from [GitHub](https://github.com/Sensory/alexa-rpi) and placed in `<source dir>/KWD/inputs/SensoryModels` for the integration tests to run properly:

* [`spot-alexa-rpi-31000.snsr`](https://github.com/Sensory/alexa-rpi/blob/master/models/spot-alexa-rpi-31000.snsr)

### Run Unit Tests with KITT.ai Enabled

In order to run unit tests for the KITT.ai wake word detector, the following files must be downloaded from [GitHub](https://github.com/Kitt-AI/snowboy/tree/master/resources) and placed in `<source dir>/KWD/inputs/KittAiModels`:
* [`common.res`](https://github.com/Kitt-AI/snowboy/tree/master/resources)
* [`alexa.umdl`](https://github.com/Kitt-AI/snowboy/tree/master/resources/alexa/alexa-avs-sample-app) - It's important that you download the `alexa.umdl` in `resources/alexa/alexa-avs-sample-app` for the KITT.ai unit tests to run properly.

## Run Integration Tests

Integration tests ensure that your build can make a request and receive a response from AVS.
* All requests to AVS require auth credentials
* The integration tests for Alerts require your system to be in UTC

**Important**: Integration tests reference an `AlexaClientSDKConfig.json` file, which you must create.
See the `Create the AlexaClientSDKConfig.json file` section (above), if you have not already done this.

To run the integration tests use this command:
`TZ=UTC make all integration`

### Network Integration Test  

If your project is built on a GNU/Linux-based platform (Ubuntu, Debian, etc.), there is an optional integration test which tests the ACL for use on slow networks. To enable this test, use this option with CMake:

```
cmake <absolute-path-to-source> -DNETWORK_INTEGRATION_TESTS=ON –DNETWORK_INTERFACE=eth0
```

**Note**: The name of the network interface can be located with this command `ifconfig -a`.
**IMPORTANT**: This test requires root permissions.  

### Run Integration Tests with Sensory Enabled

If the project was built with the Sensory wake word detector, the following files must be downloaded from [GitHub](https://github.com/Sensory/alexa-rpi) and placed in `<source dir>/Integration/inputs/SensoryModels` for the integration tests to run properly:

* [`spot-alexa-rpi-31000.snsr`](https://github.com/Sensory/alexa-rpi/blob/master/models/spot-alexa-rpi-31000.snsr)

### Run Integration Tests with KITT.ai

If the project was built with the KITT.ai wake word detector, the following files must be downloaded from [GitHub](https://github.com/Kitt-AI/snowboy/tree/master/resources) and placed in `<source dir>/Integration/inputs/KittAiModels` for the integration tests to run properly:
* [`common.res`](https://github.com/Kitt-AI/snowboy/tree/master/resources)
* [`alexa.umdl`](https://github.com/Kitt-AI/snowboy/tree/master/resources/alexa/alexa-avs-sample-app) - It's important that you download the `alexa.umdl` in `resources/alexa/alexa-avs-sample-app` for the KITT.ai integration tests to run properly.

## Run the Sample App   

**Note**: Building with PortAudio and GStreamer is required.

Before you run the sample app, it's important to note that the application takes two arguments. The first is required, and is the path to `AlexaClientSDKConfig.json`. The second is only required if you are building the sample app with wake word support, and is the path of the of the folder containing the wake word engine models.

Navigate to the `SampleApp/src` folder from your build directory. Then run this command:

```
TZ=UTC ./SampleApp <REQUIRED-absolute-path-to-config-json> <OPTIONAL-absolute-path-to-wake-word-engine-folder-enclosing-model-files> <OPTIONAL-log-level>
```

**Note**: The following logging levels are supported with `DEBUG9` providing the highest and `CRITICAL` the lowest level of logging: `DEBUG9`, `DEBUG8`, `DEBUG7`, `DEBUG6`, `DEBUG5`, `DEBUG4`, `DEBUG3`, `DEBUG2`, `DEBUG1`, `DEBUG0`, `INFO`, `WARN`, `ERROR`, and `CRITICAL`.  

**IMPORTANT**: After you launch the sample app, you must wait several seconds for the app to load before making your first request. This is a known issue, which will be addressed in a future release.  

## Install the AVS Device SDK  

These instructions assume that you've followed the instructions to create an out-of-source build. When running the `cmake` command, you must specify the install path with the `DCMAKE_INSTALL_PREFIX` option.  

This is a sample CMake command:

```
cmake <absolute-path-to-source>
-DCMAKE_INSTALL_PREFIX=<absolute-path-to-out-of-source-build>
```

After you've run `cmake`, run `make install` to install the SDK.  

The library and header files will be installed to the path specified. Additionally, `AlexaClientSDK.pc` is generated and can be used on systems that support `pkg-config`.  

To build your application with the SDK, the prefix path for your installation must be specified to CMake. For example:  

```  
cmake -DCMAKE_PREFIX_PATH=<absolute-path-to-install>  
```  

**NOTE**: You may need to specify the `rpath` to link the SDK to your application.  
**NOTE**: In your application, you will need to add the include path to `RapidJSON`.  

## AVS Device SDK for C++ API Documentation

To build API documentation locally, run this command from your build directory: `make doc`.

## Resources and Guides

* [Step-by-step instructions to optimize libcurl for size in `*nix` systems](https://github.com/alexa/alexa-client-sdk/wiki/optimize-libcurl).
* [Step-by-step instructions to build libcurl with mbed TLS and nghttp2 for `*nix` systems](https://github.com/alexa/alexa-client-sdk/wiki/build-libcurl-with-mbed-TLS-and-nghttp2).  
