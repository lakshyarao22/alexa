The AVS Device SDK for C++ provides a modern C++ (11 or later) interface for the Alexa Voice Service (AVS) that allows developers to add intelligent voice control to connected products. It is modular and abstracted, providing components to handle discrete functionality such as speech capture, audio processing, and communications, with each component exposing APIs that you can use and customize for your integration.  

This guide assumes familiarity with Linux and requires you to build dependencies from source. If you are looking to get up and running quickly, please use one of our quickstart guides:  

* [Ubuntu](https://github.com/alexa/avs-device-sdk/wiki/Ubuntu-Linux-Quick-Start-Guide)
* [macOS](https://github.com/alexa/avs-device-sdk/wiki/macOS-Quick-Start-Guide)
* [Raspberry Pi](https://github.com/alexa/avs-device-sdk/wiki/Raspberry-Pi-Quick-Start-Guide-with-Script)

## Register
Follow these instructions to [register your product and create a security profile](https://github.com/alexa/alexa-avs-sample-app/wiki/Create-Security-Profile). You can skip this set if you have a registered product you'd like to test with.

**IMPORTANT**: The allowed origin under web settings should be http://localhost:3000 and https://localhost:3000. The return URL under web settings should be http://localhost:3000/authresponse and https://localhost:3000/authresponse.

Make note of your **Product ID**, **Client ID**, and **Client Secret** -- you'll need these later.

## Dependencies
See [Dependencies](https://github.com/alexa/avs-device-sdk/wiki/Dependencies) for version details and installation instructions.

## AuthServer setup
This project uses `AuthServer`, a minimal authorization service built in Python using Flask. It's a quick way to obtain your first refresh token that is required by the sample app, unit and integration tests. In this section you'll install `pip`  and other `AuthServer` dependencies.

**IMPORTANT**: `AuthServer` is for testing purposes only. A commercial product is expected to obtain Login with Amazon (LWA) credentials using the instructions provided on the Amazon Developer Portal for *Remote Authorization* and *Local Authorization*. For additional information, see [AVS Authorization](https://developer.amazon.com/public/solutions/alexa/alexa-voice-service/content/avs-api-overview#authorization).

If `pip` isn't installed on your system, follow the detailed install instructions [here](https://packaging.python.org/installing/#install-pip-setuptools-and-wheel).

Then install Flask, requests, and commentjson:
```
pip install --user flask requests commentjson
```

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

## Run AuthServer  

The authorization service uses data in `AlexaClientSDKConfig.json` to obtain a refresh token, which along with your client ID, client secret, and product ID, will be exchanged with LWA for access tokens. An access token is included in the header of every request made to Alexa.  

Using your favorite editor, open `~/sdk-folder/sdk-build/Integration/AlexaClientSDKConfig.json` and replace the contents with the JSON provided below. You'll need to do two things:
* Update the config file with your **Product ID**, **Client ID**, and **Client Secret** from the Developer Console.
* Replace all instances of `{PATH TO APPLICATION NECESSITIES}`.

*Each instance of the SDK requires a unique **Device Serial Number**. This is provided by you and in some instances may match your product's SKU. For this sample, it's pre-populated with `123456`.*

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

Navigate to your *BUILD* folder, then run:
```
python AuthServer/AuthServer.py
```  

Navigate to the URL provided and follow the on-screen instructions.

**NOTE**: It's a good idea to save a backup copy of the `AlexaClientSDKConfig.json` file somewhere safe, since subsequent builds will clear its contents.

## Run tests
After you've obtained a refresh token, run integration and unit tests to ensure that the AVS Device SDK is functioning as designed.

* Use this command to run integration tests:
   ```
   make all integration
   ```
* Use this command to run unit tests:  
   ```
   make all test
   ```

For additional details, see [Unit and Integration Tests](https://github.com/alexa/avs-device-sdk/wiki/Unit-and-Integration-Tests).

## Run the Sample App
Navigate to your *BUILD* folder, then run this command:
```
./SampleApp ~/{PATH TO BUILD FOLDER}/Integration/AlexaClientSDKConfig.json
```

You can enable debugging with the debug flag. `debug1` through `debug9` are accepted values, with `debug1` providing the least and `debug9` providing the most information.  

```
./SampleApp /{PATH TO BUILD FOLDER}/Integration/AlexaClientSDKConfig.json debug9
```  

**TIP**: The Sample App is an implementation of the [`DefaultClient`](https://github.com/alexa/avs-device-sdk/blob/1b712a1e978dc3fc6b5f4d31d95e6b3741e47f2a/ApplicationUtilities/DefaultClient/src/DefaultClient.cpp) class.  

## Setup shortcuts

Now that you've built the Sample App, you can set up some shortcuts to make launching the Sample App easier:   

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
3. Now to launch the Sample App, just run this command:  
   ```
   alexac  
   ```
