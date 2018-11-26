This guide provides step-by-step instructions to set up the Alexa Voice Service (AVS) Device SDK on a Raspberry Pi running [Raspbian Stretch with Desktop](https://www.raspberrypi.org/downloads/raspbian/). When finished, you'll have a working sample app to test interactions with Alexa.  

**IMPORTANT**: If your Raspberry Pi is not running Raspbian Stretch With Desktop use [these instructions to upgrade](https://www.raspberrypi.org/blog/raspbian-stretch/). If you choose to build with Raspbian Jessie, you need to build certain dependencies from source (see commit [a5646fc](https://github.com/alexa/avs-device-sdk/wiki/Raspberry-Pi-Quick-Start-Guide/a5646fc9e6dde8128c940b86a9bbece7f65c1ace) for instructions).  

## Register a product

Follow these [instructions](https://github.com/alexa/avs-device-sdk/wiki/Create-Security-Profile) to register your product and create a security profile. You can skip this step if you have a registered product you'd like to test with.

## Setup your environment

Let's create a few folders to organize our files. This guide presumes that everything is built in `/home/pi`, which we will presume is your home directory. If you choose to use different folder names, please update the commands throughout this guide accordingly.

Open terminal and run this command from your home directory (~/):

```sh
cd /home/pi/
mkdir sdk-folder

cd sdk-folder
mkdir sdk-build sdk-source third-party application-necessities

cd application-necessities
mkdir sound-files
```

### Install dependencies

The AVS Device SDK requires libraries to:  

1. Maintain an `HTTP2` connection with AVS  
2. Play Alexa TTS and music  
3. Record audio from the microphone  
4. Store records in a database (persistent storage)  

The first step is to update `apt-get`. This ensures that you have access to required dependencies.

```sh
sudo apt-get update
```

Then run:

```sh
sudo apt-get -y
install git gcc

cmake build-essential libsqlite3-dev libcurl4-openssl-dev libfaad-dev \
libsoup2.4-dev libgcrypt20-dev libgstreamer-plugins-bad1.0-dev \
gstreamer1.0-plugins-good libasound2-dev doxygen
```

PortAudio is required to record microphone data. Run this command to install and configure PortAudio:  

```sh
cd /home/pi/sdk-folder/third-party

wget -c http://www.portaudio.com/archives/pa_stable_v190600_20161030.tgz
tar zxf pa_stable_v190600_20161030.tgz

cd portaudio
./configure --without-jack

make
```  

commentjson is required to parse comments in `AlexaClientSDKConfig.json`. Run this command to install commentjson:

```sh
pip install commentjson
```

### Clone the AVS Device SDK and the Sensory wake word engine

1. Let's clone the SDK into the sdk-source folder:
```sh
cd /home/pi/sdk-folder/sdk-source
git clone git://github.com/alexa/avs-device-sdk.git
```
2. Next, let's clone the sensory wake word engine into our third-party directory:   
```sh
cd /home/pi/sdk-folder/third-party
git clone git://github.com/Sensory/alexa-rpi.git
```
3. Now let's run the licensing script and accept the licensing agreement. This is required to use Sensory's wake word engine:
```sh
cd /home/pi/sdk-folder/third-party/alexa-rpi/bin/
./license.sh
```

## Build the SDK

1. Run cmake to generate the build dependencies. This command declares that the wake word engine and gstreamer are enabled, and provides paths to the wake word engine and PortAudio:
```sh
cd /home/pi/sdk-folder/sdk-build
cmake /home/pi/sdk-folder/sdk-source/avs-device-sdk \
-DSENSORY_KEY_WORD_DETECTOR=ON \
-DSENSORY_KEY_WORD_DETECTOR_LIB_PATH=/home/pi/sdk-folder/third-party/alexa-rpi/lib/libsnsr.a \
-DSENSORY_KEY_WORD_DETECTOR_INCLUDE_DIR=/home/pi/sdk-folder/third-party/alexa-rpi/include \
-DGSTREAMER_MEDIA_PLAYER=ON \
-DPORTAUDIO=ON \
-DPORTAUDIO_LIB_PATH=/home/pi/sdk-folder/third-party/portaudio/lib/.libs/libportaudio.a \
-DPORTAUDIO_INCLUDE_DIR=/home/pi/sdk-folder/third-party/portaudio/include
```  

   **NOTE**: This is sample CMake command configured for the sample app. There are other CMake options available for use outside of this project, here's the full list of [Build Options](https://github.com/alexa/avs-device-sdk/wiki/Build-Options).  

2. Here's the fun part, let's build! For this project we're only building the sample app. Run this command:
```ch
make SampleApp -j2
```
   **NOTE**: The `j2` flag allows you to run 2 proccesses in parallel, which should speed up the build. Try the `-j3` or `-j4` option if you're feeling bold, but you run the risk of overheating the Pi (maybe even melting it).

If you want to build the rest of the SDK, including unit and integration tests, run `make` instead of `make SampleApp`.

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
    * **{{device serial number}}**: You can provide your own device serial number (DSN), or use the system default 
    (`123456`). The DSN can be any unique alpha-numeric string (up to 64 characters). You should use this string 
    to identify your product or application instance. Many developers choose to use a product's SKU for this 
    value. If you don't supply a DSN, then the default value `123456` will be generated by the SDK.

        For example:

        ```sh
        sudo bash setup.sh config.json -s 998987
        ```
    * **{{path to database}}**: Specifies where the databases will be located.
    * **{{path to source folder}}**: Specifies the root directory where the avs-device-sdk source code is located.
    * **{{path to build}}**: Specifies where the build folder is, which contains the **Integration/AlexaClientSDKConfig.json** file.

**IMPORTANT**: Create a backup of your `AlexaClientSDKConfig.json` file. Subsequent builds will reset the contents of this file.

## Test the microphone  

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

```sh
sudo apt-get install sox -y && rec test.wav
```

If everything works, you should see a message indicating that audio is recording. To exit, hit **Control + C**.

## Run and authorize  
Navigate to your *BUILD* folder, then:
1. Start the sample app:
```sh
./SampleApp/src/SampleApp ./Integration/AlexaClientSDKConfig.json
```
2. Wait for the sample app to display a message like this:
```sh
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
7. Wait for the sample app to report that it is authorized, and that Alexa is idle.  It will look something like this:
```sh
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
* If you want to move this authorization to another sample app installation, you need to copy the **deviceInfo** object within `AlexaClientSDKConfig.json` to the new installation. You will also need to copy the file `"/home/pi/sdk-folder/application-necessities/cblAuthDelegate.db"` to the new installation, and update **AlexaClientSDKConfig.json** in the new installation so that the **cblAuthDelegate's databaseFilePath** property points to it.

## Setup shortcuts

Run this command to launch the sample app. This includes the path to your configuration file and the Sensory wake word model:

```sh
cd /home/pi/sdk-folder/sdk-build/SampleApp/src
./SampleApp/src/SampleApp ./Integration/AlexaClientSDKConfig.json ../third-party/alexa-rpi/models
```  

## Enabling debug logs
**IMPORTANT**: If you encounter any issues, use the debug option for additional information. `debug1` through `debug9` are accepted values. For example:  

```sh
./SampleApp /home/pi/sdk-folder/sdk-build/Integration/AlexaClientSDKConfig.json /home/pi/sdk-folder/third-party/alexa-rpi/models debug9
```  

## Bluetooth

Building with Bluetooth is optional, and is currently limited to Linux and Raspberry Pi. This release supports `A2DP-SINK` and `AVRCP` profiles. In order to use Bluetooth for these platforms, you must install all Bluetooth [dependencies]() and **disable any processes which obtain an incoming Bluetooth audio stream**, such as:

### BlueALSA

1. If you are using `BlueALSA`, you must disable Bluetooth by running this command:
    ```sh
    ps aux | grep bluealsa
    sudo kill <bluealsa pid>
    ```

### PulseAudio
If you are using `PulseAudio`, you **must** disable `PulseAudio` Bluetooth plugins. To do this:

1. Navigate to `/etc/pulse/default.pa` (or equivalent file), and comment out the following lines:
    ```sh
    ### Automatically load driver modules for Bluetooth hardware
    #.ifexists module-bluetooth-policy.so
    #load-module module-bluetooth-policy
    #.endif

    #.ifexists module-bluetooth-discover.so
    #load-module module-bluetooth-discover
    #.endif
    ```

2. Next, stop and restart PulseAudio with these commands (if auto-respawn is disabled):
    ```sh
    pulseaudio --kill
    pulseaudio --start
    ```

## Next Steps  

* [Build Options](https://github.com/alexa/avs-device-sdk/wiki/Build-Options)  
* [Unit and Integration Tests](https://github.com/alexa/avs-device-sdk/wiki/Unit-and-Integration-Tests)  
