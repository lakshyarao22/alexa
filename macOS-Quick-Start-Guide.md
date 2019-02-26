
This guide provides step-by-step instructions to set up the Alexa Voice Service (AVS) Device SDK on macOS. When finished, you'll have a working sample app to test interactions with Alexa.   

**WARNING**: This guide doesn't include instructions to enable wake word.

## Prerequisites

**Software**

| Tool | Minimum Version    |
| :------------- | :------------- |
| [Python](https://cmake.org/download/) | 2.7.x |
| [Xcode](https://itunes.apple.com/us/app/xcode/id497799835?mt=12) | - |
| [Homebrew](https://raw.githubusercontent.com/Homebrew/install/master/install) | - |

To install Homebrew:

```shell
  /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

## Get started

1. [Register an AVS Product and Create a Security Profile](https://github.com/alexa/avs-device-sdk/wiki/Create-Security-Profile), if you haven't already. It must be enabled for [Code-Based Linking](https://developer.amazon.com/docs/alexa-voice-service/code-based-linking-other-platforms.html#step1).

2. Make sure that Python 2.7.x is installed on your system:

    ```sh
    python -V
    ```

3. Update Homebrew:

    ```sh
    brew update
    ```

4. Set up your build environment. Create a **my_project** folder, and these subfolders: **build**, **source**, **third-party**, **application-necessities** > **sound-files**.

    ```shell
    cd ~/
    mkdir my_project

    cd {my_project}
    mkdir build source third-party application-necessities

    cd application-necessities
    mkdir sound-files

    cd ~/{my_project}
    ```

5. Install [curl-openssl](https://formulae.brew.sh/formula/curl-openssl), which is required to connect to AVS:

     ```shell
     cd ~
     brew install curl-openssl
     ```

6. Install the core [SDK dependencies](https://github.com/alexa/avs-device-sdk/wiki/Dependencies):

     ```shell
     cd ~
     brew install gstreamer gst-plugins-base gst-plugins-good gst-plugins-bad gst-libav sqlite3 repo cmake clang-format doxygen wget git
     ```  

     **IMPORTANT**: Make sure that command ran successfully, and that no errors were thrown. If for any reason the install command fails, run brew install for each dependency individually.  


7. Install and configure PortAudio. Portaudio is required to stream microphone input/output data.

     ```shell
     cd ~/my_project/third-party
     wget -c http://www.portaudio.com/archives/pa_stable_v190600_20161030.tgz
     tar xf pa_stable_v190600_20161030.tgz

     cd portaudio
     ./configure

     make
     ```  

     **IMPORTANT**: If you see `"error: cannot find 10.5 to 10.12 SDK"`, then run:
     ```shell
     ./configure --disable-mac-universal

     make
     ```

8. Clone the SDK into the **~/my_project/source** folder:  

    ```shell
    cd ~/{my_project}/source
    git clone git://github.com/alexa/avs-device-sdk.git
    ```

## Build the SDK

1. Use the [CMake](https://cmake.org/) parameters in this section to customize how the SDK builds. To get debug logs from the sample app, include the `-DCMAKE_BUILD_TYPE=DEBUG` option. For more information, see [Debug builds](https://github.com/alexa/avs-device-sdk/wiki/cmake-options#debug-builds).

In this example it is declared that gstreamer is enabled, and provides the path to PortAudio, and also enables debug logging (which is optional):

```shell
cd ~/my_project/build

cmake /~/{my_project}/source/avs-device-sdk \
-DGSTREAMER_MEDIA_PLAYER=ON \
-DCURL_LIBRARY=/usr/local/opt/curl-openssl/lib/libcurl.dylib \
-DCURL_INCLUDE_DIR=/usr/local/opt/curl-openssl/include \
-DPORTAUDIO=ON \
-DPORTAUDIO_LIB_PATH=/~/{my_project}/third-party/portaudio/lib/.libs/libportaudio.a \
-DPORTAUDIO_INCLUDE_DIR=/~/{my_project}/third-party/portaudio/include
-DCMAKE_BUILD_TYPE=DEBUG

make
```

2. Now, you can build the SDK by running this command:

```shell
 make SampleApp -j2
 ```

**Note**: You can use `-j3` or `j4` to run processes in parallel during make.

   If you want to build the full SDK, including unit and integration tests, run `make` instead of `make SampleApp`.

## Authorization

After you have built the SDK, you'll need to configure and authorize it to access AVS using Login with Amazon (LWA). To do this, follow [these authorization instructions](https://github.com/alexa/avs-device-sdk/wiki/Authorization#macOS,-Ubuntu-Linux-16.04-LTS,-and-Raspberry-Pi).

## Set up launch shortcut

You can designate aliases to launch the sample app. To do this:

1. Open `~/.bash_profile`. Add these aliases and **save**:  
   **IMPORTANT**: Make sure you update the paths to match your folder structure.  
   ```sh
   alias alexac="~/{my_project}/{build}/SampleApp/src/SampleApp {build}/Integration/AlexaClientSDKConfig.json"
   alias alexacdebug="~/{my_project/build}/SampleApp/src/SampleApp ~/{my_project}/{build}/Integration/AlexaClientSDKConfig.json DEBUG9"
   ```
2. After you've added these aliases, make sure to activate your `~/.bash_profile`:  
   ```shell
   source ~/.bash_profile
   ```
3. Now the sample app can be launched with this single command:
   ```shell
   alexac  
   ```

## Common issues  

See the [Troubleshooting Guide](https://github.com/alexa/avs-device-sdk/wiki/Troubleshooting-Guide).

## Additional resources  

* [Cmake parameters](https://github.com/alexa/avs-device-sdk/wiki/cmake-options)  
* [Unit and Integration Tests](https://github.com/alexa/avs-device-sdk/wiki/Unit-and-Integration-Tests)  
