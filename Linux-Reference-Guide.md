The AVS Device SDK for C++ provides a modern C++ (11 or later) interface for the Alexa Voice Service (AVS) that allows developers to add intelligent voice control to connected products. It is modular and abstracted, providing components to handle discrete functionality such as speech capture, audio processing, and communications, with each component exposing APIs that you can use and customize for your integration.  

This guide assumes familiarity with Linux and requires you to build dependencies from source. If you are looking to get up and running quickly, please use one of our quick start guides:  

* [Ubuntu](https://github.com/alexa/avs-device-sdk/wiki/Ubuntu-Linux-Quick-Start-Guide)
* [macOS](https://github.com/alexa/avs-device-sdk/wiki/macOS-Quick-Start-Guide)
* [Raspberry Pi](https://github.com/alexa/avs-device-sdk/wiki/Raspberry-Pi-Quick-Start-Guide-with-Script)

## Register
Follow these instructions to [register your product and create a security profile](https://github.com/alexa/avs-device-sdk/wiki/Create-Security-Profile).

Make sure you save the **Product ID** from the **Product information** tab, and your **Client ID** from the **Other devices and platforms** tab from within the **Security Profile** section.

Note: If you already have a registered product that you can use for testing, you may use it but it must be enabled for use with Code Based Linking (CBL). You can find steps for enabling CBL for your device [Here](https://developer.amazon.com/docs/alexa-voice-service/code-based-linking-other-platforms.html#step1) (Step 1).

**IMPORTANT**: When you capture the **Client ID**, make sure it is from the **Other devices and platforms** tab within the **Security Profile** section, and **NOT** from the **Client ID** from the top of the **Product information**, **Security Profile**, or **Capabilities** tabs. The **Client ID** generated within your **Security Profile** is the required **Client ID**.

## Dependencies
See [Dependencies](https://github.com/alexa/avs-device-sdk/wiki/Dependencies) for version details and installation instructions.

## Create an out-of-source build
The AVS Device SDK must be built out-of-source. Creating sub-folders within your project for *BUILD*, *SOURCE*, *THIRD-PARTY*, and *APPLICATION NECESSITIES* is recommended.

### Clone the AVS Device SDK  
Clone the AVS Device SDK into your *SOURCE* folder:  
```
git clone https://github.com/alexa/avs-device-sdk.git
```

### Build the AVS Device SDK   
Navigate to your *BUILD* folder. This sample command does a few things:  
* It declares that PortAudio is used to capture microphone data and points to its lib path and includes directory
   * It declares that gstreamer is installed and will be used when you build the SampleApp
   * It declares that the wake word detector is **OFF**  

*Linux supports wake word detectors from Sensory and Kitt.ai. Each requires a license from the provider. For instructions to build with a wake word detector, please see [Build Options](https://github.com/alexa/avs-device-sdk/wiki/Build-Options).*  

```
cmake /{PATH TO SOURCE FOLDER}/avs-device-sdk -DSENSORY_KEY_WORD_DETECTOR=OFF -DGSTREAMER_MEDIA_PLAYER=ON -DPORTAUDIO=ON -DPORTAUDIO_LIB_PATH=/{PATH TO PORTAUDIO}/lib/.libs/libportaudio.a -DPORTAUDIO_INCLUDE_DIR=/{PATH TO PORTAUDIO}/include
```

Then run:
```
make
```

**IMPORTANT**: See [Build Options](https://github.com/alexa/avs-device-sdk/wiki/Build-Options) for a additional configuration options.

## Setup your configuration

The sample app uses data in `AlexaClientSDKConfig.json` to obtain a refresh token which, along with your **Client ID** and **Product ID**, will be exchanged with LWA for access tokens. An access token is included in the header of every request made to Alexa.  

Using your favorite editor, open `{PATH TO BUILD FOLDER}/Integration/AlexaClientSDKConfig.json` and replace the contents with the JSON provided below. You'll need to do two things:
* Update the **deviceInfo** object in the config file with your **Product ID** and **Client ID** from the Developer Console.
* Replace all instances of `"databaseFilePath":"/{HOME}/sdk-folder/application-necessities/{{dataBase}}.db"` with the absolute path to that file.

Note: Each instance of the SDK requires a unique **Device Serial Number** (also found in the **deviceInfo** object). This is provided by you, and in some instances may match your product's SKU. For this sample, it's pre-populated with `123456`.

```
{
  "deviceInfo":{
    "deviceSerialNumber":"123456",
    "clientId":"YOUR_CLIENT_ID",
    "productId":"YOUR_PRODUCT_ID"
  },
  "cblAuthDelegate":{
      "databaseFilePath":"/{PATH TO APPLICATION NECESSITIES}/cblAuthDelegate.db"
  },
  "miscDatabase":{
      "databaseFilePath":"/{PATH TO APPLICATION NECESSITIES}/miscDatabase.db"
  },
  "alertsCapabilityAgent":{
      "databaseFilePath":"/{PATH TO APPLICATION NECESSITIES}/alerts.db"
  },
  "settings":{
      "databaseFilePath":"/{PATH TO APPLICATION NECESSITIES}/settings.db",
      "defaultAVSClientSettings":{
          "locale":"en-US"
      }
  },
  "certifiedSender":{
     "databaseFilePath":"/{PATH TO APPLICATION NECESSITIES}/certifiedSender.db"
  },
  "notifications":{
      "databaseFilePath":"/{PATH TO APPLICATION NECESSITIES}/notifications.db"
  }
}
```

**IMPORTANT:** Save a backup copy of your edited `AlexaClientSDKConfig.json`. Subsequent builds will reset the contents of this file.

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
* If you want to move this authorization to another sample app installation, you need to copy the **deviceInfo** object within `AlexaClientSDKConfig.json` to the new installation.  You will also need to copy the file `"/{PATH TO APPLICATION NECESSITIES}/cblAuthDelegate.db"` to the new installation, and update **AlexaClientSDKConfig.json** in the new installation so that the **cblAuthDelegate's databaseFilePath** property points to it.

## Enabling debug logs

Debug logs are available in DEBUG builds.  You can enable them by adding the following to the end of your cmake command line:
```
cmake < other cmake parameters > -DCMAKE_BUILD_TYPE=DEBUG
```
An additional sample app command line parameter allows you to select the level of debug logging to be output.  The allowed values are DEBUG0, DEBUG1, ... DEBUG9, where DEBUG9 provides the most information.  For example:
```
./SampleApp/src/SampleApp ./Integration/Integration/AlexaClientSDKConfig.json DEBUG9
```  

## Run tests
After you've authorized your sample app, run integration and unit tests to ensure that the AVS Device SDK is functioning as designed.

* Use this command to run integration tests:
   ```
   make all integration
   ```
* Use this command to run unit tests:  
   ```
   make all test
   ```

For additional details, see [Unit and Integration Tests](https://github.com/alexa/avs-device-sdk/wiki/Unit-and-Integration-Tests).

## Setup shortcuts

Now that you've built the sample app, you can set up some shortcuts to make launching the sample app easier:   

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
3. Now to launch the sample app, just run this command:  
   ```
   alexac  
   ```

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