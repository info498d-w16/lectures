## 03-01 Wearables

### Admin
- Any questions/issues with projects?

## Wearables
Today we're going to talk a little bit about building apps for [Android Wear](https://www.android.com/wear/).
- Why? Not because I think that wearables are the "next big thing" (I don't). But because it is a testable example of how the Android platform can be used across lots of different form-factors. I don't think smart-watches are all that great (but I could be wrong; I am not a futurist). But I do think embedded systems are a pretty cool, and I expect that knowing the wearable API is going to be informative for those systems as they appear.
- Anyone have a wearable and want to argue otherwise?
- Note that I don't have a watch, and have only begun to play with this SDK so today will be a bit "experimental" (but that's okay!)

### Setup
We need to get [setup](http://developer.android.com/training/wearables/apps/creating.html) First we need to make sure the Wear SDK is installed
- Update SDK tools (`Tools > Android > SDK Manager`)
  - Install wearable SDK (or update to `4.4W.2`)
  - Android SDK tools need to be at 23.0.0+

Next we need to set up a wearable device! Go to `Tools > Android > AVD Manager` and create a new device.
- Pick a wear model (I'm going with circle cause interface difference, and useful for showing how UI changes dramatically). Run Marshmallow so we have latest features.
- Then run the emulator!

It's common to have Wearable apps be partnered with a "traditional" phone app; for example, with the phone app running background services that the Wearable can use. This allows the wearable to offload a lot of the "heavy lifting" (things like network connections, etc) to the phone, acting a a kind of "thin client."
- We can [pair the emulator with an actual device](http://developer.android.com/training/wearables/apps/creating.html#SetupEmulator) to get these two things to connect. I tested it this way and it let me get a network connection. If we're not using network can get away without the phone... but being able to access the internet is more fun :p
  - To do this, will need to install the `Android Wear` app from Google Play, plug in your phone over USB, and then forward the communication from the emulator via
  ```
  adb -d forward tcp:5601 tcp:5601
  ```
  - Start up Android Wear app, and click "Connect Emulator" in the settings menu.

Finally, we can actually create a project to start building our app.
- We can build the two different pieces (wearable + mobile) at once!
- Make the mobile an `Empty` Activity, and the wearable a `Blank` Activity (will give us some starter code we can look at)

Now can launch app on the watch... and voila!

So what did we get? (Things to look at):
- Two separate "projects", each with own `Manifest`, `Activities`, `res/`, etc. We're going to focus on the **wear** one.

### Screen Shape
Remember how when we creates the emulator we had to pick square or round (or chin)? Watches can have drastically different screen shapes (unlike phones, which while the resolution/sizes may be different are still basically rectangles). So our UI work has to accommodate that.
- What happens if we try to just apply a square layout to a round screen?

![square vs. round image](http://developer.android.com/wear/images/01_uilib.png)

How might we handle these different layouts? Wear gives us two options. First, we can specify different layout resources for `rect` and `round` screens, and then simply inflate the appropriate one at runtime. This is the equivalent to defining different portrait and landscape (`land`) resources.
- But rather than specifying a different configuration folder, we're instead going to use a [`WatchViewStub`](http://developer.android.com/reference/android/support/wearable/view/WatchViewStub.html). Does everyone remember the `ViewStub` View you (should have) used in the Sunspotter assignment? This is the same kind of thing--a "placeholder" for a view that will be inflated into the layout.
- In particular, the `WatchViewStub` specifies two attributes
  ```xml
  <android.support.wearable.view.WatchViewStub
      app:rectLayout="@layout/rect_activity_wear"
      app:roundLayout="@layout/round_activity_wear">
  </android.support.wearable.view.WatchViewStub>
  ```
  Which then specify which view should get inflated depending on the screen configuration.
- (The provided code also does some work to lookup the view inside those resources only _after_ they've been loaded. You can actually use the same pattern in the regular SDK).

The second solution I like a little more, because it's overall less work. This is useful if we want the layouts to be the same, but we want everything to automatically fit inside the round screen and not get cut off. Effectively, we want to lay things out only in the **inscribed rectangle**.
- This is equivalent to deciding that a round screen is just a square screen with lots of border space, so we'll just use the square portion and treat them as having different resolutions just like we have in the past.
- If we were trying to do this work ourselves, how would we make it so that the content was only inside the inscribed rectangle? _Padding!_
  - Of course, this padding would need to be different depending on the configuration

Luckily, Android gives us a class to applies this padding automatically, called [`BoxInsetLayout`](http://developer.android.com/reference/android/support/wearable/view/BoxInsetLayout.html). This acts as a `ViewGroup`-esque "container" for our content, and will apply padding to the contained elements to make them fit on round screens
- But to do that, we need to specify an attribute of the _inside_ element, so clarify what kind of padding we want! This is the `app:layout_box="all"` property. For example:
```xml
<android.support.wearable.view.BoxInsetLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_height="match_parent"
    android:layout_width="match_parent">
    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_box="all">
        <TextView
            android:gravity="center"
            android:layout_height="wrap_content"
            android:layout_width="match_parent"
            android:text="Hello World"
            android:textColor="@color/white" />
    </FrameLayout>
</android.support.wearable.view.BoxInsetLayout>
```
- The docs suggest to use `layout_gravity` to position elements, we can use `|` to combine values together (e.g., `center|right`).
- Note the slick part of this is that the `BoxInsetLayout` fills the screen, but the `FrameLayout` content does not! If we assign colored backgrounds, we can see how this works.


### Cards
There are [a number][ui library] of different wear-specific layouts we can use, and I'd encourage you to look through those options. We're only going to demo a few of them today (for time).

One simple example of a UI specific to wearables is [Cards](http://developer.android.com/training/wearables/ui/cards.html), which are basically chunks of content that usually appear along the bottom of the watch (a common UI element). Luckily, cards are pretty easy to create... and again, there are two options:

First, we can define the card in code by using a `CardFragment`. This will actually work really similarly to fragments we've already worked with (and so is good review).
- First we'll want to make sure our `FrameLayout` (inside our `BoxInsetLayout`) has an id so we can inflate the fragment into it.
- We can then create the `Fragment` in code. While we could probably subclass `CardFragment`, it's easier to just use a [factory](http://developer.android.com/reference/android/support/wearable/view/CardFragment.html) method to produce a generic card for us:
```java
CardFragment cardFragment =
    CardFragment.create("title", "description", R.drawable.p);
```
- Finally, we can use a `FragmentManager` (just like before) to put our fragment into the layout:
```java
FragmentManager fragmentManager = getFragmentManager();
FragmentTransaction fragmentTransaction = fragmentManager.beginTransaction();
fragmentTransaction.add(R.id.frame_layout, cardFragment);
fragmentTransaction.commit();
```

Note: alternatively, we can specify the Card inside the `xml` resource [directly](http://developer.android.com/training/wearables/ui/cards.html#card-layout): Create a `android.support.wearable.view.CardFrame` to hold custom content (the `CardFrame` does stuff like sizing and adding shadow borders, etc). This should then go inside a `android.support.wearable.view.CardScrollView`, which "detects the shape of the screen and displays the card differently on round and square devices, using wider side margins on round screens".
- Per the documentation: "However, placing the `<CardScrollView>` element inside a `<BoxInsetLayout>` and using the `layout_box="bottom"` attribute is useful to align the card to the bottom of round screens without cropping its content."

### ViewPager
If we want to have multiple screens, we could use multiple Activities and just send `Intents` between them to start the new piece. But an alternative structure is to use _Fragments_ to do this work.
- So how do we swipe between Fragments? Luckily, Android provides a class that does a lot of this work: called [`ViewPager`](http://developer.android.com/training/animation/screen-slide.html). We can use this class on mobile (and there is a tutorial for making it work), but we can also use it on mobile as well... so let's get it setup (and then you can adapt this work for your final project potentially!)

A good way of learning to use these classes (and really, to use _any_ class or library at all) is to look at an example. Google provides a number of [samples](http://developer.android.com/samples/index.html) (in a painful to navigate list) that you can work off of.

Let's look at one called [Flashlight][flashlight sample] to try and figure out how to use a `ViewPager`
- Check out the `MainActivity`. Can you find the `ViewPager`? Notice it's part of the layout (since we find it), so we'll need to add that in.
- We then need an `Adapter` (similar to what we did with `ListViews`). In this case, it's an inner-class
  - The inner class is pretty generic, lets copy it over
- We then instantiate our Fragments and add them to the adapter
  - We need to create Fragments! That's okay, we've done that before. Fill in `onCreateView()` to inflate a view resource, which we can define as a simple `TextView`

And that's about it (the other stuff is "flashlight specific"). So now we can get multiple screens in our app that slide around!
- Note that wearable also includes a [`GridViewPager`](http://developer.android.com/reference/android/support/wearable/view/GridViewPager.html) which works the same way, but allows both horizontal _and_ vertical scrolling!


### ListView... if time (probably not)
We can also make lists of things that scroll nicely on wearables (letting us swipe to navigate, and "snapping" to the selected one). To do this, we can use a [`WearableListView`](http://developer.android.com/training/wearables/ui/lists.html), which works almost exactly like the `ListViews` we've talked about previously.
- Let's make a Second Activity to support this. Note that we could instead use a `ViewPager` or even a `WearableGridView`, but the second Activity is less new stuff for us to talk about (unless want to go over the `ViewPager`... if so, see [this sample][flashlight sample]).
  - We'll do more "learn from samples" later!
  - [do this, so can work through a single Activity and share data variables]

Steps for adding the list view
1. Add a `<android.support.wearable.view.WearableListView>` inside the `FrameLayout` (in the `BoxInsetLayout`).
2. Create a `list_item_layout.xml` resource (with a `TextView` for our content)
3. Create the adapter to put data in our list! We'll actually need to make our own custom one... so we can work off the [doc sample][wearable adapter]
4. And finally we connect them!


### Interaction: Speech
So far all we've done has been layout work, which is similar to what you've done in mobile (and hopefully is a useful review/extension). But interacting with a watch is more tricky: you don't have a keyboard, can only tap and swipe (and only have one hand!)... how do you do anything more complex _effectively_?
- One neat solution is to just ignore your hands entirely and just use your mouth... by which I mean [voice commands](http://developer.android.com/training/wearables/apps/voice.html). These are actually pretty easy to integrate into a wearable app (and phone apps as well).

Start with the most general process: getting free-form voice input! This can be done simply by launching an `Intent` for a separate activity (e.g., one provided on the wearable or on your phone) that responds to the `RECOGNIZE_SPEECH` action:
```java
private void displaySpeechRecognizer() {
    Intent intent = new Intent(RecognizerIntent.ACTION_RECOGNIZE_SPEECH);
    intent.putExtra(RecognizerIntent.EXTRA_LANGUAGE_MODEL,
            RecognizerIntent.LANGUAGE_MODEL_FREE_FORM);
    // Start the activity, the intent will be populated with the speech text
    startActivityForResult(intent, SPEECH_REQUEST_CODE);
}
```
And then when we get the result back, we can do further processing on what the user said!
```java
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    if (requestCode == SPEECH_REQUEST_CODE && resultCode == RESULT_OK) {
        List<String> results = data.getStringArrayListExtra(
                RecognizerIntent.EXTRA_RESULTS);
        String spokenText = results.get(0);
        // Do something with spokenText. E.g., log out.
    }
    super.onActivityResult(requestCode, resultCode, data);
}
```
This is pretty easy and actually works (if you have the phone plugged in so can perform network connections to talk to Google's recognition system...)

It's also possible to let your App be able to respond to particular commands: e.g., "call a cab". You can do this by specifying that your Activity responds to particular `Intents`, which are send when the system hears a vocal command (usually prefaced with "Ok Google").
- See the list of [wearable voice intents](http://developer.android.com/training/wearables/apps/voice.html)
- This should be trivial to set up... but I haven't managed to get it to work (in part because the emulator isn't responding to my "OK Google"... though the explicit voice recognizer was working...)
- But really, should work the same way on mobiles. See [this guide](https://developers.google.com/voice-actions/system/)


### Notifications
Notifications are a major part of the watch UI... in part because that's one of the main use-cases for this expensive piece of hardware :p In this case, notifications are expected to be paired with a phone.
- The idea is you create notifications from the phone app, and those automatically show up on the Wearable! And instead you can just add extra features to the notification as it appears on the watch
- Can "demo" this (I think) by downloading and running the [Notifications sample](http://developer.android.com/samples/Notifications/project.html)

- [[can explore]]...

### Watch Faces
Last set of samples to look at (if time): a big part of watches is what's on the face: e.g., how do you show the time? Or maybe you want to be showing other information (like current speed or heartrate if you're running). Android Wear lets you develop your own [Watch Faces](http://developer.android.com/training/wearables/watch-faces/index.html), which are basically `Services` that are run in the background to update a displayed View... which is the watch's face.
- See the [WatchFace sample](http://developer.android.com/samples/WatchFace/project.html)
  - [[explore this one too...]]

### Wrap-up
This was intended to be a fun "first look" at wearables (was for me). Lots more pieces, but wanted to give you a taste of things you might consider if you want to build apps for watches.
- Also covered things that may be useful in your projects, like `ViewPager` and voice commands
- And practice using an existing code sample to learn how things work, and hopefully demonstrated why checking out samples is actually a good idea!
  - Other fun sample to play with on watch: [Eliza](http://developer.android.com/samples/ElizaChat/project.html)
  - Wear also has a big "Data Layer Connection" to allow synchronization between mobile and wearable; check out the [Quiz sample](http://developer.android.com/samples/Quiz/project.html) as an illustration.

Next time:
- Lab is a work day again this week (since group projects; keep pushing on them!) Due a week from Thursday.
- Thursday we'll try to demo some AR stuff, though it's not as well-defined as I had hoped so our sample might not be that interesting...
  - Could do more wear stuff? Or any other topics to cover?

---

<!-- References -->
[ui library]: http://developer.android.com/training/wearables/apps/layouts.html#UiLibrary
[flashlight sample]: http://developer.android.com/samples/Flashlight/project.html
[wearable adapter]: http://developer.android.com/training/wearables/ui/lists.html#adapter

#### References
[UI Widgets][ui library]
[Flashlight Sample][flashlight sample]
[Wearable Adapter][wearable adapter]
