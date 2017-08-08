## How to build libcurl with nghttp2 for macOS

Building for macOS requires some additional setup. Specifically, you need to ensure that you are running the latest version of cURL and that cURL is linked to nghttp2 (the default installation does not).

To recompile cURL, follow these instructions:

1. Install [Homebrew](http://brew.sh/), if you haven't done so already.
2. Install cURL with HTTP2 support:
`brew install curl --with-nghttp2`
3. Force cURL to explicitly link to the updated binary:
`brew link curl --force`
4. Close and reopen terminal.
5. Confirm version and source with this command:
`brew info curl`
