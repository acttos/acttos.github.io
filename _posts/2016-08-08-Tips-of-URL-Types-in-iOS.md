---
layout: post
title:  "Tips of URL Types in iOS"
date:   2016-08-08 19:51:18 +0800
categories: iOS
---

### What is URL Types and how to use it ?
---


`URL Types` is a collection of `URL Definition`, these URL Definitions can be used by any apps if needed. `URL Types` make it possible to exchange data amoung different apps.

When we need to develop with third apps to exchange some data, such as using WeChat to open it's OpenAPI to identify an user, we need to define URL Types in our app's Info.plist file by adding URL Types through `Targets -> App -> Info -> URL Types`.

###### Here is an example:

![](/images/url-types/url-types.png)

We can also see the values in our `Info.plist` file:

![](/images/url-types/url-types-in-plist.png)

If we dig deeper, we can see the source code of `Info.plist` which we have added:

![](/images/url-types/url-types-in-plist-code.png)

The key `FBundleURLTypes` stands for URLs our app can handle, it is an array, which give us ablitity to handle more than one URL. The key `CFBundleURLName` stands for a category of URL Types in our app, it can be any words you want, just give it a name which can suitably describle the URLs. For example: we use `Wechat` or `weixin` for URLs which is called by Wechat(微信), we use `Weibo` for URLs which is called by Weibo(微博). But remember: `It can be anything you want to input`.

The key `CFBundleURLSchemes` defines an array which our app can handle. For example: we add a value `OpenDemo` to receive requests from other apps who called `OpenDemo://some-parameters` or `OpenDemo://*****`. `OpenDemo` is a key word such as `http,https,ftp` and so on.

**Remember**:
The key word `OpenDemo` is suggested to be unique from other apps. iOS will ignore the later registeration of URL Types if there is already a same key word `OpenDemo` registered.


### Thanks for visiting.

