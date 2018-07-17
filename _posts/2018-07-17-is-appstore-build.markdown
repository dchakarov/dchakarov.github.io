---
layout: post
title:  "Am I Running On A User Device?"
date:   2018-07-17
---

<span class="dropcap">A</span> common problem every iOS developer has to solve is finding out whether the app is being executed on a user device (via AppStore), a test device (via TestFlight) or a dev device (via Xcode). This is useful for deciding the level of logging, showing and hiding "admin" features, and connecting to the right backend environment.

After a few backs and forths the code I came up with for testing that is we came up with a solution which gave us a simple `isLive` boolean which we use throughout the app. Here it goes:

```swift
var isLive: Bool {
	if Device().isSimulator { return false }
	let isTestFlight = 	Bundle.main.appStoreReceiptURL?.lastPathComponent == "sandboxReceipt" // see https://stackoverflow.com/a/26113597/67667
	let isDebugConfiguration = _isDebugAssertConfiguration() // see https://stackoverflow.com/a/34532569/67667
	return !isDebugConfiguration && !isTestFlight
}
```

You can see in the comments where we got our inspiration. As for the `Device().isSimulator` part - we are using the wonderful [DeviceKit framework](https://github.com/dennisweissmann/DeviceKit).

Hope you find this helpful! If you have any suggestions for improving it, please ping me on Twitter @gimly.