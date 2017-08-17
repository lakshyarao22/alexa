## Get Started with the AVS Device SDK on Raspberry Pi

In this wiki, we'll walk through the steps to set up the AVS Device SDK and get the C++ sample app running on a Raspberry Pi 3. Before you get started, we recommend watching [Getting Started](https://youtu.be/F5DixCPJYo8) video to establish a working knowledge of how the AVS Device SDK works.

Here's what you need to do:

1. Register your device with Amazon.
2. Install and configure dependencies for AVS Device SDK on your Raspberry Pi.
3. Build the SDK and run the sample app on your Raspberry Pi.


### 1. Register Your Device with Amazon

Follow these [instructions](https://github.com/alexa/alexa-avs-sample-app/wiki/Create-Security-Profile) to register your product and create a security profile. You’ll use the client ID and client secret to retrieve access and refresh tokens that are used to communicate with Alexa. Please note that the allowed origin and return URL under web settings should be <http://localhost:3000> and <http://localhost:3000/authresponse>, respectively.

After you register the device, go to the *General* tab under *Security Profile*, and make a note of the clientID, clientSecret, and deviceTypeID. You will need this information to configure the AuthServer.



### 2. Install and Configure Dependencies for AVS Device SDK on Your Raspberry Pi

Use these instructions to setup your development environment on a Raspberry Pi 3 running Raspbian Jessie.

These instructions guide you through building the minimum required version for each dependency. If you plan to use a newer version, please validate each command, specifically configuration steps.  

**TIP**: All tests were performed on Raspbian Jessie Lite if you prefer not to install a GUI distribution.  

#### 2.1 Create a source folder for your dependencies

To get your Raspberry Pi ready to consume the AVS Device SDK, you'll need to build some dependencies from source. It's a best practice to keep source files in a pre-defined folder. Use the command to create your source folder. Feel free to modify the  `SOURCE_FOLDER` value. Keep in mind that this variable will be referenced throughout this document:

```bash
echo "export SOURCE_FOLDER=$HOME/sources" >> $HOME/.bash_aliases
echo "export LOCAL_BUILD=$HOME/local-builds" >> $HOME/.bash_aliases
echo "export LD_LIBRARY_PATH=$HOME/local-builds/lib:$LD_LIBRARY_PATH" >> $HOME/.bash_aliases
echo "export PATH=$HOME/local-builds/bin:$PATH" >> $HOME/.bash_aliases
echo "export PKG_CONFIG_PATH=$HOME/local-builds/lib/pkgconfig:$PKG_CONFIG_PATH" >> $HOME/.bash_aliases
source $HOME/.bashrc
mkdir $SOURCE_FOLDER
```

#### 2.2 Download build tools

The next step is to download some essential build tools:  

```bash
sudo apt-get install git gcc cmake build-essential
```

#### 2.3 Set up `libcurl` with `nghttp2` and `openssl`

This is the most important part of the setup, since a connection to AVS requires the use of HTTP2, and the SDK uses `libcurl` to establish that connection.

##### 2.3.1: Build `nghttp2` from source

**IMPORTANT**: The `nghttp2` development package provided in Jessie is out-of-date, so you will need to build the correct version from source. The first step is to get the source:

```bash
cd $SOURCE_FOLDER
wget https://github.com/nghttp2/nghttp2/releases/download/v1.0.0/nghttp2-1.0.0.tar.gz
tar xzf nghttp2-1.0.0.tar.gz
```

Then configure, build, and install with these commands:

```bash
cd $SOURCE_FOLDER/*nghttp2*/
./configure --prefix=$LOCAL_BUILD --disable-app
make
sudo make install
```

##### 2.3.2: Build `openssl` from source

**IMPORTANT** The `openssl` package included with Jessie is out-of-date, so you will need to build the correct version from source.

Run these commands to download and build `openssl`:

```bash
cd $SOURCE_FOLDER
wget https://www.openssl.org/source/old/1.0.2/openssl-1.0.2a.tar.gz
tar xzf openssl-1.0.2a.tar.gz
cd *openssl*/
./config --prefix=$LOCAL_BUILD --openssldir=$LOCAL_BUILD shared
make
sudo make install
```

##### 2.3.3: Build `libcurl` from source

Now that `nghttp2` and `openssl` are built from source, we can build cURL.

Run these commands:

```bash
cd $SOURCE_FOLDER
wget https://curl.haxx.se/download/curl-7.50.2.tar.gz
tar xzf curl-7.50.2.tar.gz
cd *curl*/
./configure --with-ssl=$LOCAL_BUILD --with-nghttp2=$LOCAL_BUILD --prefix=$LOCAL_BUILD
make
sudo make install
```

#### 2.4: Install `sqlite`  

SQLite is required for Alerts. To install, run this command:

```
sudo apt-get install libsqlite3-dev
```


The dependencies that follow are required to use the included `MediaPlayer` implementation and to build the sample app.


#### 2.5: Build `gstreamer` libraries

The sample app requires a media player implementation to play MP3 files; our implementation supports GStreamer. Before building GStreamer, you need to install some utilities and dependencies.

Run this command:

```
sudo apt-get install bison flex libglib2.0-dev libasound2-dev pulseaudio libpulse-dev
```

##### 2.5.1: Build `gstreamer-1.10.4`

```bash
cd $SOURCE_FOLDER
wget https://gstreamer.freedesktop.org/src/gstreamer/gstreamer-1.10.4.tar.xz
tar xf gstreamer-1.10.4.tar.xz
cd *gstreamer*/
./configure --prefix=$LOCAL_BUILD
make
sudo make install
```

##### 2.5.2: Build `gst-plugins-base-1.10.4`

```bash
cd $SOURCE_FOLDER
wget https://gstreamer.freedesktop.org/src/gst-plugins-base/gst-plugins-base-1.10.4.tar.xz
tar xf gst-plugins-base-1.10.4.tar.xz
cd *gst-plugins-base*/
./configure --prefix=$LOCAL_BUILD
make
sudo make install
```

##### 2.5.3: Build `gst-plugins-good-1.10.4`  

```bash
cd $SOURCE_FOLDER
wget https://gstreamer.freedesktop.org/src/gst-plugins-good/gst-plugins-good-1.10.4.tar.xz
tar xf gst-plugins-good-1.10.4.tar.xz
cd *gst-plugins-good*/
./configure --prefix=$LOCAL_BUILD
make
sudo make install
```

##### 2.5.4: Build `gst-libav-1.10.4`  

```bash
cd $SOURCE_FOLDER
wget https://gstreamer.freedesktop.org/src/gst-libav/gst-libav-1.10.4.tar.xz
tar xf gst-libav-1.10.4.tar.xz
cd *gst-libav*/
./configure --prefix=$LOCAL_BUILD
make
sudo make install
```

##### 2.5.5: Install iHeartRadio Dependencies

The following dependencies are required to playback audio from iHeartRadio:

```bash
sudo apt-get install libfaad-dev libsoup2.4-dev libgcrypt20-dev
```

##### 2.5.6: Build `gst-plugins-bad-1.10.4` (Needed for iHeartRadio)

```bash
cd $SOURCE_FOLDER
wget https://gstreamer.freedesktop.org/src/gst-plugins-bad/gst-plugins-bad-1.10.4.tar.xz
tar xf gst-plugins-bad-1.10.4.tar.xz
cd *gst-plugins-bad*/
./configure --prefix=$LOCAL_BUILD
make
sudo make install
```

#### 2.6: Build `portaudio`  

PortAudio provides a simple API for recording and playing sound. This is required to use the sample app.

Run these commands to download and build from source:  

```bash
cd $SOURCE_FOLDER
wget http://www.portaudio.com/archives/pa_stable_v190600_20161030.tgz
./configure --prefix=$LOCAL_BUILD
make
sudo make install
```

#### 2.7: Install Sensory  

In this guide, we'll use the Sensory wake word engine to detect the wake word Alexa. Follow the instructions below to install required packages and dependencies.  

##### 2.7.1: Install required packages for Sensory

```bash
sudo apt-get -y install libasound2-dev
sudo apt-get -y install libatlas-base-dev
sudo ldconfig
```

##### 2.7.2: Install Sensory

```bash
cd $SOURCE_FOLDER
git clone https://github.com/Sensory/alexa-rpi.git

bash alexa-rpi/bin/license.sh

cp alexa-rpi/lib/libsnsr.a $LOCAL_BUILD/lib
cp alexa-rpi/include/snsr.h $LOCAL_BUILD/include
cp alexa-rpi/models/spot-alexa-rpi-31000.snsr $LOCAL_BUILD/models
```

#### 2.8: Make sure everything is up to date

Ensure that you have the latest version of each dependency.

```
sudo apt-get update
```


### 3. Build the SDK and Run the Sample App on Your Raspberry Pi

#### 3.1: Download or clone the AVS Device SDK

[Github](https://github.com/alexa/avs-device-sdk) repository on your
Raspberry Pi.

#### 3.2: Create an SDK build with Sensory, GStreamer, and PortAudio:

Assuming $SDK\_SRC is SDK source location.

Create a build directory.

```
cd $HOME
mkdir BUILD
cd BUILD
```

Go into the build directory and create an out-of-source SDK build by copying the cmake command below in your terminal.

```
cmake $SDK_SRC -DSENSORY_KEY_WORD_DETECTOR=ON -DSENSORY_KEY_WORD_DETECTOR_LIB_PATH=$LOCAL_BUILD/lib/libsnsr.a -DSENSORY_KEY_WORD_DETECTOR_INCLUDE_DIR=$LOCAL_BUILD/include -DGSTREAMER_MEDIA_PLAYER=ON -DPORTAUDIO=ON
-DPORTAUDIO_LIB_PATH=$LOCAL_BUILD/lib/libportaudio.a -DPORTAUDIO_INCLUDE_DIR=$LOCAL_BUILD/include -DCMAKE_PREFIX_PATH=$LOCAL_BUILD -DCMAKE_INSTALL_PREFIX=$LOCAL_BUILD
```

#### 3.3 Update AlexaClientSDKConfig.json

After you create the out-of-source build, go to the build directory and use your favorite text editor to
open the **AlexaClientSDKConfig.json** file in the **Integration** folder. Fill in the information you noted during device registration.

```json
 {
    "authDelegate":{
        "clientSecret":"${SDK_CONFIG_CLIENT_SECRET}",
        "deviceSerialNumber":"${SDK_CONFIG_DEVICE_SERIAL_NUMBER}",
        "refreshToken":"${SDK_CONFIG_REFRESH_TOKEN}",
        "clientId":"${SDK_CONFIG_CLIENT_ID}",
        "deviceTypeId":"${SDK_CONFIG_DEVICE_TYPE_ID}"
     },

   "alertsCapabilityAgent":{
     "databaseFilePath":"${SDK_SQLITE_DATABASE_FILE_PATH}",
     "alarmSoundFilePath":"${SDK_ALARM_DEFAULT_SOUND_FILE_PATH}",
     "alarmShortSoundFilePath":"${SDK_ALARM_SHORT_SOUND_FILE_PATH}",
     "timerSoundFilePath":"${SDK_TIMER_DEFAULT_SOUND_FILE_PATH}",
     "timerShortSoundFilePath":"${SDK_TIMER_SHORT_SOUND_FILE_PATH}"
   }
 }
```

Make sure that you don’t have any extra characters (or spaces) in the paths. The config also contains paths to sound files and the wake word engine. For this exercise, point the wake word engine to the Sensory path (see step 2.7.2).

You can get the required sound files from the Timer and Alarms section at <https://developer.amazon.com/public/solutions/alexa/alexa-voice-service/content/alexa-voice-service-ux-design-guidelines>

A database is required to store scheduled alerts. In your config file, include the file path location to the database you wish to use to store and read alerts from. A database file will be created for you if the database does not already exist.


#### 3.4 Make Install

After the JSON file has been filled and double-checked, go into the build directory in your terminal and run “make”. This part will take anywhere between 30 to 45 minutes since we are using a Raspberry Pi.

```
cd $HOME/BUILD
make
make install
```

#### 3.5 Run AuthServer

When the build is complete, you need to get a refresh token for the device. Just run AuthServer, a local server implementation that will handle the token exchange:

```
python AuthServer/AuthServer.py
```

**Note:** If you run into errors here, which will pop up on your browser, click for more details to ensure that you’re able to go back and correct them.

#### 3.6 Update Refresh Token

From your Pi, open the browser and navigate to <http://localhost:3000>. Log in with your Amazon developer credentials and close the window when instructed.

![Login Screen](https://m.media-amazon.com/images/G/01/mobile-apps/dex/avs/sdk/3.png")

**Note:** If there are any errors, carefully review them as they will make it clear what went wrong. Incorrect device information in the JSON file is the most common reason for problems. If you want, open the JSON file and verify that there is a new line with your refresh token.

#### 3.7 Run The Sample App

You are now ready to run the sample app. Navigate to the SampleApp/src
folder from your build directory and run this command:

```
TZ=UTC ./SampleApp <REQUIRED-path-to-config-json> $LOCAL_BUILD/models
```

A command line interface will then pop up. Since you have set up a Wake Word Engine, all you need to do is say “Alexa”.

After you build your first prototype using the SDK and set up the development environment, go to our [Github](https://github.com/alexa/avs-device-sdk) page for more resources and guides.
