This document provides step-by-step instructions to optimize `libcurl` for size in `*nix` systems.

We recommend that you use separate directories to keep track of your optimized builds, especially if you use these libraries elsewhere. For example, if you are building the Alexa Client SDK on a Linux machine that you use daily, installing these optimized versions will most likely break other operations. Therefore, in this document we will assume that all sources will be in `$OPTIMIZED_SOURCE_DIR` and all installations will be in `$OPTIMIZED_TARGET_DIR`.

**Note**: To speed up compilation use the `-jN` option with `make`.


## Optimize libnghttp2

1. [Download the latest release of `libnhttp2`](https://github.com/nghttp2/nghttp2/releases) to `$OPTIMIZED_SOURCE_DIR`. Make sure the version is 1.17 or later.  

2. Extract the contents to `$OPTIMIZED_SOURCE_DIR`:

    ```
    tar xvf nghttp2-*.tar.*
    ```
3.  Change directories to `nghttp2-*` and configure the library to output shared libraries only:

    ```
    cd nghttp2-*
    CFLAGS="-Os" ./configure --enable-lib-only --disable-assert --disable-largefile --prefix=$OPTIMIZED_TARGET_DIR
    ```
4.  Make the library and install it to the prefixed folder:

    ```
    make && make install
    ```

## Optimize libssl

1.  [Download the latest release of `libssl`](https://www.openssl.org/source/) to `$OPTIMIZED_SOURCE_DIR`. Make sure the version is 1.1.0c or later.
2.  Extract the contents to `$OPTIMIZED_SOURCE_DIR`:

 ```
 tar xvf openssl-*.tar.gz
 ```
3.  Change directories to `openssl-*` and configure the library to remove unused cipher algorithms and cryptographic protocols:

    ```
    cd openssl-*
    ./config no-ssl3-method no-tls1-method no-tls1_1-method no-dtls1-method no-dtls1_2-method no-asm no-afalgeng no-capieng no-deprecated no-cms no-ct no-dgram no-dynamic-engine no-engine no-hw no-bf no-blake2 no-camellia no-cast no-chacha no-cmac no-des no-idea no-md4 no-mdc2 no-ocb no-poly1305 no-rc2 no-rc4 no-rmd160 no-scrypt no-seed no-whirlpool --prefix=$OPTIMIZED_TARGET_DIR -Os
    ```
4.  Make the library and install only library components it to the prefixed folder:

    ```
    make && make install_sw
    ```

## Optimize libcurl

1.  [Download the latest release of libcurl](https://curl.haxx.se/download.html) to `$OPTIMIZED_SOURCE_DIR`. Make sure the version is 7.50.2 or later.
2.  Extract the contents to `$OPTIMIZED_SOURCE_DIR`:

    ```
    tar xvf curl-*.tar.*
    ```
3.  Change directories to `curl-*` and configure the library to remove unused communication and cryptographic protocols:

    ```
    cd curl-*
    CFLAGS="-Os" LDFLAGS="-Wl,-R$OPTIMIZED_TARGET_DIR/lib" ./configure --enable-threaded-resolver --with-nghttp2=$OPTIMIZED_TARGET_DIR --with-ssl=$OPTIMIZED_TARGET_DIR --disable-largefile --disable-ftp --disable-file --disable-ldap --disable-ldaps --disable-rtsp --disable-proxy --disable-dict --disable-telnet --disable-tftp --disable-pop3 --disable-imap --disable-smb --disable-smtp --disable-gopher --disable-manual --disable-libcurl-option --disable-ipv6 --disable-unix-sockets --disable-cookies --disable-soname-bump --disable-sspi --disable-versioned-symbols --disable-verbose --disable-crypto-auth --disable-ntlm-wb --disable-tls-srp --prefix=$OPTIMIZED_TARGET_DIR
    ```
4.  Make the library and install it to the prefixed folder:

    ```
    make && make install
    ```
    This only installs the libraries and executables to the folder and skips the documentation files.

At this point, you should have all your libraries under `$OPTIMIZED_TARGET_DIR/lib`.

## Pass Optimized libcurl to the Alexa Client SDK

In this section, we will pass the optimized `libcurl` to the make process. We assume that the source code for Alexa Client SDK is located in `$ACSDK_SOURCE_DIR` and the build directory is `$ACSDK_BUILD_DIR`. 
```
cd $ACSDK_BUILD_DIR
cmake -DCURL_LIBRARY=$OPTIMIZED_TARGET_DIR/lib/libcurl.so -DCURL_INCLUDE_DIR=$OPTIMIZED_TARGET_DIR/include $ACSDK_SOURCE_DIR
```
**Note**: The flags we pass force `make` to use the optimized `libcurl`. Feel free to pass other `cmake` flags that your project requires.  

That is it, you have optimized libcurl for size. Click here to return to [README.md](https://github.com/alexa/alexa-client-sdk).

## Optmiziation Results

This table illustrates the size differences between standard and optimized libraries.

| Library	| Standard | Optimized |
| ------------  | -------- | --------- |
| `libnghttp2`  | 652K     | 156K      |
| `libssl`	| 520K     | 416K      |
| `libcrypto`	| 2.5M     | 1.7M      |
| `libcurl`	| 508K	   | 264K      |

## Release Notes   

| Version | Release Date | Notes |
|---------|--------------|-------|  
| v1.0 | 3/8/2017 | Initial release. |
