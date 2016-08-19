---
layout: post
title:  "NSTimer and GCD Timer in iOS"
date:   2016-08-19 16:09:27 +0800
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

#### 2.2. What is GCD Timer and How to use it ?

`GCD Timer` should be called `Timer in GCD`, because `GCD Timer` is not existed, it only is a feature of GCD. Let's look into a demo:

First, let's define two parameters in class:

```oc
let queue: dispatch_queue_t? = dispatch_queue_create("GCDTimerQueue", DISPATCH_QUEUE_CONCURRENT);//Define and initialize a dispatch_queue;
var timer: dispatch_source_t? = nil;//To be initialized later;
```

Then, let's set up the function of `_timerInGCD(repeated:)`:

```oc
    func _timerInGCD(repeated repeated:Bool) -> Void {
        NSLog("_timerInGCD invoked.");
        if (self.timer != nil) {
            dispatch_source_cancel(self.timer!);
        }
        timer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, queue);
        dispatch_source_set_timer(timer!, DISPATCH_TIME_NOW, 1 * NSEC_PER_SEC, 0);

        dispatch_source_set_event_handler(timer!, { () -> Void in
            NSLog("In GCD Timer, block is invoked ...");

            if(!repeated) {
                dispatch_source_cancel(self.timer!);
                NSLog("The timer is canceled.");
            }
        });

        NSLog("timer will be resumed.");
        dispatch_resume(timer!);
    }
```

In the console, we get these outputs:

##### GCD Timer output

```
2016-08-19 14:49:16.251 Timer[97556:2180745] _timerInGCD invoked.
2016-08-19 14:49:16.253 Timer[97556:2180745] timer will be resumed.
2016-08-19 14:49:16.255 Timer[97556:2181222] In GCD Timer, block is invoked ...
2016-08-19 14:49:16.256 Timer[97556:2181222] The timer is canceled.
```
We can see that, the Timer is fired only once, because we canceled the `dispatch_source`.

##### Repeated GCD Timer output:

```
2016-08-19 14:49:36.031 Timer[97556:2180745] _timerInGCD invoked.
2016-08-19 14:49:36.031 Timer[97556:2180745] timer will be resumed.
2016-08-19 14:49:36.032 Timer[97556:2181938] In GCD Timer, block is invoked ...
2016-08-19 14:49:37.032 Timer[97556:2181938] In GCD Timer, block is invoked ...
2016-08-19 14:49:38.033 Timer[97556:2181938] In GCD Timer, block is invoked ...
2016-08-19 14:49:39.033 Timer[97556:2181938] In GCD Timer, block is invoked ...
2016-08-19 14:49:40.033 Timer[97556:2181938] In GCD Timer, block is invoked ...
```
The Timer fires every second and it will continue firing event until we cancel the dispatch_source.

_Warning:_ The parameter `queue` and `timer` must defined in class scope, because the life scope of a function is so short, if we define the `queue` or `timer` in a function, the `GCD Timer` will not be able to find the `timer` and the `queue` when the event fired because the `queue` or `timer` is released at the end of the function scope.

For function control, I add some buttons and actions in the storyboard and code.

##### The storyboard looks like this:

![](/images/nstimer-and-gcd-timer-in-ios/timer-storyboard.png)

##### The button actions in code:

![](/images/nstimer-and-gcd-timer-in-ios/button-actions-in-code.png)


#### 2.3. Something need to know about GCD Timer

According to the document of Apple about GCD, we can find the part of GCD Timer.

##### dispatch_queue_t
> A dispatch queue is a lightweight object to which your application submits blocks for subsequent execution.

We usually use `dispatch_queue_t` to define a queue for later use.

##### dispatch_queue_create
> Creates a new dispatch queue to which blocks can be submitted.

After we define the queue, we also need to create the queue by invoking `dispatch_queue_create` with a name and type, we can pick `Serial` or `Concurrent` as the queue's type..

##### dispatch_source_t
> typealias dispatch_source_t = OS_dispatch_source;    Dispatch sources are used to automatically submit event handler blocks to dispatch queues in response to external events.

In GCD, we are recommanded to create `dispatch_source_t` in `dispatch_queue_t`, so we use `dispatch_source_t` to define a source for timer.

##### dispatch_source_set_timer
> Sets a start time, interval, and leeway value for a timer source.

We need to set the timer in the queue by `dispatch_source_set_timer`.

##### dispatch_source_set_event_handler
> Sets the event handler block for the given dispatch source.

This can set the handler when the timer fires event.

##### dispatch_resume
> Resume the invocation of block objects on a dispatch object.

All the `dispatch_source_t` are just put in the queue with handler, they will not be executed unless we resume the dispatch object which contains the queue.

##### dispatch_source_cancel
> Asynchronously cancels the dispatch source, preventing any further invocation of its event handler block.

If we want to cancel the timer, we can just cancel the dispatch object which contains the timer queue.

### 3. Compare NSTimer and GCD Timer

When we test the `NSTimer` in the codes, we know `NSTimer` needs to be run in a `NSRunLoop` which is automatically initialized and maintained in main thread but not in other background threads. When we need to setup a `NSTimer` in background threads, we need to handle the stuff of `NSRunLoop` by ourself, that is not a friendly action.

We also know, every `NSTimer` will retain a strong reference of `target`, this may cause memory leak when the `target` can not be released.

`GCD Timer` is based on `GCD (Grand Central Dispatch)`, we just leave all the rest to `GCD`, we will never need to worry about the memory issues.

But `GCD Time` is not so easy to use because of the API of it, we can modify or create a new function to work like `NSTimer`. In other words, we need to implement these functions:

```oc
func scheduledTimerWithTimeInterval(ti: NSTimeInterval, block:GCDTimerBlock, repeats yesOrNo: Bool) -> Void;
```

Yes, we need to define a closure called `GCDTimerBlock`, it can look like this:

```oc
typealias GCDTimerBlock = @convention(block) (userInfo:AnyObject?) -> Void;
```

All together:

```oc
    typealias GCDTimerBlock = @convention(block) (userInfo:AnyObject?) -> Void

    func scheduledTimerWithTimeInterval(ti: UInt64, block:GCDTimerBlock, repeats yesOrNo: Bool) -> Void {
        if (self.timer != nil) {
            dispatch_source_cancel(self.timer!);
        }
        timer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, queue);
        dispatch_source_set_timer(timer!, DISPATCH_TIME_NOW, ti * NSEC_PER_SEC, 0);

        dispatch_source_set_event_handler(timer!, { () -> Void in
            NSLog("In GCD Timer, block is invoked ...");

            if(!yesOrNo) {
                dispatch_source_cancel(self.timer!);
                NSLog("The timer is canceled.");
            }
        });

        NSLog("timer will be resumed.");
        dispatch_resume(timer!);
    }
```

### 4. Source code

##### You can find the source code [[ _Here_ ]](https://github.com/acttos/Swift-Acttos) and you are welcomed to give suggestions.
_The codes may be updated from time to time._

Thanks!
