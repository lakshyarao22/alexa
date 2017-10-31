
This guide provides step-by-step instructions to set up the Alexa Voice Service (AVS) Device SDK on macOS. When finished, you'll have a working sample app to test interactions with Alexa.   

**Table of Contents**
* [1. Install and configure dependencies for the SDK](#1-install-and-configure-dependencies-for-the-sdk)  
* [2. Build the SDK](#2-build-the-sdk)    
* [3. Set up and run the local auth server](#3-set-up-and-run-the-local-auth-server)  
* [4. Run the sample app](#4-run-the-sample-app)  
* [Common Issues](#common-issues)  
* [Next Steps](#next-steps)

**WARNING**: This guide doesn't include instructions to enable wake word.

## 1. Install and configure dependencies for the SDK  

**IMPORTANT**: This guide assumes that all commands are run from your home directory (~/), that [Xcode](https://itunes.apple.com/us/app/xcode/id497799835?mt=12) is installed, you are running Python 2.7.x, and Homebrew is your package manager.  

To install Homebrew, run this command:  

```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```


### 1.1 Create the folder structure

Let's create a few folders to organize our files:

```
cd ~/ && mkdir sdk-folder && cd sdk-folder && mkdir sdk-build sdk-source third-party application-necessities && cd application-necessities && mkdir sound-files && cd ~/sdk-folder
```

### 1.2 Install dependencies  

The AVS Device SDK requires libraries to:

* Maintain an HTTP2 connection with AVS  
* Play Alexa TTS and music  
* Record audio from the microphone  
* Store records in a database (persistent storage)  

1. Update Homebrew. This ensures that you have access to required dependencies:

    ```
    brew update
    ```

2. Run this command to configure curl with http2, this is required to connect to AVS:  

     ```
     brew install curl --with-nghttp2
     brew link curl --force
     echo 'export PATH="/usr/local/opt/curl/bin:$PATH"' >> ~/.bash_profile
     source ~/.bash_profile
     ```

3. Now run:

     ```
     brew install gstreamer gst-plugins-base gst-plugins-good gst-libav sqlite3 repo cmake clang-format doxygen wget git
     ```  

     **IMPORTANT**: Make sure that command ran successfully, and that no errors were thrown. If for any reason the install command fails, run brew install for each dependency individually.  

4. PortAudio is required to record microphone data. Run this command to install and configure PortAudio:  

     ```
     cd ~/sdk-folder/third-party && wget -c http://www.portaudio.com/archives/pa_stable_v190600_20161030.tgz && tar xf pa_stable_v190600_20161030.tgz && cd portaudio && ./configure && make
     ```  

     **IMPORTANT**: If you see "error: cannot find 10.5 to 10.12 SDK", then run:
     ```
     ./configure --disable-mac-universal && make
     ```

5. A handful of python libraries, including pip and flask, are required to run the local auth server. Run this command to install pip if it's not already installed (will prompt for your password):   

    ```
    sudo easy_install pip
    ```

    Then run this command to install flask and requests:  

    ```
    pip install --user flask requests
    ```  

6. To support timers and alarms, we need to download sound files from the amazon.developer.portal with wget:  

    ```
    cd ~/sdk-folder/application-necessities/sound-files/ && wget -c https://images-na.ssl-images-amazon.com/images/G/01/mobile-apps/dex/alexa/alexa-voice-service/docs/audio/states/med_system_alerts_melodic_02._TTH_.mp3 && wget -c https://images-na.ssl-images-amazon.com/images/G/01/mobile-apps/dex/alexa/alexa-voice-service/docs/audio/states/med_system_alerts_melodic_02_short._TTH_.wav && wget -c https://images-na.ssl-images-amazon.com/images/G/01/mobile-apps/dex/alexa/alexa-voice-service/docs/audio/states/med_system_alerts_melodic_01._TTH_.mp3 && wget -c https://images-na.ssl-images-amazon.com/images/G/01/mobile-apps/dex/alexa/alexa-voice-service/docs/audio/states/med_system_alerts_melodic_01_short._TTH_.wav
    ```

### 1.3 Clone the AVS Device SDK

Let's clone the SDK into the sdk-source folder:  

```
cd ~/sdk-folder/sdk-source && git clone git://github.com/alexa/avs-device-sdk.git
```

## 2. Build the SDK  

1. Run cmake to generate the build dependencies. This command declares that gstreamer is enabled, and provides the path to PortAudio.

    **IMPORTANT**: Replace all instances of `{HOME}` with the absolute path to your home directory. For example: `/Users/johnsmith/`  

    ```
    cd ~/sdk-folder/sdk-build && cmake /{HOME}/sdk-folder/sdk-source/avs-device-sdk -DGSTREAMER_MEDIA_PLAYER=ON -DPORTAUDIO=ON -DPORTAUDIO_LIB_PATH=/{HOME}/sdk-folder/third-party/portaudio/lib/.libs/libportaudio.a -DPORTAUDIO_INCLUDE_DIR=/{HOME}/sdk-folder/third-party/portaudio/include
    ```

2. Here's the fun part, let's build! For this project we're only building the sample app.

    Run this command:
    ```
    make SampleApp -j2
    ```

   **Note**: Try the `-j3` or `j4` to run processes in parallel during make.

   If you want to build the full SDK, including unit and integration tests, run `make` instead of `make SampleApp`.

## 3. Obtain credentials and set up your local auth server  

In this section we are going to setup and run a local authorization server, which we'll use to obtain a refresh token. This refresh token, along with your **Client ID** and **Client Secret** are exchanged for an access token, which the sample app needs to send to Alexa with each event (request).  

### 3.1 Register your product with Amazon

Follow these [instructions](https://github.com/alexa/alexa-avs-sample-app/wiki/Create-Security-Profile) to register your product and create a security profile. You can skip this set if you have a registered product you'd like to test with. **Make sure the Allowed Origin and Return URL are `http` *not* `https`.

**IMPORTANT ENOUGH TO REPEAT**: The allowed origin and return URL under web settings should be <http://localhost:3000> and <http://localhost:3000/authresponse>, respectively.

Make sure you save the **Product ID** from the **Product information** tab, and your **Client ID** and **Client Secret** from the **Security Profile** tab. You'll need these params to configure the authorization server.  

### 3.2 Update AlexaClientSDKConfig.json

1. Use your favorite text editor to open `AlexaClientSDKConfig.json`.  

   For example:   
   * Terminal editor: `nano ~/sdk-folder/sdk-build/Integration/AlexaClientSDKConfig.json`  
   * GUI-based editor: `open ~/sdk-folder/sdk-build/Integration/AlexaClientSDKConfig.json`  

2. Copy and paste this snipped into your `AlexaClientSDKConfig.json` file:

    **IMPORTANT**: Replace all instances of `{HOME}` with the absolute path to your home directory. For example: `/Users/johnsmith/`.

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
            "databaseFilePath":"/{HOME}/sdk-folder/application-necessities/alerts.db",
            "alarmSoundFilePath":"/{HOME}/sdk-folder/application-necessities/sound-files/med_system_alerts_melodic_01._TTH_.mp3",
            "alarmShortSoundFilePath":"/{HOME}/sdk-folder/application-necessities/sound-files/med_system_alerts_melodic_01_short._TTH_.wav",
            "timerSoundFilePath":"/{HOME}/sdk-folder/application-necessities/sound-files/med_system_alerts_melodic_02._TTH_.mp3",
            "timerShortSoundFilePath":"/{HOME}/sdk-folder/application-necessities/sound-files/med_system_alerts_melodic_02_short._TTH_.wav"
       },
       "settings":{
            "databaseFilePath":"/{HOME}/sdk-folder/application-necessities/settings.db",
            "defaultAVSClientSettings":{
                "locale":"en-US"
            }
       },
       "certifiedSender":{
            "databaseFilePath":"/{HOME}/sdk-folder/application-necessities/certifiedSender.db"
       }
    }
    ```

2. Enter the `clientId`, `clientSecret`, and `productId` that you obtained during registration, then save.  

   **NOTE** - `deviceSerialNumber` is pre-populated for this project, however, a commercial product should use a serial number or other unique identified for the device.  

Locale is set to US English by default in the sample JSON, however, British English (en-GB) and German (de-DE) are supported. Feel free to test each language.

**IMPORTANT**: It is a good idea to save a backup of this file. Subsequent builds may overwrite the values in `AlexaClientSDKConfig.json`.  

### 3.3 Obtain a refresh token  

After you've updated `AlexaClientSDKConfig.json`, run `AuthServer.py` to kick-off the token exchange:  

```
cd ~/sdk-folder/sdk-build && python AuthServer/AuthServer.py
```

Then, open your browser and navigate to <http://localhost:3000>. Login with your Amazon credentials and follow the instructions provided.  

![Login Screen](https://m.media-amazon.com/images/G/01/mobile-apps/dex/avs/sdk/3.png")

**Note:** Incorrect information in `AlexaClientSDKConfig.json` is the most common reason for errors.  

## 4. Run the sample app

Now you're ready to run the sample app. This command sets the time zone to UTC and references your `AlexaClientSDKConfig.json`:

**IMPORTANT**: Replace all instances of `{HOME}` with the absolute path to your home directory. For example: `/Users/johnsmith/`:  

```
cd ~/sdk-folder/sdk-build/SampleApp/src && TZ=UTC ./SampleApp /{HOME}/sdk-folder/sdk-build/Integration/AlexaClientSDKConfig.json
```  

You can enable debugging with the debug flag. `debug1` through `debug9` are accepted values, with `debug1` providing the least and `debug9` providing the most information.  

```
TZ=UTC ./SampleApp /{HOME}/sdk-folder/sdk-build/Integration/AlexaClientSDKConfig.json debug9
```  

## Common Issues  

This is a list of common issues (or got'chas) and workarounds/resolutions. Please, let us know what we can improve by creating a [new issue](https://github.com/alexa/avs-device-sdk/issues/new).  

| Issue | Workaround/Resolution |
|--------|--------------|  
| SDK fails to build because of missing dependencies. | Sometimes, the `brew install` command that is run during **step 1.2.3** will fail if a dependency is already installed. To get around this issue, run a `brew install` command for each dependency individually. |  
| SampleApp fails to build because your version of curl doesn't support HTTP/2. | It's possible that curl didn't link correctly. To fix this issue, run `brew uninstall curl`, then repeat the steps in **1.2.2**. |  
| AuthServer.py throws this error: `File "AuthServer/AuthServer.py", line 67 print 'The file "' + \ SyntaxError: Missing parentheses in call to 'print'`. | This is a known issue with Python 3.x. The workaround is to use Python 2.7.x. |  

## Next Steps  

* [Build Options](https://github.com/alexa/avs-device-sdk/wiki/Build-Options)  
* [Unit and Integration Tests](https://github.com/alexa/avs-device-sdk/wiki/Unit-and-Integration-Tests)  
