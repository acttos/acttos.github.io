---
layout: post
title:  "Open a settings page in iOS"
date:   2016-08-12 20:12:23 +0800
categories: iOS
---

### 1. General opening method

Sometime, we need to access data may contains privacy(such as:camera,microphone,photo and so on) in user's mobile device, for that, iOS requires us to acquire an authorization from the device user. We can show up an AlertView for the request, and what we can do more is `Open a certain settings page` for user. Let's figure it out.

##### Common opening URL in iOS:

```oc
if ([[UIApplication sharedApplication] canOpenURL:url]) {
    [[UIApplication sharedApplication] openURL:url];
}
```
Basically, we can open any legal URLs in iOS. For settings, we use a pattern `prefs:root=RootName&path=PathName`, the RootName and PathName can be found in paragraph 2 below.

### 2. URLs of settings in iOS

```oc
Camera - prefs:root=Privacy&path=CAMERA;
Microphone - prefs:root=Privacy&path=MICROPHONE;
About — prefs:root=General&path=About;
Accessibility — prefs:root=General&path=ACCESSIBILITY;
Airplane Mode On — prefs:root=AIRPLANE_MODE;
Auto-Lock — prefs:root=General&path=AUTOLOCK;
Brightness — prefs:root=Brightness;
Bluetooth — prefs:root=General&path=Bluetooth;
Date & Time — prefs:root=General&path=DATE_AND_TIME;
FaceTime — prefs:root=FACETIME;
General — prefs:root=General;
Keyboard — prefs:root=General&path=Keyboard;
iCloud — prefs:root=CASTLE;
iCloud Storage & Backup — prefs:root=CASTLE&path=STORAGE_AND_BACKUP;
International — prefs:root=General&path=INTERNATIONAL;
Location Services — prefs:root=LOCATION_SERVICES;
Music — prefs:root=MUSIC;
Music Equalizer — prefs:root=MUSIC&path=EQ;
Music Volume Limit — prefs:root=MUSIC&path=VolumeLimit;
Network — prefs:root=General&path=Network;
Nike + iPod — prefs:root=NIKE_PLUS_IPOD;
Notes — prefs:root=NOTES;
Notification — prefs:root=NOTIFICATIONS_ID;
Phone — prefs:root=Phone;
Photos — prefs:root=Photos;
Profile — prefs:root=General&path=ManagedConfigurationList;
Reset — prefs:root=General&path=Reset;
Safari — prefs:root=Safari;
Siri — prefs:root=General&path=Assistant;
Sounds — prefs:root=Sounds;
Software Update — prefs:root=General&path=SOFTWARE_UPDATE_LINK;
Store — prefs:root=STORE;
Twitter — prefs:root=TWITTER;
Usage — prefs:root=General&path=USAGE;
VPN — prefs:root=General&path=Network/VPN;
Wallpaper — prefs:root=Wallpaper;
Wi-Fi — prefs:root=WIFI;
```

### 3. AlertView in iOS

According to the document of Apple, we find that the `UIAlertView` is deprecated in `iOS 9.0 and later`, for safety we show the AlertView with different ways due to certain systemVersions:

```oc
if ([[[UIDevice currentDevice] systemVersion] floatValue] >= 8.0) {
    UIAlertController* alert = [UIAlertController alertControllerWithTitle:@"Authorization Request"
                                                                  message:requestMsg
                                                           preferredStyle:UIAlertControllerStyleAlert];
    UIAlertAction* defaultAction = [UIAlertAction actionWithTitle:@"OK"
                                                            style:UIAlertActionStyleDefault
                                                          handler:^(UIAlertAction * action) {
                                                                [self _openURL:_settingsItemURL];
                                                          }];

    [alert addAction:defaultAction];
    [[self _getPresentedViewController] presentViewController:alert animated:YES completion:nil];
} else {
    UIAlertView *alertView = [[UIAlertView alloc] initWithTitle:@"Authorization Request" message:tips delegate:self cancelButtonTitle:nil otherButtonTitles:@"OK", nil];
    [alertView show];
}
```
`UIAlertController` is only available since iOS 8.0, so we set the condition of `if` statement is `systemVersion > 8.0`.
