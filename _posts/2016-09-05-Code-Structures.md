---
layout: post
title:  "The code structures in projects"
date:   2016-09-05 16:53:29 +0800
categories: Structure
---

> Summary:<br/>For years, I was working with many old projects handed over from previous coders. Most of the time, I feel upset very much at some of them which have badly designed code structures. I do NOT like it and I want to make a change. That is why this article comes out.

## 1. Goals of code structures

I believe every programmer prefers a clear, simple and scalable code structure in the project. A good design of structure is a good start of a project, it can brings very fast coding, searching, refactoring and also brings a pleasant feeling of breath which I think is very important while coding.

#### 1.1. Team Work

Most of us are not working alone. We have colleagues or co-workers doing the coding job. If we write codes independently without enough communication or there is not so much of time while codeing, the good design of code structures is the key of collaboration. Based on the same code structure, everyone is codeing inside the part of himself, and provides well designed interfaces among parts of others'.

#### 1.2. Codes Handover

We can't be sure that we will work at the same palce or do jobs with only one project, and we also will take over projects from former coders. A well designed code structure here is strongly needed, otherwise, we will need much more time dealing with reading and understanding the codes, and with well knowing of codes, there might come with modifying and even refactoring next.

#### 1.3. Maintenance

On special condition, you are always at the same place with the same job, but what about the time? You can't stop time, so your codes all will become a history, one day you need to locate an old postion to handle an issue possibly caused by some codes you wrote before, do you need a well designed code structure? Ofcourse you do.


## 2. Frequently used designs
I can't say I am so good at code structures designing, I only have some real practises which I think can give some ideas. I know there are so many ways to design the structure, I am not able to figure all of them out. Please allow me show you the three frequently used designs below:

#### 2.1. K.I.S.S

I always belive and insist that `The simpler your code is, the easier you job will be`. No one want to put himself into a complicated situation. So, rember to keep every thing simple, the simpler the better.

`K.I.S.S` means `Keep it simple, stupid`, the word `stupid` here does not mean `Write codes stupidly`, `stupid` is the same as `simple`, `K.I.S.S` uses `simple` and `stupid` to tell us how to write codes.

*Here is a example of Objective-C to fill an user info:*

```
#pragma mark - Private Methods for only Self-using
-(void)_fillViewWithUserInfo:(SPUserInfo *)userInfo {
    UIImage *iconImage = [UIImage imageWithData:[NSData dataWithContentsOfURL:[NSURL URLWithString:userInfo.portraitURL]]];
    self.userIconBackgroundImageView.image = iconImage;
    self.userIconImageView.image = iconImage;
    self.userNicknameLabel.text = userInfo.nickName;
    self.userIdLabel.text = userInfo.userId;
    self.userIntroLabel.text = @"";
    self.userFunCountLabel.text = [NSString stringWithFormat:@"%ld",userInfo.fansNum];
    self.userFocusCountLabel.text = [NSString stringWithFormat:@"%ld",userInfo.subsNum];
    self.userStarTicketCountLabel.text = [NSString stringWithFormat:@"%ld",userInfo.totalStarTickets];
    self.userIncomeCountNumberLabel.text = [NSString stringWithFormat:@"%f",userInfo.userIncome];
    self.userStarBeanCountNumberLabel.text = [NSString stringWithFormat:@"%ld",userInfo.userStarCoins];

}
```  
The method above fills an user info, and it is the only thing it does. `Simple` does not mean there is less lines of codes, it means `Very easy to understand and Designed for only one purpose`.

Let's see more:
*Control method and sub-control methods*

```
// I am the control method for deciding route
-(void)_switchBroadcastingLeftItems {
    if (!self.isLeftItemsButtonShowing) {
        //Show them
        [self _showBroadcastingLeftItems];
    } else {
        //Hide them
        [self _hideBroadcastingLeftItems];
    }
}

// I am the sub-control method for showing thing
-(void)_showBroadcastingLeftItems {
	//Do the showing thing
}
// I am the sub-control method for hiding thing
-(void)_hideBroadcastingLeftItems {
	//Do the hiding thing
}
```

I usually see some one writing the `showing` and `hiding` methods in only one statement. Ofcourse, that is OK, we can completely do lots of things in one place except we are willing to keep simple and clear.

