# Brightcove iOS 1

## Steps to Reproduce

1. Clone Repository
2. Navigate to Root of Offline Sample Project for Swift
3. Run `pod install` to Install Dependencies
4. Build and Run Application on Physical Device
5. Download Videos 1 and 2
6. Wait for Downloads to Complete
7. Download Video 3
8. Push Application to Background Before Download of Video 3 Completes
9. Force Quit Application
10. Build and Run Application on Physical Device
11. Download Video 4

## Diagnostic Information

This issue has been reproduced using the latest version of the Brightcove SDK (**6.3.11**) on the following devices:
- **iPad Pro** running **iOS 12.1**
- **iPhone 7** running **iOS 12.1.2**
- **iPhone 6** running **iOS 11.4**

## Issue

The sudden termination of the application results in several issues. The focus of this case is the inability to request the video download status of any download (pending, paused, completed, cancelled, ...) when a download has been interrupted due to the sudden termination of the application.

The problem is clearly visible when the user launches the application after force quitting it (step 10). The application indicates that videos 1, 2, and 3 have not been downloaded. The problem becomes evident when the download of video 4 is initiated (step 11). A few print statements have been added to the `offlineVideoToken(_:downloadTask:didProgressTo:)` method, a method defined by the `BCOVOfflineVideoManagerDelegate` protocol. Every time the Brightcove SDK invokes this method, the `DownloadManager` instance asks the Brightcove SDK for the video download status of each download and prints it to the console.

```
func offlineVideoToken(_ offlineVideoToken: String?, downloadTask: AVAssetDownloadTask?, didProgressTo progressPercent: TimeInterval) {
		// This delegate method reports progress for the primary video download
		let percentString = String(format: "%0.2f", progressPercent)
		print("Offline download didProgressTo: \(percentString) for token: \(offlineVideoToken!)")

		DispatchQueue.main.async {
				NotificationCenter.default.post(name: OfflinePlayerNotifications.UpdateStatus, object: nil)
				let downloadsVC = AppDelegate.current().tabBarController.downloadsViewController()
				downloadsVC?.updateInfoForSelectedDownload()
				downloadsVC?.refresh()
		}

		print("++++++++++++++ LIST OFFLINE VIDEO TOKENS ++++++++++++++")
		for offlineVideoToken in BCOVOfflineVideoManager.shared()!.offlineVideoTokens {
				if let _ = BCOVOfflineVideoManager.shared()!.offlineVideoStatus(forToken: offlineVideoToken) {
						print("✅ OFFLINE VIDEO STATUS PRESENT FOR \(offlineVideoToken)")
				} else {
						print("🚨 NO OFFLINE VIDEO STATUS PRESENT FOR \(offlineVideoToken)")
				}
		}
		print("+++++++++++++++++++++++++++++++++++++++++++++++++++++++")
}
```

The Brightcove SDK is only able to return the video download status of downloads that are initiated after the sudden termination of the application. Even though videos 1 and 2 have been successfully downloaded by the application, the Brightcove SDK is unable to return the video download status of these downloads.

```
Offline download didProgressTo: 85.36 for token: 1E658EC7-867D-4153-8679-A02C75274E07
++++++++++++++ LIST OFFLINE VIDEO TOKENS ++++++++++++++
🚨 NO OFFLINE VIDEO STATUS PRESENT FOR 537F6B03-7D0C-4E3F-9FA4-7D8232C182CC
🚨 NO OFFLINE VIDEO STATUS PRESENT FOR 02562534-E78A-409D-B8AA-BAF31133C58B
🚨 NO OFFLINE VIDEO STATUS PRESENT FOR 5F033E40-0395-4D72-AE4E-16A8FE6F6E64
✅ OFFLINE VIDEO STATUS PRESENT FOR 39E59A1C-C0AB-4708-A911-077CEE0B615A
+++++++++++++++++++++++++++++++++++++++++++++++++++++++
```

### Consequences

Because of this issue, it's impossible for the application to know what the video download status of a download is. This means that the application isn't able to show the user the status of each download. It also doesn't know whether a video is available for offline playback. Having the ability to request the video download status of a download is essential for video downloads.

### Other Findings

If you browse the application's container, then you find the property list that includes the status of the interrupted download. This is what it looks like. It shows that the download wasn't completed successfully due to an error. The reason of the failure isn't specified.

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
