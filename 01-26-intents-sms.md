## 01-26 Intents and SMS

### Admin [5min]
- Homework check-in
  - How did the self-tracker end up? I apologize for errors/issues with the writeup. I'm designing these assignments from scratch and haven't had time to fully test them out (though all have "proof of concept")...
  - This week we're going to one more "broader" assignment (and I've tested this one a bit more), and then next week's will be more focused (read: smaller).
  - But you can do this one in pairs! About making a messaging client, so nice to be able to send messages to another person :)
- Fork and clone the repo for today! No more movies (we'll do other stuff)
  - Open up AVD and let's hook up the camera (since we'll play with it briefly)
    - Choose "webcam" from list of options (for front camera, makes sense).
    - When start up the camera app, should see it working!

## Context [15min]
Before we begin, I want to address one concept that was flagged in the _last_ homework that I've been remiss about: [`Context`](http://developer.android.com/reference/android/content/Context.html).
- Context is an **abstract class** that acts as a reference for information about the current running environment; it represents environmental data (stuff like "what OS is running? Is there a keyboard plugged in?"). You can _almost_ think of it as representing the "Application", though it's broader than that (since `Application` is actually a subclass of `Context`!)
- The context is _used_ to do "application-level" actions: mostly working with resources (accesing/loading), but also communicating between Activities (with `Intents`, which we'll talk about today), or otherwise accessing system services. Effectively, it lets us refer to the state in which we are running: the "context" for our code (e.g., "where is this occuring?"). It's a kind of _reflection_ or meta-programming, in a way.

There are a couple of different kinds of Contexts we might use:
- The Application context (e.g., the `Application`) references the state of the entire application. It's basically the Java object that is built out of the Manifest (and so contains that level of information)
- The Activity context (e.g., the `Activity`) that references the state of that activity. Again, this would be the `<activity>` tags from the Manifest.

In general, we tend to use the Activity context for loading resources, which is usually fine. But it's worth noting that each of these `Context` objects exist for the life of their respective component: that is, an `Activity` Context is around as long as the Activity exists (disappearing after destroy), where as `Application` Contexts survive as long as the application does (and are in fact a __singleton__).
- Where it starts to matter is when you want to _save a reference_ to a Context, such as by making a static variable that holds a View (that references its Context, and so keeps the entire Activity in member). Thus it can be safer to use the `Activity` context, which is a singleton so can be referenced without causing memory leaks (see [here](http://android-developers.blogspot.com/2009/01/avoiding-memory-leaks.html)).

Note that as was pointed out last week, the `Application` and `Activity` can have different themes/stylings associated with them. This will effect any Views loading in that `Context`. Similar in spirit to "am I opening Chrome on a Mac or on Windows?"

We're not going to get into `Context` in a lot more detail, we mostly need to pass it as a reference into various functions that are doing resource or system work, like loading Views or sending Intents. Which is what we'll talk about now!


## Intents
What is an [Intent](http://developer.android.com/guide/components/intents-filters.html)? A **message** that is sent between app components, allowing them to communicate!
- Most object communication we do is via _direct method call_; you have a reference to an Object and then you call a method on it. We've also seen _event callbacks_, where on an event one of our callbacks gets executed by the system (really just a wrapper around _direct method call_ via Observer pattern)
- Intents step outside of this a little bit: they allow us to create objects that can be "given" to another component (read: Activity), who can then respond upon receiving that. Similar to an event callback, but working at a slightly higher system level.

You can think of Intents as like envelopes: they are addressed to a particular target (e.g., another Activity--more properly a `Context`), and have room for some data called **extras** to go inside (held in a `Bundle`). When the envelope arrives, the recipient can get that data out and do something with it... possibly sending a response back.

Note that there are couple of different kinds of Intents; we'll go through examples of each.


### Intents for another Activity (Implicit) [10-15min]
The most basic kind of Intent is one we've already seen: an Intent sent to a specific Activity/Context, such as for telling that Activity to open.

So just like we did during the first week, we can use an Intent to start our Second Activity from our first. First we'll construct the [`Intent`](http://developer.android.com/reference/android/content/Intent.html), and then we send it off using the `startActivity()` method:

```java
//                         context, target
Intent intent = new Intent(MainActivity.this, SecondActivity.class);
startActivity(intent);
```

This is called an **Explicit Intent** because we're _explicit_ about what target we want to receive it. It's a letter to a specific Activity.

#### Extras
We can also specify some extra data inside our envelope. This data is referred to as **Extras**. This is basically a `Bundle` (so a set of key-value pairs) that we can use to pass _limited_ information around!

```java
intent.putExtra("package.name.key","value");
```
- Docs say that best practice is to include the full package name on keys, so avoid any collisions or misreading of data. There are also some pre-defined values (constants) that you can use in the `Intent` class.

We can then get the extras from the Intent in the Activity that receives it:
```java
//in onCreate();
Bundle extras = getIntent().getExtras();
String value = extras.getString("key");
```

So we can have Activities communicate, and even share information between them! Yay!


### Intents for another App (Explicit) [10-15mins]
We can send Intents to our own Activities, or even to other Apps. When calling on other apps, we usually use **Implicit Activities**.
- This is a little bit like letters that have weird addresses, but still get delivered. "For that guy at the end of the block with the red mailbox."

An Implicit Intent includes an **Action** and some **Data**. The __Action__ says what the target should _do_ upon receiving the intent (a Command), and the ___Data___ gives more detail about what to run that action on.
- **Actions** can be things like `ACTION_VIEW` to view some data, or `ACTION_PICK` to choose an item from a list. See a full list under ["Standard Action Activities"](http://developer.android.com/reference/android/content/Intent.html).
  - `ACTION_MAIN` is the most common (just start the Activity as if it were a "main" launching point). So when we don't specify anything else, this is used!
- **Data** gives detail about what to do with the action (e.g., the Uri to `VIEW` or the Contact to `DIAL`).
  - Extras then support this data!

For example, if we specify a `DIAL` action, then we're saying that we want our Intent to be delivered to an App that is capable of dialing a telephone number.
- _If there is more than one app that supports this action, the user will pick one!_ This is key: we're not saying exactly what app to use, just what kind of functionality we need to be supported! It's a kind of abstraction!

```java
Intent intent = new Intent(Intent.DIAL);
intent.setData(Uri.parse("tel:206-685-1622"));
if (intent.resolveActivity(getPackageManager()) != null) {
  startActivity(intent);
}
```

Here we've specified the Action (`DIAL`) for our Intent, as well as some Data (a phone number, converted into a Uri). The `resolveActivity()` method looks up what Activity is going to receive our action--we check that it's not null before trying to start it up.
- This should allow us to "dial out" !

Note that we can open up all sorts of apps. See [Common Intents](http://developer.android.com/guide/components/intents-common.html) for a list of common implicit events (with examples!).


#### About 11:15 ?


### Intents for a response [10-15min]
We've been using intents to start Activities, but what if we'd like to get a result _back_ from the Activity? That is, what if we want to look up a Contact or take a Picture, and then be able to use the Contact or show the Picture?

To do this, we're going to create Intents in the same way, but use a different method to launch them: [`startActivityForResult()`](http://developer.android.com/guide/components/activities.html#StartingAnActivityForResult). This will launch the resolved Activity. But once that Action is finished, the launched Activity will send _another_ Intent back to us, which we can then react to in order to handle the result.

For fun, let's do it with the Camera--we'll launch the Camera to take a picture, and then get the picture and show it in an `ImageView` we have.
- Note that your Emulator will need to have Camera emulation on!
- See [Taking Photos Simply](http://developer.android.com/training/camera/photobasics.html) for walkthrough.

In the activity, we can specify an intent that uses the `MediaStore.ACTION_IMAGE_CAPTURE` action (the action for "take a still picture and return it").
- The "request code" is used to distinguish this intent from others we may send (kind of like a "tag").
- Note that we could pass an Extra for where we want to save the large picture file to. However, we're going to leave that off and just work with the thumbnail for this demonstration. See the [guide](http://developer.android.com/training/camera/photobasics.html#TaskPath) for details; if time we can walk through it!

```java
static final int REQUEST_IMAGE_CAPTURE = 1;

private void dispatchTakePictureIntent() {
    Intent takePictureIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
    if (takePictureIntent.resolveActivity(getPackageManager()) != null) {
        startActivityForResult(takePictureIntent, REQUEST_IMAGE_CAPTURE);
    }
}
```

In order to handle the "response" Intent, we need to provide a callback that will get executed when that Intent arrives. Called `onActivityResult()`.
- We can get information about the Intent we're receiving from the params. And we can get access to the returned data (e.g., the image) by getting the `"data"` field from the extras.
- Note that this is a [`Bitmap`](http://developer.android.com/reference/android/graphics/Bitmap.html), which is the Android class representing a raster image. We'll play with Bitmaps more in a couple weeks, because I like graphics.

```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    if (requestCode == REQUEST_IMAGE_CAPTURE && resultCode == RESULT_OK) {
        Bundle extras = data.getExtras();
        Bitmap imageBitmap = (Bitmap) extras.get("data");
        mImageView.setImageBitmap(imageBitmap);
    }
}
```


### Listening for Intents [10-15min]
We're able to send implicit Intents that can be heard by other Apps, but what if we wanted to receive implicit Intents ourselves? What if _we_ want to be able to handle phone dialing?!

In order to receive an implicit Intent, we need to declare that our Activity is able to handle that request. Since we're specifying an aspect of our application, we'll do this in the `Manifest` using what is called an `<intent-filter>`.
- The idea is that we're "hearing" all the intents, and we're "filtering" for the ones that are relevant to us. Like sorting out the junk mail.

An `<intent-filter>` tag is nested inside the element that it applies to (e.g., the `<activity>`). In fact, you can see there is already one there: that responds to the `MAIN` action sent with the `LAUNCHER` category (meaning that it responds to intents from the app launcher).

Similarly, we can specify three "parts" of the filter:
- a `<action android:name="action">` filter, which describes the Action we can respond to.
- a `<data>` filter, which specifies aspects of the data we accept (e.g., only respond to Uri's that look like telephone numbers)
- a `<category android:name="category">` filter, which is basically a "more information" piece. You can see the ["Standard Categories"](http://developer.android.com/reference/android/content/Intent.html) in the documentation.
  - Note that you _must_ include the `DEFAULT` category to receive implicit intents. This is the category used by `startActivity()` and `startActivityForResult`.

Note that you can include multiple actions, data, and category tags. You just need to make sure that you can handle all possible combinations (they are "or" not "and" filters!)

Responding to that dial command:
```xml
<activity android:name="SecondActivity">
    <intent-filter>
        <action android:name="android.intent.action.DIAL"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <data android:mimeType="text/plain"/>
    </intent-filter>
</activity>
```  

You can see many more examples in the [`Intent`](http://developer.android.com/reference/android/content/Intent.html) documentation.


//about 11:50?


### Broadcast Receivers [10min]
There is one other kind of Intent I want to talk about: [**Broadcasts**](http://developer.android.com/reference/android/content/BroadcastReceiver.html). A broadcast is a message that _any_ app can receive. Unlike Explicit and Implicit Intents, broadcasts are heard by the entire system--anything you "shout" with a broadcast is publicly available (security concerns!)

Other than who receives them, broadcasts work much like normal implicit intents! We create an Intent with an Action and Data (and Category and Extras...). But instead of using the `startActivity()` method, we use the `sendBroadcast()` method. That intent can now be heard by all Activities on the phone,
- (can skip demo for time and motivation)

But more common than sending broadcasts will be _receiving_ broadcasts; that is, we want to listen and respond to System broadcasts that are produced (things like power events, wifi status, etc). Or more germane to our homework--to incoming text messages!!

We can receive broadcasts by using a [`BroadcastReceiver`](http://developer.android.com/reference/android/content/BroadcastReceiver.html). This is a base class that is used by an class that can receive broadcast Intents. We **subclass** it and implement the `onReceive(Context, Intent)` callback in order to handle when broadcasts are received.

```java
public void onReceive(Context context, Intent intent)
{
    Log.v("TAG", "received! "+intent.toString());
    if (intent.getAction() == SMS_RECEIVED) {
        SmsMessage[] messages = Intents.getMessagesFromIntent(intent); //CHECK?!
    }
}
```

But in order to register our receiver, we also need to specify it in the `Manifest`. We do this by including a `<receiver>` attribute inside our `<application>`. Note that this is _not_ an Activity, but a separate component! We can put an `<intent-filter>` inside of this to filter for broadcasts we care about.

```xml
<receiver android:name=".SMSReceiver">
    <intent-filter>
        <action android:name="android.provider.Telephony.SMS_RECEIVED" />
        <!-- no category because not for an activity! -->
    </intent-filter>
</receiver>
```

## SMS
To show this off, let's get it responding to incoming text messages (which is part of what you need to do for your homework). Specifically, we're responding to SMS (Short Messaging Service) messages, the most popular form of data communication in the world.
- _Important note:_ the SMS APIs changed *drastically* in KitKat (API 19). So we're going to make sure that is our minimum so we can get all the helpful methods and support newer stuff (check gradle to confirm!).

#### Receiving
We can specify that our receiver filters for SMS messages that have been `RECEIVED` (not _DELIVERED_), per example above.
- We also will need to include permission: `<uses-permission android:name="android.permission.RECEIVE_SMS" />`

Then in our `BroadcastReceiver`, we can check the Intent's action (with `getAction()`), and if it's the `RECEIVE_SMS` we can fetch the `SmsMessage` objects:
```java
SmsMessage[] messages = Intents.getMessagesFromIntent(intent);
for(SmsMessage msg : messages) {
  Log.v(TAG, msg.getOriginatingAddress() + ": " +msg.getMessageBody());
}
```

How do we test this? If you have a phone, good times. But the emulator doesn't have phone service, so can't get SMS messages! But luckily, we can fake it!
- Note that when we started up the emulator it had a number. This is the _port_ it is running on. This is a basic networking port... so we can connect to it via something like `telnet`!
  - `telnet localhost 5554`
  - `sms send <from-number> "<message>"`
- More details about sending and receiving SMS through the Emulator in the homework write-up (I tested this extensively!)

#### Sending
What about sending SMS? It's actually not too hard! The main thing to note is that as of KitKat, each system has a _default_ messaging client---who is the only one who can actually send messages. Luckily, the API lets you get access to that messaging client's services in order to send a message _through_ it:

```java
SmsManager smsManager = SmsManager.getDefault();
smsManager.sendTextMessage("5554", null, "This is a test message!", null, null);
//                         target,       message
```
We will need permission: `<uses-permission android:name="android.permission.SEND_SMS" />`

Can see this works by looking at the inbox in the Messages app.

#### Reading
In order to see messages that have already been sent, we can query a [`ContentProvider`](http://developer.android.com/guide/topics/providers/content-providers.html) provided by the system that "provides" the "content" of the [SMS Inbox](http://developer.android.com/reference/android/provider/Telephony.Sms.Inbox.html). It holds the "database" of messages received. (This is how we _should_ have interacted with our SQLite database)!
- A `ContentProvider` is defined by it's `URI` (like a webpage url), which usually has the protocol `content://` (instead of `http://`).

We can query this directly by using the `contentResolver` and specifying the URI. This will give us back a `Cursor` we can iterate through, much like how you did with your Database (I expect).

```java
Uri uri = Telephony.Sms.Inbox.CONTENT_URI;
Cursor c = getContentResolver().query(uri, null, null, null, null);

c.moveToFirst();
while(!c.isAfterLast()){
    String address = c.getString(c.getColumnIndexOrThrow("address"));
    String subject = c.getString(c.getColumnIndexOrThrow("subject"));
    String body = c.getString(c.getColumnIndexOrThrow("body"));
    long date = c.getInt(c.getColumnIndexOrThrow("date"));
    int type = c.getInt(c.getColumnIndexOrThrow("type"));

    Log.v(TAG, address + ", "+subject+", "+body+", "+date+", "+type);
    c.moveToNext();
}
```
Note we'll need permission: `<uses-permission android:name="android.permission.READ_SMS" />`


A better strategy though is to use a `Loader` to be able to load data in the background! his is basically a wrapper around `ASyncTask`, but one that let's you quickly specify what action should occur in the background thread. In particular, Android provides a [`CursorLoader`](http://developer.android.com/reference/android/content/CursorLoader.html) specifically used to load Cursors (like what you'd get from a database, or from the SMS Inbox).
  - To use a `Loader`, you need to specify that your _Fragment_ implements the [`LoaderManager.LoaderCallback`](http://developer.android.com/reference/android/support/v4/app/LoaderManager.LoaderCallbacks.html) interface---basically saying that this fragment can react to Loader events.
    - (Loaders need to work with Fragments. If you have your Activity extends [`FragmentActivity`](http://developer.android.com/reference/android/support/v4/app/FragmentActivity.html), you can get the "Fragment" capabilities to use a Loader in an Activity). Note that `AppCompatActivity` is a subclass of this so should work.
  - You can then specify what happens the Loader should _do_ in the `onCreateLoader()` callback. Here you would instantiate and return a `CursorLoader` that queries the `ContentProvider` (e.g., the SMS Inbox). Note that the third parameter is a "projection", which is just a list of columns you want to get from the data store (you can use constants from the [`Telephony.Sms.Conversations`](http://developer.android.com/reference/android/provider/Telephony.TextBasedSmsColumns.html) class; most of the ones of note are inherited from [`TextBasedSMSColumns`](http://developer.android.com/reference/android/provider/Telephony.TextBasedSmsColumns.html)).
    - Be sure to include the `_ID` as well!
  - Finally, in the `onLoadFinished()` callback, you can `swap()` the `Cursor` into your `SimpleCursorAdapter` in order to feed that model data into your controller (for display in the view). See the [guide](http://developer.android.com/guide/components/loaders.html) for more details.

```java
String[] columnToFetch = {Telephony.Sms.Conversations._ID, Telephony.Sms.Conversations.ADDRESS, Telephony.Sms.Conversations.BODY};

//runs the query
return new CursorLoader(
        getActivity(),   // Parent activity context
        uri,        // Table to query
        columnToFetch,     // Projection to return
        null,            // No selection clause
        null,            // No selection arguments
        null             // Default sort order
);
```

### Other Connectivity
Before you go, I want to flag that this structure of working with the "manager" for SMS is actually pretty common among all kinds of different connectivity. For example **Bluetooth**, which I think we'll play around with for lab on Thursday :)
- You'll need to have a physical device (no service needed). You can check one out from IT down the hall if necessary!! You'll want that device for our _following_ week's assignment as well (doing location-based stuff).

Quickly: what is Bluetooth? [Bluetooth](http://developer.android.com/guide/topics/connectivity/bluetooth.html) is a communication standard (like "wifi") used for low-power communication between devices that are _co-located_, or physically nearby.
  - Co-location is huge! The whole point of "mobile" development is that you're working with devices that are mobile--they can move. Their position and location matters. It's what separates them from desktops/laptops, and it's why we do all this work to make things run.
  - My first notable Android project was playing around with this stuff

Bluetooth is used for all kinds of things: headsets, music players, quick communication, sharing contact cards, etc.


With Bluetooth, there are a couple of pieces to connecting. First you need to find other devices nearby to connect to them. This is called _discovery_. You basically send out a "ping" over the radio, and any devices that are listening (are set to be "discoverable") will respond with their [MAC address](https://en.wikipedia.org/wiki/MAC_address) address (and can answer a second request about  a list of services they provide). Once you've found a device, you can send a request to "pair" with that device--basically to establish a communication stream between them.
- Pairing is "master/slave" based, where one is the "master" (the server) and one is the "slave" (the client), though the roles can be switched over the protocol.
- Bluetooth spec actually allows you to connect to up to 7 devices at once (in a "piconet"), though hardware antennas may limit this. There's also way to have some of those devices act as masters for their _own_ piconets, forming a full scatter-net!

Tomorrow in lab we'll play around with finding and connecting to other devices. Because it's related to SMS (another communication medium!) and it's fun ;p
