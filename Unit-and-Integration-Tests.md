## Run Unit Tests

Unit tests for the AVS Device SDK for C++ use the [Google Test](https://github.com/google/googletest) framework. Use this command to run all unit tests: 

```
make all test
```

Make sure that all tests pass before you begin integration testing.

### Run Unit Tests with Sensory Enabled

If the project was built with the Sensory wake word detector, the following files must be downloaded from [GitHub](https://github.com/Sensory/alexa-rpi) and placed in `<source dir>/KWD/inputs/SensoryModels` for the integration tests to run properly:

* [`spot-alexa-rpi-31000.snsr`](https://github.com/Sensory/alexa-rpi/blob/master/models/spot-alexa-rpi-31000.snsr)

### Run Unit Tests with KITT.ai Enabled

If the project was built with the KITT.ai wake word detector, the following files must be downloaded from [GitHub](https://github.com/Kitt-AI/snowboy/tree/master/resources) and placed in `<source dir>/KWD/inputs/KittAiModels`:
* [`common.res`](https://github.com/Kitt-AI/snowboy/tree/master/resources)
* [`alexa.umdl`](https://github.com/Kitt-AI/snowboy/tree/master/resources/alexa/alexa-avs-sample-app) - It's important that you download the `alexa.umdl` in `resources/alexa/alexa-avs-sample-app` for the KITT.ai unit tests to run properly.

## Run Integration Tests

Integration tests ensure that your build can make a request and receive a response from AVS.
* All requests to AVS require auth credentials
* The integration tests for Alerts require your system to be in UTC

**Important**: Integration tests reference an `AlexaClientSDKConfig.json` file, which you must create.
See the `Create the AlexaClientSDKConfig.json file` section (above), if you have not already done this.

To run the integration tests use this command: 

```
TZ=UTC make all integration
```

### Network Integration Test  

If your project is built on a GNU/Linux-based platform (Ubuntu, Debian, etc.), there is an optional integration test that tests the ACL for use on slow networks. 

This `cmake` option is required when you build the SDK:

```
cmake <absolute-path-to-source> -DNETWORK_INTEGRATION_TESTS=ON –DNETWORK_INTERFACE=eth0
```

**Note**: The name of the network interface can be located with this command `ifconfig -a`.
**IMPORTANT**: This test requires root permissions.  

### Run Integration Tests with Sensory Enabled

If the project was built with the Sensory wake word detector, the following files must be downloaded from [GitHub](https://github.com/Sensory/alexa-rpi) and placed in `<source dir>/Integration/inputs/SensoryModels` for the integration tests to run properly:

* [`spot-alexa-rpi-31000.snsr`](https://github.com/Sensory/alexa-rpi/blob/master/models/spot-alexa-rpi-31000.snsr)

### Run Integration Tests with KITT.ai

If the project was built with the KITT.ai wake word detector, the following files must be downloaded from [GitHub](https://github.com/Kitt-AI/snowboy/tree/master/resources) and placed in `<source dir>/Integration/inputs/KittAiModels` for the integration tests to run properly:
* [`common.res`](https://github.com/Kitt-AI/snowboy/tree/master/resources)
* [`alexa.umdl`](https://github.com/Kitt-AI/snowboy/tree/master/resources/alexa/alexa-avs-sample-app) - It's important that you download the `alexa.umdl` in `resources/alexa/alexa-avs-sample-app` for the KITT.ai integration tests to run properly.