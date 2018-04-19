
This guide provides step-by-step instructions to set up the Alexa Voice Service (AVS) Device SDK on macOS. When finished, you'll have a working sample app to test interactions with Alexa.   

**WARNING**: This guide doesn't include instructions to enable wake word.

## Prerequisites  
This guide assumes that:

* All commands are run from your home directory (~/)
* You are running Python 2.7.x  
* [Xcode](https://itunes.apple.com/us/app/xcode/id497799835?mt=12) is installed
* Homebrew is your package manager  

  To install Homebrew, run this command:  

```
  /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

## Register a product  
Follow these instructions to [register your product and create a security profile](https://github.com/alexa/avs-device-sdk/wiki/Create-Security-Profile).

Make sure you save the **Product ID** from the **Product information** tab, and your **Client ID** from the **Other devices and platforms** tab from within the **Security Profile** section.

Note: If you already have a registered product that you can use for testing, you may use it but it must be enabled for use with Code Based Linking (CBL). You can find steps for enabling CBL for your device [Here](https://developer.amazon.com/docs/alexa-voice-service/code-based-linking-other-platforms.html#step1) (Step 1).

**IMPORTANT**: When you capture the **Client ID**, make sure it is from the **Other devices and platforms** tab within the **Security Profile** section, and **NOT** from the **Client ID** from the top of the **Product information**, **Security Profile**, or **Capabilities** tabs. The **Client ID** generated within your **Security Profile** is the required **Client ID**.

## Setup

Let's create a few folders to organize our files:

```
cd ~/ && mkdir sdk-folder && cd sdk-folder && mkdir sdk-build sdk-source third-party application-necessities && cd application-necessities && mkdir sound-files && cd ~/sdk-folder
```

### Install dependencies  

The AVS Device SDK requires libraries to:

* Maintain an HTTP2 connection with AVS  
* Play Alexa Text-to-Speech (TTS) and music  
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

### Clone the AVS Device SDK

Let's clone the SDK into the sdk-source folder:  

```
cd ~/sdk-folder/sdk-source && git clone git://github.com/alexa/avs-device-sdk.git
```

## Build the SDK  

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

## Setup your configuration

The sample app uses data in `AlexaClientSDKConfig.json` to obtain a refresh token which, along with your **Client ID** and **Product ID**, will be exchanged with LWA for access tokens. An access token is included in the header of every request made to Alexa.  

Using your favorite editor, open `~/sdk-folder/sdk-build/Integration/AlexaClientSDKConfig.json` and replace the contents with the JSON provided below. You'll need to do two things:
* Update the **deviceInfo** object in the config file with your **Product ID** and **Client ID** from the Developer Console.
* Replace all instances of `"databaseFilePath":"/{HOME}/sdk-folder/application-necessities/{{dataBase}}.db"` with the absolute path to that file.

Note: Each instance of the SDK requires a unique **Device Serial Number** (also found in the **deviceInfo** object). This is provided by you and in some instances may match your product's SKU. For this sample, it's pre-populated with `123456`.

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

Navigate to your *BUILD* folder, then:
1. Start the sample app:
```
./SampleApp/src/SampleApp ./Integration/AlexaClientSDKConfig.json
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
7. Wait (it may take as few seconds) for the sample app to report that it is authorized, and that Alexa is idle.  It will look something like this:
```
###########################
#       Authorized!       #
###########################
########################################
#       Alexa is currently idle!       #
########################################
```
8. You are now ready to use the sample app. The next time you start the sample app, you will not need to go through the authorization process.

A couple more details:
* If you exit out of sample app via the `k` command, the `CBLAuthDelegate` database will be cleared and you will need to reauthorize your client.
* If you want to move this authorization to another sample app installation, you need to copy the **deviceInfo** object within `AlexaClientSDKConfig.json` to the new installation. You will also need to copy the file `"/{HOME}/sdk-folder/application-necessities/cblAuthDelegate.db"` to the new installation, and update **AlexaClientSDKConfig.json** in the new installation so that the **cblAuthDelegate's databaseFilePath** property points to it.

## Enabling debug logs

Debug logs are available in DEBUG builds. You can enable them by adding the following to the end of your cmake command line:
```
cmake < other cmake parameters > -DCMAKE_BUILD_TYPE=DEBUG
```
An additional sample app command line parameter allows you to select the level of debug logging to be output. The allowed values are DEBUG0, DEBUG1, ... DEBUG9, where DEBUG9 provides the most information. For example:
```
./SampleApp/src/SampleApp ./Integration/Integration/AlexaClientSDKConfig.json DEBUG9
```  

## Setup shortcuts

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
