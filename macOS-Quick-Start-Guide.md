
This guide provides step-by-step instructions to set up the Alexa Voice Service (AVS) Device SDK on macOS. When finished, you'll have a working sample app to test interactions with Alexa.   

**WARNING**: This guide doesn't include instructions to enable wake word.

## Prerequisites  
This guide assumes that:

* All commands are run from your home directory (~/)
* You are running Python 2.7.x  
* [Xcode](https://itunes.apple.com/us/app/xcode/id497799835?mt=12) is installed
* Homebrew is your package manager

  To install Homebrew, run this command:  

```shell
  /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

## Register your product and create security profile

Follow these instructions to [register your product and create a security profile](https://github.com/alexa/avs-device-sdk/wiki/Create-Security-Profile).

During this process, you'll download a **config.json** file to your machine that will be used by the SDK to authorize your build of the SDK with Login With Amazon (LWA).

**Note**: If you already have a registered product that you can use for testing, you may use it, but it must be enabled for use with Code Based Linking (CBL).

Here are the instructions on [how to enable CBL for your device](https://developer.amazon.com/docs/alexa-voice-service/code-based-linking-other-platforms.html#step1).

## Setup your environment

Let's create a few folders to organize our files:

```shell
cd ~/
mkdir sdk-folder

cd sdk-folder
mkdir sdk-build sdk-source third-party application-necessities

cd application-necessities
mkdir sound-files

cd ~/sdk-folder
```

### Install dependencies  

The AVS Device SDK requires libraries to:

* Maintain an HTTP2 connection with AVS  
* Play Alexa Text-to-Speech (TTS) and music  
* Record audio from the microphone  
* Store records in a database (persistent storage)  

1. Update Homebrew. This ensures that you have access to required dependencies:

    ```shell
    brew update
    ```

2. Now use Homebrew to install and link openSSL:

    ```sh
    brew install openssl
    brew link --force openssl
    ```

On macOS, when you try to link openSSL, you may encounter this error:

    ```sh
    $ brew link --force openssl
    Warning: Refusing to link: openssl
    Linking keg-only openssl means you may end up linking against the insecure,
    deprecated system OpenSSL while using the headers from Homebrew's openssl.
    Instead, pass the full include/library paths to your compiler e.g.:
       -I/usr/local/opt/openssl/include -L/usr/local/opt/openssl/lib
    ```

If so, you'll need to manually link openSSL to your local profile, **/usr/local/<include>**, which is where the compiler looks during the linking process. To do this, follow these steps:

    1. Manually link openSSL to your profile:

    ```sh
    echo export PKG_CONFIG_PATH="/usr/local/opt/openssl/lib/pkgconfig:$PKG_CONFIG_PATH" >> ~/.bash_profile
    source ~/.bash_profile
    ```
    2. Delete your <sdk build> directory.
    3. Start a new shell session, then continue with the steps below.

3. Run this command to configure `curl` with **http2**. This is required to connect to AVS:  

     ```shell
     brew install curl --with-nghttp2
     brew link curl --force

     echo export PATH="/usr/local/opt/curl/bin:$PATH" >> ~/.bash_profile
     source ~/.bash_profile
     ```

4. Now install the [SDK dependencies](https://github.com/alexa/avs-device-sdk/wiki/Dependencies):

     ```shell
     brew install gstreamer gst-plugins-base gst-plugins-good gst-plugins-bad gst-libav sqlite3 repo cmake clang-format doxygen wget git
     ```  

     **IMPORTANT**: Make sure that command ran successfully, and that no errors were thrown. If for any reason the install command fails, run brew install for each dependency individually.  


5. PortAudio is required to record microphone data. Run this command to install and configure PortAudio:

     ```shell
     cd ~/sdk-folder/third-party
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

### Clone the AVS Device SDK

Let's clone the SDK into the `sdk-source` folder:  

```shell
cd ~/sdk-folder/sdk-source
git clone git://github.com/alexa/avs-device-sdk.git
```

## Build the SDK  

1. Run CMake to generate the build dependencies. To get debug logs from the sample app, add `-DCMAKE_BUILD_TYPE=DEBUG` to the end of your command. For more information, see [Debug builds](https://github.com/alexa/avs-device-sdk/wiki/Build-Options#debug-builds).

In this example it is declared that gstreamer is enabled, and provides the path to PortAudio, and also enables debug logging (which is optional):

    **IMPORTANT**: Replace all instances of `{home}` with the absolute path to your home directory. For example: `/Users/courtneybarnett/`.

    ```shell
    cd ~/sdk-folder/sdk-build

    cmake /{home}/sdk-folder/sdk-source/avs-device-sdk \
    -DGSTREAMER_MEDIA_PLAYER=ON \
    -DPORTAUDIO=ON \
    -DPORTAUDIO_LIB_PATH=/{home}/sdk-folder/third-party/portaudio/lib/.libs/libportaudio.a \
    -DPORTAUDIO_INCLUDE_DIR=/{home}/sdk-folder/third-party/portaudio/include
    -DCMAKE_BUILD_TYPE=DEBUG

    make
    ```

2. Here's the fun part, let's build! For this project we're only building the sample app.

    Run this command:
    ```shell
    make SampleApp -j2
    ```

   **Note**: Try the `-j3` or `j4` to run processes in parallel during make.

   If you want to build the full SDK, including unit and integration tests, run `make` instead of `make SampleApp`.

## Setup your configuration

The sample app uses data in `AlexaClientSDKConfig.json` to obtain a refresh token which, along with your **Client ID** and **Product ID**, will be exchanged with Login With Amazon (LWA) to gain access tokens.

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

2. Navigate to **{{path to source folder}}/avs-device-sdk/tools/Install** and run `genConfig.sh`, including the following arguments:

    ```sh
    cd {{path to source folder}}/avs-device-sdk/tools/Install

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
    value.

        For example:

        ```sh
        sudo bash config.json -s 998987
        ```
        **Note**: If you don't supply a DSN, then the default value `123456` will be generated by the SDK.
    * **{{path to database}}**: Specifies where the databases will be located.
    * **{{path to source folder}}**: Specifies the root directory where the avs-device-sdk source code is located.
    * **{{path to build}}**: Specifies where the build folder is, which contains the **Integration/AlexaClientSDKConfig.json** file.

**IMPORTANT**: Create a backup of your `AlexaClientSDKConfig.json` file. Subsequent builds will reset the contents of this file.

## Run and authorize

Navigate to your *BUILD* folder, then:
1. Start the sample app:
```shell
./SampleApp/src/SampleApp ./Integration/AlexaClientSDKConfig.json
```
2. Wait for the sample app to display a message like this:
```shell
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
```shell
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
* If you want to move this authorization to another sample app installation, you need to copy the **deviceInfo** object within `AlexaClientSDKConfig.json` to the new installation. You will also need to copy the file `"/{home}/sdk-folder/application-necessities/cblAuthDelegate.db"` to the new installation, and update **AlexaClientSDKConfig.json** in the new installation so that the **cblAuthDelegate's databaseFilePath** property points to it.

## Setup shortcuts

Now that you've built the sample app, let's setup some shortcuts so that you don't have to remember the full command to launch the app.  

1. Use your favorite text editor to open `~/.bash_profile`. Then add these aliases and **save**:  
   **IMPORTANT**: Make sure you update the paths to match your folder structure.  
   ```sh
   alias alexac="[path_to_build_folder]/SampleApp/src/SampleApp [path_to_build_folder]/Integration/AlexaClientSDKConfig.json"
   alias alexacdebug="[path_to_build_folder]/SampleApp/src/SampleApp [path_to_build_folder]/Integration/AlexaClientSDKConfig.json DEBUG9"
   ```
2. After you've added these aliases, make sure to activate your `~/.bash_profile`:  
   ```shell
   source ~/.bash_profile
   ```
3. Now to launch the sample app just run this command:  
   ```shell
   alexac  
   ```

## Common issues  

See the [Troubleshooting Guide](https://github.com/alexa/avs-device-sdk/wiki/Troubleshooting-Guide).

## Next steps  

* [Build Options](https://github.com/alexa/avs-device-sdk/wiki/Build-Options)  
* [Unit and Integration Tests](https://github.com/alexa/avs-device-sdk/wiki/Unit-and-Integration-Tests)  
