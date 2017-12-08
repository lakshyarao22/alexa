
These are step-by-step instructions to cross-compile and build the AVS Device SDK for iOS. If you encounter any errors or have questions, please check our issues list before creating a new issue. 

1. Install [Xcode](https://itunes.apple.com/us/app/xcode/id497799835?mt=12). Skip to the next step if previously installed.  
2. Install [Homebrew](https://brew.sh/). Skip to the next step if previously installed. 

   ```
   /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
   ```
3. Install additional dependencies: 
  
   ```
   brew install cmake
   brew install pkg-config
   ``` 
4. Create a folder for your project. In this example project, we've named the folder `cross-compile`, however you can use whatever you'd like:  

   ```
   cd ~
   mkdir cross-compile
   cd cross-compile
   ```
5. Download the CMake toolchain for iOS: 

   ```
   cd ~/cross-compile
   git clone https://github.com/leetal/ios-cmake.git
   ```
6. Download and cross-compile Google Test: 

   ```
   cd ~/cross-compile
   git clone https://github.com/google/googletest.git
   mkdir googletest_build && cd googletest_build
   cmake ../googletest -DIOS_DEPLOYMENT_TARGET=10.0 -DCMAKE_TOOLCHAIN_FILE=../ios-cmake/ios.toolchain.cmake -DIOS_PLATFORM=OS
   make
   ```
7. Download and cross-compile OpenSSL and curl. For this step, we're going to use an open source script from GitHub. Heads-up, this may take a while: 
   * Clone the repository: 
     ```
     cd ~/cross-compile
     git clone https://github.com/jasonacox/Build-OpenSSL-cURL.git
     cd Build-OpenSSL-cURL/
     ```
   * Open `build.sh` with your favorite test editor and change the curl version to 7.55.1 and **save**.   
     ```
     ########################################
     # EDIT this section to Select Versions #
     ########################################

     OPENSSL="1.0.2l"
     LIBCURL="7.55.1"
     NGHTTP2="1.24.0"

     ########################################
     ```
   * Run the script:
     ```
     ./build.sh
     ```
8. Download the AVS Device SDK and make a few minor adjustments:  
   * Clone the repository:  
     ```
     cd ~/cross-compile
     git clone https://github.com/alexa/avs-device-sdk.git  
     cd avs-device-sdk/ 
     ```
   * Add this code between lines 98 and 99 of `/AVSCommon/Utils/src/LibcurlUtils/LibcurlUtils.cpp`:   
     ```
     + setopt(handle, CURLOPT_CAINFO, caPath.c_str(), "CURLOPT_CAINFO", caPath.c_str());
     ```
   * After you add the line, it should look like this: 
     ```
     if (configuration::ConfigurationNode::getRoot()[LIBCURLUTILS_CONFIG_KEY].getString(CAPATH_CONFIG_KEY, &caPath)) {
       + setopt(handle, CURLOPT_CAINFO, caPath.c_str(), "CURLOPT_CAINFO", caPath.c_str());
         return setopt(handle, CURLOPT_CAPATH, caPath.c_str(), "CURLOPT_CAPATH", caPath.c_str());
     }
     ```
   * The SDK must be run as a STATIC library. Run this command to make it global in all subfolders:  
     ```
     cd ~/cross-compile/avs-device-sdk/
     find . -type f -name '*.txt' -exec sed -i '' s/SHARED/STATIC/ {} +
     ```
9. Now we'll build the AVS Device SDK.  
   * Run these commands to build for iOS:  
     ```
     cd ~/cross-compile
     mkdir avs_build
     cd avs_build
     cmake ../avs-device-sdk -DIOS_DEPLOYMENT_TARGET=10.0 -DCMAKE_TOOLCHAIN_FILE=../ios-cmake/ios.toolchain.cmake -DIOS_PLATFORM=OS -DGSTREAMER_MEDIA_PLAYER=OFF -DCURL_LIBRARY=../Build-OpenSSL-cURL/archive/libcurl-7.54.1-openssl-1.0.2l-nghttp2-1.24.0/libcurl_iOS.a -DCURL_INCLUDE_DIR=../Build-OpenSSL-cURL/curl/curl-7.55.1/include -DGTEST_LIBRARY=../googletest_build/googlemock/libgmock.a -DGTEST_MAIN_LIBRARY=../googletest_build/googlemock/libgmock_main.a -DGTEST_INCLUDE_DIR=../googletest/googletest/include/ -DPKG_CONFIG_EXECUTABLE=/usr/local/bin/pkg-config
     ```  
   * Run these commands to build with the ability to run in the iOS Simulator:  
     ```  
     cd ~/cross-compile
     mkdir avs_build_sim
     cd avs_build_sim
     cmake ../avs-device-sdk -DIOS_DEPLOYMENT_TARGET=10.0 -DCMAKE_TOOLCHAIN_FILE=../ios-cmake/ios.toolchain.cmake -DIOS_PLATFORM=OS -DGSTREAMER_MEDIA_PLAYER=OFF -DCURL_LIBRARY=../Build-OpenSSL-cURL/archive/libcurl-7.54.1-openssl-1.0.2l-nghttp2-1.24.0/libcurl_iOS.a -DCURL_INCLUDE_DIR=../Build-OpenSSL-cURL/curl/curl-7.55.1/include -DGTEST_LIBRARY=../googletest_build/googlemock/libgmock.a -DGTEST_MAIN_LIBRARY=../googletest_build/googlemock/libgmock_main.a -DGTEST_INCLUDE_DIR=../googletest/googletest/include/ -DPKG_CONFIG_EXECUTABLE=/usr/local/bin/pkg-config
     ```  
   * To build what's required for the iOS proof of concept run these commands:  
     ```
     make AuthDelegate
     make DefaultClient
     make KWD
     make PlaylistParser
     ```
   * Make sure that each make command reaches `100%`. 
10. That's it, you're done. The AVS Device SDK has been cross-compiled for iOS.  