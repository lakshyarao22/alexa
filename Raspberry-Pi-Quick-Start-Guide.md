This guide provides step-by-step instructions to set up the Alexa Voice Service (AVS) Device SDK on a Raspberry Pi running [Raspbian Stretch with Desktop](https://www.raspberrypi.org/downloads/raspbian/). When finished, you'll have a working sample app to test interactions with Alexa.  

**IMPORTANT**: If your Raspberry Pi is not running Raspbian Stretch With Desktop use [these instructions to upgrade](https://www.raspberrypi.org/blog/raspbian-stretch/). If you choose to build with Raspbian Jessie, you need to build certain dependencies from source (see commit [a5646fc](https://github.com/alexa/avs-device-sdk/wiki/Raspberry-Pi-Quick-Start-Guide/a5646fc9e6dde8128c940b86a9bbece7f65c1ace) for instructions).  

**Table of Contents**
* [1. Install and configure dependencies for the SDK](#1-install-and-configure-dependencies-for-the-sdk)  
* [2. Build the SDK](#2-build-the-sdk)    
* [3. Obtain credentials and set up your local auth server](#3-obtain-credentials-and-set-up-your-local-auth-server)  
* [4. Run the sample app](#4-run-the-sample-app)
* [Next Steps](#next-steps)

## 1. Install and configure dependencies for the SDK  

**IMPORTANT**: This guide assumes that all commands are run from your home directory (/home/pi). If you choose a different destination for your files, it's important that

### 1.1 Create the folder structure

Let's create a few folders to organize our files. Open terminal and run this command from your home directory (~/):

```
cd /home/pi/ && mkdir sdk-folder && cd sdk-folder && mkdir sdk-build sdk-source third-party application-necessities && cd application-necessities && mkdir sound-files
```

### 1.2 Install dependencies

The AVS Device SDK requires libraries to:  

1. Maintain an `HTTP2` connection with AVS  
2. Play Alexa TTS and music  
3. Record audio from the microphone  
4. Store records in a database (persistent storage)  

The first step is to update apt-get. This ensures that you have access to required dependencies.

```
sudo apt-get update
```

Then run:

```
sudo apt-get -y install git gcc cmake build-essential libsqlite3-dev libcurl4-openssl-dev libfaad-dev libsoup2.4-dev libgcrypt20-dev libgstreamer-plugins-bad1.0-dev gstreamer1.0-plugins-good libasound2-dev doxygen
```

PortAudio is required to record microphone data. Run this command to install and configure PortAudio:  

```
cd /home/pi/sdk-folder/third-party && wget -c http://www.portaudio.com/archives/pa_stable_v190600_20161030.tgz && tar zxf pa_stable_v190600_20161030.tgz && cd portaudio && ./configure --without-jack && make
```  

commentjson is required to parse comments in `AlexaClientSDKConfig.json`. Run this command to install commentjson:

```
pip install commentjson
```

### 1.3 Clone the AVS Device SDK and the Sensory wake word engine

1. Let's clone the SDK into the sdk-source folder:  
    `cd /home/pi/sdk-folder/sdk-source && git clone git://github.com/alexa/avs-device-sdk.git`
2. Next, let's clone the sensory wake word engine into our third-party directory:   
    `cd /home/pi/sdk-folder/third-party && git clone git://github.com/Sensory/alexa-rpi.git`
3. Now let's run the licensing script and accept the licensing agreement. This is required to use Sensory's wake word engine:   
     `cd /home/pi/sdk-folder/third-party/alexa-rpi/bin/ && ./license.sh`

## 2. Build the SDK

1. Run cmake to generate the build dependencies. This command declares that the wake word engine and gstreamer are enabled, and provides paths to the wake word engine and PortAudio:
     ```
     cd /home/pi/sdk-folder/sdk-build && cmake /home/pi/sdk-folder/sdk-source/avs-device-sdk -DSENSORY_KEY_WORD_DETECTOR=ON -DSENSORY_KEY_WORD_DETECTOR_LIB_PATH=/home/pi/sdk-folder/third-party/alexa-rpi/lib/libsnsr.a -DSENSORY_KEY_WORD_DETECTOR_INCLUDE_DIR=/home/pi/sdk-folder/third-party/alexa-rpi/include -DGSTREAMER_MEDIA_PLAYER=ON -DPORTAUDIO=ON -DPORTAUDIO_LIB_PATH=/home/pi/sdk-folder/third-party/portaudio/lib/.libs/libportaudio.a -DPORTAUDIO_INCLUDE_DIR=/home/pi/sdk-folder/third-party/portaudio/include
     ```  

   **NOTE**: This is sample cmake command configured for the sample app. There are other cmake options available for use outside of this project -- for the full list [click here](https://github.com/alexa/avs-device-sdk/wiki/Build-Options).  

2. Here's the fun part, let's build! For this project we're only building the Sample App. Run this command:
    `make SampleApp -j2`

   **NOTE**: The `j2` flag allows you to run 2 proccesses in parallel, which should speed up the build. Try the `-j3` or `-j4` option if you're feeling bold, but you run the risk of overheating the Pi (maybe even melting it).

If you want to build the rest of the SDK, including unit and integration tests, run `make` instead of `make SampleApp`.

## 3. Obtain credentials and set up your local auth server

In this section we are going to setup and run a local authorization server, which we'll use to obtain a refresh token. This refresh token, along with your **Client ID** and **Client Secret** are exchanged for an access token, which the sample app needs to send to Alexa with each event (request).  

### 3.1 Register your product with Amazon

Follow these [instructions](https://github.com/alexa/alexa-avs-sample-app/wiki/Create-Security-Profile) to register your product and create a security profile. You can skip this step if you have a registered product you'd like to test with.

**IMPORTANT**: The allowed origin under web settings should be <http://localhost:3000> or <https://localhost:3000>.
The return URL under web settings should be <http://localhost:3000/authresponse> or <https://localhost:3000/authresponse>.

Make sure you save the **Product ID** from the **Product information** tab, and your **Client ID** and **Client Secret** from the **Security Profile** tab. You'll need these params to configure the authorization server.  

### 3.2 Update AlexaClientSDKConfig.json

Use your favorite text editor to open `/home/pi/sdk-folder/sdk-build/Integration/AlexaClientSDKConfig.json`.

**TIP**: If you prefer to use a GUI-based text editor, run this command to install **gedit**: `sudo apt-get install gedit`.  

For example:   
* Terminal editor: `nano /home/pi/sdk-folder/sdk-build/Integration/AlexaClientSDKConfig.json`  
* Gedit-based editor: `gedit /home/pi/sdk-folder/sdk-build/Integration/AlexaClientSDKConfig.json`  

Now fill in your product-specific values and save. Alternatively, you can use the template provided below, which includes paths to the database files required for the sample app.  

If you choose to use the template, follow these instructions:  

1. Replace the contents of `AlexaClientSDKConfig.json` with this JSON blob:  

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
            "databaseFilePath":"/home/pi/sdk-folder/application-necessities/alerts.db"
       },
       "settings":{
            "databaseFilePath":"/home/pi/sdk-folder/application-necessities/settings.db",
            "defaultAVSClientSettings":{
                "locale":"en-US"
            }
       },
       "certifiedSender":{
            "databaseFilePath":"/home/pi/sdk-folder/application-necessities/certifiedSender.db"
       },
       "notifications":{
           "databaseFilePath":"/home/pi/sdk-folder/application-necessities/notifications.db"
       }
    }
    ```

2. Enter the `clientId`, `clientSecret`, and `productId` that you saved during device registration and save.

   **NOTE**: Do not remove the quotes and make sure there are no extra characters or spaces! The required values are strings.  

   **NOTE 2**: `deviceSerialNumber` is pre-populated for this project, however, a commercial product should use a serial number or other unique identified for the device.  

The locale is set to US English by default in the sample JSON, however, British English, (en-GB), German (de-DE) and Indian English (en-IN) are supported. Feel free to test each language.

**IMPORTANT**: It is a good idea to save a backup of this file. Subsequent builds may overwrite the values in `AlexaClientSDKConfig.json`.  

### 3.3 Obtain a refresh token  

After you've updated `AlexaClientSDKConfig.json`, run `AuthServer.py` to kick-off the token exchange:  

```
cd /home/pi/sdk-folder/sdk-build && python AuthServer/AuthServer.py
```

**NOTE:** You may need to change the **locale** settings for your Pi, as some Raspbian images default to `amazon.co.uk` to `amazon.com`.

Open your browser and navigate to <http://localhost:3000>. Login with your Amazon credentials and follow the instructions provided.  

![Login Screen](https://m.media-amazon.com/images/G/01/mobile-apps/dex/avs/sdk/3.png")

**TIP**: If you are running Raspbian Lite, run `AuthServer.py` on a machine with access to a web browser and copy the refresh token into `AlexaClientSDKConfig.json` on your Pi.  

#### Common Issues  

These are the most common issues encountered when trying to obtain a refresh token:

* Incorrect information in `AlexaClientSDKConfig.json`.

### 3.4 Test the microphone  

A fresh Raspbian Stretch image requires updates to `/.asoundrc` before we can test the microphone. With your favorite text editor, open `~/.asoundrc`. For example:

* Terminal editor:  `nano ~/.asoundrc`  
* GUI-based editor:  `gedit ~/.asoundrc`  

Then add these lines to the file and save:  

```
pcm.!default {
  type asym
   playback.pcm {
     type plug
     slave.pcm "hw:0,0"
   }
   capture.pcm {
     type plug
     slave.pcm "hw:1,0"
   }
}
```

To ensure the microphone is capturing audio data, run this command:  

```
sudo apt-get install sox -y && rec test.wav
```

If everything works, you should see a message indicating that audio is recording. To exit, hit **Control + C**.

## 4. Run the sample app

Run this command to launch the sample app. This includes the path to your configuration file and the Sensory wake word model:

```
cd /home/pi/sdk-folder/sdk-build/SampleApp/src && TZ=UTC ./SampleApp /home/pi/sdk-folder/sdk-build/Integration/AlexaClientSDKConfig.json /home/pi/sdk-folder/third-party/alexa-rpi/models
```  

**IMPORTANT**: If you encounter any issues, use the debug option for additional information. `debug1` through `debug9` are accepted values. For example:  

```
TZ=UTC ./SampleApp /home/pi/sdk-folder/sdk-build/Integration/AlexaClientSDKConfig.json /home/pi/sdk-folder/third-party/alexa-rpi/models debug9
```  

## Next Steps  

* [Build Options](https://github.com/alexa/avs-device-sdk/wiki/Build-Options)  
* [Unit and Integration Tests](https://github.com/alexa/avs-device-sdk/wiki/Unit-and-Integration-Tests)  
