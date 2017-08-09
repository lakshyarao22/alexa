## How to setup your Raspberry Pi development environment

Use these instructions to setup your development environment. These instructions assume that you are using a Raspberry Pi 3 that is running Raspbian Jessie.

**IMPORTANT**: This guide will guide you through building the minimum required version for each dependency. If you plan to use a newer version, please confirm that each command is valid, specifically configuration steps.  

**TIP**: All tests were performed on Raspbian Jessie Lite if you prefer not to install a GUI distribution.  

## Step 1: Create a source folder for your dependencies

To get your Raspberry Pi ready to consume the Alexa Device SDK, you'll need to build some dependencies from source. It's a best practice to keep source files in a pre-defined folder. Use the command to create your source folder. Feel free to modify the  `SOURCE_FOLDER` value. Keep in mind that this variable will be referenced throughout this doucment:

```bash
echo "export SOURCE_FOLDER=$HOME/sources" >> $HOME/.bash_aliases
source $HOME/.bashrc
mkdir $SOURCE_FOLDER
```

## Step 2: Download build tools

The next step is to download some essential build tools. Run this command:  

```bash
sudo apt-get install git gcc cmake build-essential
```

## Step 3: Set up `libcurl` with `nghttp2` and `openssl`

This is the most important part of setup, since a connection to AVS requires the use of HTTP2, and the SDK uses `libcurl` to establish that connection.

### Step 3.1: Build `nghttp2` from source

Unfortunately, the `nghttp2` development package of provided in Jessie is out-of-date, so we'll need to build the correct version from source. The first step is to get the source:

```bash
cd $SOURCE_FOLDER
wget https://github.com/nghttp2/nghttp2/releases/download/v1.0.0/nghttp2-1.0.0.tar.gz
tar xzf nghttp2-1.0.0.tar.gz
```

Then configure, build, and install with these commands:

```bash
cd $SOURCE_FOLDER/*nghttp2*/
./configure --prefix=/usr --disable-app
make
sudo make install
```

### Step 3.2: Build `openssl` from source

The `openssl` package included with Jessie is out-of-date, so we'll need to build the correct version from source.

Run these commands:

```bash
cd $SOURCE_FOLDER
wget https://www.openssl.org/source/old/1.0.2/openssl-1.0.2a.tar.gz
tar xzf openssl-1.0.2a.tar.gz
cd *openssl*/
./config --prefix=/usr --openssldir=/usr shared
make
sudo make install
```

### Step 3.3: Build `libcurl` from source

Now that `nghttp2` and `openssl` are built from source, we can build cURL.

Run these commands:

```bash
cd $SOURCE_FOLDER
wget https://curl.haxx.se/download/curl-7.50.2.tar.gz
tar xzf curl-7.50.2.tar.gz
cd *curl*/
./configure --with-ssl=/usr --with-nghttp2=/usr
make
sudo make install
```

## Step 4: Install `sqlite`  

SQLite is required for Alerts. Run this command to install:

```
sudo apt-get install libsqlite3-dev
```
---  
**IMPORTANT**: The following dependencies are required to use the included `MediaPlayer` implementation, as well as to build the sample app.  
---  

## Step 5: Build `gstreamer` libraries

The sample app requires a media player implementation to play MP3 files; our implementation uses GStreamer. Before we build GStreamer, we need to install some utilities and dependencies.

Run this command to install:

```
sudo apt-get install bison flex libglib2.0-dev pulseaudio libpulse-dev
```

### Step 5.1: Build `gstreamer-1.10.4`

```bash
cd $SOURCE_FOLDER
wget https://gstreamer.freedesktop.org/src/gstreamer/gstreamer-1.10.4.tar.xz
tar xf gstreamer-1.10.4.tar.xz
cd *gstreamer*/
./configure --prefix=/usr
make
sudo make install
```

### Step 5.2:  Build `gst-plugins-base-1.10.4`

```bash
cd $SOURCE_FOLDER
wget https://gstreamer.freedesktop.org/src/gst-plugins-base/gst-plugins-base-1.10.4.tar.xz
tar xf gst-plugins-base-1.10.4.tar.xz
cd *gst-plugins-base*/
./configure --prefix=/usr
make
sudo make install
```

### Step 5.3: Build `gst-plugins-good-1.10.4`  

```bash
cd $SOURCE_FOLDER
wget https://gstreamer.freedesktop.org/src/gst-plugins-good/gst-plugins-good-1.10.4.tar.xz
tar xf gst-plugins-good-1.10.4.tar.xz
cd *gst-plugins-good*/
./configure --prefix=/usr
make
sudo make install
```

### Step 5.4: Build `gst-libav-1.10.4`  

```bash
cd $SOURCE_FOLDER
wget https://gstreamer.freedesktop.org/src/gst-libav/gst-libav-1.10.4.tar.xz
tar xf gst-libav-1.10.4.tar.xz
cd *gst-libav*/
./configure --prefix=/usr
make
sudo make install
```

## Step 6: Build `portaudio`  

PortAudio provides a simple API for recording and/or playing sound. This is required to use the sample app.

Run these commands to download and build from source:  

```bash
cd $SOURCE_FOLDER
wget http://www.portaudio.com/archives/pa_stable_v190600_20161030.tgz
tar xzf pa_stable_v19060020161030.tgz
cd portaudio
./configure --prefix=/usr
make
sudo make install
```
