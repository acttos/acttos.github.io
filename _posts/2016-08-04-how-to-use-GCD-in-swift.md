---
layout: post
title:  "How to use GCD in Swift ?"
date:   2016-08-04 22:06:18 +0800
categories: iOS
---

# How to use GCD in Swift ?
---

## #1. Issue
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

//TODO:
## #3. Source Code:

You can find all the codes [Here](https://github.com/majinshou/ColorfulLabel){:target="_blank"}.
