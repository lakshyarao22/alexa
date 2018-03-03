
These instructions will get you up and running quickly on a machine running **Ubuntu Linux 16.04 LTS**. For all other distributions, please review dependencies, build options and make commands for Generic Linux.  

## Prerequisites  
This guide assumes that:

* you are running Ubuntu Linux 16.04 LTS  
* you have a microphone
* your Ubuntu box is capable of audio output through headphones or a speaker  

## Register a device
Follow these [instructions](https://github.com/alexa/alexa-avs-sample-app/wiki/Create-Security-Profile) to register your product and create a security profile. You can skip this set if you have a registered product you'd like to test with.

**IMPORTANT**: The allowed origin under web settings should be http://localhost:3000 and https://localhost:3000. The return URL under web settings should be http://localhost:3000/authresponse and https://localhost:3000/authresponse.

Make sure you save the **Product ID** from the **Product information** tab, and your **Client ID** and **Client Secret** from the **Security Profile** tab.

## Quickstart instructions

1. The first step is to make sure your machine has the latest package lists and then install the latest version of each package in that list:
   ```
   sudo apt-get update && sudo apt-get upgrade -y
   ```
2. We need somewhere to put everything, so let's create some folders. This guide presumes that everything is built in `{HOME}`, which we will presume is your home directory. If you choose to use different folder names, please update the commands throughout this guide accoringly:
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
9. The AVS Device SDK uses a sample authorization service build in Python to obtain refresh and access tokens. In this step, we'll install PIP and other dependencies for the authorization service:  
   * Run this command to determine if PIP is installed:  
      ```
      pip --version
      ```
   * If PIP isn't installed, run:   
      ```
      sudo easy_install pip
      ```
   * Then, install Flask, requests, and commentjson:
      ```
      pip install --user flask requests commentjson
      ```
10. The authorization service uses data in `AlexaClientSDKConfig.json` to obtain a refresh token and access tokens.  

    Using your favorite editor, open `~/sdk-folder/sdk-build/Integration/AlexaClientSDKConfig.json` and replace the contents with the JSON provided below. You'll need to do two things:
      * Update the config file with your **Product ID**, **Client ID**, and **Client Secret** from the Developer Console.
      * Replace all instances of `{HOME}` with the path to your home directory (do not use `~/`).
    Each instance of the SDK requires a unique **Device Serial Number**. This is provided by you, and may in some instances match your product's SKU. For this sample, it's pre-populated with `123456`.  
    ```
    {
        "authDelegate":{
            "clientSecret":"YOUR_CLIENT_SECRET",
            "deviceSerialNumber":"123456",
            "refreshToken":"",
            "clientId":"YOUR_CLIENT_ID",
            "productId":"YOUR_PRODUCT_ID"
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

11. Now, run the authorization service. Open your browser and navigate to [http://localhost:3000](http://localhost:3000). Log in with your Amazon credentials and follow the instructions provided:
    ```
    cd ~/sdk-folder/sdk-build && python AuthServer/AuthServer.py
    ```  

    Save a backup copy of `AlexaClientSDKConfig.json`. Subsequent builds will clear the contents of this file.

12. Run integration and unit tests to ensure that the AVS Device SDK is functioning as designed.
    * Use this command to run integration tests:
       ```
      make all integration
       ```
    * Use this command to run unit tests:  
       ```
       make all test
       ```
    For additional details, see [Unit and Integration Tests](https://github.com/alexa/avs-device-sdk/wiki/Unit-and-Integration-Tests).  

13. If we've still got your attention, here's the best part! Run this command to launch the sample app, which allows you to interact with Alexa:  
    ```
    cd ~/sdk-folder/sdk-build/SampleApp/src
    ./SampleApp ~/sdk-folder/sdk-build/Integration/AlexaClientSDKConfig.json
    ```
    **TIP**: The sample app is an implementation of the [`DefaultClient`](https://github.com/alexa/avs-device-sdk/blob/1b712a1e978dc3fc6b5f4d31d95e6b3741e47f2a/ApplicationUtilities/DefaultClient/src/DefaultClient.cpp) class.  
