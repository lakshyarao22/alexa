These instructions will get you up and running quickly on a machine running **Ubuntu Linux 16.04 LTS**. For all other distributions, please review dependencies, build options and make commands for Generic Linux.  

## Prerequisites  
This guide assumes that:

* you are running Ubuntu Linux 16.04 LTS  
* you have a microphone
* your Ubuntu box is capable of audio output through headphones or a speaker  

## Register a product
Follow these instructions to [register your product and create a security profile](https://github.com/alexa/avs-device-sdk/wiki/Create-Security-Profile).

Make sure you save the **Product ID** from the **Product information** tab, and your **Client ID** from the **Other devices and platforms** tab from within the **Security Profile** section.

Note: If you already have a registered product that you can use for testing, you may use it but it must be enabled for use with Code Based Linking (CBL). You can find steps for enabling CBL for your device [Here](https://developer.amazon.com/docs/alexa-voice-service/code-based-linking-other-platforms.html#step1) (Step 1).

**IMPORTANT**: When you capture the **Client ID**, make sure it is from the **Other devices and platforms** tab within the **Security Profile** section, and **NOT** from the **Client ID** from the top of the **Product information**, **Security Profile**, or **Capabilities** tabs. The **Client ID** generated within your **Security Profile** is the required **Client ID**.

## Setup

1. The first step is to make sure your machine has the latest package lists and then install the latest version of each package in that list:
   ```
   sudo apt-get update && sudo apt-get upgrade -y
   ```
2. We need somewhere to put everything, so let's create some folders. This guide presumes that everything is built in `{HOME}`, which we will presume is your home directory. If you choose to use different folder names, please update the commands throughout this guide accordingly:
   ```
   mkdir sdk-folder && cd sdk-folder && mkdir sdk-build sdk-source third-party application-necessities
   ```  
3. Now, let's set up our toolchain:  
   ```
   sudo apt-get install -y git gcc cmake openssl clang-format
   ```
4. Next, we need to download dependencies available from apt-get:
   ```
   sudo apt-get install -y openssl libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev gstreamer1.0-plugins-good libgstreamer-plugins-good1.0-dev libgstreamer-plugins-bad1.0-dev gstreamer1.0-libav pulseaudio doxygen libsqlite3-dev repo libasound2-dev
   ```  
5. The AVS Device SDK relies on an HTTP2 connection with the Alexa service. To address this requirement, we're going to build curl with nghttp2 and openssl. If you would prefer to build with mbed TLS, [click here for instructions](https://github.com/alexa/avs-device-sdk/wiki/Build-libcurl-with-mbed-TLS-and-nghttp2):
   * Let's install nghttp2. **Note:** This command will download and install dependencies for nghttp2:  
     ```
     sudo apt-get install -y g++ make binutils autoconf automake autotools-dev libtool pkg-config zlib1g-dev libcunit1-dev libssl-dev libxml2-dev libev-dev libevent-dev libjansson-dev libjemalloc-dev cython python3-dev python-setuptools  
     ```
   * Now, let's build nghttp2 from source:
     ```
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
     ```
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
     ```
     curl -I https://nghttp2.org/
     ```
     If the request succeeds, you will see a message like this:  
     ```
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
   ```
   cd ~/sdk-folder/third-party
   wget -c http://www.portaudio.com/archives/pa_stable_v190600_20161030.tgz && tar zxf pa_stable_v190600_20161030.tgz && cd portaudio && ./configure --without-jack && make  
   ```
7. Now it's time to clone the AVS Device SDK. Navigate to your sdk-source folder and run this command:  
   ```
   cd ~/sdk-folder/sdk-source && git clone git://github.com/alexa/avs-device-sdk.git
   ```
8. Assuming the download succeeded, the next step is to build the SDK. This command does a few things:  
   * It declares that PortAudio is used to capture microphone data and points to its lib path and includes directory
   * It declares that gstreamer is installed and will be used when you build the SampleApp
   * It declares that the wake word detector is **OFF**  
     * Linux supports wake word detectors from Sensory and Kitt.ai. Each requires a license from the provider. For instructions to build with a wake word detector, please see [Build Options](https://github.com/alexa/avs-device-sdk/wiki/Build-Options).  

   **IMPORTANT**: Replace `{HOME}` with the full path to your home directory (do not use `~/`):  
   ```
   cd /{HOME}/sdk-folder/sdk-build && cmake /{HOME}/sdk-folder/sdk-source/avs-device-sdk -DSENSORY_KEY_WORD_DETECTOR=OFF -DGSTREAMER_MEDIA_PLAYER=ON -DPORTAUDIO=ON -DPORTAUDIO_LIB_PATH=/{HOME}/sdk-folder/third-party/portaudio/lib/.libs/libportaudio.a -DPORTAUDIO_INCLUDE_DIR=/{HOME}/sdk-folder/third-party/portaudio/include && make
   ```   
### Bluetooth

## Setup your configuration
The sample app uses data within `AlexaClientSDKConfig.json` to obtain a refresh token, which along with your **Client ID** and **Product ID**, will be exchanged with LWA for access tokens. An access token is included in the header of every request made to Alexa.  

Using your favorite editor, open `~/sdk-folder/sdk-build/Integration/AlexaClientSDKConfig.json` and replace the contents with the JSON provided below. You'll need to do two things:
* Update the **deviceInfo** object in the config file with your **Product ID** and **Client ID** from the Developer Console.
* Replace all instances of `"databaseFilePath":"/{HOME}/sdk-folder/application-necessities/{{dataBase}}.db"` with the absolute path to that file.

*Each instance of the SDK requires a unique **Device Serial Number** (also found in the **deviceInfo** object). This is provided by you and in some instances may match your product's SKU. For this sample, it's pre-populated with `123456`.*

```
{
   "deviceInfo":{
     "deviceSerialNumber":"123456",
     "clientId":"YOUR_CLIENT_ID",
     "productId":"YOUR_PRODUCT_ID"
   },
   "cblAuthDelegate":{
       "databaseFilePath":"/{HOME}/sdk-folder/application-necessities/cblAuthDelegate.db"
   },
   "miscDatabase":{
       "databaseFilePath":"/{HOME}/sdk-folder/application-necessities/miscDatabase.db"
   },
   "alertsCapabilityAgent":{
       "databaseFilePath":"/{HOME}/sdk-folder/application-necessities/alerts.db"
   },
   "settings":{
       "databaseFilePath":"/{HOME}/sdk-folder/application-necessities/settings.db",
       "defaultAVSClientSettings":{
           "locale":"en-US"
       }
   },
   "certifiedSender":{
      "databaseFilePath":"/{HOME}/sdk-folder/application-necessities/certifiedSender.db"
   },
   "notifications":{
       "databaseFilePath":"/{HOME}/sdk-folder/application-necessities/notifications.db"
   }
}
```

IMPORTANT: Save a backup copy of your edited `AlexaClientSDKConfig.json`. Subsequent builds will reset the contents of this file.

## Run and authorize

Navigate to your build folder (~/sdk-folder/sdk-build), then:
1. Start the sample app:
```
./SampleApp/src/SampleApp Integration/AlexaClientSDKConfig.json
```
2. Wait for the sample app to display a message like this:
```
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
7. Wait (it may take as few seconds) for `CBLAuthDelegate` to successfully get an access token and refresh token from Login With Amazon (LWA). At this point the sample app will print a message like this:
```
########################################
#       Alexa is currently idle!       #
########################################
```
8. You are now ready to use the sample app. The next time you start the sample app, you will not need to go through the authorization process.

A couple more details:
* If you exit out of sample app via the `k` command, the `CBLAuthDelegate` database will be cleared and you will need to reauthorize your client.
* If you want to move this authorization to another sample app installation, you need to copy the **deviceInfo** object within `AlexaClientSDKConfig.json` to the new installation.  You will also need to copy the file `"/{HOME}/sdk-folder/application-necessities/cblAuthDelegate.db"` to the new installation, and update **AlexaClientSDKConfig.json** in the new installation so that the **cblAuthDelegate's databaseFilePath** property points to it.

## Integration and unit tests

1. Run integration and unit tests to ensure that the AVS Device SDK is functioning as designed.
    * Use this command to run integration tests:
       ```
      make all integration
       ```
    * Use this command to run unit tests:  
       ```
       make all test
       ```
    For additional details, see [Unit and Integration Tests](https://github.com/alexa/avs-device-sdk/wiki/Unit-and-Integration-Tests).

## Bluetooth

Building with Bluetooth is optional, and is currently limited to Linux and Raspberry Pi. This release supports `A2DP-SINK` and `AVRCP` profiles. In order to use Bluetooth for these platforms, you must install all Bluetooth [dependencies]() and **disable any processes which obtain an incoming Bluetooth audio stream**, such as:

### PulseAudio
If you are using `PulseAudio`, you **must** disable `PulseAudio` Bluetooth plugins. To do this:

1. Navigate to `/etc/pulse/default.pa` (or equivalent file), and comment out the following lines:
      ```
      ### Automatically load driver modules for Bluetooth hardware
      #.ifexists module-bluetooth-policy.so
      #load-module module-bluetooth-policy
      #.endif

      #.ifexists module-bluetooth-discover.so
      #load-module module-bluetooth-discover
      #.endif
      ```

2. Next, stop and restart PulseAudio with these commands (if auto-respawn is disabled):
      ```
      pulseaudio --kill
      pulseaudio --start
      ```