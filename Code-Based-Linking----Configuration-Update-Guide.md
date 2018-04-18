This guide is intended to help you adopt changes related to Code Based Linking (CBL) if you have a v1.7 or earlier build of the SDK.

There have been changes made to the configuration file, which you need to be aware of. Specifically:

1. `deviceInfo` - This is a new configuration object that specifies device identifying information for use by the Device Capability Framework (DCF) and for authorization (`CBLAuthDelegate`). This object has three required values:
   * clientId (from the "Alexa Dashboard" > "My products" > "manage" > "Security Profiles" > "Other devices and platforms")
   * deviceSerialNumber (unique string, supplied by developer)
   * productId (from "Alexa Dashboard" > "My Products")  

2. `cblAuthDelegate` - This is a new configuration object that specifies parameters for CBLAuthDelegate. This object has one required value:
   * databaseFilePath - The absolute path / file-name specifying where CBLAuthDelegate should persist data.

For more detailed instructions, please consult the quick start guide for your platform: 
* [Linux Reference Guide](https://github.com/alexa/avs-device-sdk/wiki/Linux-Reference-Guide)
* [Ubuntu Linux Quick Start Guide](https://github.com/alexa/avs-device-sdk/wiki/Ubuntu-Linux-Quick-Start-Guide)
* [macOS Quick Start Guide](https://github.com/alexa/avs-device-sdk/wiki/macOS-Quick-Start-Guide)
* [Raspberry Pi Quick Start Guide with Script](https://github.com/alexa/avs-device-sdk/wiki/Raspberry-Pi-Quick-Start-Guide-with-Script)
* [Windows Quick Start Guide with Script](https://github.com/alexa/avs-device-sdk/wiki/Windows-Quick-Start-Guide-with-Script)



