---
layout: post
title:  "What does \"File's Owner\" mean?"
date:   2016-08-09 21:21:07 +0800
categories: iOS
---

### 1. What is "File's Owner" ?

`File's Owner`, the owner of a file, that is the direct meaning of the words. Every xib file in Xcode has a `File's Owner`. Most of the time,we just ignore the settings, and it works well. That is why we did not pay enough attention to it. When there is an error occured, we do not know what is happening and why.

##### An error possibly caused by `File's Owner`:

![](/images/201608-files-owner/files-owner-issue.png)

We searched the possible reasons associated with this error, and found most of the results are about `File's Owner`, so we checked the codes and the settings in xib file and found:

##### The codes in Xcode:

![](/images/201608-files-owner/files-owner-issue-create-from-nib.png)

##### The settings of `File's Owner` in xib file:

![](/images/201608-files-owner/files-owner-issue-nib-setting.png)

<br/>
The method `loadNibNamed:owner:options` will unarchive the contents of a nib file in the bundle and return an array containing the top-level objects in the nib file. All the objects in the array are instantiated when the nib file is unarchived. But the objects do NOT contain references to `File's Owner` object or any proxy objects.
<br/>
Now, Let's pay more attention to the param `owner`, the owner here means *The object to assign as the nib's `File's Owner` object*. What is that? I can not understand the meaning of *nib's `File's Owner` object*. Can the object be the Class bound with the nib file? Just the same as the class of the current view? I don't know ...

So,I begin to dig deeper to find what `File's Owner` really is and what exactly it can do.

### 2. File's Owner in Apple's document

I checked the documents of `File's Owner` in Apple official document, and here is the screenshot:

##### File's Owner in Apple official documents:

![](/images/201608-files-owner/apples-files-owner.png)

We can see that `File's Owner` object is one of the most important objects in a nib file, and `File's Owner` is a placeholder object, and is NOT created when the nib file is loaded. `File's Owner` object must be created by yourself and you need to pass it to the nib-loading code to recreate the link of the code and the contents of a nib file. We can tell the `File's Owner` object can be `NSObject` in the Xcode and when you link your UI elements with your code (most of the time it is the ViewController class), the `File's Owner` object is specified (and it is your ViewController instance).

### 3. What does "File's Owner" do in XIB?

From Apple's official documents we know `File's Owner` object plays a role of 'port' between your UI elements in xib file and the codes which you designed to control the UI elements. When the xib file is loaded by methods like `loadNibNamed:owner:options`, we should pass the `File's Owner` object to these methods, and these methods will recreate the connections you designed in *Interface Builder*. After the connections are established, your objects in the codes can reference the UI elements object in the xib file and you can control the UI elements through your codes.

### 4. What should we do using "File's Owner" ?

So far as we know, We just leave the value of `File's Owner` to be `NSObject` in the settings of `Identity Inspector` in Xcode, when the `Referencing Outlets` connections are established the `File's Owner` is automatically specified.

##### The default value of `File's Owner` in Xcode:

![](/images/201608-files-owner/files-owner-issue-nib-setting.png)

##### The value of the root view in xib file is specified to be an existed class name:

![](/images/201608-files-owner/files-owner-issue-nib-setting-2.png)

<br/>

### 5. At last:

> The best help documents are Apple's official programming guide, if we really read them earnestly, we will not get stuck so much.

<br/>

### Thanks for visiting.
