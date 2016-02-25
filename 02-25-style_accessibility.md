## 02-25 Styles & Accessibility

### Admin
- Back to lectures!
  - Thanks for patience/support for guest lectures :)
- Homeworks should all be returned by this point. You've done great work: now all that's left is to synthesize it into final projects!
  - Any questions/issues with projects?
  - If there are any grading issues, let us know (we can make sure everything is correct and up-to-date). Check Canvas to make sure you agree with the numbers!

- Fork & clone repo for today, as always (so we have something to play with, though going to focus on resources).

## Styles
This week has been about developing more effective user interfaces: the "front end" part of an Android app. Guest lecture about reviewing some overall UI design stuff, and we'll get into a few more aspects today. In particular, we're going to talk about [styles and themes](http://developer.android.com/guide/topics/ui/themes.html).

First, consider some of the layouts that we've used in previous examples (e.g., from the Services or Intents demos). We defined the layouts in XML, with lots of repeated `Buttons` or `TextViews`.
- These buttons have a lot of the same attributes: larger text, similar sizing... if I decided I wanted to make the text bigger, I'd have to change it in three places. Wouldn't it be nice to separate that out?

We can define a collection of properties (e.g., XML attributes) by defining what is called a [Style](http://developer.android.com/guide/topics/ui/themes.html#DefiningStyles). Styles are _also_ defined as an XML resource, but one that we can then reference inside our layout.

[Styles resources](http://developer.android.com/guide/topics/resources/style-resource.html) are defined inside the `res/values/styles.xml` file. Note that when we had Android Studio create projects for us, this was already defined (and even includes some content!)
- Style resources use a `<resources>` tag as the top-level element, declaring that this XML contains (generic-ish) resources for us to use

Inside that tag we define `<style>` elements, which will represent a single style. You can _almost_ think of this like a CSS class, but without the "cascading" part.
  - The `<style>` tag is given a `name`, similar to how we'd define a CSS class name. We label this using [PascalCase](http://c2.com/cgi/wiki?PascalCase) (needs to be legal Java identifier since will be compiled into `R`, and it's a class so capitalize I guess).
  - We can also declare a `parent` attribute in order to inherit style; we'll come back to that in a second

Finally, _inside_ the `<style>` tag we declare the individual properties as `<item>` elements&mdash;these would be the properties in a CSS file, or the attributes of a `View` element in a layout.
- Each property gets its own `<item>`. This is not exactly compact.
- `<item>` elements get a `name` attribute, which is the name of the property we want to include. For example `name="android:layout_width"`. Note that we usually need to include the package prefix.
- The content of the `<item>` tag is then the value we want to be associated with that property, e.g., `wrap_content`.

For example (from the [docs](http://developer.android.com/guide/topics/resources/style-resource.html))
```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <style name="CustomText" parent="@style/Text">
        <item name="android:textSize">20sp</item>
        <item name="android:textColor">#008</item>
    </style>
</resources>
```

Finally, we can specify that we want a particular `View` (in the layout) to use the style by giving that view element a `style` property, referencing the style we just defined:
```xml
<TextView
    style="@style/MyStyle"
    android:text="Hello World" />
```
- The `style` attribute does not use the `android` namespace!
- Note that we're referencing another resource with the `@` symbol, and that resource is the `style` resource (with the `MyStyle` item).
- Now any attributes we've put in the styles will be included automatically, but we can still add extra attributes if we want to customize that element further!
- This means I can change an attribute in one place (in the `style.xml` file), but have it apply throughout my application. Which is handy!

### Style Inheritance
Since we're trying to save ourselves work (e.g., make it easier to modify things in one place), it would be nice to not have to redefine each style if it only changes a little bit. Luckily, we can **inherit** and "extend" other existing styles in order to reuse them! There are two options for this (and they work a little differently): we can inherit from _built-in styles_ (e.g., from someone else), or we can inherit from _custom styles_ (e.g., ones that we made).

Let's start with inheriting our own styles. We can use the `parent` attribute of the `<style>` tag to specify what "parent" style we should inherit from (self-definitional I know, but we've done a lot of programmatic inheritance already). For example:
```xml
<style name="ChildStyle" parent="@style/ParentStyle"> ... </style>
```
This will cause our `ChildStyle` to include all of the `<item>` elements of `ParentStyle`.
- We can then "override" them by redefining the `<item>` that we want to change. (Example: `android:textColor`).

Note that when inheriting our own styles, we don't _have_ to use the `parent` tag. Instead, we can use **dot notation** name-spacing to define child classes.
- For example, naming a style `ParentStyle.ChildStyle` will define a style (`ChildStyle`) that inherits from `ParentStyle`. This would be referenced in the layout as `@style/ParentStyle.ChildStyle`.
- We can chain these together as much as we want, similar to CSS: `MyStyle.Red.Big` (though that would be distinct from the `MyStyle.Big.Red` style! It's not doing CSS combining, but Java class inheritance!)

Android also includes a large number of [built-in platform styles](http://developer.android.com/guide/topics/ui/themes.html#PlatformStyles) that we can apply and/or inherit from. These **must** be inherited via the `parent` attribute (you can't use dot notation for them). They are referenced with the format:
```xml
<style name="MyStyle" parent="@android:style/StyleName">...</style>
```
There are a bunch of these... and Android's recommendation is to understand them by browsing the [source code][styles.xml].
- You can also look at a list of options in [R.style][R.style]
- This doesn't help with discoverability much. My recommendation: if you want a style to "look like" an element that already exists, search for that particular element (e.g., want it to look like a `Toast`? Search for `Toast`). And then can look "up" the inheritance chain for more details
  - This is a lot like trying to learn the Bootstrap classes by reading the css file :p
- As point of reference, the docs use `TextAppearance` as a style to inherit


### Themes
Unlike CSS, styles do not normally _cascade_: that is, if you apply a style to a `ViewGroup`, that style will not be applied to all components of that `ViewGroup`. But what we can do is define a style that should apply to _every_ `View` in an **Activity** or even **Application** by applying that style as a [Theme](http://developer.android.com/guide/topics/ui/themes.html#ApplyATheme)
- Themes are styles that are applied to the whole application or to a particular Activity; you can't get any finer granularity (without moving to per-View styles). Note that Theme styles apply to **every** View in the context (though we can overwrite specific styling for a View as normal)

Themes _are_ styles, and so are defined the exact same way (as `<style>` elements inside the `styles.xml` file).
- You can of course define them in a separate file (e.g., `theme.xml`) since resource filenames are arbitrary; it will still be loaded in as a style resource and accessed the same way

Themes are then applied to an Activity or Application by specifying a `android:theme` attribute in the `Manifest` (where the Activity/Application is defined). For example, we could define the theme:
```xml
<color name="custom_theme_color">#b0b0ff</color>
<style name="CustomTheme" parent="android:Theme.Light"> <!-- ignore parent -->
    <item name="android:windowBackground">@color/custom_theme_color</item>
    <item name="android:colorBackground">@color/custom_theme_color</item>
</style>
```
and then apply it with
```xml
<activity android:theme="@style/CustomTheme">
```

You may notice that the Studio-generated projects already have a theme! In fact, that is the style that is provided.
- Experiment with removing the theme: how does the layout change?

Along with styles, Android provides a number of platform-specific Themes. And again, the recommendation is to understand these by browsing the [source code][themes.xml].
- It's too bad there isn't a more visible set of (automatically generated) documentation. That would be a neat project...
- But this is a good place to look for themes to apply to your application!
  - example: `.Light` and `.Dark` versions,

Note that we can also [reference](http://developer.android.com/guide/topics/resources/accessing-resources.html#ReferencesToThemeAttributes) Theme-level attributes with the `?` symbol in xml (e.g., `?android:textColorPrimary` will refer to the value of the `textColorPrimary` attribute in the current _Theme_)

#### Material Themes
One of the most important set of Android-provided themes are the [Material Themes](http://developer.android.com/training/material/theme.html). These are themes that support Google's [Material Design](http://www.google.com/design/spec/material-design/introduction.html), a visual design language Google uses (or aims to use) across its products, including Android.
- This includes a set of themes, widgets (lists and cards), animations, etc. See the [design docs](http://developer.android.com/design/material/index.html) for more details.

There are two main Material themes
- `@android:style/Theme.Material` a Dark version of the theme
- `@android:style/Theme.Material.Light` a Light version of the theme

And a bunch of variants:
- `@android:style/Theme.Material.Light.DarkActionBar` a Light version with a Dark action bar
- `@android:style/Theme.Material.Light.LightStatusBar` Variant of the material (light) theme that has a light status bar background with dark status bar contents.
- `@android:style/Theme.Material.Light.NoActionBar` Variant of the material (light) theme with no action bar.
- ... etc. See [R.style](http://developer.android.com/reference/android/R.style.html) for more (ctrl-f `Material`)

(Can experiment with the effects of changing the theme!)

The easiest way to customize these themes is by adjusing the color scheme. You can do this by inheriting from one of the Material themes and then _overriding_ the color resource definitions:
```xml
<resources>
  <!-- inherit from the material theme -->
  <style name="AppTheme" parent="android:Theme.Material">
    <!-- Main theme colors -->
    <!--   your app branding color for the app bar -->
    <item name="android:colorPrimary">@color/primary</item>
    <!--   darker variant for the status bar and contextual app bars -->
    <item name="android:colorPrimaryDark">@color/primary_dark</item>
    <!--   theme UI controls like checkboxes and text fields -->
    <item name="android:colorAccent">@color/accent</item>
  </style>
</resources>
```
See http://developer.android.com/training/material/images/ThemeColors.png for a diagram of what color variables to use.

You can specify an `android:theme` for any element in a layout, allowing that theme (specifically, it's color scheme) to apply to that any any child Views. This only works for the color variables, not for generic attribute definition (it's because the styling is already referencing the theme, so you're just overriding what those variables mean).

So style and theme is a useful way of defining a visual appearance that can be shared across an entire app. Any questions on this?

#### Break ~5min

## Accessibility
The second topic I want to cover today also deals with user interface---but instead of looking at aesthetics (style), I want to talk about functionality in terms of **Accessibility**.

I approach this topic from the point of view of [Universal Design](https://en.wikipedia.org/wiki/Universal_design), which is the goal of designing products that are _inherently accessible_. The premise here is that designing for accessibility benefits not just those with some form of limitation or disability, but _everyone_.
  - The classic example is [curb cuts](https://en.wikipedia.org/wiki/Curb_cut): the "slopes" built into curbs to accommodate those in wheelchairs. However, these cuts end up supporting _everyone_: people with rollerbags, strollers, temporary injuries, etc.
  - There was a great infographic I couldn't locate again that points out that if you design for someone with one arm, then you support people with disabilities, but also people with temporary disability (e.g., an arm in a sling/cast) and people who are just massively inconvenienced (e.g., a person holding a baby in that arm).
  - This is just as important in the mobile design space: if we can support people with vision impairment (e.g., by supporting touch or voice controls), then we also support people who want to use the app while driving or otherwise visually occupied.
  - Like we talked about yesterday: supporting low-end devices with limited data allowance (e.g., for people or communities that can't afford unlimited 4G connections) also supports people who are currently without data connection (in the woods, on an airplane, over their data plan, etc).
  - See also a [recent article](http://www.fastcodesign.com/3054927/the-big-idea/microsofts-inspiring-bet-on-a-radical-new-type-of-design-thinking) about how Microsoft is approaching this problem.

The main point: accessibility is important and worth considering. Not that people with disabilities aren't worth helping for their own right, but a little bit of effort to support those who are normally disenfranchised makes the world a better place for everyone.

[Accessibility in Android](http://developer.android.com/guide/topics/ui/accessibility/index.html) is not as effective as it could be, but there are a number of simple steps you can take to support users with disabilities _especially_ if you're just using the built-in `View` elements (like we've been doing in this class). So we're going to go over some of the "low-hanging fruit" today.
- There are basically two goals supported by the framework itself:
  1. Enable the application to be used without "sighted assistance" (e.g., for visually-impaired)
  2. Support interaction without the touch-screen (e.g., with directional controls for other external devices)

To test all this stuff out, we're actually going to want a _new_ emulator hardware profile
- Clone the Nexus 5, enable Keyboard and D-Pad support. This will let us control the emulator/device with alternative devices, such as those used for accessibility support (will come back to this in a moment, and makes testing a bit more sane)


### TalkBack
These capabilities are provided by [accessibility services](http://developer.android.com/guide/topics/ui/accessibility/services.html), or background services that are able to respond to "accessibility events" (more later). The two most common built-in services are [TalkBack](http://developer.android.com/tools/testing/testing_accessibility.html#testing-talkback), which "speaks" the name of UI elements as the user focuses on them, and [Explore by Touch](http://developer.android.com/training/accessibility/testing.html#testing-ebt) (4.0+), which allows the user to drag a finger around a screen and get verbal feedback of what is there.
- These can be turned on by going to `Settings > Accessibility > TalkBack`... but we need to install it on the emulator (`adb install package.apk` will work if `adb` is available...)

(Experiment with the TalkBack demo)

A lot of interface design gives usability hints (e.g., what a button does) through visual cues: icons, labels, etc. This isn't effective for visually impaired--as you can see, our "icon" buttons are just explored as "Button 59".
- We can improve this by adding an `android:contentDescription` attribute to our button, which specifies how TalkBack refers to it!
- Super easy addition that does a lot to support accessibility :)

For the second piece, we'd also like to support interaction that _doesn't_ use the Touch Screen. The way to do this is to make sure that input elements can get _focus_: that is, they can be "currently selected" supporting interaction without the touch screen.
- To do this, we need to add another attribute: `android:focusable=true`
  - We can also specify `android:nextFocusDown|Up|Left|Right=@+id/...` to adjust the "tab order" of inputs. This is a thing to do in web development as well!
- Note that in order to test this, we need to enable our device to support a physical keyboard and/or D-Pad

Finally, I want to flag that Android supports a wider variety of accessibility services&mdash;and indeed we can write our own!
- We can support existing services in our _custom_ views by firing off [Accessibility Events](http://developer.android.com/training/accessibility/accessible-app.html#events). For example,
```java
AccessibilityManager accessibilityManager =
        (AccessibilityManager) MainActivity.this.getSystemService(Context.ACCESSIBILITY_SERVICE);
if (accessibilityManager.isEnabled()) {
    View v = //... findViewById(R.id.masterLayout);
    v.sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_TEXT_CHANGED);
}
```
- More details [here](http://developer.android.com/guide/topics/ui/accessibility/apps.html#send-events). Be sure to check out information about [supporting custom touch views](http://developer.android.com/guide/topics/ui/accessibility/apps.html#custom-touch-events)
  //can do this if time?!

But more than that, we can also build our _own_ accessibility services! We do this by extending the `AccessibilityService` class and overriding the appropriate methods:
```java
public class MyAccessibilityService extends AccessibilityService {
    @Override
    public void onAccessibilityEvent(AccessibilityEvent event) {
    }

    @Override
    public void onInterrupt() {
    }
}
```
- We declare this as a `<service>` in the `Manifest`, with an `<intent-filter>` for `<action android:name="android.accessibilityservice.AccessibilityService" />` so that it can respond to the events generated.
  - We also need to give it `android.permission.BIND_ACCESSIBILITY_SERVICE` permissions, both as a `<uses-permission>` and as a `android:permission` attribute
- Will need to configure the service, per [these instructions](http://developer.android.com/training/accessibility/service.html#configure). Easiest way is to do this in XML (provided in starter code), and then include in manifest as a meta attribute:
```xml
<meta-data android:name="android.accessibilityservice"
                android:resource="@xml/accessibility_service_config" />
```

- And then finally we can have the `onAccessibilityEvent()` method do something helpful when the event occurs!

For example, we can provide spoken feedback using the [text-to-speech engine](http://android-developers.blogspot.com/2009/09/introduction-to-text-to-speech-in.html) (untested):
```java
private TextToSpeech mTts;

  mTts = new TextToSpeech(this, this);
  mTts.setLanguage(Locale.US); //optional?
  String myText1 = "Did you sleep well?";
  String myText2 = "I hope so, because it's time to wake up.";
  mTts.speak(myText1, TextToSpeech.QUEUE_FLUSH, null);
  mTts.speak(myText2, TextToSpeech.QUEUE_ADD, null);

mTts.shutdown(); //in onDestroy
```

Can check for TTS capabilities if needed:
```java
Intent checkIntent = new Intent();
checkIntent.setAction(TextToSpeech.Engine.ACTION_CHECK_TTS_DATA);
startActivityForResult(checkIntent, MY_DATA_CHECK_CODE);

protected void onActivityResult(
        int requestCode, int resultCode, Intent data) {
    if (requestCode == MY_DATA_CHECK_CODE) {
        if (resultCode == TextToSpeech.Engine.CHECK_VOICE_DATA_PASS) {
            // success, create the TTS instance
        } else {
            // missing data, install it
            Intent installIntent = new Intent();
            installIntent.setAction(
                TextToSpeech.Engine.ACTION_INSTALL_TTS_DATA);
            startActivity(installIntent);
        }
    }
}
```

It's also worth noting that Android has a [Vibrator](http://developer.android.com/reference/android/os/Vibrator.html) class that can be used to vibrate the phone.

#### Checklist
This is just a taste of how we might support accessibility: the `contentDescriptions` and `focusable` are the easy to support ones.

Android also provides a general [checklist][accessibility checklist] of things you should check for as you're developing/accessorizing your app, including links to design guidelines
- check this out! A lot of it may not be relevant when using the standard widgets, but as soon as you make your own custom stuff you'll want to start including this to support as many different users/situations as possible.

Any questions?

#### Next Week
We're going to break into the fun stuff and away from "phone apps", looking at software for watches and virtual reality!
- Keep working on projects, and if there are any questions let me know!


<!-- references -->
[styles.xml]: https://android.googlesource.com/platform/frameworks/base/+/refs/heads/master/core/res/res/values/styles.xml
[R.style]: http://developer.android.com/reference/android/R.style.html
[themes.xml]: https://android.googlesource.com/platform/frameworks/base/+/refs/heads/master/core/res/res/values/themes.xml
[theme colors]: http://developer.android.com/training/material/images/ThemeColors.png
[accessibility checklist]: http://developer.android.com/guide/topics/ui/accessibility/checklist.html
