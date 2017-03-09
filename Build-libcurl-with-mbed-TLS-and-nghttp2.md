This document provides step-by-step instructions to build libcurl with mbed TLS and nghttp2 in `*nix` systems.

We recommend that you use separate directories to keep track of your optimized builds, especially if you use these libraries elsewhere. For example, if you are building the Alexa Client SDK on a Linux machine that you use daily, installing these optimized versions will most likely break other operations. Therefore, in this document we will assume that the principle working directory is `$WD`.

**Note**: These instructions are verified for Ubuntu 16.04.  

## Build Instructions

1. Run the following commands to install CMake. You can skip this step if CMake is already installed.  
   * `wget https://cmake.org/files/v3.8/cmake-3.8.0-rc1.tar.gz`  
   * `tar -xvf cmake-3.8.0-rc1.tar.gz`  
   * `cd cmake-3.8.0-rc1/`  
   * `./bootstrap && make && sudo make install`
     **Note**: If you encounter issues bootstrapping or finding the appropriate C/C++ compiler, try this command: `sudo apt-get install build-essential`.  
2. Install the [latest version of nghttp2](https://github.com/nghttp2/nghttp2/releases) to your principle working directory `$WD`. Run these commands:  

   **Note**: This example presumes you have downloaded v1.19.0. Make sure you adjust the commands below to match the version of nghttp2 you have downloaded.  

   * `wget https://github.com/nghttp2/nghttp2/releases/download/v1.19.0/nghttp2-1.19.0.tar.gz`  
   * `tar -xvf nghttp2-1.19.0.tar.gz`
   * `cd nghttp2-1.19.0`  
   * `autoreconf -i`  
     **Note**: You may need to run the following command to obtain the right tools: `sudo apt-get update && sudo apt-get install g++ make binutils autoconf automake autotools-dev libtool pkg-config zlib1g-dev libcunit1-dev libssl-dev libxml2-dev libev-dev libevent-dev -y`
   * `automake`
   * `autoconf`  
   * `./configure --prefix=$WD/nghttp2/`  
   * `make`  
   * `sudo make install`  
3. Install the latest version of [mbed TLS](https://github.com/ARMmbed/mbedtls/releases) to your principle working directory `$WD`. Run these commands:  

   **Note**: This example presumes you have downloaded v2.4.0. Make sure you adjust the commands below to match the version of mbed TLS you have downloaded.   

   * `wget https://tls.mbed.org/download/mbedtls-2.4.0-apache.tgz`  
   * `tar -xvf mbedtls-2.4.0-apache.tgz`  
   * `cd mbedtls-2.4.0/`  
   * Build with CMake:
     * `cmake -DCMAKE_INSTALL_PREFIX=$WD/mbedtls/ -DUSE_SHARED_MBEDTLS_LIBRARY=On`  
     * `make`  
     * `sudo make install`  
4. Install the latest version of [libcurl (7.50.2 or later)](https://curl.haxx.se/download.html). Run these commands:      

   **Note**: This example presumes you have downloaded v7.53.0. Make sure you adjust the commands below to match the version of libcurl you have downloaded.  

   * `wget https://curl.haxx.se/download/curl-7.53.0.tar.gz`  
   * `tar -xvf curl-7.53.0.tar.gz`  
   * `cd curl-7.53.0/`  
   * `LIBS="-lpthread" LDFLAGS="-Wl,-R$WD/mbedtls/lib" ./configure --with-nghttp2=$WD/nghttp2/ --without-ssl --with-mbedtls=$WD/mbedtls --prefix=$WD/curl/`  
   * The configuration summary should be printed to your terminal. Make sure that SSL support (mbed TLS) and HTTP/2 support (nghttp2) are enabled.  
   * `make`  
   * `sudo make install`  
5. Build the Alexa Client SDK.
   * Pull the latest version of the SDK. From your project directory run: `git pull`.  
   * Pass the recently built libcurl to the make process. **Note**: This assumes that the source code for the Alexa Client SDK is located in `$ACSDK_SOURCE_DIR` and the build directory is `$ACSDK_BUILD_DIR`.  
     * Change directories: `cd $ACSDK_BUILD_DIR`  
     * Then run these commands:
       * `cmake -DCURL_LIBRARY=$WD/curl/lib/libcurl.so -DCURL_INCLUDE_DIR=$WD/curl/include/ $ACSDK_SOURCE_DIR`  
       **Note**: If Doxygen is not found, you will need to run the following command: `sudo apt-get install doxygen`  
       * `make`  
       * `make test`  
     * After creating your Integration/AuthDelegate.config file (see [README.md](https://github.com/alexa/alexa-client-sdk)), run integration tests with this command: `make integration`.

That is it, you have build libcurl with mbed TLS and nghttp2. Click here to return to [README.md](https://github.com/alexa/alexa-client-sdk).

## Release Notes   

| Version | Release Date | Notes |
|---------|--------------|-------|  
| v1.0 | 3/8/2017 | Initial release. |
