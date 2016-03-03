## 03-03 VR

### Admin
Today I want to do a quick concept discussion and demo about **Virtual Reality**. No code generation (though we'll look through a sample)
- This is intended to flag a domain that you maybe having considered before, and show how Android development fits into that.
- It's basically "here is another cool thing you can do with a phone"!

This shouldn't take too long, and then can let you get back to work :)


### Virtual Reality
_What is virtual reality?_ is the creation of a virtual environment in which the user is (physically and perceptually) immersed. The **immersion** part of this is what makes it important: virtual reality is about "feeling like you are there", and letting your physical actions _directly_ translate to changes in the world or your perception thereof (rather than mediated through things like a mouse or button click). You turn your head in the real world, and your "head" turns in the game as well.
- This gives us more [_embodied interactions_](http://www.dourish.com/embodied/), which is awesome.

Virtual reality is on the edge of becoming an actual genre of technology, with lots of big-name developers getting into the field:
- [Oculus Rift](https://www.oculus.com/en-us/)
- [HTC Vive](http://www.htcvive.com/us/)
- [Playstation VR](https://www.playstation.com/en-us/explore/playstation-vr/) (a.k.a. Project Morpheus)
- [MS Hololens](https://www.microsoft.com/microsoft-hololens/en-us)
- [Samsung Gear VR](http://www.samsung.com/global/galaxy/wearables/gear-vr/)
- [Google Cardboard](https://www.google.com/get/cardboard/)


### Augmented Reality
Note that there are different forms of "virtual reality", depending on how you mix the real and virtual. Milgram et al. (1994) placed **mixed reality** on a continuum, from a "real environment" to a "virtual environment."

In the middle (to one side), you have **Augmented Reality**, which is the addition of virtual elements into a real environment. This is most commonly done by _overlaying_ virtual characters onto a video of a real environment.
- Like an interactive, 3D version of _Who Framed Roger Rabbit_.
- This is what Hololens is doing.

How this works is to develop an application that basically shows what the camera sees on the screen, like a View-finder (or if you were looking through a lens). You then draw virtual elements on top of that view. If you can animate (change) the drawing to correspond with movements of the camera, then it creates the illusion of the virtual object "existing" within the real world!
- Again, like opening a real door but then drawing the character doing the opening

So the question becomes: how do you make the drawing match the camera movement? There are two major strategies: **location-based** and **target-based** AR.

In **location-based** AR systems, you use the phones physical position (determined through GPS and/or compass sensors) to figure out what the camera is pointing at. You then draw the appropriate content that matches that perspective.
- One common use for this is in advertising systems: for example, Yelp has (had?) a system that let's you view ratings of nearby restaurants overlayed over their _actual_ location. It's not practical, but it's kind of fun.
- These are also useful for generating things like walking directions that may actually show you where to go... like a "heads-up-display" in a video game.
- There are existing APIs that help perform this kind of tracking. I've managed to use [Layar](https://www.layar.com/) in the past, with some success.

The second option is **target-based** AR. In this case you let the camera identify objects/points in the real world (called _features_), and then use that view to determine what the camrea is looking at. This involves significant Computer Vision processing, since you need to recognize a _feature_ in a picture, calculate your position base on the orientation/perspective of that _feature_, and then draw the object as appropriate---and do this every single frame!
- _Target-based_ AR is used for fun demos and some games, though it never really took off
- Ikea had [an app](http://www.gizmag.com/ikea-augmented-reality-catalog-app/28703/) where you could make a virtual image of their furniture to see how it looked in your home!
- One of my favorite examples (though kind of old at this point): [ARhrrrr (2009)](https://www.youtube.com/watch?v=cNu4CluFOcw)
- Other systems as well!

Again, you usually would use a third-party library to do target-based augmented reality. There use to be a few open-source options, but many of them have been abandoned. There are still a few useful systems out there.
- One of the biggest is called [Vuforia](https://developer.vuforia.com/), which is pretty neat but a pain to set up. But I do have a demo of it so you can see this stuff in action!

[[tag demos]]

Actually modifying these samples to implement something is beyond what we can do in an hour (or at least, what I'm prepared for at this point).


### Google Cardboard
I think Augmented Reality apps are fun, but they never really took off: the processing is expensive (so couldn't run on most phones for a long time), and the applications are somewhat limited.

But what _is_ on the verge of taking off (surprisingly) is full-blown _virtual reality_, where the user is immersed within a completely virtual environment (rather inserting virtual objects into the real world).
- This is what the Occulus, Vive, etc. are about
- Haven't totally figured out what these will be used for, but there is a lot of interest in developing games and other experiences for this kind of setup (and the hardware that enables that!)

These systems usually work with a _headset_ that contains a screen right in front of your eyes. The image of the virtual world is shown on the screen, and since that's all you can see, it gives the impression that you're inside that world! It's like you're just sitting way to close to the TV.
- In addition, by tracking the _head movement_ (e.g., where the camera goes), the scene can be changed to reflect that new perspective--just like how location-based AR would work!
  - If the headset detects that it's turning left, it will adjust the scene by moving it to the right.
- Finally these systems use [stereoscopic displays](https://en.wikipedia.org/wiki/Stereoscopy) to produce "3D" worlds. This involves showing each eye a slightly different image, like a picture taken from a camera moved 3 inches to the side. Because each eye sees an image _from a different perspective_, the brain believes that the object has real depth!
  - This is the same way that 3D movies and TV works as well (using polarization to show a different image to each eye),

How does this relate to Android? Well to do this kind of VR, you need a screen that can cover your eyes (about "this big"), and powerful computer to do all this processing. What kind of devices might provide those capabilities? Phones!
- You can create a Virtual Reality system by showing the view onto your phone and holding the screen right up to your face.
- And the phone already has things like motion sensors that can detect movement for head tracking, as well as speakers for playing audio... brilliant!
- This is sort of what Samsung's Gear VR does--it's basically a system that holds your phone to your face, using that phone as a screen (with extra hardware to make position tracking more effective)

But if you basically need to strap a phone to your face, there isn't actually a bunch of hardware needed for that. Enter [Google Cardboard](https://www.google.com/get/cardboard/): the cheapest VR headset you can find. It's basically a cardboard (literally) version of the Gear VR, where all the work is done by the phone. Less powerful than
- And along with this basic setup, Google provides an API for developing virtual reality apps. It helps with some of the hard work of (a) doing stereoscopic displays and (b) performing headtracking
- Note that development of apps is also usually done through Unity 3D (a game engine that's great for modeling virtual worlds); but I am not yet a Unity developer so we're going to leave that piece to the side.

Check out the demo app! [[do so]]

### Google Cardboard Code [if time]
Let's look at this [source code](https://github.com/googlesamples/cardboard-java) and try to see what kinds of capabilities are being offered
- Note that most of this code is just doing the hard work of rendering 3D graphics using OpenGL. OpenGL is _very_ verbose, and so will try to skim away those pieces.

Start with the `Manifest`
- Uses OpenGL
- Specify screen orientation
- `NFC` is ["Near Field Communication"](https://en.wikipedia.org/wiki/Near_field_communication), a set of communication protocols that let devices talk when they are in close proximity. Skipped that programming because hardware dependent.

Look at the `MainActivity`
- Extends `CardboardActivity`, which gives us event callbacks for that device (namely its button)
- Implements `StereoRenderer`, which does the heavy lifting figuring out the distance between the eyes so you can "move the camera" and create a stereoscopic display. So you just draw a View, and that View will be told the "position" of the eye associated with it.

The View that we're drawing is a `CardboardView` (specified in the layout), and setup in `onCreate()`
- This view also does the work of setting up a OpenGL Drawable Surface, which is really similar in principle to the `SurfaceView` we used before.
- We actually render the view using two methods: `onNewFrame()` (like our `render()` method, says what to do on every frame), and `onDrawEye()` which is called for each eye with a different set of "location" parameters

I'm being vague about location because it's using computer graphics concepts. An eye holds a _transformation_, which tells you how to "move/rotate/etc" and object (technically, to get from one coordinate system to another)
- Idea: have a square, and then if want to move and rotate it, can specify changes to the origin and the axes, which is equivalent to moving the square in the _other_ frame of reference.
- These transformations are stored as Matrices, which are (mathematically) sets of linear transformations to coordinate spaces. And we'll not go any further than that.

`onNewFrame()` gets a `HeadTransformation`, which includes the movements from "start" that get to the head's current position. This transformation is determined by the SDK, so we don't need to worry about it!

And `onDrawEye()` does the actual rendering _for each eye_--this method is called twice! The rendering code goes in here (and this is more OpenGL work). From the docs:
- The treasure comes into eye space.
- We apply the projection matrix. This provides the scene rendered for the specified eye.
- The Cardboard SDK applies distortion automatically, to render the final scene.

Cardboard also provides an [audio system](https://developers.google.com/cardboard/android/latest/reference/com/google/vrtoolkit/cardboard/audio/CardboardAudioEngine) that allows for "3D sound"--that is, sound that is localized in space. So if the sound is coming from your right, then it's louder in the right speaker. This is nifty (though hard to tell from the demo)

That's a basic demo of getting started with Cardboard. I apologize for not being more on top of things to actually _get_ some headsets we could use... but they're pretty cheap (like $15) if you want to try this out at some point!
- Again, a lot of the graphical/world development is done through Unity, which then exports to an Android device

Questions?
