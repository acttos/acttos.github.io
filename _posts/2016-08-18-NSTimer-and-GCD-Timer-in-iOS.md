---
layout: post
title:  "NSTimer and GCD Timer in iOS"
date:   2016-08-18 11:07:27 +0800
categories: iOS
---

### 1. How to use NSTimer ?

NSTimer can schedule an event in a later time. The most common way is :

```oc
class ViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        self._setupNSTimer();
    }

    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }

    func _setupNSTimer() -> Void {
        NSTimer.scheduledTimerWithTimeInterval(3, target: self, selector: #selector(_timerAction(_:)), userInfo: nil, repeats: true);
    }

    func _timerAction(timer: NSTimer) -> Void {
        NSLog("In '_timerAction(_:)': The timer is fired.");
    }
}
```

This class method can return a NSTimer instance and schedule the event too. We set the timer with time interval 3, which means this timer will `fire in 3 seconds`, and the `target` of the timer is `self`, the action is `_timerAction(_:)`, pay attention, this `:` here is required, method with pattern `selectorName(param:)` is called when the timer fires the event, the parameter `userInfo` here you can put some key-value pairs for the later use, the parameter `repeats` with value `true` means the timer we scheduled is repeatable, the event of the timer will triggered repeatedly.

As we say, we defined a method `_timerAction(timer:)` to execute the logic when the timer event is triggered:

```oc
func _timerAction(timer: NSTimer) -> Void {
    NSLog("In '_timerAction(_:)': The timer is fired.");
}
```
We use `NSLog` instead of `print` because we want print the system time in the console, and we can know the precise time of the events.

Now, let's run to see what will happen:

```
2016-08-18 15:01:11.618 Timer[58520:1287697] In '_timerAction(_:)': The timer is fired.
2016-08-18 15:01:14.556 Timer[58520:1287697] In '_timerAction(_:)': The timer is fired.
2016-08-18 15:01:17.555 Timer[58520:1287697] In '_timerAction(_:)': The timer is fired.
2016-08-18 15:01:20.556 Timer[58520:1287697] In '_timerAction(_:)': The timer is fired.
2016-08-18 15:01:23.555 Timer[58520:1287697] In '_timerAction(_:)': The timer is fired.
```
As we expected, the timer works fine, it fires every 3 seconds.

### 2. The GCD Timer

Most of the time, I use NSTimer for my scheduled events, and NSTimer seems to be the best option until I got an issue: The NSTimer does NOT work...

#### 2.1. Why NSTimer does NOT work ?

Here is the thing, I want to set a NSTimer for a background thread to see how long the thread costs, unfortunately I did not get what I want. Now let's see a familiar demo:

```oc
class ViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad();
        self._backgroundTimer();
    }

    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
        // Dispose of any resources that can be recreated.
    }

    func _backgroundTimer() -> Void {
        NSLog("_backgroundTimer invoked.");
        /**
         *  The thread I used is a background thread, dispatch_async will set up a background thread to execute the code in the block.
         */
        dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0)) {
            NSLog("NSTimer will be scheduled...");
            NSTimer.scheduledTimerWithTimeInterval(1, target: self, selector: #selector(self._backgroundTimerAction(_:)), userInfo: nil, repeats: true);
            NSLog("NSTimer scheduled...");
        }

    }

    func _backgroundTimerAction(timer: NSTimer) -> Void {
        NSLog("In '_backgroundTimerAction(_:)': The timer is fired.");
    }
}
```
In the console, we get this:

```
2016-08-18 15:27:22.575 Timer[59305:1314553] _backgroundTimer invoked.
2016-08-18 15:27:22.579 Timer[59305:1314860] NSTimer will be scheduled...
2016-08-18 15:27:22.580 Timer[59305:1314860] NSTimer scheduled...
```
We will never see `In '_backgroundTimerAction(_:)': The timer is fired.` printed.

Why is that ?

