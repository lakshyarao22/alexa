These instructions will get you up and running on a machine running **Ubuntu Linux 16.04 LTS**. For all other distributions, please review dependencies and Cmake parameters for Generic Linux.

## Prerequisites  

* Ubuntu Linux 16.04 LTS  
* Hardware audio input/output (microphone and speakers or headphones)

## Get started

1. [Register an AVS Product and Create a Security Profile](https://github.com/alexa/avs-device-sdk/wiki/Create-Security-Profile), if you haven't already. It must be enabled for [Code-Based Linking](https://developer.amazon.com/docs/alexa-voice-service/code-based-linking-other-platforms.html#step1).

2. Update and install the latest packages:
   ```shell
   sudo apt-get update && sudo apt-get upgrade -y
   ```
3. Set up your build environment. Create a **my_project** folder, and these subfolders: **build**, **source**, **third-party**, **application-necessities** > **sound-files**.

    ```shell
    mkdir my_project

    cd {my_project}
    mkdir build source third-party application-necessities

    cd application-necessities
    mkdir sound-files

    cd {home}/{my_project}
    ```

4. Install toolchain: git, gcc, cmake, openssl, clang-format:
   ```shell
   sudo apt-get install -y git gcc cmake openssl clang-format
   ```
5. Install the core [SDK dependencies](https://github.com/alexa/avs-device-sdk/wiki/Dependencies):

    ```shell
    sudo apt-get install -y libgstreamer1.0-dev \
    libgstreamer-plugins-base1.0-dev gstreamer1.0-plugins-good \
    libgstreamer-plugins-good1.0-dev libgstreamer-plugins-bad1.0-dev \
    gstreamer1.0-libav pulseaudio doxygen libsqlite3-dev repo libasound2-dev
    ```

     **IMPORTANT**: Make sure that command ran successfully, and that no errors were thrown. If for any reason the install command fails, run brew install for each dependency individually.  

6. The AVS Device SDK relies on an HTTP2 connection with the Alexa service. To address this requirement, we're going to build curl with nghttp2 and openssl. If you would prefer to build with mbed TLS, [click here for instructions](https://github.com/alexa/avs-device-sdk/wiki/Build-libcurl-with-mbed-TLS-and-nghttp2):
   * Let's install nghttp2. **Note:** This command will download and install dependencies for nghttp2:  
     ```shell
     sudo apt-get
     install -y g++ \
     make binutils autoconf automake autotools-dev libtool \
     pkg-config zlib1g-dev libcunit1-dev libssl-dev libxml2-dev libev-dev \
     libevent-dev libjansson-dev libjemalloc-dev cython python3-dev \
     python-setuptools  
     ```
   * Now, let's build nghttp2 from source:
     ```shell
     cd ~/sdk-folder/third-party
     git clone https://github.com/tatsuhiro-t/nghttp2.git

     cd nghttp2
     autoreconf -i
     automake
     autoconf
     ./configure

     make
     sudo make install
     ```
   * Download the latest version of curl and configure with support for nghttp2 and openssl:
     ```shell
     cd ~/sdk-folder/third-party
     wget http://curl.haxx.se/download/curl-7.54.0.tar.bz2
     tar -xvjf curl-7.54.0.tar.bz2

     cd curl-7.54.0
     ./configure --with-nghttp2=/usr/local --with-ssl

     make
     sudo make install

     sudo ldconfig     
     ```

   * Test curl:
     ```shell
     curl -I https://nghttp2.org/
     ```
     If the request succeeds, you will see a message like this:  
     ```shell
     HTTP/2 200
     date: Fri, 15 Dec 2017 18:13:26 GMT
     content-type: text/html
     last-modified: Sat, 25 Nov 2017 14:02:51 GMT
     etag: "5a19780b-19e1"
     accept-ranges: bytes
     content-length: 6625
     x-backend-header-rtt: 0.001021
     strict-transport-security: max-age=31536000
     server: nghttpx
     via: 2 nghttpx
     x-frame-options: SAMEORIGIN
     x-xss-protection: 1; mode=block
     x-content-type-options: nosniff
     ```
7. Install and configure PortAudio. Portaudio is required to stream microphone input/output data.

     ```shell
     cd {home}/my_project/third-party
     wget -c http://www.portaudio.com/archives/pa_stable_v190600_20161030.tgz
     tar xf pa_stable_v190600_20161030.tgz

     cd portaudio
     ./configure --without-jack && make

     make
     ```  
8. Clone the SDK into the **~/my_project/source** folder:  

    ```shell
    cd {home}/{my_project}/source
    git clone git://github.com/alexa/avs-device-sdk.git
    ```

## Build the SDK

1. Use the [CMake](https://cmake.org/) parameters in this section to customize how the SDK builds. To get debug logs from the sample app, include the `-DCMAKE_BUILD_TYPE=DEBUG` option. For more information, see [Debug builds](https://github.com/alexa/avs-device-sdk/wiki/cmake-options#debug-builds).

In this example it is declared that gstreamer is enabled, and provides the path to PortAudio, and also enables debug logging (which is optional):

```shell
cd {home}/my_project/build

cmake /{home}/{my_project}/source/avs-device-sdk \
-DGSTREAMER_MEDIA_PLAYER=ON \
-DCURL_LIBRARY=/usr/local/opt/curl-openssl/lib/libcurl.dylib \
-DCURL_INCLUDE_DIR=/usr/local/opt/curl-openssl/include \
-DPORTAUDIO=ON \
-DPORTAUDIO_LIB_PATH=/{home}/{my_project}/third-party/portaudio/lib/.libs/libportaudio.a \
-DPORTAUDIO_INCLUDE_DIR=/{home}/{my_project}/third-party/portaudio/include
-DCMAKE_BUILD_TYPE=DEBUG

make
```

2. Build the SDK:

```shell
 make SampleApp -j2
 ```

 **Note**: You can use `-j3` or `j4` to run processes in parallel during make.

    If you want to build the full SDK, including unit and integration tests, run `make` instead of `make SampleApp`.

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

## Authorization

After you have built the SDK, you'll need to configure and authorize it to access AVS using Login with Amazon (LWA). To do this, follow [these authorization instructions](https://github.com/alexa/avs-device-sdk/wiki/Authorization#macOS,-Ubuntu-Linux-16.04-LTS,-and-Raspberry Pi).

## Integration and unit tests

1. After you've built the AVS Device SDK, we recommend running integration and unit tests to make sure that the SDK is functioning as designed.
    * Use this command to run integration tests:
       ```shell
      make all integration
       ```
    * Use this command to run unit tests:  
       ```shell
       make all test
       ```
    For additional details, see [Unit and Integration Tests](https://github.com/alexa/avs-device-sdk/wiki/Unit-and-Integration-Tests).