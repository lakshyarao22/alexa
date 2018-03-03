
This guide provides step-by-step instructions to set up the Alexa Voice Service (AVS) Device SDK on macOS. When finished, you'll have a working sample app to test interactions with Alexa.   

**Table of Contents**
* [1. Install and configure dependencies for the SDK](#1-install-and-configure-dependencies-for-the-sdk)  
* [2. Build the SDK](#2-build-the-sdk)    
* [3. Obtain credentials and set up your local auth server](#3-obtain-credentials-and-set-up-your-local-auth-server)  
* [4. Run the sample app](#4-run-the-sample-app)  
* [5. Setup shortcuts](#5-setup-shortcuts)
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
     brew install gstreamer gst-plugins-base gst-plugins-good gst-plugins-bad gst-libav sqlite3 repo cmake clang-format doxygen wget git
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

    Then run this command to install flask, requests, and commentjson:  

    ```
    pip install --user flask requests commentjson
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

Follow these [instructions](https://github.com/alexa/alexa-avs-sample-app/wiki/Create-Security-Profile) to register your product and create a security profile. You can skip this step if you have a registered product you'd like to test with.

**IMPORTANT**: The allowed origin under web settings should be http://localhost:3000 and https://localhost:3000. The return URL under web settings should be http://localhost:3000/authresponse and https://localhost:3000/authresponse.

Make sure you save the **Product ID** from the **Product information** tab, and your **Client ID** and **Client Secret** from the **Security Profile** tab. You'll need these params to configure the authorization server.  

### 3.2 Update AlexaClientSDKConfig.json

Use your favorite text editor to open `AlexaClientSDKConfig.json`.  

   For example:   
   * Terminal editor: `nano ~/sdk-folder/sdk-build/Integration/AlexaClientSDKConfig.json`  
   * GUI-based editor: `open ~/sdk-folder/sdk-build/Integration/AlexaClientSDKConfig.json`  

Now fill in your product-specific values and save. Alternatively, you can use the template provided below, which includes paths to the database files required for the sample app.  

If you choose to use the template, follow these instructions:  

1. Replace the contents of `AlexaClientSDKConfig.json` with this JSON blob:

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

2. Enter the `clientId`, `clientSecret`, and `productId` that you saved during device registration and save.

    **NOTE**: Do not remove the quotes and make sure there are no extra characters or spaces! The required values are strings.  

    **NOTE 2**: `deviceSerialNumber` is pre-populated for this project, however, a commercial product should use a serial number or other unique identified for the device.  

The locale is set to US English by default in the sample JSON, however other [locales are supported](https://developer.amazon.com/docs/alexa-voice-service/settings.html#settingsupdated). Feel free to test each language.

**IMPORTANT**: It is a good idea to save a backup of this file. Subsequent builds may overwrite the values in `AlexaClientSDKConfig.json`.  

### 3.3 Obtain a refresh token  

After you've updated `AlexaClientSDKConfig.json`, run `AuthServer.py` to kick-off the token exchange:  

```
cd ~/sdk-folder/sdk-build && python AuthServer/AuthServer.py
```

Then, open your browser and navigate to <http://localhost:3000>. Login with your Amazon credentials and follow the instructions provided.  

![Login Screen](https://m.media-amazon.com/images/G/01/mobile-apps/dex/avs/sdk/3.png")

#### Common issues  

These are the most common issues encountered when trying to obtain a refresh token:

* Incorrect information in `AlexaClientSDKConfig.json`.

## 4. Run the sample app

Now you're ready to run the sample app. This command sets the time zone to UTC and references your `AlexaClientSDKConfig.json`:

**IMPORTANT**: Replace all instances of `{HOME}` with the absolute path to your home directory. For example: `/Users/johnsmith/`:  

```
cd ~/sdk-folder/sdk-build/SampleApp/src
./SampleApp /{HOME}/sdk-folder/sdk-build/Integration/AlexaClientSDKConfig.json
```  

You can enable debugging with the debug flag. `debug1` through `debug9` are accepted values, with `debug1` providing the least and `debug9` providing the most information.  

```
./SampleApp /{HOME}/sdk-folder/sdk-build/Integration/AlexaClientSDKConfig.json debug9
```  

## 5. Setup shortcuts

Now that you've built the sample app, let's setup some shortcuts so that you don't have to remember the full command to launch the app.  

1. Use your favorite text editor to open `~/.bash_profile`. Then add these aliases and **save**:  
   **IMPORTANT**: Make sure you update the paths to match your folder structure.  
   ```
   alias alexac="[path_to_build_folder]/SampleApp/src/SampleApp [path_to_build_folder]/Integration/AlexaClientSDKConfig.json"
   alias alexacdebug="[path_to_build_folder]/SampleApp/src/SampleApp [path_to_build_folder]/Integration/AlexaClientSDKConfig.json DEBUG9"
   ```
2. After you've added these aliases, make sure to activate your `~/.bash_profile`:  
   ```
   source ~/.bash_profile
   ```
3. Now to launch the sample app just run this command:  
   ```
   alexac  
   ```

## Common issues  

See the [Troubleshooting Guide](https://github.com/alexa/avs-device-sdk/wiki/Troubleshooting-Guide).

## Next steps  

* [Build Options](https://github.com/alexa/avs-device-sdk/wiki/Build-Options)  
* [Unit and Integration Tests](https://github.com/alexa/avs-device-sdk/wiki/Unit-and-Integration-Tests)  