We checked the official document of Apple about NSTimer, we know NSTimer must run in a `NSRunLoop`, when the NSTimer is scheduled in the main thread, it will be added to the `Main Thread NSRunLoop` automatically. But in other threads, we must get the `NSRunLoop` first and add the timer to it and run the current `NSRunLoop` ...

Let's see the modified codes:

```oc
class ViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad();
        self._backgroundTimer();
    }

    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
        // Dispose of any resources that can be recreated.
    }

    func _backgroundTimer() -> Void {
        NSLog("_backgroundTimer invoked.");
        /**
         *  The thread I used is a background thread, dispatch_async will set up a background thread to execute the code in the block.
         */
        dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0)) {
            NSLog("NSTimer will be scheduled...");
            //Define a NSTimer
            let timer:NSTimer = NSTimer.scheduledTimerWithTimeInterval(1, target: self, selector: #selector(self._backgroundTimerAction(_:)), userInfo: nil, repeats: true);
            //Get the current RunLoop
            let runLoop:NSRunLoop = NSRunLoop.currentRunLoop();
            //Add the timer to the RunLoop
            runLoop.addTimer(timer, forMode: NSDefaultRunLoopMode);
            //Invoke the run method of RunLoop manually
            runLoop.run();
            NSLog("NSTimer scheduled...");
        }
    }

    func _backgroundTimerAction(timer: NSTimer) -> Void {
        NSLog("In '_backgroundTimerAction(_:)': The timer is fired.");
    }
}
```
In the console, we finally get what we want:

```
2016-08-18 15:51:16.343 Timer[60011:1337525] _backgroundTimer invoked.
2016-08-18 15:51:16.344 Timer[60011:1337800] NSTimer will be scheduled...
2016-08-18 15:51:17.344 Timer[60011:1337800] In '_backgroundTimerAction(_:)': The timer is fired.
2016-08-18 15:51:18.344 Timer[60011:1337800] In '_backgroundTimerAction(_:)': The timer is fired.
2016-08-18 15:51:19.344 Timer[60011:1337800] In '_backgroundTimerAction(_:)': The timer is fired.
2016-08-18 15:51:20.345 Timer[60011:1337800] In '_backgroundTimerAction(_:)': The timer is fired.
```
But `NSTimer scheduled...` is not printed, why?
Because the code `NSLog("NSTimer scheduled...");` is invoked only after the NSRunLoop becomes disactive, when the `NSTimer` is over, the codes under it will be invoked.

We change the initialization of the timer as:

```oc
let timer:NSTimer = NSTimer.scheduledTimerWithTimeInterval(1, target: self, selector: #selector(self._backgroundTimerAction(_:)), userInfo: nil, repeats: false);
```
And again, we check the console, we get this:

```
2016-08-18 15:58:05.484 Timer[60228:1346311] _backgroundTimer invoked.
2016-08-18 15:58:05.486 Timer[60228:1346618] NSTimer will be scheduled...
2016-08-18 15:58:06.489 Timer[60228:1346618] In '_backgroundTimerAction(_:)': The timer is fired.
2016-08-18 15:58:06.489 Timer[60228:1346618] NSTimer scheduled...
```
See? `NSTimer scheduled...` is printed.

Although we get the right result, I still feel it is a little complicated and there is some hidden dangers in `NSTimer`.

1. `NSTimer` needs one live `NSRunLoop` to execute it's events. In main thread, the `NSRunLoop` is always live and will never stop until the app is terminated, but in other threads, you must invoke `run()` to active the `NSRunLoop`;

2. `NSTimer` must invoke `invalidate()` to release the current timer, otherwise, the timer will retain a strong reference of the current instance of `target`, and it will remain in memory until app terminated;

3. `NSTimer` must created and invalidate in the same thread, and a lot of times, we may forget that.

And let me show you GCD Timer.

#### 2.2. What is GCD Timer ?

`GCD Timer` should be called `Timer in GCD`, because `GCD Timer` is not existed, it only is a feature of GCD. Let's look into a demo:

```oc
// To be continued. @2016-08-18 20:28
```

#### 2.3. How to use GCD Timer ?
// To be continued. @2016-08-18 20:28
