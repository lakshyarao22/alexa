Occasionally, you may encounter issues related to unit and integration tests provided with the SDK. To understand your issue and to pinpoint where the failure has occurred, please follow these instructions:

**NOTE**: These instructions assume that you are working at the top level of the build directory, where you run `make test` and `make integration`.

## Integration Tests

The integration tests rely on your platform's internet connection and the server's availability. Therefore, the failures you might see during integration tests might be intermittent. The general approach for integration tests should be to isolate the test bench (rf Step 1 below) and repeat the test bench using the `--gtest_repeat=n` flag with at least 10 tries. If the failure is persistent, you can also filter the test (rf Step 3 below) and repeat with at least 10 tries. The main difference between the unit and integration tests is that integration tests are not included in the CTest command. Therefore, you should pass the correct parameters to the executable to run a single test bench. There are seven integration tests, the following commands are used to run these tests:

* `./Integration/test/AlexaAuthorizationDelegateTest /path/to/sdk/config/AlexaClientSDKConfig.json `
* `./Integration/test/AlexaCommunicationsLibraryTest /path/to/sdk/config/AlexaClientSDKConfig.json /path/to/sdk/source/Integration/inputs`
* `./Integration/test/AlexaDirectiveSequencerLibraryTest /path/to/sdk/config/AlexaClientSDKConfig.json /path/to/sdk/source/Integration/inputs`
* `./Integration/test/AudioInputProcessorIntegrationTest /path/to/sdk/config/AlexaClientSDKConfig.json /path/to/sdk/source/Integration/inputs`
* `./Integration/test/SpeechSynthesizerIntegrationTest /path/to/sdk/config/AlexaClientSDKConfig.json /path/to/sdk/source/Integration/inputs`
* `./Integration/test/AlertsIntegrationTest /path/to/sdk/config/AlexaClientSDKConfig.json /path/to/sdk/source/Integration/inputs`
* `./Integration/test/AudioPlayerIntegrationTest /path/to/sdk/config/AlexaClientSDKConfig.json /path/to/sdk/source/Integration/inputs`

Once you have used `--gtest_filter`, you can include that output to the issue report.

## Unit Tests  

### How to Find Which Test is Responsible

Using the build instructions, you may encounter failures during the `make test` step. Here we summarize what to do and how to re-run just the test that had failed:  

1. **Find the test bench and the individual tests that failed:** When a test fails, CTest or GoogleTest will print the name of the test that has failed at the end of the test run, e.g. `make test` will print something like this if there's a failed test:  
    ```
    The following tests FAILED:
            364 - MediaPlayerTest_test (Failed)
            365 - MediaPlayerTest.testStartPlayWaitForEnd (Failed)
            366 - MediaPlayerTest.testStartPlayForUrl (Failed)
    ```
   In this example, we see that the main test bench that caused the test failure is `MediaPlayerTest` since that's the one that ends with `_test` and each individual test printed below it is prefixed by `MediaPlayerTest.`. Also, the two failed tests are `MediaPlayerTest.testStartPlayWaitForEnd` and `MediaPlayerTest.testStartPlayForUrl`.  
2. **Run the test bench:** The tests rely on your platform's internet connection and the server's availability. Therefore, the failures you might see during these tests might be intermittent. Therefore, running the test bench on its own before reporting it will show the nature of the failure. Continuing with the same example, let's run `MediaPlayerTest`, located in `MediaPlayer/test/` (you can find any failed test in the build tree by `find . -name <test_name_here>`, e.g. `find . -name MediaPlayerTest`). In this case, if you run:   
    ```
    ctest -R "^MediaPlayerTest_test$" -V
    ```
    It prints a different output, which is followed by a block that lists the failed tests again, e.g.:  
    ```
    [==========] 16 tests from 1 test case ran. (***** ms total)
    [  PASSED  ] 14 tests.
    [  FAILED  ] 2 tests, listed below:
    [  FAILED  ] MediaPlayerTest.testStartPlayWaitForEnd
    [  FAILED  ] MediaPlayerTest.testStartPlayForUrl
    ```
    In this example, we see that the same two tests have failed again.
    
