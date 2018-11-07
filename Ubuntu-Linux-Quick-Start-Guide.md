These instructions will get you up and running quickly on a machine running **Ubuntu Linux 16.04 LTS**. For all other distributions, please review dependencies, build options, and make commands for Generic Linux.  

## Prerequisites  
This guide assumes that:

* you are running Ubuntu Linux 16.04 LTS  
* you have a microphone
* your Ubuntu box is capable of audio output through headphones or a speaker  

## Register a product

Follow these instructions to [register your product and create a security profile](https://github.com/alexa/avs-device-sdk/wiki/Create-Security-Profile). During this process, you will download a config.json to your machine that is used to complete configuration of the sample app.

Note: If you already have a registered product that you can use for testing, you may use it, but it must be enabled for use with Code Based Linking (CBL).

Here are the instructions on [how to enable CBL for your device](https://developer.amazon.com/docs/alexa-voice-service/code-based-linking-other-platforms.html#step1).

## Setup

1. The first step is to make sure your machine has the latest package lists and then install the latest version of each package in that list:
   ```shell
   sudo apt-get update && sudo apt-get upgrade -y
   ```
2. We need somewhere to put everything, so let's create some folders. This guide presumes that everything is built in `{home}`, which we will presume is your home directory. If you choose to use different folder names, please update the commands throughout this guide accordingly:
   ```shell
   mkdir sdk-folder

   cd sdk-folder
   mkdir sdk-build sdk-source third-party application-necessities
   ```  
3. Now, let's set up our toolchain:  
   ```shell
   sudo apt-get install -y git gcc cmake openssl clang-format
   ```
4. Next, we need to download dependencies available from apt-get:
   ```shell
   sudo apt-get install -y openssl libgstreamer1.0-dev \
   libgstreamer-plugins-base1.0-dev gstreamer1.0-plugins-good \
   libgstreamer-plugins-good1.0-dev libgstreamer-plugins-bad1.0-dev \
   gstreamer1.0-libav pulseaudio doxygen libsqlite3-dev repo libasound2-dev
   ```  
5. The AVS Device SDK relies on an HTTP2 connection with the Alexa service. To address this requirement, we're going to build curl with nghttp2 and openssl. If you would prefer to build with mbed TLS, [click here for instructions](https://github.com/alexa/avs-device-sdk/wiki/Build-libcurl-with-mbed-TLS-and-nghttp2):
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
6. PortAudio is required to record microphone data. Run this command to install and configure PortAudio:  
   ```shell
   cd ~/sdk-folder/third-party
   wget -c http://www.portaudio.com/archives/pa_stable_v190600_20161030.tgz \
   tar zxf pa_stable_v190600_20161030.tgz

   cd portaudio
   ./configure --without-jack && make  
   ```
7. Now it's time to clone the AVS Device SDK. Navigate to your sdk-source folder and run this command:  
   ```shell
   cd ~/sdk-folder/sdk-source
   git clone git://github.com/alexa/avs-device-sdk.git
   ```
8. Assuming the download succeeded, the next step is to build the SDK. This command does a few things:  
   * It declares that PortAudio is used to capture microphone data and points to its lib path and includes directory
   * It declares that gstreamer is installed and will be used when you build the SampleApp
   * It declares that the wake word detector is **OFF**  
     * Linux supports wake word detectors from Sensory and Kitt.ai. Each requires a license from the provider. For instructions to build with a wake word detector, please see [Build Options](https://github.com/alexa/avs-device-sdk/wiki/Build-Options).  

   **IMPORTANT**: Replace `{home}` with the full path to your home directory (do not use `~/`):  
   ```shell
   cd /{home}/sdk-folder/sdk-build

   cmake /{home}/sdk-folder/sdk-source/avs-device-sdk \
   -DSENSORY_KEY_WORD_DETECTOR=OFF \
   -DGSTREAMER_MEDIA_PLAYER=ON \
   -DPORTAUDIO=ON \
   -DPORTAUDIO_LIB_PATH=/{home}/sdk-folder/third-party/portaudio/lib/.libs/libportaudio.a \
   -DPORTAUDIO_INCLUDE_DIR=/{home}/sdk-folder/third-party/portaudio/include

   make
   ```   

## Setup your configuration

### `genConfig.sh`

`genConfig.sh` is a configuration file generator script that is included with v1.10.0 or greater of the SDK, located in the **tools/Install** folder. It uses data from **config.json** and the arguments below, to populate `AlexaClientSDKConfig.json` in order to authorize with LWA.

Follow these steps to setup `AlexaClientSDKConfig.json` using `genConfig.sh`:

1. Move the **config.json** you downloaded when you [created your security profile](https://github.com/alexa/avs-device-sdk/wiki/Create-Security-Profile#create-a-security-profile) into the **tools/Install** folder of the SDK.

    ```sh
    mv ~/{{download location}}/config.json
    {{path to source folder}}/avs-device-sdk/tools/Install
    ```

    **config.json** should be populated with the `clientID` and `productID` values generate in the Security Profile process.

    Example:

    ```sh
        {
         "deviceInfo": {
          "clientId": "amzn1.application-oa2-client.{{string}}",
          "productId": "Test_123"
         }
        }
    ```

2. Run `genConfig.sh`, including the following as arguments:

    ```sh
    cd {{path to source folder}}/avs-device-sdk/tools/Install

    bash genConfig.sh config.json {device serial number} \
    /{{path to database}} \
    /{{path to source folder}}/avs-device-sdk \
    {{path to build}}/Integration/AlexaClientSDKConfig.json
    ```

    Parameters:

    * `config.json`: Downloaded when you created your security profile. It should contain the following:
    ```
    "clientId": "{OAuth client ID}"
    "productId": "{product name for the device}"
    ```
    * **{{device serial number}}**: Each instance of the SDK requires a unique **Device Serial Number** (also found in the **deviceInfo** object). This is provided by you, and in some instances may match your product's SKU. By default the value is `123456`.
    * **{{path to database}}**: Specifies where the databases will be located.
    * **{{path to source folder}}**: Specifies the root directory where the avs-device-sdk source code is located.
    * **{{path to build}}**: Specifies where the build folder is, which contains the **Integration/AlexaClientSDKConfig.json** file.

**IMPORTANT**: Create a backup of your `AlexaClientSDKConfig.json` file. Subsequent builds will reset the contents of this file.

## Run and authorize

Navigate to your build folder (~/sdk-folder/sdk-build), then:
1. Start the sample app:
```shell
./SampleApp/src/SampleApp Integration/AlexaClientSDKConfig.json
```
2. Wait for the sample app to display a message like this:
```shell
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
7. Wait for `CBLAuthDelegate` to successfully get an access token and refresh token from Login With Amazon (LWA). At this point the sample app will print a message like this:
```shell
########################################
#       Alexa is currently idle!       #
########################################
```
8. You are now ready to use the sample app. The next time you start the sample app, you will not need to go through the authorization process.

A couple more details:
* If you exit out of sample app via the `k` command, the `CBLAuthDelegate` database will be cleared and you will need to reauthorize your client.
* If you want to move this authorization to another sample app installation, you need to copy the **deviceInfo** object within `AlexaClientSDKConfig.json` to the new installation.  You will also need to copy the file `"/{home}/sdk-folder/application-necessities/cblAuthDelegate.db"` to the new installation, and update **AlexaClientSDKConfig.json** in the new installation so that the **cblAuthDelegate's databaseFilePath** property points to it.

## Integration and unit tests

1. Run integration and unit tests to ensure that the AVS Device SDK is functioning as designed.
    * Use this command to run integration tests:
       ```shell
      make all integration
       ```
    * Use this command to run unit tests:  
       ```shell
       make all test
       ```
    For additional details, see [Unit and Integration Tests](https://github.com/alexa/avs-device-sdk/wiki/Unit-and-Integration-Tests).

## Bluetooth

Building with Bluetooth is optional, and is currently limited to Linux and Raspberry Pi. The `A2DP-SINK`, `A2DP-SOURCE`, `AVRCPTarget`, and `AVRCPController` profiles are supported for Linux/Ubuntu.

To enable Bluetooth on Linux, follow these steps:

### 1. Install Dependencies

In order to use Bluetooth, you must install these dependencies:

* Core [Bluetooth dependencies](https://github.com/alexa/avs-device-sdk/wiki/Dependencies#bluetooth-dependencies)
* PulseAudio
* PulseAudio bluetooth modules:
```
sudo apt-get install pulseaudio-module-bluetooth
```

### 2. Initialize the SDK

To use Bluetooth with the SDK, you must build and initialize the SDK before you load the PulseAudio bluetooth modules. This establishes order-of-priority for sink audio handing, setting the SDK as the controller.

### 3. Load bluetooth modules

After the SDK has been initialized, you'll need to unload and then load the PulseAudio bluetooth modules.

For example:

```
pactl unload-module module-bluetooth-discover; pactl unload-module module-bluetooth-policy; pactl load-module module-bluetooth-discover; pactl load-module module-bluetooth-policy
```

Note: When installing these modules, if they fail upon unload (because they weren't originally loaded), this is not a critical error -- and should not affect your Bluetooth integration.

### 4. AVRCPTarget

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
