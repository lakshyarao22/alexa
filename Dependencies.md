This is a list of requirements and dependencies for the AVS Device SDK. Quick start guides are available for the following platforms:  

* [Raspberry Pi](https://github.com/alexa/avs-device-sdk/wiki/Raspberry-Pi-Quick-Start-Guide)  
* [macOS](https://github.com/alexa/avs-device-sdk/wiki/macOS-Quick-Start-Guide)  
* [Ubuntu Linux](https://github.com/alexa/avs-device-sdk/wiki/Ubuntu-Linux-Quick-Start-Guide)  

### Core dependencies  
* C++ 11 or later
* [GNU Compiler Collection (GCC) 4.8.5](https://gcc.gnu.org/) or later **OR** [Clang 3.9](http://clang.llvm.org/get_started.html) or later
* [CMake 3.1](https://cmake.org/download/) or later
* [libcurl 7.50.2](https://curl.haxx.se/download.html) or later
* [nghttp2 1.0](https://github.com/nghttp2/nghttp2) or later
* [OpenSSL 1.0.2](https://www.openssl.org/source/) or later
* [Doxygen 1.8.13](http://www.stack.nl/~dimitri/doxygen/download.html) or later (required to build API documentation)  
* [SQLite 3.19.3](https://www.sqlite.org/download.html) or later (required for Alerts)  
* For Alerts to work as expected:  
  * The device system clock must be set to UTC time. We recommend using `NTP` to do this   
  * A file system is required  

### MediaPlayer reference implementation dependencies
Building the reference implementation of the `MediaPlayerInterface` (the class `MediaPlayer`) is optional, but requires:  
* [GStreamer 1.10.4](https://gstreamer.freedesktop.org/documentation/installing/index.html) (or later) and the following GStreamer plug-ins:  
* [GStreamer Base Plugins 1.10.4](https://gstreamer.freedesktop.org/releases/gst-plugins-base/1.10.4.html) or later.
* [GStreamer Good Plugins 1.10.4](https://gstreamer.freedesktop.org/releases/gst-plugins-good/1.10.4.html) or later.
* [GStreamer Libav Plugin 1.10.4](https://gstreamer.freedesktop.org/releases/gst-libav/1.10.4.html) or later **OR**
* [GStreamer Ugly Plugins 1.10.4](https://gstreamer.freedesktop.org/releases/gst-plugins-ugly/1.10.4.html) or later, for decoding MP3 data.

**NOTE**: The plugins may depend on libraries which need to be installed for the GStreamer based `MediaPlayer` to work correctly.  

### Sample app dependencies  
Building the sample app is optional, but requires:  
* [PortAudio v190600_20161030](http://www.portaudio.com/download.html)
* GStreamer

**NOTE**: The sample app will work with or without a wake word engine. If built without a wake word engine, hands-free mode will be disabled in the sample app.  

### Music provider dependencies  

The following codecs and packages are required:  
* [GStreamer Bad Plugins 1.10.4](https://gstreamer.freedesktop.org/releases/gst-plugins-bad/1.10.4.html) or later.  
* [Crypto Library](https://gnupg.org/software/libgcrypt/index.html) for HLS demuxers.
* [libsoup]( https://wiki.gnome.org/Projects/libsoup) an HTTP client/server library used by GStreamer.
* [libfaad-dev](https://github.com/dsvensson/faad2) - AAC and HE-AAC decoding.