#### 2.2. MVC & MVP

We also want an clear structure in our project, I saw many projects with bad codes hierarchy, you can feel it:

*A codes hierarchy:*

```
SAViewController
|____SAImagePicker
|____SACameraController
|____SADashboardController
| |____dashboardImage
| |____PremiumFeatures
|____SAFBLoginController
|____SAMediaController
|____SAMenuController
|____SAMessenger
| |____Chat
| |____Friend
|____SAPhotoBrowser
| |____CustomNavBar
| |____EGORefreshTable
| |____HPGrowingTextView
| |____MBProgressHUD
| |____Popover
| |____SAActivityIndicatorView
| |____SAAlertView
| |____SAEmojiKeyboard
| |____SALabel
| |____SALazyImageView
| |____SAProgressBar
| |____SASeparateView
| |____StyledPageControl
```
Why there are so many un-controller class folders under `SAViewController`? Why there is a model `SAMessenger` under `SAViewController`? I was totally lost in it the first time I saw it.

In my opinion, we should place the same type of files in a folder, the same functioning of files in a folder, even the same type of language in a folder and so on.

In one words:`We always put files which have something in common into a folder`.

*Here is another example which places the file with a better designed hierarchy:*

```
SAStore
|____Controller
| |____SAStoreViewController.h
| |____SAStoreViewController.m
| |____SAProductWebViewController.h
| |____SAProductWebViewController.m
|____Model
| |____SAStoreProduct.h
| |____SAStoreProduct.m
| |____SAStoreProductListResponse.h
| |____SAStoreProductListResponse.m
|____SAStore.storyboard
|____SAStore.xib
|____View
| |____NJKWebViewProgress
| | |____NJKWebViewProgress.h
| | |____NJKWebViewProgress.m
| | |____NJKWebViewProgressView.h
| | |____NJKWebViewProgressView.m
| |____SAProductCollectionViewCell.h
| |____SAProductCollectionViewCell.m
```
*Here is the code structure in Xcode:*

![](/images/201609-code-structure/sastore-mvc-structure.png)


Now, let's see what is `MVC`:

> *MVC* means *Model,View,Controller*, it is a design pattern of flow control or data control.  

> *MVC* is used widely in J2EE projects, the source codes in the *Model* layer response for the logic such as `database invoke,data cache`, the source codes in `Controller` response for the logic of `business logic control`, and the source codes in `View` response for the `user interface`.   

*Role of Model, View, Controller:*

```
Model: Handle most of the data logic in a system, such as database access, remote http invoking and so on.

View: Handle the 'User Interface' thing in a system.

Controller: Control the business logic.
```
*Here is the invoking flow of MVC:*

![](/images/201609-code-structure/mvc-invoke-flow.png)

The user can interactive with these two flow-chats below:

![](/images/201609-code-structure/mvc-invoke-flow-with-user.png)

The left one shows up that the user interactives through the `View` layer, it is seen in a web application: *User Registration or Signing In*. Most of this type requires the user to input some forms for the data to transfer and then the *Controller* handles the business logic with the data, at last,the *Model* gives the result to the *View* and the user can get a result of inputs.

The right one shows up that the user passes data directly to the *Controller* layer, then, *Controller* passes the request data to *Model*, the *Model* returns the result to *View*. This kind of *MVC* can be seen in Apps linked with an App-Server. Most of the time, there is no input views for the user, the pages in the App can handle the request data as input key-value(s).



There is another Design Partten called *MVP* which is familiar to *MVC*.

*Here is the invoking flow of MVP:*

![](/images/201609-code-structure/mvp-invoke-flow.png)

In *MVP*, we can't see the layer *Controller*, instead of, we get *Presenter*.

The key difference between *MVP* and *MVC* is the data transmission. All the data must be handled by *Presenter* in *MVP*. *Presenter* is the main door and only door of data.

I prefer *MVP* as my design partten, there is no reason, or it is much clearer.

## 3. Thanks
<br/>
*Thank [RuanYifeng](http://www.ruanyifeng.com/blog/2015/02/mvcmvp_mvvm.html) very much for the introduction and pictures of MVC and MVP*
