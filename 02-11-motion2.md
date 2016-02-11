## 02-09 Motion & animation

### Admin
- Homework check-in, get it finished? Assignments taking a bit too long (15hrs instead of 10 target)?
  - For next assignment: I'd be okay moving it to due Friday, though that's the same day your project proposal is due. Thoughts?
- Fork & clone repo for today, as always. Actually has starter code we'll go over!
  - We'll be doing a lot of "play-developing"; less pre-written out than it could be, so hopefully is okay...

#### Group selection! [10mins?]
- Want to make sure I have a list of project groups by _today_. So everyone stand up!
  - If you have a group, go stand with your team members!
  - Who is still missing people? Do you have a quick pitch/idea for a project you want to recruit with? Or do you want me to randomly assign?


### Motion Sensors
Yesterday we talked about using gesture detection to support (more) complex touch interfaces. Touch gestures are good and well, but lots of modern laptops have touch screens too... and we're interested in the capabilities that make mobile devices _unique_. Because they are mobile, you can pick up and move the devices easy: shake them around, toss them in the air, etc. Android devices often come with [**sensors**](http://developer.android.com/guide/topics/sensors/sensors_motion.html) that are able to measure these movements.

Android supports lots of generic and specific sensors (see [the guide](http://developer.android.com/guide/topics/sensors/sensors_overview.html)), using an overall "sensor framework" to listen for and react to information detected by particular sensors. This is so that apps can harness what might be built into the device (from motion sensors like an _accelerometer_, to environmental sensors like _thermometers_ or _barometers_). Additionally, the system is structured so you can develop and attach your own (hardware) sensors if needed, though that's much more low level than we'll be going.

- We're going to focus on the **accelerometer** today, which is a motion sensor used to detect acceleration force (e.g., how fast the device is moving). This sensor is found on most devices and is useful for detecting motions, and is low-power! Note that the **gravity sensor** is also useful for determine device tilt and works in a similar manner, though is a bit less common.

- Note also that you'll need to have physical hardware for this demo; if you're just on the emulator today, find a friend with a device to partner with!


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


### Animation (if time?)
Yesterday we talked about how we could perform animation simply by updating our drawing that occurs each frame. This is great for games or other complex animations... but sometimes we want to have some simpler, platform-specific effects. Android actually involves a number of different animation systems that can be used within and _across_ Views:

[Property Animation](http://developer.android.com/guide/topics/graphics/prop-animation.html) is a general animation framework, that basically performs animated _linear interpolation_ on whatever values you give it to change.

[Scene Transitions](http://developer.android.com/training/transitions/index.html) A framework for transitioning between layouts; the material animations use this indirectly. See also [Animations](http://developer.android.com/training/animation/index.html)
- These are useful and able to be added to really any application... but are "extra flair" rather than being core to something like games, so I'll leave you to look up these systems on your own.
- [Material Animations](http://developer.android.com/training/material/animations.html) include animations built into the "material design" style (found in Lollipop and later), and include various effects and transitions in response to user interaction (e.g., ripples, sliding fragments, etc).

[OpenGL Animations](http://developer.android.com/guide/topics/graphics/opengl.html) are available for doing 3D animated systems.

...and there might be a few others out there as well. A good place to start if you're interested in adding animations and transitions to your app is the [Material Design Guide](http://developer.android.com/design/index.html), particularly the [animation section](https://www.google.com/design/spec/animation/authentic-motion.html).

We're going to focus on **Property Animation** as an example; potentially useful for homework and for doing more "game-like" experiences. Also less setup. [Property Animation](http://developer.android.com/guide/topics/graphics/prop-animation.html) is an animation system where you specify a start state, an end state, and a duration, and the Android systems changes from the start to the end over that length of time--thereby producing animation!
- This is calculated using a concept called [interpolation](https://en.wikipedia.org/wiki/Interpolation); most usually a variation of **linear interpolation**.
  - Anyone done linear algebra? Fun aside! Basically, we figure out how far along we are in the animation, and then take that a weighted average of the start and end values. We can get _non-linear interpolation_ by adjusting the weights.

The main engine for doing this kind of interpolated animation in Android is the [`ValueAnimator`](http://developer.android.com/guide/topics/graphics/prop-animation.html#value-animator) class. This class lets you specify the start, end, and duration, and then can be run to calculate all the values in between. It has a number of static methods (e.g., `.ofInt`, `.ofFloat`, `.ofArgb`) which creates "animators" for interpolating different _values_. For example:
```java
ValueAnimator animation = ValueAnimator.ofFloat(0f, 1f);
animation.setDuration(1000);
animation.start();
```
- Of course, just running this doesn't do anything we can see, since it's change the numbers but those don't relate to anything! We can get the values by setting a [listener](http://developer.android.com/guide/topics/graphics/prop-animation.html#listeners) and overriding a callback we're interested in (e.g., `onAnimationUpdate()` from `ValueAnimator.AnimatorUpdateListener`).

But more commonly, we want to have our animation change the _property_ of some object--for example, the color of a view, the position of an object, etc. We can do this easily using the [`ObjectAnimator`](http://developer.android.com/guide/topics/graphics/prop-animation.html#object-animator) subclass. Basically, this subclass runs an animation like the `ValueAnimator`, but has the built-in functionality to change the property of an object on each step.
- It does this by calling a **setter** for that property---thus the object needs to have a property setter for us to animate it! (If not, we can make a wrapper or just develop our on `ValueAnimator`).
```java
//change the "alpha" property of foo (e.g., call foo.setAlpha())
ObjectAnimator anim = ObjectAnimator.ofFloat(foo, "alpha", 0f, 1f);
anim.setDuration(1000);
anim.start();
```

We can use this to (say) change the ball's radius or position using an interpolated animation.
- Make a _getter_ and only pass the Animator an ending value if we want
- use `.setRepeatCount(INFINITE)` and `.setRepeatMode(REVERSE)` to cause it to repeat!
- Note that we're changing the property, and because our `SurfaceView` is refreshing all the time, we get to see the updates in action :)

If we want to include multiple animations in sequence, we can use an [`AnimatorSet`](http://developer.android.com/guide/topics/graphics/prop-animation.html#choreography), which gives us methods about the ordering:
```java
//example from docs
ObjectAnimator animX = ObjectAnimator.ofFloat(obj, "x", 50f);
ObjectAnimator animY = ObjectAnimator.ofFloat(obj, "y", 100f);
AnimatorSet animSetXY = new AnimatorSet();
animSetXY.playTogether(animX, animY);
animSetXY.start();
```

We can also start defining these animations in XML as [resources](http://developer.android.com/guide/topics/graphics/prop-animation.html#declaring-xml). These are put into the `/res/animator` directory.
  ```xml
  <set android:ordering="together"> <!-- together is default -->

    <objectAnimator
             android:propertyName="x"
             android:duration="500"
             android:valueTo="400"
             android:valueType="intType"/>
    <!-- ... -->
  </set>
  ```
  Note that you'll need to **inflate** the Animator resource, just like we did with layouts:
  ```java
  ObjectAnimator anim = (ObjectAnimator) AnimatorInflater.loadAnimator(myContext, R.anim.animator);
  anim.setTarget(myObject);
  anim.start();
  ```

Note that we can also use this same framework to animate changes to `Views`: buttons, images, etc. Views are objects and have properties (along with getters and setters we can use)... so we can use just an `ObjectAnimator`! See [the docs](http://developer.android.com/guide/topics/graphics/prop-animation.html#views) for a list of properties we can change (includes `x, y, rotation, alpha, scaleX, scaleY`, etc.)
- Or to make this even simpler, Views provide a `ViewPropertyAnimator` which modifies multiple properties together, but does so in a much more efficient manner. We can access this with the `.animate()` method on a `View`, then call relevant methods that we want to change:
```java
myView.animate().x(50f).y(100f);
```
(But really, if you want to animate layout changes on a modern device, you should use [transitions](http://developer.android.com/training/transitions/index.html)).

There are [lots more ways](http://developer.android.com/guide/topics/graphics/prop-animation.html) to customize exactly what you want your animation to be, but this should give you a lot of the basics! You can also look at [official demos](https://android.googlesource.com/platform/development/+/master/samples/ApiDemos/src/com/example/android/apis/animation).




<!-- References -->
[Accelerometer Demo]: https://github.com/android/platform_development/blob/master/samples/AccelerometerPlay/src/com/example/android/accelerometerplay/AccelerometerPlayActivity.java
[Sensor Types]: https://source.android.com/devices/sensors/sensor-types.html
