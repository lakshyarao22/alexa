In this guide, we will be walking you through the steps to set up the Alexa Voice Service (AVS) Device SDK on a Raspberry Pi running [Raspbian Stretch](https://www.raspberrypi.org/downloads/raspbian/). By following these steps, you can set up your development environment, and run the AVS sample app on your Pi.

**IMPORTANT**: This guide assumes your Raspberry Pi is running Raspbian Stretch. If your Raspberry Pi is not running Raspbian Stretch, you'll need to upgrade. Please follow the instructions located [here](https://www.raspberrypi.org/blog/raspbian-stretch/) to do so.

## 1. Configure and Install Dependencies for the SDK

### 1.1 Set Up Folders

To start with, we'll create a few folders to encapsulate the SDK source folder, the build folder where the sample app and compiled libraries will be built, as well as a source folder for any third party libraries we may need.

1. Navigate to /home/pi. To do this you can open a terminal window and enter:
    `cd /home/pi`
2. Let's create a folder to encapsulate everything we will be doing in this guide. To do this, enter:
    `mkdir sdk-folder`
3. Next, navigate to that folder by entering:
    `cd /home/pi/sdk-folder`
4. Let's create a few more folders so that we can separate the SDK source code, the build artifacts that we'll be creating, and third party libraries. Enter the following:
    `mkdir sdk-build sdk-source third-party`
    
### 1.2 Set Up Necessary Dependencies

The SDK requires the use of a few additional libraries in order to do things including, but not limited to:
1. Maintaining an `HTTP2` connection required for an AVS connection
2. Playing Alexa speech and music
3. Recording microphone audio
4. Storing records into a database for persistent storage

To install the dependencies, enter the following command:

`sudo apt-get -y install git gcc cmake build-essential libsqlite3-dev libcurl4-openssl-dev libfaad-dev libsoup2.4-dev libgcrypt20-dev libgstreamer-plugins-bad1.0-dev gstreamer1.0-plugins-good libasound2-dev`

We'll also need to install PortAudio to record microphone data. Enter the following commands into a terminal to do so:  

```
cd /home/pi/sdk-folder/third-party

wget -c http://www.portaudio.com/archives/pa_stable_v190600_20161030.tgz

tar zxf pa_stable_v190600_20161030.tgz

cd portaudio

./configure --without-jack

make
```

    
### 1.2 Clone Necessary Repositories

Here, we will be cloning the SDK and the `Sensory Wake Word Engine` library from Github.

1. Navigate to `sdk-source` by entering the following:
    `cd /home/pi/sdk-folder/sdk-source`
2. Clone the AVS Device SDK from Github by entering the following:
    `git clone git://github.com/alexa/avs-device-sdk.git`
3. Navigate to `third-party` by entering the following:
    `cd /home/pi/sdk-folder/third-party`
4. Clone the `Sensory Wake Word Engine` from Github by entering the following:
    `git clone git://github.com/Sensory/alexa-rpi.git`
5. In order to license the Sensory engine for use, we'll need to run the licensing script, as follows:
    `./alexa-rpi/bin/license.sh` and accept the license agreement
    
## 2. Build the SDK

In this step, we'll be building the SDK using the dependencies we just installed.

1. Navigate to `sdk-build` by entering the following:
    `cd /home/pi/sdk-folder/sdk-build`

2. To build the SDK, first we'll need to run a `CMake` command that will generate our build dependencies. To do so, enter the following:
     ```
     cmake /home/pi/sdk-folder/sdk-source/avs-device-sdk -DSENSORY_KEY_WORD_DETECTOR=ON -DSENSORY_KEY_WORD_DETECTOR_LIB_PATH=/home/pi/sdk-folder/third-party/alexa-rpi/lib/libsnsr.a -DSENSORY_KEY_WORD_DETECTOR_INCLUDE_DIR=/home/pi/sdk-folder/third-party/alexa-rpi/include -DGSTREAMER_MEDIA_PLAYER=ON -DPORTAUDIO=ON -DPORTAUDIO_LIB_PATH=/home/pi/sdk-folder/third-party/portaudio/lib/.libs/libportaudio.a -DPORTAUDIO_INCLUDE_DIR=/home/pi/sdk-folder/third-party/portaudio/include
     ```
    
3. Next, let's build our SDK! For the purposes of this demo, we'll be building the Sample App only. Enter the following:
    `make SampleApp -j2`
    
Note that you can also try adding a `-j3` or `-j4` if you're feeling really bold, but you run the risk of overheating the Pi.

Note that if you wish to build the rest of the SDK, including unit and integration tests, you can run `make` instead of `make SampleApp`
    
## 3. Set Up Authentication
At this point, we'll need to update the AlexaClientSDKConfig.json file. This file will contain the necessary authentication credentials in order to run the sample app and run any integration tests.


### 3.1 Register your Device with Amazon
Follow these [instructions](https://github.com/alexa/alexa-avs-sample-app/wiki/Create-Security-Profile) to register your product and create a security profile. You’ll use the client ID and client secret to retrieve access and refresh tokens that are used to communicate with Alexa. Please note that the allowed origin and return URL under web settings should be <http://localhost:3000> and <http://localhost:3000/authresponse>, respectively.

After you register the device, go to the *General* tab under *Security Profile*, and make a note of the clientID, clientSecret, and deviceTypeID (also known as ProductID). You will need this information to configure the AuthServer.

### 3.2 Enter Credentials into the JSON File
Use your favorite text editor to
open and edit the **AlexaClientSDKConfig.json** file located in **/home/pi/sdk-folder/sdk-build/Integration**. Fill in the clientID, clientSecret, and deviceTypeID that you noted during device registration.

```json
 {
    "authDelegate":{
        "clientSecret":"",
        "deviceSerialNumber":"",
        "refreshToken":"",
        "clientId":"",
        "deviceTypeId":""
     }
 }
```

Make sure that you don’t have any extra characters (or spaces) in the paths. Note that for the purposes of this demo, the `deviceSerialNumber` isn't strictly required. However, when multiple devices are using the same set of credentials, this value is used to uniquely identify each device within the same device family.


### 3.3 Update Refresh Token
When the build is complete, you need to get a refresh token for the device. Just run AuthServer, a local server implementation that will handle the token exchange:

```
cd /home/pi/sdk-folder/sdk-build
python AuthServer/AuthServer.py
```

**Note:** If you run into errors here, which will pop up on your browser, click for more details to ensure that you’re able to go back and correct them.

**Note:** You may need to change the "locale" settings of the Pi, as some Raspberry Pis by default go directly to the `.co.uk` versions of Amazon instead of the `.com` version.

From your Pi, open the browser and navigate to <http://localhost:3000>. Log in with your Amazon developer credentials and close the window when instructed.

![Login Screen](https://m.media-amazon.com/images/G/01/mobile-apps/dex/avs/sdk/3.png")

**Note:** If there are any errors, carefully review them as they will make it clear what went wrong. Incorrect device information in the JSON file is the most common reason for problems. If you want, open the JSON file and verify that there is a new line with your refresh token.

### 3.4 Set Up Other Sections of Config

The JSON config file also requires several other sections to be filled out prior to Alexa being able to fully function. See below:

```json
 {
   "alertsCapabilityAgent":{
        "databaseFilePath":"",
        "alarmSoundFilePath":"",
        "alarmShortSoundFilePath":"",
        "timerSoundFilePath":"",
        "timerShortSoundFilePath":""
   },

   "settings":{
        "databaseFilePath":"",
        "defaultAVSClientSettings":{
            "locale":""
          }
    },

   "certifiedSender":{ 
        "databaseFilePath":""
    }
 }
```


One such requirement are the paths to sound files that will be used to play sounds for Alarms and Timers. You can get the required sound files from the Timer and Alarms section at <https://developer.amazon.com/public/solutions/alexa/alexa-voice-service/content/alexa-voice-service-ux-design-guidelines>

For the purposes of the demo, we will be downloading them directly using the following commands:
```
cd /home/pi/sdk-folder
mkdir application-necessities
cd application-necessities
mkdir sound-files
cd sound-files
wget -c https://images-na.ssl-images-amazon.com/images/G/01/mobile-apps/dex/alexa/alexa-voice-service/docs/audio/states/med_system_alerts_melodic_02._TTH_.mp3
wget -c https://images-na.ssl-images-amazon.com/images/G/01/mobile-apps/dex/alexa/alexa-voice-service/docs/audio/states/med_system_alerts_melodic_02_short._TTH_.wav
wget -c https://images-na.ssl-images-amazon.com/images/G/01/mobile-apps/dex/alexa/alexa-voice-service/docs/audio/states/med_system_alerts_melodic_01._TTH_.mp3
wget https://images-na.ssl-images-amazon.com/images/G/01/mobile-apps/dex/alexa/alexa-voice-service/docs/audio/states/med_system_alerts_melodic_01_short._TTH_.wav
```

We'll need to point the SDK to these audio files. So, in the config file, enter the following:

1. For the `alarmSoundFilePath` section under `alertsCapabilityAgent` , enter the following: 
    `/home/pi/sdk-folder/application-necessities/sound-files/med_system_alerts_melodic_01._TTH_.mp3`
2. For the `alarmShortSoundFilePath` section under `alertsCapabilityAgent` , enter the following: 
    `/home/pi/sdk-folder/application-necessities/sound-files/med_system_alerts_melodic_01_short._TTH_.wav`
3. For the `timerSoundFilePath` section under `alertsCapabilityAgent` , enter the following: 
    `/home/pi/sdk-folder/application-necessities/sound-files/med_system_alerts_melodic_02._TTH_.mp3`
4. For the `timerShortSoundFilePath` section under `alertsCapabilityAgent` , enter the following:
    `/home/pi/sdk-folder/application-necessities/sound-files/med_system_alerts_melodic_02_short._TTH_.wav`


A database is required to read and store records that the SDK requires. In your config file, include the file path location to the database you wish to use to store and read alerts from. A database file will be created at that location if the database does not already exist. For example "/home/anExistingFolder/anotherExistingFolder/alerts.db". The below steps outline a sample for the purposes of this demo:

In the config file, enter the following:

1. For the `databaseFilePath` section under `alertsCapabilityAgent` , enter the following: 
    `/home/pi/sdk-folder/application-necessities/alerts.db`
2. For the `databaseFilePath` section under `settings`, enter the following: 
    `/home/pi/sdk-folder/application-necessities/settings.db`
3. For the `databaseFilePath` section under `certifiedSender`, enter the following: 
    `/home/pi/sdk-folder/application-necessities/certifiedSender.db`


Lastly, we'll need to set the locale for Alexa to start. To do so, do the following:

1. Under the `locale` section under `settings`, enter the following: `en-US`. This sets Alexa to run in American English.

Note that the following are also acceptable: `en-GB` for British English, and `de-DE` for German :)

### 3.5 Check the Microphone
We'll need to make sure the microphone is set up properly. To test this out, run the following commands:

```
sudo apt-get install sox

rec test.wav
```

If everything went well, you should see some message indicating that audio is being recorded. To exit out, hit `Control+C`. If this step went as expected, then great - your microphone is all set up!

If not, we'll need to modify the `~/.asoundrc` file. To do this, run the following commands:
```
sudo apt-get -y install gedit

gedit ~/.asoundrc
```

Enter the following when editing the file:
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

Our microphone should now be ready for use after this.



## Run the Sample App
At this point, your development environment using the AVS Device SDK is fully set up and you are ready to test the AVS sample app.

Before you run the Sample App, make sure the following items are in order:
* The Alerts Capability Agent will need to be calibrated to the Coordinated Universal Time Zone (UTC).
* The path to the JSON file we filled out, which contains all the information to get Alexa started.
* The path to the wake word model file, used to wake up Alexa whenever the wake word is heard.

The following command makes sure all that is done:

```
cd /home/pi/sdk-folder/sdk-build/SampleApp/src

TZ=UTC ./SampleApp /home/pi/sdk-folder/sdk-build/Integration/AlexaClientSDKConfig.json /home/pi/sdk-folder/third-party/alexa-rpi/models
```

