---
layout: post
title:  "Using UICollectionView Step by Step"
date:   2016-09-08 20:11:19 +0800
categories: iOS
---

> Summary: <br/><br/> `UICollectionView` allows us to display more than one cell in the same line which is not allowed in `UITableView`. We can design Apps such as *Photo Gallery*, *Video Wall* using `UICollectionView`. We will create a target `CollectionView` in [Swift-Acttos](https://github.com/acttos/Swift-Acttos) step by step. Keeping the main usages of `UICollectionView` in mind is OK, no one can remember all the features of a thing, not just `UICollectionView`.

## Let us just begin to build this target CollectionView:

### 1. Create a target

*Click the plus button to add a target 'CollectionView'*

![](/images/201609-uicollection-view/000.png)

*Choose Single View Application*

![](/images/201609-uicollection-view/001.png)

*Fill in the blanks with values you want*

![](/images/201609-uicollection-view/002.png)

After all these steps above, we have a new target `CollectionView` with *Code Structure* looks like this:

![](/images/201609-uicollection-view/003.png)

### 2. Drag a Collection View to the Storyboard

Let's select *Main.stroyboard* in the left navigator. We will see a blank Scene in the main window.

In the *View Controller Scene* we drag a *Collection View* to the Document Outline and just under *View*:

*The Collection View in the Object library:*

![](/images/201609-uicollection-view/0031.png)

*The hierarchy of Views in the scene:*

![](/images/201609-uicollection-view/0032.png)


### 3. Set the constraints

We need to add some constraints to make the page beautiful.
Do `Ctrl + Drag` from *Collection View* to *View*, with `Shift + Option` we add four constraints at once:

![](/images/201609-uicollection-view/004.png)

Then you get a *View Controller Scene* with a UICollectionView in it.
And the values of constraints are all zeros:

![](/images/201609-uicollection-view/005.png)

Now, let's modify the *Collection View Cell* by adding an UIImageView with a picture 'wechat.png':

![](/images/201609-uicollection-view/wechat.png)

The constraints of the *UIImageView* and the *Collection View Cell* is like the *View* and *Collection View*, all the values are 0.

![](/images/201609-uicollection-view/0033.png)

The scene now looks like this:

![](/images/201609-uicollection-view/0034.png)

### 4. Adjust the inner-layout of Collection View

Now let's modify the inner-layout of *Collection View*.

Select *Collection View* in the *Document Outline* and modify the layout in *Size Inspector*:

![](/images/201609-uicollection-view/0035.png)

> After all those steps above, we finally get a *UICollectionView* in *Main.storyboard*. The effective way of developing an App is **Design the user interface first, then write the codes**. So, we make the user interface first and write codes later.  

> Now, if you run `CMD + R` to see the face of your App, you probably will get a Black page on the screen. we will discuss about it later.

### 5. Define a class for Collection View Cell if needed

If you want to control the appearance of the cell dynamically, you'd better define a class for the cell in the storyboard.

Just click `CMD + N` -> `iOS` -> `Source` -> `Cocoa Touch Class` to open the guide of creating new file, give a name you want, choose a place to put the class file, and then you may get this:

![](/images/201609-uicollection-view/0036.png)

Don't forget to change the class type of the `Collection View Cell` in the Scene and set the ReuseIdentifier as *CollectionViewCell* in the *Attributes Inspector*. Here is my settings:

*Class type:*

![](/images/201609-uicollection-view/0037.png)

*ReuseIdentifier:*

![](/images/201609-uicollection-view/00371.png)

You may want to modify the appearance of the cell, here is just a ImageView, let's drag a link between the storyboard and the class file:

![](/images/201609-uicollection-view/0038.png)

At last, we set the DataSource and Delegate of CollectionView by dragging a link through InterfaceBuilder:

![](/images/201609-uicollection-view/00381.png)

Do the link again to the `delegate` in the *Connections Inspector*

### 6. Now let's do the coding part of CollectionView

First of all, we let `ViewController` implement the Protocol of `UICollectionViewDataSource` and `UICollectionViewDelegate`:

```
class ViewController: UIViewController, UICollectionViewDataSource, UICollectionViewDelegate {
    ...
}
```

You may get an error `Type 'ViewController' does not conform to protocol 'UICollectionViewDataSource'`, that is because we have not implemented the methods in `UICollectionViewDataSource` yet, don't worry, we will add them later.

#### 6.1 The DataSource methods of CollectionView

The DataSource methods define the number of sections, the number of cells and the cell of a `UICollectionView`, so we need to implement these methods to tell iOS how to handle our `UICollectionView`.

```
//MARK: - DataSource Methods of UICollectionView
func numberOfSectionsInCollectionView(collectionView: UICollectionView) -> Int {
    return 3;
}

func collectionView(collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
    return 7;
}

func collectionView(collectionView: UICollectionView, cellForItemAtIndexPath indexPath: NSIndexPath) -> UICollectionViewCell {
    let cell: CollectionViewCell = collectionView.dequeueReusableCellWithReuseIdentifier("CollectionViewCell", forIndexPath: indexPath) as! CollectionViewCell;

    if (indexPath.section % 2 == 0) {
        cell.backgroundImageView.image = UIImage.init(data: NSData(contentsOfURL: NSURL(string: "https://avatars2.githubusercontent.com/u/6056509?v=3&s=460")!)!);
    } else {
        cell.backgroundImageView.image = UIImage(named: "wechat.png");
    }

    return cell;
}
```

Don't forget to set the *Collection Resuable View's Identifier* as *CollectionViewCell*, just the same as the ReuseIdentifier in `func collectionView(collectionView: UICollectionView, cellForItemAtIndexPath indexPath: NSIndexPath)`.

That is enough for now, you can run `CMD + R` to check the result.

#### 6.2 The Delegate methods of CollectionView

The delegate methods define the actions of *UICollectionView*, there are more than one method in the Delegate protocol. we just implement some of them to show the usage, you can try the else yourself to test more.

*Here is the delege methods in this target:*

```
//MARK: - Delegate Methods of UICollectionView
func collectionView(collectionView: UICollectionView, didSelectItemAtIndexPath indexPath: NSIndexPath) {
    NSLog("The cell at section : \(indexPath.section), row : \(indexPath.row) has been clicked....");
}

func collectionView(collectionView: UICollectionView, didHighlightItemAtIndexPath indexPath: NSIndexPath) {
    let cell: CollectionViewCell = collectionView.cellForItemAtIndexPath(indexPath) as! CollectionViewCell;
    let frame: CGRect = cell.backgroundImageView.frame;
    cell.backgroundImageView.frame = CGRectMake(frame.origin.x + 4, frame.origin.y + 4, frame.size.width - 8, frame.size.height - 8);
}

func collectionView(collectionView: UICollectionView, didUnhighlightItemAtIndexPath indexPath: NSIndexPath) {
    let cell: CollectionViewCell = collectionView.cellForItemAtIndexPath(indexPath) as! CollectionViewCell;
    let frame: CGRect = cell.backgroundImageView.frame;
    cell.backgroundImageView.frame = CGRectMake(frame.origin.x - 4, frame.origin.y - 4, frame.size.width + 8, frame.size.height + 8);
}
```

When you click on the cell in the CollectionView, you'll get a zooming animation and an output in console:

```
2016-09-08 19:59:45.204 CollectionView[50014:2032904] The cell at section : 0, row : 5 has been clicked....
2016-09-08 19:59:46.418 CollectionView[50014:2032904] The cell at section : 0, row : 5 has been clicked....
2016-09-08 19:59:46.830 CollectionView[50014:2032904] The cell at section : 0, row : 5 has been clicked....
2016-09-08 19:59:47.709 CollectionView[50014:2032904] The cell at section : 0, row : 5 has been clicked....
2016-09-08 19:59:48.343 CollectionView[50014:2032904] The cell at section : 0, row : 5 has been clicked....
2016-09-08 19:59:48.823 CollectionView[50014:2032904] The cell at section : 0, row : 5 has been clicked....
2016-09-08 19:59:49.064 CollectionView[50014:2032904] The cell at section : 0, row : 5 has been clicked....
2016-09-08 19:59:49.278 CollectionView[50014:2032904] The cell at section : 0, row : 5 has been clicked....
```

### 7. Epilogue

Until now, there must be a clear known about the usages of UICollectionView. you can write codes by yourself to verify the content of this article. You will find some mistakes maybe, please let me know if you do.

### Thanks for visiting!
