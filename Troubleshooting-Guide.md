# Troubleshooting
Having trouble with the AVS Device SDK? Here's a list of workarounds for common issues reported by AVS developers.

Let us know what we can do to help you by creating a [new issue](https://github.com/alexa/avs-device-sdk/issues/new) when you run into a problem.

## All Platforms

| Issue | Workaround/Resolution |
|--------|--------------|
| Playback of Alexa TTS is truncated |  If you are using the `GStreamer-based` `MediaPlayer`, change `g_object_set(m_pipeline.audioSink, "sync", FALSE, NULL);` to `TRUE` on line [671](https://github.com/alexa/avs-device-sdk/blob/master/MediaPlayer/src/MediaPlayer.cpp#L671). Alternatively, change the sleep duration `static const std::chrono::milliseconds SLEEP_AFTER_END_OF_AUDIO{300};` on line [93](https://github.com/alexa/avs-device-sdk/blob/master/MediaPlayer/src/MediaPlayer.cpp#L93). Audio timeline synchronization is turned off to account for poor MPEG-TS audio packet timestamping, so you will need to adjust one or more of these values. |
| The SDK fails to load | The `AlexaClientSDKConfig.json` must reference database files or the SDK will fail to load. |
| No audio output from the SDK | The SDK uses the `autoaudiosink` element, which automatically selects the best sink to use based on your system's configuration. **Step 1).** Run this command: `gst-launch-1.0 -m audiotestsrc ! autoaudiosink`. If the system *is* working, you will hear a test tone. If the system *is not* working, you will hear no test tone. `-m` will help you determine which sink is selected, and allow you to troubleshoot that sink. Example: `autoaudiosink0-actual-sink-osxaudio" (tag): GstMessageTag, taglist=(taglist)"taglist\,\ description\=\(string\)\"audiotest\\\ wave\"\;;`. In this example, the log suggests that `osxaudio` is used.  **Step 2).** Run this command `gst-launch-1.0 -m audiotestsrc ! alsasink`. If you hear a test tone, modify the value on line [599](https://github.com/alexa/avs-device-sdk/blob/master/MediaPlayer/src/MediaPlayer.cpp#L599) from `m_pipeline.audioSink = gst_element_factory_make("autoaudiosink", "audio_sink");` to `m_pipeline.audioSink = gst_element_factory_make("alsasink", "audio_sink");`.|
| The device fails to play Amazon Music, and the following error message is generated: `curlEasyPerformFailed:error=Peer certificate cannot be authenticated with given CA certificates` | If the device doesn't have the correct date you will experience certificate issues when contacting Amazon servers, so you'll need to change the date. To change the date (for example, to Jan 31, 2018): `date +%Y%m%d -s "20180131"`. To change the time: `date +%T -s "16:11:00"`. |


## Linux

| Issue | Workaround/Resolution |
|--------|--------------|
| No sound - Plantronix Speakerbox | `sudo apt install gstreamer1.0-alsa`. Additionally, there is some configuration work that needs to be done. See [Issue 212](https://github.com/alexa/avs-device-sdk/issues/212) for details. |
| Can't play more than one sound simultaneously | Install the [`dmix plugin`](https://alsa.opensrc.org/Dmix). See [Issue 415](https://github.com/alexa/avs-device-sdk/issues/415) for details.  |


## macOS

| Issue | Workaround/Resolution |
|--------|--------------|
| The SDK fails to build because of missing dependencies | Sometimes, the `brew install` command that is run during step [**1.2.3**](https://github.com/alexa/avs-device-sdk/wiki/macOS-Quick-Start-Guide#12-install-dependencies) will fail if a dependency is already installed. To get around this issue, run a `brew install` command for each dependency individually. |
| SampleApp fails to build because your version of curl doesn't support HTTP/2 | It's possible that curl didn't link correctly. To fix this issue, run `brew uninstall curl`, then repeat the steps in [**1.2.2**](https://github.com/alexa/avs-device-sdk/wiki/macOS-Quick-Start-Guide#12-install-dependencies). |
| AuthServer.py throws this error: `File "AuthServer/AuthServer.py", line 67 print 'The file "' + \ SyntaxError: Missing parentheses in call to 'print'` | This is a known issue with Python 3.x. The workaround is to use Python 2.7.x. |

## RaspberryPi

| Issue | Workaround/Resolution |
|--------|--------------|
| No USB soundcard option | The SampleApp uses the default input and output for audio. To use a USB soundcard, install the `dsnoop` and `dmix` plugins. Paste [this code](https://github.com/shivasiddharth/Assistants-Pi/blob/master/audio-drivers/USB-DAC/scripts/asound.conf) into `.asoundrc`, and into `asound.conf`. |

## Windows

| Issue | Workaround/Resolution |
|--------|--------------|
| The system can't locate `libwinpthread-1.dll` when running the sample app | This means that Windows can't find the library either because it's not installed, or it's not in the path environment variable. To troubleshoot, verify that the library exists, and that it is nested within the proper file path, such as `c:\msys64\mingw64\bin`. |
| The sample app hangs at startup, and ~35% of CPU is being used. | This likely means that Pacman isn't updated. Follow the steps **1.1.2** and **1.1.3** above. |
