This is a list of requirements and dependencies for the Alexa Voice Service (AVS) Device SDK for C++.

## Requirements

The AVS Device SDK runs on Raspberry Pi, macOS, Ubuntu Linux, Android, and Windows 64-bit. It requires **C++11 or later**.

### Compiler

The AVS Device SDK libraries are tested with the following compilers:

*For Ubuntu, Raspberry Pi, and macOS*:

| Compiler | Minimum Version     |
| :------------- | :------------- |
| [GNU Compiler Collection (GCC)](https://gcc.gnu.org/)     | 4.8.5 |
| [Clang](http://clang.llvm.org/get_started.html)     | 3.9  |

### Build tools

[CMake](https://cmake.org/) is a cross-platform set of tools that can be used to manage your application's dependencies and create native makefiles and workspaces suitable for the platform you're building on. It's an easy way to create, build, and test projects using the AVS Device SDK for C++.

The minimum version we test with is:

| Tool | Minimum Version    |
| :------------- | :------------- |
| [CMake](https://cmake.org/download/)       | 3.1|

### Libraries

The SDK depends on libcurl, nghttp2, OpenSSL and the dependencies of those libraries. The AVS Device SDK client libraries are tested with the following versions of these dependencies:

| Library     | Minimum versions    |
| :------------- | :------------- |
| [libcurl](https://curl.haxx.se/download.html)      | 7.50.2       |
| [nghttp2](https://github.com/nghttp2/nghttp2)      | 1.0       |
| [OpenSSL](https://www.openssl.org/source/) | 1.0.2       |
| [Doxygen](http://www.stack.nl/~dimitri/doxygen/download.html)      | 1.8.13       |

### Core music provider dependencies  

Your build of the SDK is required to support playback of music. To enable support for [AVS music service providers](https://developer.amazon.com/docs/alexa-voice-service/music-service-providers.html) the following libraries and packages, and their dependencies, are required:

| Library     | Minimum version    |
| :------------- | :------------- |
| [Crypto library](https://gnupg.org/software/libgcrypt/index.html) - For HLS demuxers | 1.8.4 or earlier     |
| [libsoup](https://wiki.gnome.org/Projects/libsoup) - An HTTP client/server library used by GStreamer. | 2.6.5 or earlier       |
| [libfaad-dev](https://github.com/dsvensson/faad2) - AAC and HE-AAC decoding | 1.10.4       |

| Plug-In     | Minimum version    |
| :------------- | :------------- |
| [GStreamer Bad Plug-ins](https://gstreamer.freedesktop.org/releases/gst-plugins-bad/1.10.4.html) | 1.10.4       |

## Optional features and their dependencies

In additional to the requirements above, if you choose to build any of the following features, they will require the following libraries and components:

### Alexa alerts

If you choose to use Alexa alerts, the SQLite library is required, and your client must meet these additional requirements:

* The device system clock must be set to UTC time. We recommend using the [network time protocol (NTP)](http://support.ntp.org/bin/view/Main/WebHome).
* A file system is required.

| Library     | Minimum version    |
| :------------- | :------------- |
| [SQLite](ttps://www.sqlite.org/download.html) | 3.19.3       |

### AVS sample app

Building the AVS sample app is optional. If you are building for Mac, Linux, Raspberry Pi, Windows, or Android (without using the Android Microphone and Media Player), these libraries and their dependencies are required:

| Library     | Minimum version    |
| :------------- | :------------- |
| [PortAudio](https://www.sqlite.org/download.html) | v190600_20161030       |
| [GStreamer](https://gstreamer.freedesktop.org/releases/1.8/) | 1.8.3       |

**NOTE**: The sample app will work with or without a wake word engine (WWE). If the SDK is built without a WWE, hands-free mode will be disabled in the sample app.

### Bluetooth

Building with Bluetooth is optional and is currently limited to Linux and Raspberry Pi. `A2DP-SINK`, `A2DP-SOURCE` (Linux only), `AVRCPTarget` (Linux only), and `AVRCPController` profiles are supported.

If you choose to build with Bluetooth, these libraries and modules, and their dependencies, must be installed:

| Library     | Minimum version    |
| :------------- | :------------- |
| [SBC Library](https://git.kernel.org/pub/scm/bluetooth/sbc.git/tree/) |   1.3     |
| [BlueZ 5.37](http://www.bluez.org/download/) |   5.37     |
| [libpulse-dev](https://packages.debian.org/sid/libpulse-dev) - Only required if enabling this [Cmake variable](https://github.com/alexa/avs-device-sdk/wiki/Dependencies#bluetooth): `BLUETOOTH_BLUEZ_PULSEAUDIO_OVERRIDE_ENDPOINTS` | 8.0 |

Note: You may substitute PulseAudio with a comparable application of your choice.

| Module     | Minimum version    |
| :------------- | :------------- |
| [PulseAudio](https://www.freedesktop.org/wiki/Software/PulseAudio/) and [PulseAudio Bluetooth](https://www.freedesktop.org/wiki/Software/PulseAudio/Documentation/User/Bluetooth/) *(or equivalent)* - Used to handle audio routing |   12.2 or earlier     |

### MediaPlayerInterface

Building the reference implementation of the `MediaPlayerInterface` is optional; but requires these libraries and plug-ins, and their dependencies:

| Library     | Minimum version    |
| :------------- | :------------- |
| [GStreamer](https://gstreamer.freedesktop.org/releases/1.8/) | 1.8.3       |

| Plug-in     | Minimum version    |
| :------------- | :------------- |
| [GStreamer Base Plug-ins](https://gstreamer.freedesktop.org/releases/gst-plugins-base/1.8.3.html) | 1.8.3       |
| [GStreamer Good Plug-ins](https://gstreamer.freedesktop.org/releases/gst-plugins-good/1.8.3.html) | 1.8.3       |
| [GStreamer Libav Plug-ins](https://gstreamer.freedesktop.org/releases/gst-plugins-good/1.8.3.html) OR <Br> [GStreamer Ugly Plug-ins](https://gstreamer.freedesktop.org/releases/gst-plugins-ugly/1.8.3.html) <br>For decoding MP3 data | 1.8.3       |

### Opus encoding

[Opus](http://opus-codec.org/) encoding is optional. To enable Opus, [libopus](http://opus-codec.org/docs/) must be installed, and you must include the [`-DOPUS=ON`](https://github.com/alexa/avs-device-sdk/wiki/cmake-parameters#additional-options) flag in your CMake command.

| Library     | Minimum version    |
| :------------- | :------------- |
| [libopus](http://opus-codec.org/) |   1.2.1     |

### SAMPLE-AES decryption

SAMPLE-AES decryption is optional. To enable SAMPLE-AES decryption for audio streamed to your device, the FFMPEG library is required, and this option [must be enabled in your CMake command](https://github.com/alexa/avs-device-sdk/wiki/cmake-parameters#additional-options).

This option is available for all platforms, however, FFMEG is required for Android builds.

| Library     | Minimum version    |
| :------------- | :------------- |
| [FFmpeg](https://www.ffmpeg.org/download.html) |   4.0.0     |


## Build for Android

In additional to the requirements above, to build for Android the following libraries and packages, and their dependencies, are required:

| Library     | Minimum version    |
| :------------- | :------------- |
| [Android SDK tools](https://developer.android.com/studio/#comand-tools) - Included in [Android Studio](https://developer.android.com/studio/#downloads) by default. |   1.0.39     |
| [Android Native Development Kit (NDK)](https://developer.android.com/ndk/downloads/). Required for the `MediaPlayerInterface` and sample app integration. |   -     |
| [FFmpeg](https://www.ffmpeg.org/download.html) |   4.0.0     |
| [OpenSL ES](https://developer.android.com/ndk/guides/audio/opensl/opensl-for-android) *(with equalizer support)* - Any full-featured Android OS or a third-party equalizer software should meet these requirements. |   -    |
