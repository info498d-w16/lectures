## 01-07 Layouts

### Admin [5min]
- Any issues getting the homework finished? Sorry for typos/errors (first time running the class and using these assignments).
- This week's homework is (has been) online; we'll be covering the pieces you need for it this week.

#### Setup
- Fork and clone my repo for today!
  - make sure to **import** the project into Android Studio (other can get weird errors)
- **Get Android Studio Booted Up**

## Resources
Today we're going to talk about *Resources* and how to use them to define *Layouts*. We talked about Activities yesterday that are the Java, now we're going to talk about the XML.

[Resources](http://developer.android.com/guide/topics/resources/overview.html) can be found in the `res` folder, and represent elements or data that is "external" to the code. You can think of them as "media content": often images, but also things like text clippings (or short String constants!). Textual resources are usually defined in XML files.
- Why there? Because resources represent elements (e.g., content) that is _separate_ from the code (the behavior of the app). This is about **Separation of Concerns**
- By defining them in XML, they can be developed (worked on) _without_ coding tools (e.g., with systems like the "layout design" tab). So you could have a Graphic Designer create these resources, which can then be integrated into the code without the Designer needing to do a lick of Java.
- Similarly, you can choose what resources to include _dynamically_. You can choose to show different images based on device screen resolution. Or pick different Strings based on the language of the device (Internationalization!)--the behavior of the app is the same, but the "content" is different!
  - Web terms: same JavaScript, different HTML!

What should be a resource? In general:
- Layouts should **always** be resources
- UI controls (buttons, etc) should mostly be defined as resources (part of layouts)
- Any graphic images (drawables) should be resources
- Any user-facing strings should be resources
- Styles are _always_ resources

As we mentioned last week, there are a number of different [resource types](http://developer.android.com/guide/topics/resources/available-resources.html), many of which you can see in the `res` folder of a default Android project
- `res/drawable/` : graphics (PNG, JPEG, etc)
- `res/layout/` : UI XML layout files (we're going to talk about these a lot today!)
- `res/mipmap/` : launcher icon files
- `res/values/` : general constants
  - `/strings` : short strings
  - `/colors` : color constants
  - `/dimen` : dimensions (like default margins)
  - `/styles` : [style and theme](http://developer.android.com/guide/topics/ui/themes.html) details

The documentation for these is hard to find and scattered throughout the documentation, but [Resource Types](http://developer.android.com/guide/topics/resources/available-resources.html) is a good place to start.


### Alternate Resources
These aren't the only names for folders---as I said, part of the goal of resources is that they can be _localized_ (changed depending on the device)!
- You can specify folders for "alternative" resources (e.g., special handling for another language, or low-res phones).
- At runtime, Android will check the config of the device, and try to find an alternative resource that matches that config. If it it _can't_ find one, it will fall back to the "default" resource

So what kinds of configurations can we include? See [this list](http://developer.android.com/guide/topics/resources/providing-resources.html) (highlighting a fewl or scavenger hunt?)
- Language and region (two-letter ISO codes)
- Screen size (small, normal, medium, large, xlarge)
- Screen orientation (port, land)
- Screen pixel density (dpi) (ldpi, mdpi, hdpi, xhdpi, ...)
  - note: dpi is "dots per inch", so pixels across _relative_ to the device size!
  - xxhdpi is pretty common for high-end devices
- Platform version (v1, v4, v7... for each API number)

Configurations are indicated using the **directory name**, giving them the form `<resources_name>(-<config_qualifier>)+`\
- [[demo/practice: make Welcome Message be in `fr` or `ko` or whatever. https://www.webucator.com/blog/2010/03/saying-hello-world-in-your-language-using-javascript/]]
  - Using "New Resource" gives a wizard; switch to "Package" view to see the files

### XML Details
Resources are usually defined as XML (which is similar in syntax to HTML).
- The `strings.xml` resource is pretty simple, but more complex details can be seen in the `activity_main` layout.
- Android-specific attributes are namespaced with a `android:` prefix, to avoid any potential conflicts
- We can use the `@` symbol to reference one resource from another: `@[<package_name>:]<resource_type>/<resource_name>`
- We can also use the `+` symbol to create a _new_ resource that we can refer to (like declaring a variable inside an attribute)
  - usually used with `android:id="+@id/identifier"`; we'll see that later

## R
So how does this XML get integrated into Android? How do we mix it into the Java? (See [here](http://developer.android.com/guide/topics/resources/accessing-resources.html))

When your application is compiled, the build tools **generate** an additional class called `R` (for "resource"). This class contains what is basically a ton of "constants"---one for each resource!
- These constants are organized into subclasses, one for each resource allowing you to refer to `[(package_name).]R.resource_type.identifier`---syntax almost like a JSON object!
  - e.g., `R.string.hello`; `R.drawable.icon`, `R.layout.activity_main`
  - for most resources, the identifier is the identifier (id)
  - for layouts, the "identifier" is the filename (without the xml)
  - that `@` symbol goes to `R` to look things up.
- This class gets regenerated all the time; with Eclipse, often a lot of issues involved needing to regenerate this class so the IDE could find stuff.
- Can find the file in `app/build/generated/source/r/debug/...` for fun

Note that these static values are often just `int`s---they're pointers to element references!!
- If you've done C work, it's like passing a pointer* around!
- So in the Java, it's almost like we're working with `int` as the data type for a resource (because it's just a pointer TO a resource!). Think the "key" or "index" of that resource.
  - Android does the hard work of taking that `int`, looking it up in an internal resource table, finding the associated XML file, and then getting the right element out of it.

So how can we access these in Java? Well that `R` class is included, so we can call it as such.
- example: `setContentView()` takes in a resource `int`.
- Other common method will be `findViewById(int)`, which lets us grab a View element (e.g., a button) from the resource to talk about it in the Java.

### **Break time?**

## Views and Layouts
The most common resource type we'll be working with (at least this week!) will be [Views](http://developer.android.com/reference/android/view/View.html). View is the "superclass" for visual elements: and visual component on the screen is a View.
- Examples: [TextView](http://developer.android.com/reference/android/widget/TextView.html), [ImageView](http://developer.android.com/reference/android/widget/ImageView.html), [Button](http://developer.android.com/reference/android/widget/Button.html), etc.
- Why a super class? Because it means we can use __polymorphism__ to be able to treat all these visual elements the same way!
  - We can lay them out, draw them, click on them, move them, etc. And all the behavior will be the same--though subclasses will have "extra" features

Here's the big magic trick: one subclass of `View` is [`ViewGroup`](http://developer.android.com/reference/android/view/ViewGroup.html). A `ViewGroup` can contain other "child" Views. But since `ViewGroup` is a `View`... it can contain more `ViewGroups` inside it! Thus we can **nest** Views within Views
- This ends up working a lot of HTML! You can have elements (e.g, `<div>`) inside of other elements!
- This is how we'll do complex layouts!

Views are defined inside of [Layouts](http://developer.android.com/guide/topics/ui/declaring-layout.html)---that is, inside a layout resource, which is an XML file describing Views.
- These resources are "inflated" (rendered) into UI objects that are part of the application.
  - "inflate" like "unpacked/expanded" into a Java object

An important note: GUI design tools (like the "design tab" of Android Studio) are _new_.
- Early developers (hi!) only used XML; that's all we had ("back in my day...")
  - and early design tools were pathetic
- Android Studio's tool is a lot less shabby. But because I'm old-school, I'm going to push you to write the layouts by hand in XML. Good for understanding the pieces, and you should be able to do it anyway.
  - a lot like how I push using git from the command-line (or my colleagues push Java from command-line). Grognards, all around

Note that `Layout`s are `ViewGroups` that provide "ordering" and "positioning" information for the Views inside of them
- They let the system lay out the Views intelligently/efficiently
- Views shouldn't know their position! (good OOP design there)

### View Components and Properties
But before we get into how to group Views, let's focus on the individual, basic View classes.
- Example: look at the provided [`TexView`](http://developer.android.com/reference/android/widget/TextView.html)
- Example: make an [`ImageView`](http://developer.android.com/reference/android/widget/ImageView.html) that contains a picture!
  - note the XML attributes!
  - Then we'll specify the content of that image in the code (e.g., if we wanted to dynamically pick a picture)
  ```java
  ImageView imageView = (ImageView) findViewById(R.id.myimageview);
  imageView.setImageResource(R.drawable.myimage);
  ```

Note that all views have 3 basic components:
- They have **properties** which define the state
- They have **methods** which the Java code can call to manipulate them (outside to in)
- They can produce **events** which they can use to notify the Java code (inside to out)

We're mostly going to focus on the first one today: Properties. These are usually defined within the resource as XML attributes. So what are some properties?
- `android:id`
  - a unique identifier; unique within the layout (and ideally the whole app)
  - legal java variable name
  - lower_case by convention
  - style: can prefix with type (`btn`, `edt`) for easy reference!
  - **note** give each View and id, and then it will be automatically "saved" in the `Bundle` when the Activity is destroyed! [ref]
- Size: `android:layout_width` and `android:layout_height` (see [ViewGroup.LayoutParams](http://developer.android.com/reference/android/view/ViewGroup.LayoutParams.html) for docs )
  - This can be either be a dimension/value (12dp), or more commonly one of two special values:
    - `wrap_content` (be as big as content, plus padding)
    - `match_parent` (be as big as parent, minus padding). Used to be `fill_parent`
    - also
  - A note on [dimensions](http://developer.android.com/guide/topics/resources/more-resources.html#Dimension) and [units](https://www.google.com/design/spec/layout/units-measurements.html#)!
    - `dp` is a "density-independent pixel". On a 160dpi screen, `1dp` == `1px`. But as dpi increases, the #`px` per `dp` goes up
    - `px` actual screen pixels. NOT RECOMMENDED.
    - `sp` is a "scale-independent pixel". Like `dp`, but scaled by font preference (think `px` vs. `pt` in CSS)
    - `pt` is 1/72 of an inch of the physical screen. Also `mm` and `in`.
- Padding & Margin: `android:padding`, `android:paddingLeft`, `android:margin`, `android:marginLeft`, etc.
  - These work the same way they do in CSS: padding is between the content and the "edge" of the View; margin is between views.
  - Margin doesn't collapse!
- Lots of others as well! Check out the listing in [`View`](http://developer.android.com/reference/android/view/View.html#lattrs), or look at the options the "design tab" in Android Studio gives you!

Those are some of the main _visual properties_, since we're interesting in developing layouts and interfaces. Note that almost all of these properties can be modified with methods (e.g., `setPadding()`) from within the Java code; but you'd only do that to make things dynamic--specify the layout in the XML!
- Other methods are things like `isVisible`, `hasFocus`, etc. We'll point at those as we need them
- And do more on events tomorrow!


## Layouts
As mentioned above, a [Layout](http://developer.android.com/guide/topics/ui/declaring-layout.html) is grouping of Views (specifically, a `ViewGroup`). They act as containers for other views, to help organize things.
- Layouts are all subclasses of `ViewGroup`! So can get a list of them from there.

### LinearLayout
Probably the simplest Layout to understand is the [`LinearLayout`](http://developer.android.com/guide/topics/ui/layout/linear.html). This simply orders the children Views in a line ("linearly")
- All children are laid out in a single direction, but you can specify the orientation (`android:orientation`)
- See [LinearLayout.LayoutParams](http://developer.android.com/reference/android/widget/LinearLayout.LayoutParams.html) for a list of all options!
  - Remember, as `Views` you can also use all of the properties we talked about earlier

The other piece you might want to control is how much of any remaining space the element should occupy (e.g., should it expand)? This is done with the `android:layout_weght` property.
- after all sizes are calculated, the remaining space is divided up proportionally to the weight of each element (default 0)
- use `0dp` for width and height and `1` for weight to make everything the same size
- Example in Text

You can also use [`android:layout_gravity`](http://developer.android.com/reference/android/widget/LinearLayout.LayoutParams.html#attr_android:layout_gravity) to encourage more details on how elements might be put in the layout.

_Important Point_ You can also nest `LinearLayouts` inside each other! So you make "grids" by putting a vertical layout containing "rows" of horizontal layouts (containing Views).
- There are lots of ways to achieve any layout!


### RelativeLayout
A [`RelativeLayout`](http://developer.android.com/guide/topics/ui/layout/relative.html) is the default layout giving by Android Studio, but it's a bit more complex to use. In a `RelativeLayout`, children are positioned "relative" to the parent **OR** _to each other_.
- All children default to the top-left of the View
- Can instead give them properties from [`RelativeLayout.LayoutParams`](http://developer.android.com/reference/android/widget/RelativeLayout.LayoutParams.html) about where to go
  - ex: `android:layout_verticalCenter` centers vertically within parent
  - ex: `android:layout_toRightOf` centers to the right of View with resource id
    - Use the `@` symbol to refer to the resource, with the `+` after it (before the id) to define a new `id`
      ```xml
      <EditText
          android:id="@+id/name"
          android:layout_width="match_parent"
          android:layout_height="wrap_content"
          android:hint="@string/reminder" />
      <Spinner
          android:id="@+id/dates"
          android:layout_width="0dp"
          android:layout_height="wrap_content"
          android:layout_below="@id/name"
          android:layout_alignParentLeft="true"
          android:layout_toLeftOf="@+id/times" />
      ```
    - You don't need to specify "toRightOf" and "toLeftOf". Think about putting down one element, and then putting down the next relative to what came before.
- This can get tricky; I'm honestly a fan of using LinearLayouts more (since you can always reproduce this kind of work by dividing into enough Linears)
- [[practice: make an image with a textbox and submit button below it (aligned)]]


### Other Layouts
There are other layouts as well, though we won't go over in depth (they work in similar ways; just check the documentation!)
- [TableLayout](http://developer.android.com/guide/topics/ui/layout/grid.html) acts like an HTML table: define `TableRow` layouts which you can fill with content
- [GridLayout](http://developer.android.com/reference/android/widget/GridLayout.html) puts things into a Grid (like Linear, but fills the grid first)
  - This is different than a [GridView](http://developer.android.com/guide/topics/ui/layout/gridview.html), which is a scrollable, adaptable list (like a `ListView`, which we'll talk about tomorrow).
- [FrameLayout](http://developer.android.com/reference/android/widget/FrameLayout.html) is a sort of "placeholder" layout that holds a single child view; can think of it as a way of adding a simple container to use


### Combining Layouts
Last piece: what if we want to combine layouts? Maybe we want to dynamically change what Views are included, or maybe we just want to refactor chunks of the layout into different XML files to improve the organization.

It's possible to dynamically load views "manually" (e.g., in code) using the [`LayoutInflator`](http://developer.android.com/reference/android/view/LayoutInflater.html). This is a class that has the job of "inflating" (rendering) Views; it is implicitly used in that `setConventView` method. It has the following syntax:
```java
LayoutInflator inflator = getLayoutInflator(); //access the inflator
View myLayout = inflator.inflate(R.layout.my_layout, parentViewGroup, true); //to attach
```
- Note that we never instantiate the `LayoutInflator`, we just access the one that exists (Factory Pattern!)
- the [`inflate`](http://developer.android.com/reference/android/view/LayoutInflater.html#inflate(int, android.view.ViewGroup, boolean)) method takes a couple of arguments
  - The first is the resource to inflate (an int!)
  - The second is a a group to act as the "parent" for this View--e.g., what layout should it go inside? Can be null for no context.
  - The third (optional) is whether to actually attach it to that parent (if not, the parent just provides context/layout params to use). If not assigning to parent, can attach to view using method in `ViewGroup` (e.g., `addView(View)`).
- [[demo?]]

This method works, but it tends to be messy and hard to maintain (UI should be specified in the XML!); so it isn't as common in modern development. A much cleaner solution is to use a [`ViewStub`](http://developer.android.com/training/improving-layouts/loading-ondemand.html). A ViewStub is like an "on deck" (baseball) layout: it is written into the XML, but isn't actually shown until you call it in code.
- Android inflates the View at runtime, but then removes it from the parent (leaving a "stub" in its place). When you call `inflate()` (or `setVisible(View.VISIBLE)`) on that stub, it is reattached to the View tree and displayed.
```xml
<ViewStub android:id="@+id/stub"
          android:inflatedId="@+id/subTree"
          android:layout="@layout/mySubTree"
          android:layout_width="120dip"
          android:layout_height="40dip" />
```
```java
ViewStub stub = (ViewStub) findViewById(R.id.stub);
View inflated = stub.inflate();
```
Handy!

Finally, note that you can statically include XML layouts inside other layouts by using the [`<include>`](http://developer.android.com/training/improving-layouts/reusing-layouts.html) tag:
```xml
<include layout="@layout/sub_layout">
```

#### If Time
- Go over Tasks & Toasts from last week

### Action Items
- get started on the SunSpotter; setting up the app and creating a basic layout e.g., try to make the "static" layout.
  - We'll talk more about `ListViews` and how to work with external data tomorrow.