3. **Run only the failed tests and report:** Now that we are sure which tests failed, we can run this test:
    ```
    ctest -R "^MediaPlayerTest.testStartPlayWaitForEnd$|^MediaPlayerTest.testStartPlayForUrl$" -V
    ```
    Here, notice that the tests are separated by `|`. Now the output is much smaller, and contains all the info needed for the run: 
    
    ```
    Note: Google Test filter = MediaPlayerTest.testStartPlayWaitForEnd:MediaPlayerTest.testStartPlayForUrl
    [==========] Running 2 tests from 1 test case.
    [----------] Global test environment set-up.
    [----------] 2 tests from MediaPlayerTest
    [ RUN      ] MediaPlayerTest.testStartPlayWaitForEnd
    2017-08-07 15:58:59.187 [  1] E MediaPlayer:setupPipelineFailed:reason=createConverterElementFailed
    2017-08-07 15:58:59.187 [  1] E MediaPlayer:handleSetAttachmentReaderSourceFailed:reason=setupPipelineFailed
    /path/to/SDK/source/MediaPlayer/test/MediaPlayerTest.cpp:398: Failure
    Expected: (MediaPlayerStatus::FAILURE) != (m_mediaPlayer->setSource( std::unique_ptr<AttachmentReader>(new MockAttachmentReader(iterations, receiveSizes)))), actual: 4-byte object <02-00 00-00> vs 4-byte object <02-00 00-00>
    2017-08-07 15:58:59.188 [  2] E MediaPlayer:playFailed:reason=sourceNotSet
    /path/to/SDK/source/MediaPlayer/test/MediaPlayerTest.cpp:413: Failure
    Expected: (MediaPlayerStatus::FAILURE) != (m_mediaPlayer->play()), actual: 4-byte object <02-00 00-00> vs 4-byte object <02-00 00-00>

    (MediaPlayerTest:970): GStreamer-CRITICAL **: gst_element_get_state: assertion 'GST_IS_ELEMENT (element)' failed
    2017-08-07 15:58:59.188 [  1] E MediaPlayer:doStopFailed:reason=gstElementGetStateFailed

    (MediaPlayerTest:970): GStreamer-CRITICAL **: gst_object_unref: assertion 'object != NULL' failed

    (MediaPlayerTest:970): GLib-CRITICAL **: g_source_remove: assertion 'tag > 0' failed
    [  FAILED  ] MediaPlayerTest.testStartPlayWaitForEnd (7 ms)
    [ RUN      ] MediaPlayerTest.testStartPlayForUrl
    2017-08-07 15:58:59.188 [  3] E MediaPlayer:setupPipelineFailed:reason=createConverterElementFailed
    2017-08-07 15:58:59.188 [  3] E MediaPlayer:handleSetSourceForUrlFailed:reason=setupPipelineFailed
    2017-08-07 15:58:59.188 [  2] E MediaPlayer:playFailed:reason=sourceNotSet
   /path/to/SDK/source/MediaPlayer/test/MediaPlayerTest.cpp:427: Failure
    Expected: (MediaPlayerStatus::FAILURE) != (m_mediaPlayer->play()), actual: 4-byte object <02-00 00-00> vs 4-byte object <02-00 00-00>

    (MediaPlayerTest:970): GStreamer-CRITICAL **: gst_element_get_state: assertion 'GST_IS_ELEMENT (element)' failed
    2017-08-07 15:58:59.188 [  4] E MediaPlayer:doStopFailed:reason=gstElementGetStateFailed

    (MediaPlayerTest:970): GStreamer-CRITICAL **: gst_object_unref: assertion 'object != NULL' failed

    (MediaPlayerTest:970): GLib-CRITICAL **: Source ID 100 was not found when attempting to remove it
    [  FAILED  ] MediaPlayerTest.testStartPlayForUrl (0 ms)
    [----------] 2 tests from MediaPlayerTest (7 ms total)

    [----------] Global test environment tear-down
    [==========] 2 tests from 1 test case ran. (7 ms total)
    [  PASSED  ] 0 tests.
    [  FAILED  ] 2 tests, listed below:
    [  FAILED  ] MediaPlayerTest.testStartPlayWaitForEnd
    [  FAILED  ] MediaPlayerTest.testStartPlayForUrl

     2 FAILED TESTS
    ```
    
    Before you report the failure, it's important that you observe how frequently these tests fail. You can have the tests repeat themselves by passing the ` --repeat-until-fail n` flag where `n` is the number of tries. If you observe the failures intermittently, then please note that in your issue report after attaching the final printout. For example:  
        
    **Note:** If you have `ACSDK_EMIT_SENSITIVE_LOGS` flags enabled and your printout contains sensitive information like user name, client ID, or refresh token, please remove those from the logs before submitting an issue.