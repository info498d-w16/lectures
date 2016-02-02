## 02-02 Location

### Admin [5min]
- Homework check-in
  - Hey we have a chance to ask questions about homework before it's due! Any problems?
  - Next one is online, though may fill in details/testing notes for the last part. _Not_ a partner assignment!
- Thursday's lab we're going to have a guest speaker--Nadia Payet, who is an Engineer on Google Maps for Android--so the _official_ app that uses these APIs! She's going to talk to us more about the Map APIs, and walk us through a demo/exercise.
  - This is a special industry guest: I expect you to _show up on time_ (we start at 9:30, if you've forgotten; be here early!), and be respectful and engaged! Participate, ask questions, and basically show off how awesome iSchool students are (so that industry people will want to come back).
  - This is also a good chance to meet someone in the industry and possibly ask questions; great networking opportunity. SO BE THERE.
- Fork and clone the repo for today. We're going to tweak the SDK/Emulator again, but in a little bit.

## Localization
Today we're going to talk about **Location** and **Mapping**. But before we get into the Android code, I want to give a little bit of background/context for some of the technologies that are in play.

**Localization** in this context is the process of determining _location_. Why is this important?

- Android was originally developed as an operating system for _mobile_ devices. Key word: **mobile**. What makes phones and tablets special, and different from desktops, is that they can and do move around. Their position and location matters _significantly_ for how they are used; it's a major part of what separates the functionality of Android apps from any other computer application.

- Localization gives apps the ability to create new kinds of user experiences, and to adjust their functionality to fit the user's _context_ (classic example: it can determine if you are at home, in the office, or on the bus)! In fact, one of the winners of the _first_ Android Developer Challenge (2008) was [Ecorio](http://web.archive.org/web/20100209012355/http://www.ecorio.org/), an app that figured out whether you were driving, walking, or busing and calculated your carbon footprint from that. Localization lets us know about the user's situation (though mobile phone location is [not necessarily](http://dx.doi.org/10.1007/11853565_8) a proxy for user location).

- Note that this perspective is coming out of the the realm of [Ubiquitous Computing](https://en.wikipedia.org/wiki/Ubiquitous_computing), a discipline that considers technology that is _ubiquitous_ or everywhere, to the point that it "blends into the surroundings." That's _my_ take on why phone development is important; so that you can compute without thinking about it.

### Localization Techniques
Ubicomp researchers have been developing localization systems for _years_ (Ubicomp "officially" started with a vision paper by Mark Weiser at PARC in 1991). The classical reference is a [survey paper](http://dx.doi.org/10.1109/2.940014) by Jeff Hightower (who was getting his PhD in the CSE department here!)

- Early Example: _Active Badge_ (AT&T) has a name-badge emit an infrared message, that is picked up by sensors in a room to determine where the wearer is! This is room-level precision; improvements and _triangulation_ (calculating angles to what you see) got it down to about _10cm_ precision! However, this requires a lot of infrastructure to be built into a particular room. With Android, we're interested in more general-purpose localization. Mobile devices use a couple of different kinds of localization (either independently or together).

#### GPS
**GPS** is the most common, and what most people think of when they think of localization. [GPS](https://en.wikipedia.org/wiki/Global_Positioning_System) stands for "Global Position System"---and yes, it can work anywhere on the globe.

_How does it work?_
GPS's functionality depends on satellites: 24 satellites in high orbit (not geo-synchronous) around the earth. Satellites are distributed so that 4 to 12 are visible from any point on Earth, and their locations are known with high precision. These satellites are each equipped with an atomic, synchronized clock that "ticks" every nanosecond. At every tick, the satellite broadcasts its current time and position. You can almost think of them as really loud alarm clocks.

The thing in your phone (and your car and whatever) that you call as "GPS" or a "GPS device" is actually a _GPS_ ___receiver___. It is able to listen for the messages broadcast by these satellites, and determine it's position based on that information.

How? First, the receiver calculates the _time of arrival_ (TOA) based on its own clock and comparing time-codes from the satellites. It then uses the announced _time of transmission_ (TOT; what the satellite was shouting) to calculate the [time of flight](https://en.wikipedia.org/wiki/Time_of_flight), or how long it took for the satellite's message to reach the receiver. Because these messages are sent at (basically) the speed of light, the [time of flight] is equivalent to the distance from the satellite! And once it has distances from the satellites, the receiver can use [trilateration](https://en.wikipedia.org/wiki/Trilateration) to determine it's position based on the satellites it "sees" (Trilateration is like Triangulation, but relies on measuring distances rather than measuring angles. Basically you construct three spheres of a given radius, and then look to see where they intersect). Whew!

- Important terms here: _time of flight_, _trilateration_

GPS precision is generally about 5 meters; however, by repeatedly calculating the position (since ticks every nanosecond), can use differential positioning extrapolate position with even higher precision, increasing precision to less than 1 meter! This is how Google can determine where you're walking.

But the biggest problem with GPS: you need to be able to see the satellites! This means that it doesn't work indoors, as building walls block the signals. Similarly, in very urban areas (think downtown Seattle), the buildings can bounce the signal around and throw off the TOF calculations, making it harder to pinpoint position.

- Also: requires a lot of energy to constantly listen for the alarm, so can be a big hit to battery life, which is something we need to worry about with mobile devices!

#### Cell Tower Localization
But your phone can also give you a rough estimate of your location even _without_ GPS. It does this through a couple of techniques, such as relying on the cell phone towers that provide the phone network service. This is also known as [**GSM localization**](https://en.wikipedia.org/wiki/Mobile_phone_tracking#Network-based) (Global System for Mobile Communications; the standard for cell phone communication used by many service providers). The location of these towers are known, so we can determine location based off them in a couple of ways:
- If you're connected to a tower, you must be within range of it. So that gives you some measure of localization right off the bat.
- If you can see multiple towers (important for handoff purposes), you can trilaterate the position between them. This can give accuracy within 50m in urban areas, with more towers producing better precision.

#### WiFi Localization
But wait there's more! What other kinds of communication antennas do you have in your phone? **WiFi**. As WiFi has became more popular, efforts have been made to identify the _location_ of WiFi hotspots so that they too can be used for [trilateration and localization](https://en.wikipedia.org/wiki/Wi-Fi_positioning_system). This is often done through crowdsourced databased, with information gathered through [war driving](https://en.wikipedia.org/wiki/Wardriving).

War driving involves driving around with a GPS receiver and a laptop, and simply recording what WiFi routers you see at what locations. This then all gets compiled into a database that can be queried--given that you see _these_ routers, where must you be?

- Google got in a lot of trouble for doing this as it was driving around to capture Street-View images.

WiFi localization can then be combined with Cell Tower localization to produce a pretty good estimate of your location, even without GPS.


I want to flag that just like the old _Active Badge_ systems, all of these localizations systems rely on some kind of existing infrastructure: GPS requires satellites; GSM requires cell towers, and WiFi needs the database of routers. All these systems require and react to the world around them, making localization dependent on the location as well! And in fact, Google provides the ability to automatically use all of these different techniques, abstracted into a single method call!

### Representing Location
So once we have a location, how do we represent it?

First, note that there is a philosophical difference between a "place" and a "space." A _space_ is a location, but without any social connotations. For example, GPS coordinates, or Cartesian xy-coordinates will all indicate a "space." A _place_ on the other hand is somewhere that has social meaning: Mary Gates Hall; University of Washington; my kitchen. Space is a computational construct; place is a human construct. When we talk about localization with a mobile device, we'll be mostly talking about _space_. But often _place_ is what we're really interested in, and we may have to translate (Google does provide a few ways to move between the two).

Our space locations will generally be reported as two coordinates: **Latitude** (lat) and **Longitude** (long) (there's also **altitude** or height, but that isn't super relevant for us). _What are latitude and longitude?_

- Latitude is the _angle_ between the equatorial plane and a line that passes through a point and the center of the earth--the angle you have to go up (north-south) the earth's surface from the equator. Effectively, it's a measure of "north/south". Latitude is usually measured in _degrees north_, so going south of the equator leads to a negative latitude (though this can be expressed positively as "degrees south").
- Longitude is the _angle_ between the prime meridian plane and a line that passes through a point and teh center of the earth--the angle you have to go across (east-west) the eaarth's surface from the meridian. Effectively, it's a measure of "east/west". Latitude is measured in _degrees east_, so going east of the meridian. That mean that the western hemisphere has "negative longitude" (though this can be expressed as positive "degrees west").

Example: [UW's GPS Coordinates](https://www.google.com/search?q=uw+gps+coordinates) are N/W, so this would be expressed as N (positive) and E (negative).

The distance between degrees and miles depends on where you you are (particularly for longitude!) However, as a very rough estimate in this part of the world, .01 degrees is _about_ a mile (not accurate, but so you have a sense of scale).


### Break: 5min before we get back to programming, as things download!


## Android Location
Okay, let's go ahead and make an app that will access the phone's location. We'll just log it out for now, and then hook up a map shortly.

First thing's first: we need to make sure we can access [Google Play Services](http://developer.android.com/google/play-services/setup.html); these are a special set of libraries that provide additional functionality to Android. That functionality will include the location and mapping tools we're interested in (Much of this functionality was originally built into code Android, but Google has since been moving it into a separate app that can be more easily distributed and updated!)

There are a few steps to this...
1. First we need to make sure we've downloaded the proper library. Go to `Tools > Android > SDK Manager` to open up the manager for the various versions of Android we've installed.
  - Should have 6.0 installed for this week's assignment, but not for today...
  - Under `SDK Tools`, select `Google Play Service` and `Google Play Repository` to download those
2. Next, we want to make sure our emulator will support these services. Go to the `AVD Manager`, and make sure the _target_ platform include the `Google APIs` (we may need to download it).
3. We'll also need to modify your `build.gradle` file so that you can get access to the Location classes. In the ___module-level___ `build.gradle` file, under `dependencies` add
  ```
  compile 'com.google.android.gms:play-services-location:8.4.0'`
  ```
  This will load in the location services (but not the other services, which require keys)

Tada, now we have access to this set of libraries! Yay!

Lastly, we'll need to request permission to access the location. There are two permission levels we can ask for: `ACCESS_COARSE_LOCATION` (for GSM/WiFi level precision), and `ACCESS_FINE_LOCATION` (for GPS level precision). We'll use the later because we want GPS-level precision
- This is a **dangerous** permission, so in Marshmallow we'd need to ask for permission at run-time. We'll talk about how to do that more on Thursday.

### Google Play Services
We're going to use Google Play Services to get our location. The Google APIs provide a nice set of methods for [accessing location](http://developer.android.com/training/location/retrieve-current.html) (without us needing to specify the source of that localization, GPS or GSM), and is the recommended API to use.

The first thing we need to do is get access to the API; we do this with a [`GoogleApiClient`](http://developer.android.com/reference/com/google/android/gms/common/api/GoogleApiClient.html) object. We construct this object in the `onCreate()` callback, using a `GoogleApiClient.Builder`:
```java
if (mGoogleApiClient == null) {
    mGoogleApiClient = new GoogleApiClient.Builder(this)
            .addConnectionCallbacks(this)
            .addOnConnectionFailedListener(this)
            .addApi(LocationServices.API)
            .build();
}
```
We need to set up what are called the Connection Callbacks, which are callbacks that will occur when we connect to the Google APIs. We do this by implementing the `GoogleApiClient.ConnectionCallbacks` and `GoogleApiClient.OnConnectionFailedListener` interfaces. Each have a single method that we can fill in; in particular, the `onConnected()` method is where we can "start" our API usage (like asking for location!)
- `onSuspended` and `onFailed` are for when the connection is stopped (like `onStop`) or if we fail to connect. See [Accessing Google APIs](https://developers.google.com/android/guides/api-client) for details.

We also specify that we want to access the LocationServices API, before creating the client.

Finally, we need to actually connect to the client. We do this in our `onStart()` method (and disconnect in `onStop()`):
```java
protected void onStart() {
    mGoogleApiClient.connect();
    super.onStart();
}
```
This of course, will lead to our `onConnected()` callback being executed.


### Location
Once we have the the client connected, we can start getting the location! We can do this using the following method:
```java
LocationServices.FusedLocationApi.getLastLocation(GoogleApiClient)
```
- [FusedLocationApi](https://developers.google.com/android/reference/com/google/android/gms/location/FusedLocationProviderApi) is a "unified" interface for accessing location. It fuses together all of the different ways of getting location, providing which-ever one best suits our specified needs. Can think of it as a "wrapper" around more detailed location services.
- The `getLastLocation()` will return the _last_ location that our phone's location sensor received. Note that this may be null if we've never gotten a location before! (it doesn't request a location, just looks up one that we've already gotten).

So how do we make sure the phone has gotten a location? Particularly if we're indoors?
- We can give a "fake" location to the emulator! To do this, open up the **Android Device Monitor** (`Tools > Android > Android Device Monitor`). This is an application that basically lets you inspect and interact with your Android device (like the emulator). Note that when this is open, Logcat stuff goes here (I think).
  - We can use this to "send" the phone a location, as if it had calculated it from the GPS!
  - Test by giving it UW (47.6550 N, -122.3080 E), and then can open Maps to see it work!
  - And can re-run our application (or use the button to test it!)
- the `FusedLocationApi` class also has a `setMockLocation()` method, but I haven't tested that.

The `getLastLocation()` returns a `Location` object, which we can `get` the `Latitude` and `Longitude` of!

### Updating Location
That's great, but I'd really like to be able to get location _whenever_ it updates. Well our `FusedLocationApi` provides a method for this: `requestLocationUpdates()`.

This method is going to take `GoogleApiClient`, but also a [`LocationRequest`](https://developers.google.com/android/reference/com/google/android/gms/location/LocationRequest) object, which specifies the details of our request (e.g., how we want to have our phone search for it's location).
- We create the object, then specify the "interval" that we want to check for updates. We can also specify the "fastest" interval, which is the maximum rate we want updates (assuming they are available). It's a bit like a minimum and maximum. (5s - 10s is good for real-time navigation).
- We also specify the [priority](https://developers.google.com/android/reference/com/google/android/gms/location/LocationRequest#constant-summary), which is the indicator to the FusedLocation about what kind of precision we want. `HIGH_ACCURACY` basically means GPS.
```java
mLocationRequest = new LocationRequest();
mLocationRequest.setInterval(10000);
mLocationRequest.setFastestInterval(5000);
mLocationRequest.setPriority(LocationRequest.PRIORITY_HIGH_ACCURACY);
```

The last parameter for the `requestLocationUpdates()` method is a `LocationListener`--an object with a callback that can be executed when the location is updated. So we'll make ourselves into one of those by implementing the interface and filling in the `onLocationChanged()` method. For example, we can log out the location received here!

And with that we can now send different locations to the emulator, and watch our position update! Yay!

That's most of what there is to working with location. Lots of pieces (because callbacks all over the place!), but that does the work.


## Google Maps
Now that we have location, how can we show it off? Well how about a map? This is Google, so we have an super robust [Maps API for Android](https://developers.google.com/maps/documentation/android-api/) that we can use to easily add an interactive map to our application.

Getting Google Maps setup takes a little bit of work, because again it is part of an external library. Moreover, in order to show the map, you need to [register and get an API key](https://developers.google.com/maps/documentation/android-api/signup#key). You'll have to do this for your homework, but for today's demo I'm just giving you access to my demo key.
- It's pretty self-explanatory to get setup: if you go to add a new `Google Map` Activity, all the infrastructure is added for you; you just need to follow a link and copy-paste.

Once we've got our key, we can show a map by using a [`SupportMapFragment`](https://developers.google.com/android/reference/com/google/android/gms/maps/SupportMapFragment). This is just a basic Fragment, so we add it to our layout like always (the default template codes it into the XML; we'll leave it there for now).
- And after creating the fragment, we call `.getMapAsync()`, which will fetch the `GoogleMap` object (including all the controls, interactions, tiles, etc) asynchronously. And when that's done, it will issue a callback to `this`, because we've implemented `OnMapReadyCallback`. And once our map is created, we can do things to it!

For example: we can position it... but if we want a default position, we should instead do that in the UI fragment settings (see [Controls and Gestures](https://developers.google.com/maps/documentation/android-api/controls) for details).
```xml
<fragment
...
map:cameraTargetLat="47.6550"
map:cameraTargetLng="-122.3080"
map:cameraZoom="16"
map:uiZoomControls="true"
tools:ignore="MissingPrefix" />
<!-- hack fix linter -->
```

Camera targets where we're centered, and "zoom" is how far in we're zoomed (notice this is a `LatLng` object). See [Camera and View](https://developers.google.com/maps/documentation/android-api/views) for details.
- We can also do thing like `setMapType()` (e.g., `MAP_TYPE_HYBRID`)


### Drawing
The map is great, but we'd like to be able to customize what it shows: that is, we want to draw on it! There are a couple of things we can draw:

#### Markers
Markers indicate single locations on the map. We can add them, as demonstrated, using the `addMarker()` method. We can [customize](https://developers.google.com/maps/documentation/android-api/marker#customize_a_marker) these as well:
- `.title()` specifies text that will appear when user clicks on the marker
- `.snippet()` for additional text
- `.anchor()` for exactly where we want the marker image to appear
- and more!

You can also create [custom info windows](https://developers.google.com/maps/documentation/android-api/infowindows) if you want to add more detail to the marker pop-ups.

#### Shapes
We can also draw free-form shapes on the map, anchored to particular locations. A nice generic option (that you'll use in your homework) is the [`Polyline`](https://developers.google.com/android/reference/com/google/android/gms/maps/model/Polyline), which is a series of connected line segments (like a "path" in SVG/d3).

In order to create a line, what we _actually_ define is a set of [`PolylineOptions`](https://developers.google.com/android/reference/com/google/android/gms/maps/model/PolylineOptions). This contains all the information about the line.
- We can `.add()` multiple `LatLng` objects to it, each one representing an end-point of a segment (in order, like an List).
- We can also specify the color with `.color()` (takes a `Color` object)
- And specify the width with `.width`. Note that width is in pixels, and is not dependent on zoom level!

We then take these options and pass them as a parameter to the `GoogleMap.addPolyline()` method. This creates the line on the map, and returns a reference that we can use later.
- We can do things like get the list of `LatLng` points from the `Polyline` if we want to change them. Note that once the line is drawn it is stable; if you want to add more points you need to re-add the entire set.


This is only the most basic demo of what you can do with the Map. We'll do more practice in lab tomorrow (special guest! Remember to come on time!), and you'll be able to play around with this more for your homework.

Questions?
