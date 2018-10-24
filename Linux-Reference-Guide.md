This guide assumes familiarity with Linux and requires you to build dependencies from source.

## Register your product and create security profile

Follow these instructions to [register your product and create a security profile](https://github.com/alexa/avs-device-sdk/wiki/Create-Security-Profile). During this process, you will download a config.json to your machine that is used to complete configuration of the sample app.

Note: If you already have a registered product that you can use for testing, you may use it, but it must be enabled for use with Code Based Linking (CBL).

Here are the instructions on [how to enable CBL for your device](https://developer.amazon.com/docs/alexa-voice-service/code-based-linking-other-platforms.html#step1).

## Dependencies

See [Dependencies](https://github.com/alexa/avs-device-sdk/wiki/Dependencies) for version details and installation instructions.

## Create an out-of-source build

The AVS Device SDK must be built out-of-source. Creating sub-folders within your project for *BUILD*, *SOURCE*, *THIRD-PARTY*, and *APPLICATION NECESSITIES* is recommended.

### Clone the AVS Device SDK

Clone the AVS Device SDK into your {{SOURCE}} folder:  
```shell
git clone https://github.com/alexa/avs-device-sdk.git
```

### Build the AVS Device SDK

Navigate to your {{BUILD}} folder. This sample command does a few things:  
* It declares that PortAudio is used to capture microphone data and points to its lib path and includes directory
   * It declares that gstreamer is installed and will be used when you build the SampleApp
   * It declares that the wake word detector is **OFF**  

*Linux supports wake word detectors from Sensory and Kitt.ai. Each requires a license from the provider. For instructions to build with a wake word detector, please see [Build Options](https://github.com/alexa/avs-device-sdk/wiki/Build-Options).*  

```shell
cmake /{PATH TO SOURCE FOLDER}/avs-device-sdk \
-DSENSORY_KEY_WORD_DETECTOR=OFF \
-DGSTREAMER_MEDIA_PLAYER=ON \
-DPORTAUDIO=ON \
-DPORTAUDIO_LIB_PATH=/{PATH TO PORTAUDIO}/lib/.libs/libportaudio.a \
-DPORTAUDIO_INCLUDE_DIR=/{PATH TO PORTAUDIO}/include

make
```

**IMPORTANT**: See [Build Options](https://github.com/alexa/avs-device-sdk/wiki/Build-Options) for a additional configuration options.

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
    cd {{path to source folder}}/avs-device-sdk/tools/Install

    genConfig.sh config.json {device serial number} \
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
    * **{{device serial number}}**: Each instance of the SDK requires a unique **Device Serial Number** (also found in the **deviceInfo** object). This is provided by you, and in some instances may match your product's SKU. By default the value is `123456`.
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
* If you want to move this authorization to another sample app installation, you need to copy the **deviceInfo** object within `AlexaClientSDKConfig.json` to the new installation.  You will also need to copy the file `"/{PATH TO APPLICATION NECESSITIES}/cblAuthDelegate.db"` to the new installation, and update **AlexaClientSDKConfig.json** in the new installation so that the **cblAuthDelegate's databaseFilePath** property points to it.

## Enabling debug logs

Debug logs are available in DEBUG builds.  You can enable them by adding the following to the end of your cmake command line:
```shell
cmake < other cmake parameters > -DCMAKE_BUILD_TYPE=DEBUG
```
An additional sample app command line parameter allows you to select the level of debug logging to be output.  The allowed values are DEBUG0, DEBUG1, ... DEBUG9, where DEBUG9 provides the most information.  For example:
```shell
./SampleApp/src/SampleApp ./Integration/Integration/AlexaClientSDKConfig.json DEBUG9
```  

## Run tests
After you've authorized your sample app, run integration and unit tests to ensure that the AVS Device SDK is functioning as designed.

* Use this command to run integration tests:
   ```shell
   make all integration
   ```
* Use this command to run unit tests:  
   ```shell
   make all test
   ```

For additional details, see [Unit and Integration Tests](https://github.com/alexa/avs-device-sdk/wiki/Unit-and-Integration-Tests).

## Setup shortcuts

Now that you've built the sample app, you can set up some shortcuts to make launching the sample app easier:   

1. Use your favorite text editor to open `~/.bash_profile`. Then add these aliases and **save**:  
   **IMPORTANT**: Make sure you update the paths to match your folder structure.  
   ```shell
   alias alexac="[path_to_build_folder]/SampleApp/src/SampleApp [path_to_build_folder]/Integration/AlexaClientSDKConfig.json"
   alias alexacdebug="[path_to_build_folder]/SampleApp/src/SampleApp [path_to_build_folder]/Integration/AlexaClientSDKConfig.json DEBUG9"
   ```
2. After you've added these aliases, make sure to activate your `~/.bash_profile`:  
   ```shell
   source ~/.bash_profile
   ```
3. Now to launch the sample app, just run this command:  
   ```
   alexac  
   ```

## Bluetooth

Building with Bluetooth is optional, and is currently limited to Linux and Raspberry Pi. The `A2DP-SINK`, `A2DP-SOURCE`, `AVRCPTarget`, and `AVRCPController` profiles are supported for Linux/Ubuntu.

To enable Bluetooth on Linux, follow these steps:

### 1. Install Dependencies

In order to use Bluetooth, you must install these dependencies:

* Core [Bluetooth dependencies](https://github.com/alexa/avs-device-sdk/wiki/Dependencies#bluetooth-dependencies)
* PulseAudio
* PulseAudio bluetooth modules:
```
sudo apt-get install pulseaudio-module-bluetooth
```

### 2. Initialize the SDK

To use Bluetooth with the SDK, you must build and initialize the SDK before you load the PulseAudio bluetooth modules. This establishes order-of-priority for sink audio handing, setting the SDK as the controller.

### 3. Load bluetooth modules

After the SDK has been initialized, you'll need to unload and then load the PulseAudio bluetooth modules.

For example:

```
pactl unload-module module-bluetooth-discover; pactl unload-module module-bluetooth-policy; pactl load-module module-bluetooth-discover; pactl load-module module-bluetooth-policy
```

Note: When installing these modules, if they fail upon unload (because they weren't originally loaded), this is not a critical error -- and should not affect your Bluetooth integration.

### 4. AVRCPTarget

If you are using the `AVRCPTarget` profile, you'll need to enable permissions for BlueZ to interact with `"org.mpris.MediaPlayer2.Player"`.

To do this, open **/etc/dbus-1/system.d/bluetooth.conf**  and add `<allow send_interface="org.mpris.MediaPlayer2.Player"/>` to your root policy.

For example:

```sh
<!DOCTYPE busconfig PUBLIC "-//freedesktop//DTD D-BUS Bus Configuration 1.0//EN"
 "http://www.freedesktop.org/standards/dbus/1.0/busconfig.dtd">
<busconfig>

  <!-- ../system.conf have denied everything, so we just punch some holes -->

  <policy user="root">
	...
   	<allow send_interface="org.mpris.MediaPlayer2.Player"/>
  </policy>

...

</busconfig>
```
