# Brightcove iOS 1

## Steps to Reproduce

- Clone Repository
- Navigate to Root of Offline Sample Project for Swift
- Run `pod install` to Install Dependencies
- Build and Run Application on Physical Device
- Download First Video
- Force Quit Application Before Download Ends
- Build and Run Application on Physical Device
- Download First Video

## Issues

The Brightcove SDK finds a downloaded video in the application's sandbox and assumes that the video the user selected for download has already been downloaded. This isn't true. Trying to play the downloaded video is unsuccessful since the video is only partially downloaded. A few print statements are added to the sample project to illustrate the underlying issues. The print statements are located in the `videoAlreadyProcessing(_:)` method of the `DownloadManager` class.

### Issue 1

The Brightcove SDK is unable to provide the status of the download. A print statement is added to the `videoAlreadyProcessing(_:)` method of the `DownloadManager` class to illustrate this (lines 195-199). The `offlineVideoStatus()` method of the `DownloadManager` class returns an empty array even though the application's container contains the status of the download that was interrupted.

### Issue 2

The status of the download is incorrect. On lines 189-193, the application lists the offline video tokens the Brightcove SDK is aware of. Even though it includes the offline video token of the interrupted download, requesting the status for the offline video token of the download is unsuccessful. The `offlineVideoStatus(forToken:)` method returns `nil` if the offline video token of the interrupted download is passed in as an argument.

If you browse the application's container, then you find the property list that includes the status of the interrupted downloaded. This is what it looks like. It shows that the download wasn't completed successfully due to an error. The reason of the failure isn't specified.

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>concurrentDownload</key>
	<false/>
	<key>downloadEndTime</key>
	<real>565874300.94870901</real>
	<key>downloadPercent</key>
	<real>0.0</real>
	<key>downloadStartTime</key>
	<real>565874276.75664103</real>
	<key>downloadState</key>
	<integer>5</integer>
	<key>error_code</key>
	<integer>69302</integer>
	<key>error_domain</key>
	<string>kBCOVOfflineVideoManagerErrorDomain</string>
	<key>error_failureReason</key>
	<string></string>
	<key>error_localizedDescription</key>
	<string>The video could not be downloaded</string>
	<key>error_recoverySuggestion</key>
	<string></string>
	<key>licenseRequestTime</key>
	<real>565874276.75664103</real>
	<key>networkInterfaceChanged</key>
	<false/>
	<key>offlineVideoToken</key>
	<string>1AA03DF5-513B-4565-94D0-53C61FC43D07</string>
	<key>pausedAtLeastOnce</key>
	<false/>
	<key>reportedTracksCompletionToCaller</key>
	<false/>
	<key>reportedVideoCompletionToCaller</key>
	<true/>
	<key>sentToBackground</key>
	<true/>
</dict>
</plist>
```

The folder also contains other information, such as the properties of the video, the poster and thumbnail images of the video, and the parameters of the download request.

## Problem

When the user force quits the application or the application is abruptly terminated by the system of due to a fatal issue, any downloads that are in progress can end up in an unknown state. The Brigthcove SDK currently doesn't return the correct status of such downloads. In addition, the Brightcove SDK provides incorrect information regarding the status of the download. The offline video manager returns a valid `BCOVVideo` instance if the offline video token of the problematic download is given, but it isn't able to return the status of the download.

## Conclusion

It's currently not possible to obtain the true status of a download that has been interrupted in the way described earlier using the API of the Brightcove SDK.
