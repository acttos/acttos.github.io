---
layout: post
title:  "Common usage of UIWebView"
date:   2016-10-08 19:10:39 +0800
categories: iOS
---

## 1.What is UIWebView?

`UIWebView` is a UI kit from UIKit. ^_^ We can show a web page in our apps without opening `Safari` browser in iOS. Most of the apps do not need a web page to show the extra info, sometimes, we want to display some ADs, or we need to show a recommend product list or something else, and we may want to change the contents of the page without publishing a new version of app. Now, we need a web server in remote and an `UIWebView` in our apps.

## 2.How to use UIWebView?

In iOS, we can just drag an `UIWebView` from the `Object library` in the right-bottom of Xcode. Drag an `IBOutlet` from the storyboard or xib to the controller. That is simple and clear, I will not spend much time on introducing it. Now let's meet the real magic part of `UIWebView`: The UIWebView Methods and The Delegate Methods.

## 3.Functioning Methods and Delegate Methods of UIWebView

### 3.1 Functioning Methods of UIWebView

```
/**
 * UIWebView will load a request with this method,
 * we need initialize an URLRequest instance first
 */
open func loadRequest(_ request: URLRequest);

/**
 * UIWebView can load a local html content with
 * this method, it is safer than loadRequest from a server.
 */
open func loadHTMLString(_ string: String, baseURL: URL?);

/**
 * This method can load a Data instance such
 * HTML,JPEG,PNG and some other file types, the
 * mimeType should identify the type.
 */
open func load(_ data: Data, mimeType MIMEType: String, textEncodingName: String, baseURL: URL);

/** Using this method to reload the content of the UIWebView */
open func reload();

/** Using this method to stop current loading action. */
open func stopLoading();

/** Make the UIWebView go back if it can */
open func goBack();

/** Make the UIWebView go forward if it can. */
open func goForward();

/** A value tells whether the UIWebView can go back */
open var canGoBack: Bool { get };

/** A value tells whether the UIWebView can go forward */
open var canGoForward: Bool { get };

/** A value tells whether the UIWebView is loading a content */
open var isLoading: Bool { get };

/**
 * This method can acquire the string value of the content
 * of UIWebView by evaluating the given JavaScript string
 */
open func stringByEvaluatingJavaScript(from script: String) -> String?;

```

### 3.2 Delegate Methods of UIWebView

Now let's see what are included in UIWebView's delegate methods.

All the delegate methods of `UIWebView` is defined in class `UIWebViewDelegate`:

```
public protocol UIWebViewDelegate : NSObjectProtocol {

    /**
     * Sent before a web view begins loading a frame.
     * @param webView
     * The web view that is about to load a new frame.
     * @param request
     * The content location.
     * @param navigationType
     * The type of user action that started the load request.
     *
     * @returns true if the web view should begin loading content; otherwise, false .
    */
    @available(iOS 2.0, *)
    optional public func webView(_ webView: UIWebView, shouldStartLoadWith request: URLRequest, navigationType: UIWebViewNavigationType) -> Bool
    /**
     * Sent after a web view starts loading a frame.
     * @param webView
     * The web view that has begun loading a new frame.
    */
    @available(iOS 2.0, *)
    optional public func webViewDidStartLoad(_ webView: UIWebView)
    /**
     * Sent after a web view finishes loading a frame.
     * @param webView
     * The web view has finished loading.
    */
    @available(iOS 2.0, *)
    optional public func webViewDidFinishLoad(_ webView: UIWebView)
    /**
     * Sent if a web view failed to load a frame.
     * @param webView
     * The web view that failed to load a frame.
     * @param error
     * The error that occurred during loading.
    */
    @available(iOS 2.0, *)
    optional public func webView(_ webView: UIWebView, didFailLoadWithError error: Error)
}
```
It is shown what the delegate methods do. There is no more to explain.

Using the delegate methods, we can do most of the actions of a `UIWebView`.

## 4.A demo project of using UIWebView

Here is a demo of using `UIWebView`, please [click here.](https://github.com/acttos/Swift-Acttos/tree/master/WebView){:target="_blank"}
