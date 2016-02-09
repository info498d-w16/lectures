## 02-09 Gestures & Motion

### Admin
- Homework check-in
  - Any issues to address?
    - You can send "mock" locations to the emulator via `telnet` with `geo fix lat lng`, if that helps at all--sorry I didn't see that earlier!
  - Next assignment online (or will be, shortly). Much more open-ended; pair-work. Basically an application of drawing, gestures, and motion (what we'll talk about this week).
    - These are examples of "features" you can include in Android apps (not required for _all_ apps), but it's the stuff that I think makes them novel and unique and so worth considering!
- **PROJECT!**
  - Project details are [online](https://canvas.uw.edu/courses/1023396/pages/project), we can go over briefly
  - This is a _group_ project! We can do group forming on Thursday in Lab (so you have a week to work on the first part, which is a short written proposal); I'd like to have groups setup by then!
- Fork & clone repo for today, as always. Actually has starter code we'll go over!
  - We'll be doing a lot of "play-developing"; less pre-written out than it could be, so hopefully is okay...

### Drawing
As I said, at this point we're focusing on the special capabilities (interfaces, sensors) of mobile devices. We looked at _location_ last week, and this week we'll be focusing on gestures and motion sensors (i.e., "move your hand around" functions).

But in order to demo all this stuff, we're going to take a side-trip and talk about some simple graphics/media work first! My favorite topic in programming: drawing pretty pictures!

- Have you every drawn pictures in Java, using the [`Graphics2D`](https://docs.oracle.com/javase/8/docs/api/java/awt/Graphics2D.html) library? Or the HTML5 Canvas? Well in Android it's really similar!

Android provides us a class called [`Canvas`](http://developer.android.com/reference/android/graphics/Canvas.html) that does a lot of the same work as `Graphics2D`: it represents a "drawable" context. We can call methods on it to draw rectangle, circles, and even images (represented by `Bitmaps`)!

#### Custom View
In order to draw stuff, we need to have a `View` to draw. The easiest way to get ourselves a `View` we can _programmatically_ draw on is to create our own **custom View**, which we can specify the drawing of. We can make our own special `View` by subclassing it (remember: `Buttons` and `EditTexts` are just subclasses of `View`), and then filling in a method (`onDraw()`) that specifies how that View is rendered on the screen.

Customizing a View isn't too hard, but to move things along I've provided one: `DrawingView`. Let's walk through it!
- We `extend View` to subclass
- `View` has a pile of constructors; we'll override them all (though they end up calling the last that actually does setup)
- In here we also set up a [`Paint`](http://developer.android.com/reference/android/graphics/Paint.html), which is a set of information about how we want to draw: color, stroke width, font-size, anti-aliasing options, etc. We'll mostly use them for color.
- We can then start overriding methods. There are a few we're going to focus on:
  - `onSizeChanged()` will get called when the `View` changes size: this is on inflation _and_ on rotation, so it can act a little bit like `onResume`.  
  - `onMeasure()` is recommended to be overwritten in the [docs](http://developer.android.com/guide/topics/ui/custom-components.html#custom), but we're going to skip that for time and space (and since our `View` is going to take up the entire screen)
  - `onDraw()` is where the magic happens: this method gets called whenever the `View` needs to be displayed (like `paintComponent()` in Swing). This callback is handed a `Canvas` object that we can draw on!

The `Canvas` can be drawn on in a couple of ways:
- We can call methods like `drawColor()` (to fill the background), `drawCircle()`, or `drawText()` to draw specific shapes/etc. on it.
- We can also draw a [`Bitmap`](http://developer.android.com/reference/android/graphics/Bitmap.html), which represents a graphics raster (e.g., a 2D array of pixels). If we have a `Bitmap`, we can set the colors of individual pixels, and then draw the `Bitmap` onto the Canvas (thereby double-buffering). This is useful for pixel-level drawing
  - If you've used MS Paint, it's the difference between the shape drawing and the "zoom in" drawing

Note that we can cause the `Canvas` to be "redrawn" by calling `invalidate()` on the `View`: this causes Android to need to recreate it, thereby redrawing it. By repeatedly calling `invalidate()` we can do something approximating animation!

Animation is the process of imparting life (from latin _"anima"_). We tend to mean giving motion&mdash;having images change over time
- Animation (and video in general) involves showing a sequence of images over time. If the images go fast enough, then the human brain interprets them as being part of the same successive motion.
- Each image is called a "frame". Film tends to be 20-24 frames per second (fps), video is 29.97fps, and video games aim at 60fps.
- Thus we can achieve animation by `invalidating()` and redrawing more than 16 times per second. E.g., call `invalidate()` at the end of `onDraw()` to cause a recursive loop.
  - This can be difficult, since determining _what_ to draw is computationally expensive! If we're calculating every pixel on a 600x800 display, that's half a million pixels we have to calculate! At 60fps, that's 28 million pixels per second. For scale, a 1Ghz processor can do 1 billion operations per second (and that's part of why we use the GPU!)

(Android does have other systems for doing animation as well; we'll look at those more later).

#### SurfaceView
Since all this calculation can take some time, we want to be careful it doesn't block the UI thread! We'd like to instead do the drawing in the background. Luckily, Android gives us a class that is specially designed for being drawn on in a background thread: the [`SurfaceView`](http://developer.android.com/reference/android/view/SurfaceView.html). Unlike normal `Views` that are somewhat ephemeral, a `SurfaceView` is a dedicated drawing surface that we can interact with in a separate thread.

These also take some work to setup, so let's again look at one that is provided!
- We extend `SurfaceView` and _implement_ `SurfaceHolder.Callback`. A `SurfaceHolder` is an object that "holds" (contains) the underlying drawable surface. We interact with the `SurfaceView` through the holder to make sure that we're _thread-safe_: that only one thread is interacting with the surface at a time
  - In general there will be two threads training off use of the holder: our background thread that is drawing on the surface, and then UI thread that is showing the surface to the user.
- We register the holder in the constructor with `getHolder()`, and register ourselves for callbacks when it changes. We also instantiate a new `Runnable`, which is an object that can be "run" in a separate thread (e.g., an object with a `run()` method that we can execute).
- Our `SurfaceHolder.Callback` interface requires methods about when the surface changes, and so we fill those in.
  - `onSurfaceCreated()` starts our background thread
  - `onSurfaceChanged()` ends up acting a lot like `onSizeChanged()` from the other `View`
  - `onSurfaceDestroyed()` stops the background thread in a "safe" way
- If we look at the `Runnable`, it's basically an infinite loop:
  1. Grab the Surface's Canvas, locking it so only used in this thread
  2. Draw on it
  3. Then "push" the Canvas back out to the rest of the world, basically saying "we're done drawing, you can show it to the user now"
- Overall, this process will cause us to "redraw()" as fast as possible, all without blocking the UI thread! Great for animation, which can be controlled and timed in the `update()` helper method (e.g., only `update()` variables at a particular rate)

And that gives us a drawable surface that we can interact with!


### Gestures
So let's start adding some interaction. In particular, let's start with [Touch Gestures](http://developer.android.com/training/gestures/index.html). Touch screens are a huge part of Android devices (and mobile devices in general, especially since the first iPhone) that are incredibly familiar to most users. We've already indirectly used the touch interface, with how we've had users click on buttons (theoretically using the touch screen). But here we're interested in more than just button clicks, which really could come from anywhere: instead, how we can react to the ways the user might _caress_ the screen: flings, drags, multi-touch, etc.

Android devices automatically detect various touching interactions (it's how buttons work); we can respond to these _events_ by overriding the `onTouchEvent()` callback, which is executed whenever there is something happening involving the touch screen
- We can log out the event to see the kind of details we get!

There are _lots_ of things that can cause `TouchEvents`, so much of our work involves trying to determine what semantic "gesture" the user made. Luckily, Android provides a number of utility methods and classes to help with this.

The most basic is `MotionEventCompat.getActionMasked(event)`, which extracts the "type" of the event from the motion that was recorded:
```java
int action = getActionMasked(event);
switch(action) {
        case (MotionEvent.ACTION_DOWN) : //put finger down
            Log.d(DEBUG_TAG,"Action was DOWN");
            return true;
        case (MotionEvent.ACTION_MOVE) : //move finger
        case (MotionEvent.ACTION_UP) : //lift finger up
        case (MotionEvent.ACTION_CANCEL) : //aborted gesture
        case (MotionEvent.ACTION_OUTSIDE) : //outside bounds
        default :
            return super.onTouchEvent(event);
}
```
This lets us react to basic touching. For example, we can make it so that taps (`ACTION_DOWN`) will teleport the ball to where we click! We can also use the `ACTION_MOVE` events to let us drag the ball around.
- Note that there are some other details you need to consider when doing drags with multi-touch: see [this guide](http://developer.android.com/training/gestures/scale.html#drag) for details.

#### Fling
But we can also detect and react to more complex gestures: long presses, double-taps, or flings (a "flick" or swipe on the screen. See [here](https://www.google.com/design/spec/patterns/gestures.html#gestures-drag-swipe-or-fling-details)). Android provides a [GestureDetector](http://developer.android.com/training/gestures/detector.html#detect) class that can help identify these actions.
- The easiest way to use this---particularly when interested in a particular gesture (like fling)---is to _extend_ `GestureDetector.SimpleOnGestureListener`. We can then override the gestures we're interested in responding to: e.g., `onFling()`.
- We will also need to override `onDown()` and have it return `true`. This is because all gestures begin with a `DOWN` action, and if we return `true` with whether we've "consumed" (handled) the event or not. If we haven't handled it, then it gets passed up the hierarchy to other elements, like the Notification drawer.
- We can instantiate this class with `mDetector = new GestureDetectorCompat(this, new MyGestureListener());`
- Then in our `onTouchEvent()` method, we can pass the event into our Gesture Detector to process, and if it doesn't see anything continue on it ourselves:
  ```java
  boolean gesture = this.mDetector.onTouchEvent(event); //check gestures first!
  if(gesture){
      return true;
  }
  ```
We can now add the ability to "fling" the ball, say by taking the fling velocity and assigning that to the ball (and slowing it down by 1% on update, eventually stopping when less than 1). Scaling velocity down by 10 might work---I have not tested these numbers.

#### Multi-Touch
One of the neat things about modern touch screens is that they support [multi-touch](http://developer.android.com/training/gestures/multi.html), or the ability to detect two different "contacts" (fingers) independently. Multi-touch is actually a pretty awesome interaction mode; it's very ["sci-fi"](https://www.youtube.com/watch?v=NwVBzx0LMNQ).

Android lets you react to multi-touch by responding to `ACTION_POINTER_DOWN` events, which occur when a second "pointer" joins the gesture (after `ACTION_DOWN`).
- Each pointer is in an arbitrary array, but is given an "id" which we can fetch with `getPointerId(index)`. We can then figure out where _that_ pointer is in later interactions by using `findPointerIndex(id)`. This will let you basically track individual fingers. See [the docs](http://developer.android.com/training/gestures/multi.html) for details.
- I would demo this, but emulator doesn't support multi-touch; so left as an exercise for the homework!

Note that we can respond to common multi-touch gestures (like "pinch to scale") by using _another_ kind of GestureDetector called a [`ScaleGestureDetector`](http://developer.android.com/training/gestures/scale.html#scale).
- Again, subclass the simple version (`ScaleGestureDetector.SimpleOnScaleGestureListener`), and fill in the `onScale()` method. You can get the "scale factor" from the gesture with the `.getScaleFactor()` method. We could (theoretically) use this to scale our circle!

#### Break

### Motion Sensors
Touch gestures are good and well, but lots of modern laptops have touch screens too... and we're interested in the capabilities that make mobile devices _unique_. Because they are mobile, you can pick up and move the devices easy: shake them around, toss them in the air, etc. Android devices often come with [**sensors**](http://developer.android.com/guide/topics/sensors/sensors_motion.html) that are able to measure these movements.

Android supports lots of generic and specific sensors (see [the guide](http://developer.android.com/guide/topics/sensors/sensors_overview.html)), using an overall "sensor framework" to listen for and react to information detected by particular sensors. This is so that apps can harness what might be built into the device (from motion sensors like an _accelerometer_, to environmental sensors like _thermometers_ or _barometers_). Additionally, the system is structured so you can develop and attach your own (hardware) sensors if needed, though that's much more low level than we'll be going.

- We're going to focus on the **accelerometer** today, which is a motion sensor used to detect acceleration force (e.g., how fast the device is moving). This sensor is found on most devices and is useful for detecting motions, and is low-power! Note that the **gravity sensor** is also useful for determine device tilt and works in a similar manner, though is a bit less common.

- Note also that you'll need to have physical hardware for this demo; if you're just on the emulator today, find a friend with a device to partner with


In Android, we start our work with sensors by using the [`SensorManager`](http://developer.android.com/reference/android/hardware/SensorManager.html) class, which will tell us information about what sensors are available, as well as let us register listeners for sensors. We get access to it with
```java
mSensorManager = (SensorManager) getSystemService(Context.SENSOR_SERVICE);
```
- We can then do things like get a list of sensors with `.getSensorList(Sensor.TYPE_ALL)`, which returns a `List<Sensor>`. A [`Sensor`](http://developer.android.com/reference/android/hardware/Sensor.html) represents a particular sensor, including things like it's type (not represented as subclasses).

Note that if we want to make sure that anyone installing our app (e.g., from the Play Store) has the particular sensor we're using, we can specify that in the manifest with the `<uses-feature>` tag:
```xml
<uses-feature android:name="android.hardware.sensor.accelerometer"
              android:required="true" />
```
- Note that this doesn't actually keep us from installing the app, it's just extra information.

In order to get information from the sensor, we need to _register_ a listener for events that it will produce. Again, common pattern in interactive systems!
- We get access to the sensor using `.getDefaultSensor(type)` to determine if there is a particular sensor; if it's `null`, then the sensor is missing and so we can quit. We do this is `onCreate()`
- We will want to register the sensor in `onResume()` (and unregister in `onPause()`) this will "turn on/off" the sensor only when we're definitely using it, since sensors can cause significant battery drain!
  ```java
  mSensorManager.registerListener(this, mSensor, SensorManager.SENSOR_DELAY_NORMAL);
  ```
    - The `this` needs to be a `SensorEventListener`, so we'll need to implement that interface!
    - the `DELAY_NORMAL` specifies a 200,000 microsecond (200ms) delay between samples; we can also use `DELAY_GAME` for a faster 20ms delay.

Finally, we can use the sensor information by filling in the `onSensorChanged(event)` callback. This is called _whenever_ we get a new sensor reading!

Sensor readings are stored in the `event.values` array, but what those numbers mean depends on the sensor that we're working with. With the **accelerometer**, each entry is the acceleration force on a different _axis_ [x,y,z].

- Caveat: for the accelerometer, gravity is always exerting an accelerating force, even when at rest! So we'll need to "factor out" the gravity in order to figure out actual acceleration (see [the docs](http://developer.android.com/guide/topics/sensors/sensors_motion.html#sensors-motion-accel). For a further example, see [the sample app](https://github.com/android/platform_development/blob/master/samples/AccelerometerPlay/src/com/example/android/accelerometerplay/AccelerometerPlayActivity.java).
- Or an easier solution: use the `LINEAR_ACCELERATION` Sensor, which automatically subtracts out gravity! This is a "virtual", compound sensor that combines readings from other sensors to produce useful results. See also [here](https://source.android.com/devices/sensors/sensor-types.html#linear_acceleration). But of course, if were interested in tilt, then we _care_ about the gravity!
  - Lots of options here, really, depending on the sensors available. We can also look at the [`Rotation Vector Sensor`](https://source.android.com/devices/sensors/sensor-types.html#rotation_vector) (another virtual sensor) to determine angles.
  - `SensorManager.getOrientation()` is a common solution as well, though much more complex to work with since you need to deal with rotation matrices.
  ```java
    //untested, adapted from https://github.com/kplatfoot/android-rotation-sensor-sample/blob/master/src/com/kviation/android/sample/orientation/Orientation.java
    float[] rotationMatrix = new float[9]; //3x3
    SensorManager.getRotationMatrixFromVector(rotationMatrix, rotationVector);

    //adjust rotation matrix for device orientation here

    // Transform rotation matrix into azimuth/pitch/roll
    float[] orientation = new float[3];
    SensorManager.getOrientation(adjustedRotationMatrix, orientation);  
  ```

Let's talk a little bit about the [coordinate system](http://developer.android.com/guide/topics/sensors/sensors_overview.html#sensors-coords). Sensors use a pretty standard 3D coordinate system, with x and y axis what you'd expect, and the z axis coming out of the front of the device (following the [right-hand rule](https://en.wikipedia.org/wiki/Right-hand_rule) like a good coordinate system!)
- The trick though is that the coordinate system is based on the _device's_ **frame of reference**, not on the Activity or configuration! That means when you rotate the device (e.g., into landscape mode), the coordinate system doesn't change, and you need to adjust your math accordingly. You can determine the device's current orientation with `Display#getRotation()`:
  ```java
  Display display = ((WindowManager) getSystemService(Context.WINDOW_SERVICE)).getDefaultDisplay();
  display.getRotation();
  ```
  And then use `SensorManager#remapCoordinateSystem()` or just adjust our readings (e.g., as in the [demo][Accelerometer Demo]).

With all that in place, we can try to use our accelerometer values to move the ball around (e.g., by accessing the variable in the DrawingView). We'd probably want to synchronize to make this thread-safe... but can skip for now, though may get awkward.
- Or we can go over the `synchronized` keyword and the use of lock objects.


In the end, we have a system that we can use gestures and motion to interact with, rather than just depending on "buttons"! Or at least that's the goal.

Tomorrow we'll finish up some animation and media concepts (and maybe tweak the sensors if we have time).


<!-- References -->
[Accelerometer Demo]: https://github.com/android/platform_development/blob/master/samples/AccelerometerPlay/src/com/example/android/accelerometerplay/AccelerometerPlayActivity.java
[Sensor Types]: https://source.android.com/devices/sensors/sensor-types.html
