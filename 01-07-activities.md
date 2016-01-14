# 01-07 Activities

### Admin [5min]
- And we're back!
- Check in on homework: everyone have access to GitHub? It looks like _34_ people have been invited (but only _28_ have logged in so...)
  - My assumption is you're comfortable with git to fork/clone and submit the homework. IF NOT, please let me know and I'm happy to go over it.
- Have we worked out kinks with getting Android Studio running on your machines?
  - I know we're in computer lab, but bringing your own laptops/devices might be easiest for practicing with stuff!


## What is an Activity? [10min]
Today we're going to dive right in and start talking about how Android apps run. We'll start by looking at the basic class that's given to us by a new project: Activity!
  - start by creating a new Android Project with an Empty Activity, just like you did yesterday (practice!)
  - *??? fork and clone my repo??*

What is an [Activity](http://developer.android.com/guide/components/activities.html)? According to Google:

> An Activity is an application component that provides a screen with which users can interact in order to do something


You can think of an Activity as a single Screen in your App. The equivalent of a "Window" in a GUI system (or a `JFrame` in a Swing app).
- Note that Activities don't __need__ to be full screens: they can also be floating modal windows, embedded inside other Activities (like half a screen), etc. But we'll start by thinking of them as full screens.
- In many ways, an Activity is a "bookkeeping mechanism": a place to hold state and data, and tell Android what to draw.
  - It functions a bit as a Controller (in Model-View-Controller sense) in that regard!

We can have lots of Activities (screens) in an Application, and they are loosely connected so we can navigate between them (and start new ones)

Also:

> An activity is a single, focused thing that the user can do.

Which implies a design suggestion: Activities (screens) break up your App into "tasks": each one can represent what a user is doing at once! If the user does something else, that should be a different Activity and so probably a different screen.


### Making Activities
How do we use them? We create our own activities by subclassing the provided [`Activity`](http://developer.android.com/reference/android/app/Activity.html) class.
- inheritance: make a specialized type of `Activity`
- similar to extending `JFrame` in Swing apps
- all the methods that control how the OS interacts with Activities are provided for us.

If you look at the default Blank activity, it actually subclasses [`AppCompatActivity`](http://developer.android.com/reference/android/support/v7/app/AppCompatActivity.html), which is an Activity with an [`ActionBar`](http://developer.android.com/reference/android/support/v7/app/ActionBar.html) (the toolbar at the top). If we change the class to just extend `Activity`, we can get rid of that bar.
  - You'd need to import the `Activity` class! The keyboard shortcut is `alt+return`, or you can do it by hand (look up the package)!

There are a pile of other built-in Activity subclasses that we could subclass instead. We'll mention them as they become relevant.
- Many on the books have been deprecated in favor of **Fragments**, which are sort of like "subactivities" that get nested in larger Activities. We'll talk about Fragments more once we've gotten the basics down.

Other important point to note: does this activity have a **constructor** that we call? (No!)
- We never write code that **instantiates** our Activity. There is no `main` method. Activities are created and managed by the Android operating system!

## The Activity Lifecycle [10-15min]
So if we never call the constructor or main, how do we start doing stuff?

- Activities have an _incredibly_ well-defined lifecycle&mdash;that is, a series of **events** that occur during usage
  - e.g., when created, stopped, etc.
- When each of these events occur, Android executes a **callback method**, similar to how you called `actionPerformed` to react to a "button press" event. We can override these methods in order to do special actions (read: our own code) when these events occur.

What is the lifecycle?

![lifecycle state chart](http://developer.android.com/images/activity_lifecycle.png)

(Or an alternative, simplified diagram [here](http://developer.android.com/images/training/basics/basic-lifecycle.png)).

There are 7 "events" that occur in the Activity Lifecycle, which are designated by the callback function that they execute:
- `onCreate()`: when **first** created/instantiated. This is where you initialize UI stuff (e.g., specify the layout to use)
- `onStart()`: called just before the Activity becomes visible to the user.
  - Whats the difference between this and `onCreate`? `onStart` can be called more than once (e.g., if you leave the Activity and come back).
- `onResume()`: just before user interaction starts--activity is ready to go!
  - difference between `onStart()`? When start is called we're visible, but resume is called when interaction is ready.
  - We can be visible but not interacting (like if there is a modal in front of us!)
- `onPause()`: when the system is about to start another activity (so about to lose focus). This is the "mirror" of `onResume()`. _The activity stays visible_.
  - This is where we tend to store (temporary) unsaved changes (like email drafts), stop animations, etc, since the activity might be on its way out.
- `onStop()`: the activity is no longer visible. (e.g., another activity took over, but also because might be destroyed). A mirror of `onStart()``
  - This is where we would want to persist any state information (e.g., save their game).
- `onRestart()`: called when the app is coming back from a "stopped" state
- `onDestroy()`: activity is about to be closed. This can happen because the user ended the application, or because the OS is trying to save memory and so kills you app (if it hasn't been use in a while).
  - can do more final cleanup.

Note that apps may not need to use all of these methods: for example, if there is no difference between starting from scratch and resuming from stop, then you don't need an onRestart (since onStart goes in the middle). Similarly onStart may not be needed if you just use onCreate and onResume; but these lifecycles allow for more granularity and the ability to avoid duplicate code.

### Overriding the methods
Let's look at overriding these methods to see them in action!

We already have `onCreate()` overridden for us, since that's where the layout is specified (we'll cover how soon, I promise!)
  - Notice it takes a `Bundle` as a parameter. A [`Bundle`](http://developer.android.com/reference/android/os/Bundle.html)   is an object that stores information about the activities state, so that you can restore it when the activity is recreated
    - e.g., if the system destroyed the Activity for memory, it should be restored to previous state when recreated so the user doesn't notice that!
    - It stores current layout information in it by default (if Views have ids), and since its basically a Map (key-value pairs) you can store other data in it as well.
      - called `onSaveInstanceState()` for each View, and those tend to save important bits.
    - see [Saving Activity State](http://developer.android.com/guide/components/activities.html#SavingActivityState) for details.
  - Also note that we call `super.onCreate()`. **ALWAYS CALL UP THE INHERITANCE CHAIN**, so that you can do system-level stuff!

So we can add the others: how about `onStart()`? (see [Implementing Lifecycle Callbacks](http://developer.android.com/guide/components/activities.html#ImplementingLifecycleCallbacks) for template).


## Logging & ADB [10min]
But how can we know if the lifecycle events are getting called?
- `Sysout`? we don't actually have a terminal! More specifically, our device (which is where the application is running) doesn't have access to `stdout`
  - Aside: we can get it with `adb shell stop; adb shell setprop log.redirect-stdio true; adb shell start`
- Instead, Android provides a Logging system that we can use to write out debugging information, and which is automatically accessible over the `adb` (Android Debugging Bridge).
  - Logging can be filtered, categorized, sorted, etc.
  - Can be disabled in production builds, but often isn't :p
- To perform logging, we'll use the [`android.util.Log`](http://developer.android.com/reference/android/util/Log.html) class. This class includes a number of `static` methods, which all basically wrap around `println` to print to the device's log file, which is then accessible through the `adb`.
  - The device's log file is stored... sort of. It's a 16k file, but is shared across the _entire_ system. So fills up fast. Hence filtering/searching becomes important, and you tend to watch it/debug in real time!
  - Remember to import the [`Log`](http://developer.android.com/reference/android/util/Log.html)] class!

### Log Methods
Methods correspond to different level of priority (importance) of the messages. From low to high:
- `Log.v()`: VERBOSE output. The most detail. Everyday stuff. Often our go-to level.
  - only compile into an application during development!
- `Log.d()`: DEBUG output. lower-level, but a bit less detail.
  - Debug logs _can_ be compiled but stripped at runtime using the [`BuildConfig`] generated class, which can be customized through Gradle
- `Log.i()`: INFO output. High level info
- `Log.w()`: WARN output. Warnings
- `Log.e()`: ERROR output. Errors
- Also look at the API... `Log.wtf()`!

These are used to help "filter out the noise". So you can look just at errors, at errors and warnings, at err warn info... all the way down to _everything_ with verbose.
- A HUGE amount of information is logged, so really need this filtering!

Each `Log` method takes two `Strings` as parameters. The second is the message to print. The first is a "tag"--a String that's prepended to the output which you can search and filter on.
- This is is usually the App or Class name (e.g., "AndroidDemo", "MainActivity")
- A common practice is to declare a `TAG` constant you can use throughout:
```java
private static final String TAG = "MainActivity";
```

### Logcat
You can view the logs via `adb` (the debugging bridge) and a service called `Logcat` (from "log" and "cat" from concatenation, since it concats the logs).
- The easiest way to check Logcat is to use Android Studio. The Logcat browser panel is usually found at the bottom of the screen after you launch an application. It "tails" the log, showing the latest output as it appears
  - You can use the dropdown box to filter by priority, and the search box to search (e.g., by tag if you want).
  - Android Studio also lets you filter to only show the current application, which is hugely awesome.
  - Note that you will see a lot of Logs that you didn't produce, including possibly Warnings (e.g., I see a lot of stuff about how OpenGL connects to the graphics card). _This is normal_!
- It is also possible to view Logcat through the command-line using `adb`, and includes complex filtering arguments. See [Reading and Writing Logs](http://developer.android.com/tools/debugging/debugging-log.html) for more details.

## Exercise: Log and Lifecycle Practice [15min]
As practice working with logging and exploring the Activity Lifecycle, you should modify your Android Activity so that it logs out messages on different lifecycle events. Your app should include the following output:
1. When the application is created, Log "Greetings Developer!" at the **Verbose** level
2. On each of the **7** lifecycle events, you should Log "on{eventName} fired" at the **Info** level. Log out any parameters the event handlers take as well.
3. When the application is closed, it should log "We're going down!" at the **Error** level of the log.

Once you have these in place, make sure to run and play with your application to test that the events are all fired! Use the "back" button (the arrow) or the "home" button (the circle) to leave the application, or use the "switcher" button (the square) to go to a different app (like the browser) to see how Android destroys applications that are not being used.

Something else to test:
- Cause your app to throw a runtime `Exception` in one of the handlers. For example, you could make a new local array and try to access an item out of bounds. Or just `throw new RuntimeException()` (which is slightly less interesting). _Can you see the **Stack Trace** in the logs?_

### **break around 11:30; 5-10min**

## New Activities; Intents [20min]
The whole point of considering the Activity Lifecycle is because Android applications can have multiple activities and interact with multiple other applcations. So let's talk briefly about how we could have an app use multiple Activities (and so get a sense for how this Lifecycle may affect us)
.
- We can go ahead and create a New Activity through Android Student by using `File > New > Activity`. We could also just add a new .java file with the Activity class in it
  - Make a new Empty activity `SecondActivity`
  - Note that this Activity also gets a resource XML layout
    - You should copy in a `<TextView>` element from the MainLayout so this one can have a message as well.
- _ALSO NOTE_: For every activity we make, it gets added to the **Manifest** file.
  - another `<activity>` element in the `<application>` element. We can also see the first Activity; we'll talk about its child elements in a second
  - We can also add some more configuration to this element. Add a `android:parentActivityName` attribute, with a value to set the full class name (including package **and** appname!) of your Main Activity. This will let you be able to use the "back" visual elements (e.g., of the ActionBar) to move back to the "parent" activity. See [Up Navigation](http://developer.android.com/design/patterns/navigation.html) for details.
    - This is only supported for API 16+; since our min SDK is 15, we can include backwards support with the following child XML element:
    ```xml
    <meta-data
         android:name="android.support.PARENT_ACTIVITY"
         android:value="{parent.activity.package.goes.here}" />    
    ```
    - Note that this backwards compatibility uses a [Support Library](http://developer.android.com/tools/support-library/index.html) that needs to be part of your App. Android Studio includes it automatically.
    - Annnnd to turn on this up behavior in the Action bar, add the following method call to your SecondActivity. *Think*: what event handler should it be put in?
    ```java
    getSupportActionBar().setDisplayHomeAsUpEnabled(true);
    ```
    - This adds the "back/up" caret to the ActionBar. Explanatory from the method name, or can pull the documentation.
- In order to move between the Activities, let's add an interface element: a button we can click to start the second Activity.
- In **`activity_main.xml`** (the original Activity's layout), add the following code:
```xml
<Button
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="Start Activity"
    android:onClick="launchSecondActivity" />
```
This should go inside the `<RelativeLayout>` element, after the `<TextView>` element.
  - This defines a (anyone?) Button. The `android:text` attribute says the text that is on the button, and the `android:onClick` attribute says what method should be called when the button is clicked (just like with Swing or jQuery).
  - We'll talk in a lot more detail about how exactly this XML works (and what's the deal with the layout_width/layout_height) next week, but you should be able to make a pretty good educated guess based on the names.
  - Actually, to make the button not overlap, change the `<RelativeLayout>` to a `<LinearLayout>`. More on layout tomorrow!
- Since we said that the button should call a `launchSecondActivity` method, let's add that to the `MainActivity` class now. It will have a signature of:
```java
public void launchSecondActivity(View view)
```
  - The `View` parameter is a reference to the component (the Button) that called this method. Again, more later.
  - Try logging out `"Button pressed!"` when the button is pressed to check that everything works! Yay more logging practice!

In Android, we start new Activities not by instantiating them (remember, *we never instantiate Activities*!), but by sending them a message requesting that they do something (i.e., start). These messages are called [**Intents**](http://developer.android.com/reference/android/content/Intent.html).
  - Intents are *messages* used to communicate between Activities (which run in their own threads, so wouldn't be amenable to direct method callback). Generally used to request some kind of behavior from the other App.
  - I don't have a good justification for the name, though they do suggest purposeful (intentful) actions

[`Intents`](http://developer.android.com/reference/android/content/Intent.html) are something we _can_ instantiate, so let's do that in our event handler! There are lots of different constructors, but the one you'll want to use is:
```java
Intent intent = new Intent(this, SecondActivity.class);
```
- The first parameter refers to the [**Application Context**](http://developer.android.com/reference/android/content/Context.html), you can almost think of it as a reference to the "app" which can be used when accessing global resources (e.g., the names of other classes, shared messages, etc). Since `Activity` extends `Context`, we can just use `this` (we _are_ an `Activity`)~
- The second parameter is the class we want to send the Intent to (the `.class` property fetches a reference to the class type; this is metaprogramming!)

And now that we built the intent, we can use it to start an activity using the [`startActivity`](http://developer.android.com/reference/android/app/Activity.html#startActivity(android.content.Intent)) method (inherited from `Activity`), passing it the Intent!
- Voila! we can now start a second activity, and see how that impacts our Lifecycle calls (e.g., with visibility, etc).

There are actually a couple of different kinds of `Intents` (this is an **Explicit Intent**, because it is explicit about what Activity it's talking to), and a lot more we can do with them. We'll dive into Intents in more detail later; for now we're going to focus on mostly Single Activities.
- e.g., if you look back at the Manifest, you can see that the MainActivity has an `<intent-filter>` that allows it to receive particular kinds of Intents--including ones to use it when launching the App!


## Tasks & Back [10min]
So we can have lots of Activities (even across multiple apps!) running and move between them. How exactly is that "Back" button keeping track of where to go to?
- Do you know what kind of data structure is associated with "back" or "undo"? A **stack**!
- Everytime you start a new Activity, Android creates it and puts it on the top of a stack. Then when you hit te back button, that activity is popped off the stack and you're taken to the new head.
![activity stack example](http://developer.android.com/images/fundamentals/diagram_backstack.png)

However, you might have different "sequences" of actions you're working on: maybe you start writing and email, and then go to check your Twitter timeline through a different set of Activities. Android breaks up these sequences into groups called [**Tasks**](http://developer.android.com/guide/components/tasks-and-back-stack.html). A task is a collection of activities arranged in a Stack; and there can be multiple tasks in the background.
- Tasks usually start from the Home Screen. E.g., when you launch an Application, that starts a new Task.
- When you go back to homescreen, that Task is moved to the background, so the "back" button won't let you navigate that Stack.
- Thinking of them like different tabs/browsers and webpages is a pretty good analogy

Important caveat: Tasks are distinct from one another, so you can have different copies of the same Activity on multiple stacks (e.g., the Camera activity could be part of both Facebook and Twitter app Tasks if you are on a selfie binge)
  - Though it is possible to modify this, see [Managing Tasks](http://developer.android.com/guide/components/tasks-and-back-stack.html#ManagingTasks)

{{demo via some internal apps?}}

## Extra: Toasts [5min]
Logging is fantastic and one of the the best techniques we have for debugging, but in how Activities are being used or for any kind of bug (also RuntimeExceptions)
- Back to printline debugging, which is totes legit. Android Studio does have a [debugger](http://developer.android.com/tools/debugging/debugging-studio.html) if you're comfortable with those (can be handy!)

However, sometimes you want to check some output/interaction without Logging it. You just want to see some feedback while the app is running! Android provides a number of different classes for doing visual notifications, including alert-style and customizable [Dialogs](http://developer.android.com/guide/topics/ui/dialogs.html).

But a simple, quick way of giving some short visual feedback is to use what is called a [Toast](http://developer.android.com/guide/topics/ui/notifiers/toasts.html). This is a tiny little tetxbox that pops up at the bottom of the screen for a moment.
- Toast because it pops up :p

Toasts a pretty simple to implement, as with the following example (from the docs):
```java
Context context = getApplicationContext();
String text = "Hello toast!";
int duration = Toast.LENGTH_SHORT;

Toast toast = Toast.makeText(context, text, duration);
toast.show();
```
- But since `this` Activity _is_ a context, and we can just use the Toast anonymous, we can shorten this to a one-liner:
```java
Toast.makeText(this, "Hello world", Toast.LENGTH_SHORT);
```
  - Boom, a quick visual alert method we can use for proof-of-concept stuff!
- Note that this uses a static `makeToast()` method, rather than a constructor. This is an example of a Factory method--a design pattern we'll see a lot.
- Toasts are intended to be ways to interact with the user (e.g., giving them quick feedback), but can possibly be useful for testing too! Though in the end, Logcat is going to be your best bet.


## Action Items [2min]
- Finish the Warmup
- get started on the SunSpotter if you have a chance (though a lot of it is resource work, which we'll talk about on Tuesday)
- Review the documentation as needed; lots of details out there!
