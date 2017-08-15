By default libcurl is built with paths to a CA bundle and a directory containing CA certificates.  You can direct the AVS Device SDK for C++ to configure libcurl to use an additional path to directories containing CA certificates via the [CURLOPT_CAPATH](https://curl.haxx.se/libcurl/c/CURLOPT_CAPATH.html) setting.  This is done by adding a `"libcurlUtils/CURLOPT_CAPATH"` entry to the `AlexaClientSDKConfig.json` file.  Here is an example:

```json
{
  "authDelegate" : {
    "clientId" : "INSERT_YOUR_CLIENT_ID_HERE",
    "refreshToken" : "INSERT_YOUR_REFRESH_TOKEN_HERE",
    "clientSecret" : "INSERT_YOUR_CLIENT_SECRET_HERE"
  },
  "libcurlUtils" : {
    "CURLOPT_CAPATH" : "INSERT_YOUR_CA_CERTIFICATE_PATH_HERE"
  }
}
```
**Note** If you want to assure that libcurl is *only* using CA certificates from this path you may need to reconfigure libcurl with the `--without-ca-bundle` and `--without-ca-path` options and rebuild it to suppress the default paths.  See [The libcurl documention](https://curl.haxx.se/docs/sslcerts.html) for more information.
