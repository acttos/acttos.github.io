---
layout: post
title:  "The Colorful UILabel in iOS"
date:   2016-08-02 19:36:51 +0800
categories: iOS
---

# The Colorful UILabel in iOS development
---

Once up on a time,I had a request which needed to display a colorful UILabel in the screen.

The productor said the colorful texts can catch the eyes of audiences.

OK,I sure chose to believe them.

Now,here is the question:

## 1. How to set different colors to each word in the UILabel?

---

I started my work with my own ideas:

- An UILabel can display the texts;
- The color of the texts can be set by 'UILabel.textColor';
- I add two UILabels with different colors will do;
- Excellent! (I said to myself). 

I opened the Xcode.app to edit my codes like this:

### 1.1 The way I did in the past time

```
/**
 Initializes a colorful label container view
 
 - returns: A colorful label container view instance
 */
func initColorfulLabel() -> UIView {
    let containerViewFrame:CGRect = CGRect(x: -7, y: 100, width: self.view.bounds.width, height: 20);
    let containerView:UIView = UIView(frame: containerViewFrame);
    
    let redLabelFrame = CGRect(x: 0, y: 0, width: containerView.bounds.width / 2, height: 20);
    let redLabel:UILabel = UILabel(frame: redLabelFrame);
    redLabel.text = "RED";
    redLabel.textColor = UIColor.redColor();
    redLabel.textAlignment = NSTextAlignment.Right;
    
    containerView.addSubview(redLabel);
    
    let blueLabelFrame = CGRect(x: containerView.bounds.width / 2, y: 0, width: containerView.bounds.width / 2, height: 20);
    let blueLabel:UILabel = UILabel(frame: blueLabelFrame);
    blueLabel.text = " BLUE";
    blueLabel.textColor = UIColor.blueColor();
    blueLabel.textAlignment = NSTextAlignment.Left;
    
    containerView.addSubview(blueLabel);
    
    
    return containerView;
}
```

Oh,My God,It worked! ^0^:

![colorfulLabelsInContainerView.png](/images/colorful-uilabel/colorfulLabelsInContainerView.png)


### 1.2 The suggested way of a mentor told me recently

My mentor told me that UILabel supports different texts with different colors by setting UILabel's attributedText.

Here is the newer one:

```
/**
 Initializes a real colorful UILabel instance instead of a container view,this method is guided by a mentor
 
 - returns: A real colorful UILabel instance
 */
func initColorfulLabelOfMentor() -> UILabel {
    let str:NSMutableAttributedString = NSMutableAttributedString(string: "RED BLUE");
    str.addAttribute(NSForegroundColorAttributeName, value: UIColor.redColor(), range: NSMakeRange(0, 3));
    str.addAttribute(NSForegroundColorAttributeName, value: UIColor.blueColor(), range: NSMakeRange(4, 4));
    
    let labelFrame = CGRect(x: 0, y: 140, width: self.view.bounds.width, height: 20);
    let label:UILabel = UILabel(frame: labelFrame);
    label.attributedText = str;
    label.textAlignment = NSTextAlignment.Center;
    
    
    return label;
}
```

It works too:

![colorfulLabel.png](/images/colorful-uilabel/colorfulLabel.png)

After I made the colorful texts in the screen,the productor said we still need to add a shadow to the text, and again, to catch the eyes of audiences.

OK,I'll do it. Because I always to choose to believe them.

So,I have a new question here:

## 2. How to add a shadow to the words in UILabel?

---

I started my thinking again,of course I chose to add two different UILabels to make the shadow:

- Add an UILabel to display the given texts with certain color(color of the shadow);
- Add another UILabel to display the texts with shadow color;
- Put the two UILabels together into a container UIView to make a shadow;
- Return the UIView instance called 'containerView'.

So,I put my ideas into codes:

### 2.1 The way I did in the past time

Here is the implementation of my idea:

```
/**
 Initializes a shadow label container view
 
 - returns: A UIView instance which contains two UILabels to display a shadow
 */
func initShadowLabel() -> UIView {
    
    let containerViewFrame:CGRect = CGRect(x: 0, y: 180, width: self.view.bounds.width, height: 20);
    let containerView:UIView = UIView(frame: containerViewFrame);
    
    let lowerLabelFrame = CGRect(x: 1, y: 1, width: self.view.bounds.width, height: 20);
    let lowerLabel:UILabel = UILabel(frame: lowerLabelFrame);
    lowerLabel.text = "LABEL WITH SHADOW";
    lowerLabel.textColor = UIColor.blueColor();
    lowerLabel.textAlignment = NSTextAlignment.Center;
    
    containerView.addSubview(lowerLabel);
    
    
    let upperLabelFrame = CGRect(x: 0, y: 0, width: self.view.bounds.width, height: 20);
    let upperLabel:UILabel = UILabel(frame: upperLabelFrame);
    upperLabel.text = "LABEL WITH SHADOW";
    upperLabel.textColor = UIColor.redColor();
    upperLabel.textAlignment = NSTextAlignment.Center;
    
    containerView.addSubview(upperLabel);
    
    
    return containerView;
}
```

Oh,My God,It worked again:

![shadowLabelsInContainerView.png](/images/colorful-uilabel/shadowLabelsInContainerView.png)


### 2.2 The suggested way of a mentor

Once again,a mentor occurs,And he gives me another way to implement the shadow in UILabel:

```
/**
 Initializes a real shadow UILabel instance instead of a container view,this method is guided by a mentor
 
 - returns: A UILabel which has a shadow
 */
func initShadowLabelOfMentor() -> UILabel {
    
    let labelFrame = CGRect(x: 0, y: 220, width: self.view.bounds.width, height: 20);
    let label:UILabel = UILabel(frame: labelFrame);
    label.text = "LABEL WITH SHADOW";
    label.textColor = UIColor.redColor();
    label.shadowColor = UIColor.blueColor();
    label.shadowOffset = CGSize(width: 1, height: 1);
    label.textAlignment = NSTextAlignment.Center;
    
    
    return label;
}
```

See? It is more simple and clearer.

Here is the result in picture:

![shadowLabel.png](/images/colorful-uilabel/shadowLabel.png)

## 3. Source Code:

You can find all the codes [Here](https://github.com/majinshou/ColorfulUILabel)

