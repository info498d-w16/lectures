## 01-28 UI Elements: Menus, Notifications, Settings

### Admin [5min]
- Homework check-in
  - Self-tracker got turned in with extra day? Hopefully moving deadlines to Wednesdays will match actual working schedule more.
  - Remember to get started on the next one as well!
- Fork and clone the repo for today; working off of what we had finished yesterday!

Today is going to be a bit of a grab-bag of topics. The overall theme is "user interaction"---other patterns/tools for communicating with the user.
- We already have some tools to use: input controls (`Buttons`, `EditTexts`), alerts, toasts. So today we'll look at a few more!
- Note we're aiming at exposure/familiarity, rather than in-depth mastery. So you are aware of some basic components, then you can look up in the documentation how to customize them to best fit your needs.

## Menus
Let's start with the most prominent visual element in our default app: the [___App Bar___](http://developer.android.com/training/appbar/index.html) or ___Action Bar___. This acts as the sort of "header" for your app, providing a dedicated space for navigation and interaction (e.g., menus).
- The `AppCompatActivity` that we've been using provides this, but we can also add it ourselves (e.g., if we're not using the default provided by our theme!)
- The [`ActionBar`](http://developer.android.com/reference/android/support/v7/app/ActionBar.html) is a generalization of a [`ToolBar`](http://developer.android.com/reference/android/support/v7/widget/Toolbar.html), which is basically how you can add "action bars" where-ever you want (e.g., in the middle of your app, or if you want it to stick to the bottom.
  - To add your own Action Bar, you specify a _theme_ without an ActionBar, and then include a `<android.support.v7.window.Toolbar>` element inside your layout wherever you want the toolbar to go. See [Setting up the App Bar](http://developer.android.com/training/appbar/setting-up.html) for details; we'll probably return to this later when we talk about Themes and Styles.

We can get access to the ActionBar by calling the `getSupportActionBar()` method. We can then call utility methods on it to interact with it; for example `.hide()` will hide the toolbar!

However, the main use for the ActionBar is a place to hold [___Menus___](http://developer.android.com/guide/topics/ui/menus.html). We went over this briefly the other day, but let's revisit it!
- A Menu (specifically, an [**options menu**](http://developer.android.com/guide/topics/ui/menus.html#options-menu)) is a set of items (think buttons) that appear in the ActionBar
  - pre-API 11 they appeared at the bottom of the screen!
  - Menus can be declared both in the `Activity` and in a `Fragment`; if they are in both places, they are combined into the ActionBar (so you can have menus be contextual pretty easily).

Menus are defined as `xml` resources, specifically of type `menu`.
- can create one by selecting `New > Android Resource File` and then the `Menu` Resource type. This gives you an xml file with a main [`<menu>`](http://developer.android.com/reference/android/view/Menu.html) element.

We get the menu to show up by overriding the `onCreateOptionsMenu` callback, and then using the `MenuInflater` to expand the menu:
```java
public boolean onCreateOptionsMenu(Menu menu) {
    MenuInflater inflater = getMenuInflater();
    inflater.inflate(R.menu.main_menu, menu);
    return true;
}
```

We can then add items inside the XML menu, particularly [`<item>`](http://developer.android.com/reference/android/view/MenuItem.html) elements
- Common `<item>` attributes include: `id` (to refer to it later), `icon` (image to show), `title` (text to show), and `showAsAction` (for whether it should be listed in the ActionBar or collapsed under a button). See [Menu resources](http://developer.android.com/guide/topics/resources/menu-resource.html) for the full list of options!
- Note that we can use the `android:drawable/ic_*` elements for built-in icons! [Android Drawables](http://androiddrawables.com/) includes the full list!
- Example: Let's make some "shortcuts" for our call and message buttons :)
- We can also add one level of submenus (`<menu>` inside an `<item>`). And we can also use `<group>` tags to group the items together. We can give the group an idea to refer to the "group" of icons, and if one icon is hidden then the others will be as well. `orderInCategory` is useful for ordering within the group.

We then respond to these items being selected by filling in the `onOptionsItemSelected()` callback. Use a `switch` on the `item.getItemId()` to determine what item was selected, and then act accordingly.
- On default, pass up to `super`.


But wait there's more! We can add [Action Views](http://developer.android.com/training/appbar/action-views.html) that are expandable widgets in the action bar (e.g., search example). Or we can add an [Action Provider](http://developer.android.com/training/appbar/action-views.html#action-provider) (like [`ShareActionProvider`](http://developer.android.com/training/sharing/shareaction.html)), which gives us a bunch of interaction built into the menu!
- Skip the demo, or copy source code from the guide.

```java
Intent shareIntent = new Intent();
shareIntent.setAction(Intent.ACTION_SEND);
shareIntent.putExtra(Intent.EXTRA_STREAM, uriToImage);
```

### Context Menus
In addition to ActionBar Option menus we can also use menus as contextual pop-ups (on long-press).
- start by using `registerForContextMenu()`, passing it the `View` we want to be able to create the menu for
- We then specify to menu to use through the `onCreateContextMenu()` callback, just like we did for Option menus!
  - We can even pass it the same menu if we wanted
- We respond to _those_ items being selected with the `onContextItemSelected()` (see a pattern)? So basically works the same way, but different set of callbacks.

That's a whirl-wind tour of menus. As always, there are lots more options and complex things you can do with them!

### 5min break?

## Notifications
We can let the user know what's going on by popping up and alert or a toast, but often we want to notify the user of something outside of the normal UI (e.g., when the app isn't running, or without getting in the way of stuff). To do this, we can use [Notifications](http://developer.android.com/guide/topics/ui/notifiers/notifications.html). These show up in the **notification area** (the icons at the top) and in the system's **notification drawer**, which the user can get to at any point (even when outside the app) by swiping down.

Android's documentation for UI components is overall pretty good (because Google wants to make sure people can build effective apps so the platform is worthwhile!) And because there are so many different UI elements and they change all the time, in order to do real-world Android development you need to be able to read, synthesize, and apply this documentation.
  - So for this morning, we're going to practice doing that! Everyone open up the **Notifications** page (google search "android notifications" and you'll find it). Let's walk through the process using Google's documentation, to get a sense for how we can follow their explanation.
  - OUR GOAL: when we hit another button, we want to make a notification that says we did, and how many times we hit it!

Okay, looking at the documentation we see an overview. Also a link to the **Design Guide**, which is a good place to go to figure out how to design _effective_ notifications

There is a lot of text about how to make a Notification... I personally prefer to work off of sample code, modifying until I have something that does what I want, so I'm going to scroll down **slowly** until I find an example I can copy/paste in, or at least reference.
- Then we can scroll back up to see how this woks!

The first part of this Notification is using `NotificationCompat.Builder`. Where else have we seen Builders? The `AlertBuilder`! Same idea here
  - I don't have a drawable resource that makes me want to comment out the icon... but scrolling back up I can see that it is _required_. So we can generate an Image Asset just like we did with icons (cause hey, notifications are a thing that happens!)
  - We can also set priority; see the [guide](http://developer.android.com/design/patterns/notifications.html#correctly_set_and_manage_notification_priority).

The next line makes an Intent. We've done that... but why are we creating an Intent? Well if we scroll up and look where where `Intent` is referenced, we can find out about Notification Actions (e.g., what happens when someone clicks the Notification). And since Intents are messages to open Activities, it makes sense that clicking a Notification might send an Intent, right?
- It says that the `Intent` is wrapped in a `PendingIntent`. What's that?! The details are not _super_ readable... It's basically a wrapper around an Intent that we give to _another_ class. Then when that class receives our `PendingIntent` and reacts to it, it can run the `Intent` (command) we sent it with as if that Activity was us (whew). Basically we're saying "when I call you, you can come pick me up using my car" kind of thing.
  - The idea being that the Notification "component" (which is not our Activity!) will be able to run this intent.. which wakes up our Activity! The Intent is "pending" delivery/activation by the other service.
- The notification is using the `TaskStackBuilder` to construct an "artificial" backstack--so that when you jump to the detail view (say), you'll still be able to hit back and return to the master view. Sort of like what the phone dialer was doing to us
  - We can use this to still create a `PendingIntent`. Alternatively, we can look up the `PendingIntent` class and see that there is a `getActivity()` method we could use.

Finally, we can use the `NotificationManager` to "get the notification service", and actually run the `Notification!` Boom we have notifications :)
- We can click on that notification and get taken to our app :)

Updating the notification is actually really straightforward. You simply re-issue a notification with the same **id** number you assigned, and it will "replace" the previous one!

If time, we can also add an expanded layout, following the [example](http://developer.android.com/guide/topics/ui/notifiers/notifications.html#ApplyStyle).

### 5min break?

## Settings
Last piece I want to talk about today: what if we wanted to let the user decide whether the clicking should create notifications or not? Maybe sometimes they just want to see Toasts! The cleanest way to do this is to create some [Settings](http://developer.android.com/guide/topics/ui/settings.html) using `Preferences`. Specifically, these use a `PreferenceFragment` (or `PreferenceActivity`) to display a predefined structure for app settings.

Preferences are special because they are _automatically_ connected to a persistent data store called the [SharedPreferences](http://developer.android.com/guide/topics/data/data-storage.html#pref). This is an file (an XML file, technically) that is used to persist small bits of primitive data in your app (basically, anything you could put into an XML file).
- It's not _just_ for preferences--can use it for any data you want! Not good for structured data though (since only key-value pairs), hence why we still care about SQLite

Preferences, like everything else, are defined in an `xml` resource. In this case, the "type" is actually just `XML` (generic), though our "root element" will be a `PreferenceScreen` (thanks intelligent defaults!)
- By convention, the preferences resource is named `preferences.xml`
- Inside the `PreferenceScreen`, we add more elements: one to represent each "line" of the screen. These have types such as: `<CheckBoxPreference>`, `<EditTextPreference>`, `<SwitchPreference>`, or `<ListPreference>` (for a dialog of radio buttons).
- These elements take (among others) the following XML attributes:
  - `android:key` the key to store the preference in the SharedPreferences file
  - `android:title` a user-visible name
  - `android:defaultvalue` guess :p
  - More options in the [Preferences](http://developer.android.com/reference/android/preference/Preference.html#lattrs) documentation.

Note that it is possible to have multiple screens, which end up producing "subscreens" (just like we had "submenus").

We show the settings by creating a `PreferenceFragment`, usually inside of a blank `Activity` (e.g., `SettingsActivity`).
- At the minimum, the Fragment needs to call `addPreferencesFromResource(R.xml.preferences);` to specify the "layout"
- And the Activity doesn't even need to load a layout: just specify a transaction!
```java
getFragmentManager().beginTransaction()
                .replace(android.R.id.content, new SettingsFragment())
                .commit();
```
- Note that `android.R.id.content` refers to the "root view", whatever that may be. But if we want to include other stuff (e.g., an ActionBar), we'd structure our Activity in more detail.

Finally, how do we interact with these settings? Well we can access the `SharedPreferences` file to fetch valeus from it really easily!
- Settings are stored in the "default" SharedPreferences file, which you can access via
```java
SharedPreferences sharedPref = PreferenceManager.getDefaultSharedPreferences(this);
```
Then you can query that `SharedPreferences` object like it was a bundle: `getString()`, `getBoolean()`, etc. That way we can check the current preferences before we show a notification!

## Wrap-up
Menus, notifications, and settings are most of the user-interactive "extras" that go into a "full app". There is of course more customization that can be done, particularly with themes and styling. But using these will let you be able to build out a project that feels like a full-blown application, rather than just a simple practice.
- Again, lots more details and options to use--look through the documentation!

What you may notice we're **not** talking about is the UI Design guidelines: e.g., what kind of text should I put in my notification? When should I choose to _use_ a notification? How do I best organize my settings? Android has lots of guidance on these topics in their "design" documentation, and further HCI and Mobile Design topics apply here just as well (has anyone taken the Mobile Design class? Does it cover this stuff?)
- Leaving the UI design up to you. But major guidelines apply (e.g., make actions obvious, give feedback, etc.)


### Review
For the remainder of our time (if any), I think I'd like to open things up to work/review/questions. What aspects of things we've covered so far need review/practice? Are there any pieces of the homework you'd like to make sure and go over? [take a few minutes to read over it now!] What can I best do to help at the moment?
- And if there is nothing, happy to let you work on your own or go early :)

Next time: we're going to start doing location-based stuff, and dig into the Maps API! Because location is what mobile is all about :)
