---
layout: post
title:  "Acquire Microphone and Camera authorization"
date:   2016-08-24 21:34:23 +0800
categories: iOS
---
### 1. Microphone and Camera in iPhone devices

Recently, we need to develop an app with video and sound in iOS.
It is the first time I have been play with AVFoundation Framework, so I surelly get some problems. I want talk about one of them here to avoid the second time mistake.

In iOS, we use Microphone to get the voice, use Camera to capture the pictures or videos. Also, we know iOS is a safer OS than any others in the market. Most of the Hardwares such as Microphone, Camera, Bluetooth, Accelerometer require an authorization to access, that is the way Apple protecting people's privacy. So, we need to acquire a permission in our 'Microphone and Camera app'.

### 2. Steps for acquiring authorization

`AVFoundation Framework` is required for `Audio or Video` development. So firt of all, we import AVFoundation Framework in our class, and here is the header of my class, it includes AVFoundation:

```
import UIKit
import Foundation
import AVFoundation
```

After that, we can use the classes in AVFoundation, and we can acquire authorizations we need.

Let's begin with a standalone function called `_requestForMediaAuthorization(type:String, callback:AuthorizeResultBlock)`, here is the whole func:

```
func _requestForMediaAuthorization(type: String, callback:AuthorizeResultBlock) -> Void {
    AVCaptureDevice.requestAccessForMediaType(type) { (grant) in
        if (grant) {
            NSLog("Authorize sucessfully with type:\(type)");
            callback(succeed: true);
        } else {
            NSLog("Failed to get authorization with type:\(type)");
            callback(succeed: false);
        }
    }
}
```

We define a function has two parameters: We accept AVMediaType to decide which device authorization is required, and accept a closure to pass the result waiting for the receiver to handle.

Here is the definition of the closure:

```
typealias AuthorizeResultBlock = @convention(block) (succeed:Bool) -> Void;
```
The closure has only one parameter `succeed` to identify the result of accquiring a certain type authorization of device. If `succeed` is `true`, it means we succefully acquired the authorization, otherwise failed.

In function `_requestForMediaAuthorization(type:String, callback:AuthorizeResultBlock)`, we use the following definition to do the real thing for acquiring:

```
class func requestAccessForMediaType(mediaType: String!, completionHandler handler: ((Bool) -> Void)!)
```
This function is a class function, which means we can directly call it with dot `.` appending with the class name. It is just like `AVCaptureDevice.requestAccessForMediaType(mediaType,completionBlock)`.

Basically that is the way we acquiring a permission. However, if we failed to do that, we still need to give a notice that why we can not continue.

### 3. The failure notice view

Here is the notice handler function with a notice view show up:

```
func _showAuthorizationFailureNotice(type: String) -> Void {
    dispatch_async(dispatch_get_main_queue()) {
        var tips: String?;
        let privacyItem: String = type == AVMediaTypeAudio ? "MICROPHONE" : type == AVMediaTypeVideo ? "CAMERA" : "";

        self.settingsItemURL = NSURL(string: "prefs:root=Privacy&path=\(privacyItem)");

        if (type == AVMediaTypeAudio) {
            tips = "Please allow us to use your Microphone to capture audio by opening 'Settings -> Privacy -> Microphone' to grant.";
        } else if (type == AVMediaTypeVideo) {
            tips = "Please allow us to use your Camera to capture video by opening 'Settings -> Privacy -> Camera' to grant.";
        } else {
            tips = "Please allow us to use your device by opening 'Settings -> Privacy' to grant certain authorization.";
        }

        let compareResult:NSComparisonResult = UIDevice.currentDevice().systemVersion.compare("8.0");

        if (compareResult == NSComparisonResult.OrderedSame || compareResult == NSComparisonResult.OrderedDescending) {
            let alertController: UIAlertController = UIAlertController(title: "Authorization Request", message: tips, preferredStyle: UIAlertControllerStyle.Alert);
            let confirmAction: UIAlertAction = UIAlertAction(title: "OK", style: UIAlertActionStyle.Default, handler: { (action) in
                if(UIApplication.sharedApplication().canOpenURL(self.settingsItemURL!)) {
                    UIApplication.sharedApplication().openURL(self.settingsItemURL!);
                }
            });

            let cancelAction: UIAlertAction = UIAlertAction(title: "Cancel", style: UIAlertActionStyle.Cancel, handler: nil);

            alertController.addAction(confirmAction);
            alertController.addAction(cancelAction);
            self.presentViewController(alertController, animated: true, completion: {

            });
        } else {
            let alertView: UIAlertView = UIAlertView(title: "Authorization Request", message: tips!, delegate: self, cancelButtonTitle: "Cancel", otherButtonTitles: "OK");
            alertView.show();
        }
    };
}
```
Let's go deeper into the codes above.

1. The Accquiring action is processed in the background thread by iOS, so if we want to display a failure notice view on the screen, we need to switch to the main thread.

2. First line in the statement is a `GCD` keyword `dispatch_async`, with `dispatch_get_main_queue()` we move from the background thread to the main thread.

3. For the failure notice view, we simply set a message and a button for the user. We set the value of parameter `tips` on condition of the value of `type`, because different types of accquring bring us different way to handle.

4. After we set the value of `tips`, we are ready to show it on the screen. And here we have two options:`UIAlertView` and `UIAlertController`. `UIAlertView` is the common option for the alert notice view, but from iOS version 8.0, `UIAlertView` is marked as `Deprecated`, so it is necessary to add `UIAlertController` and `UIAlertAction`s to adapt different version of iOS.

5. For opening a certain settings page, [here is a reference](/2016/08/Open-A-Settings-Page-in-iOS/){:target="_blank"}, you can find how to open a settings page.

6. Remember, `UIAlertView` needs a delegate to handle the button actions on it, so we write a simple function implementation.

```
//MARK: - UIAlertView Delegate Methods
func alertView(alertView: UIAlertView, clickedButtonAtIndex buttonIndex: Int) {
    if (buttonIndex == 1) {
        if(UIApplication.sharedApplication().canOpenURL(settingsItemURL!)) {
            UIApplication.sharedApplication().openURL(settingsItemURL!);
        }
    }
}
```

### 4. Source code

##### You can find the source code [[ _Here_ ]](https://github.com/acttos/Swift-Acttos){:target="_blank"} and you are welcomed to give suggestions.
_The codes may be updated from time to time._

Thanks!
