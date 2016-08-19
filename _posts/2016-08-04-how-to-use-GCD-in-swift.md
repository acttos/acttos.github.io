---
layout: post
title:  "How to use GCD in Swift ?"
date:   2016-08-04 20:31:18 +0800
categories: iOS
---

## #1. Condition
---

The first time GCD came to my sight,I was impressed by the beauty and efficient of it.
For a while, I used GCD in all my Network modules. I put the Network operation which always costs a little time in the background thread queue,and when the data is ready,I switch to the main thread to refresh UI. I believe most of us,the iOS developer,would act like this.

Here is an example of GCD in Objective-C:

```
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{

    //Put the codes which will cost a little more time here.
    self.filterList = [NetworkDispatchManager getFilterListWithURL:self.serverURL];

    //When the data is ready,we switch to main thread to refresh UI.
    dispatch_async(dispatch_get_main_queue(), ^{
        //Refresh UI.
        [self.tableView reloadData];
    });
});
```
Familiar? That is right, this is the common solution when we need to load a data from Network or read|write files on disk and then refresh the UI on the screen.

Now,I have an idea,what if I change the codes above in to Swift ? What would it look like ?

So I start my work to exchange:

## #2.The Code exchanged in Swift
---

Here is my implementation:

```
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    //Put the codes which will cost a little more time here.
    self.filterList = NetworkDispatchManager.getFilterListWithURL(self.serverURL);
    //When the data is ready,we switch to main thread to refresh UI.
    dispatch_async(dispatch_get_main_queue(), ^{
        //Refresh UI.
        self.tableView.reloadData();
    });
});
```
We can see that GCD in both Objective-C and Swift are almost the same.

## #3. Something about Closure in Swift
---

The closure in Swift is defined like this:

```
{ (arguments) -> Return Typs in
  statements
}
```
We can set the closure as a value of a variable like this:

`let aClosure = aClosureDefinition`

Here is an implementation:

```
// Define a closure and set it to variable `aClosure`:
let aClosure = {(location: NSURL?, response: NSURLResponse?, error: NSError?) in
    // For a demonstration here just print a message.
    print("The statement of a closure");
};

/**
 * The function who uses the closure defined before.
 */
func useClosure() -> Void {
    if let url: NSURL = NSURL(string: "http://domain.com/files/a-big-file.zip") {
        let request: NSURLRequest = NSURLRequest(URL: url, cachePolicy: NSURLRequestCachePolicy.ReturnCacheDataElseLoad, timeoutInterval: 10);
        let session:NSURLSession = NSURLSession.sharedSession();
        //We use variable `aClosure` as the argument `completionHandler`
        let task:NSURLSessionDownloadTask = session.downloadTaskWithRequest(request, completionHandler: aClosure);

        task.resume();
    } else {
        print("The URL is illegal.");
    }
}
```
<br/>

### Thanks for your visiting.
