Want to see the AVS Device SDK in action? This guide is designed to have a working sample running on a Raspberry Pi 3 in less than an hour.

This guide uses a handful of scripts to download, build, and run the AVS Device SDK with wake word detection enabled. If you'd like to build from scratch, we also provide [step-by-step instructions](https://github.com/alexa/alexa-avs-sample-app/wiki/Raspberry-Pi-Quick-Start-Guide) that will walk you through downloading dependencies, running the authorization service, and running the sample app in debug mode.

## Required Hardware

1. **Raspberry Pi 3** (Recommended) or **Pi 2 Model B** (Supported)  - Buy at Amazon - [Pi 3](https://amzn.com/B01CD5VC92) or [Pi 2](http://amzn.com/B00T2U7R7I).
2. **Micro-USB power cable** for Raspberry Pi.
3. **Micro SD Card** (Minimum 8 GB) - You need an operating system to get started. NOOBS (New Out of the Box Software) is an easy-to-use operating system install manager for Raspberry Pi. The simplest way to get NOOBS is to buy an SD card with NOOBS pre-installed - [Raspberry Pi 8GB Preloaded (NOOBS) Micro SD Card](https://www.amazon.com/gp/product/B00ENPQ1GK/ref=oh_aui_detailpage_o01_s00?ie=UTF8&psc=1). Alternatively, you can download and install it on your SD card (follow instructions [here](#step-1-setting-up-your-pi)).
4. **USB 2.0 Mini Microphone** - Raspberry Pi does not have a built-in microphone; to interact with Alexa you'll need an external one to plug in - [Buy on Amazon](http://amzn.com/B00IR8R7WQ)
5. **External Speaker** with 3.5mm audio cable - [Buy on Amazon](http://amzn.com/B007OYAVLI)
6. A **USB Keyboard & Mouse**, and an external **HDMI Monitor** - we also recommend having a USB keyboard and mouse as well as an HDMI monitor handy if you're unable to [remote(SSH)](Setup-SSH-&-VNC) into your Pi.
7. Internet connection (Ethernet or WiFi)
8. (Optional) WiFi Wireless Adapter for Pi 2 ([Buy on Amazon](http://www.amazon.com/CanaKit-Raspberry-Wireless-Adapter-Dongle/dp/B00GFAN498/)).
   **Note:** Pi 3 has built-in WiFi.

## Register a Product  
Before we get started, you'll need to register a device and create a security profile at developer.amazon.com. [Click here](https://github.com/alexa/alexa-avs-sample-app/wiki/Create-Security-Profile) for step-by-step instructions.

**IMPORTANT**: The allowed origins under web settings should be `http://localhost:3000` and  `https://localhost:3000`. The return URLs under web settings should be `http://localhost:3000/authresponse` and `https://localhost:3000/authresponse`.  

If you already have a registered product that you can use for testing, feel free to skip ahead.

## Setup and Run
1. Download the install script and configuration file. We recommend running these commands from the home directory (`~/`) or Desktop, however, you can run the script anywhere.  
    ```
    wget https://raw.githubusercontent.com/alexa/avs-device-sdk/master/tools/RaspberryPi/setup.sh && wget https://raw.githubusercontent.com/alexa/avs-device-sdk/master/tools/RaspberryPi/config.txt
    ```
2. Update `config.txt` with the **Client ID**, **Client Secret**, and **Product ID** for your registered product and **save**.   
3. Run the setup script with your configuration as an argument:
    ```
    bash setup.sh config.txt
    ```
4. After the setup script has finished running, you'll need to generate an authorization token. Run this command and follow the on-screen instructions:
    ```
    bash startauth.sh
    ```
5. Last and most importantly, let's run the Sample App:
    ```
    bash startsample.sh
    ```
6. You can also run integration and unit tests:
    ```
    bash test.sh
    ```
