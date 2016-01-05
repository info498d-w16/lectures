# 01-05 Introduction

### Welcome
- Good Morning! This is *498 D: Android Development*
- Plan for the day:
  - Do introductions & syllabus
  - Android History
  - (break)
  - Getting Started: "Hello World"

### Introductions [10min]
**Who am I?**
- "Hi, I'm Joel". New Senior Lecturer
- Grad school at UCI, studied _ubiquitous computing_ and _pervasive games_ Did a chunk of Android development (aimed at sensor sharing and tracking), but also studied a lot of HCI-style theory stuff in the mobile space
  - Caveat: this was a while ago (like Android 2.0 Froyo targets), and the platform keeps evolving
  - Have taught some courses that _used_ Android as a dev platform, but not focused on the system itself. New course for me. Will have some experimentation :)
- Also about me: I live in Tacoma, so have a commute. If I'm not responsive or in the office, probably on bus. And I rush home at end of the day to see my daughter (who is 2 and adorable)

**Who are you?**
- Stats from catalyst?
- But also introduce yourself to each other! Make a new friend
  - Who are you? Where are you from?
  - Most excited about for this class?

### Syllabus [20min]
- Course materials are on Canvas or GitHub. Syllabus is on Canvas.
- Contact info is at the top
  - Office hours in middle of the day, but in my office most days. Feel free to send me an email and make an appointment for another time!
  - Everyone get the Slack invite last week? Slack is basically a really slick chat room system with killer search, cross-device syncing, and sweet integration with other tools. It's the new hotness in the tech industry, so if you've never tried it, now is your chance
    - Really effective way of asking questions! Often easier than email.
- Description: _"This course will introduce you..."_
- Course Calendar
  - We have a rather... "ambitious" schedule--but you're iSchool students!
  - Cover overall basics of Android development in first 3 weeks, and then get more practice and development by focusing on specific fun/useful aspects for rest of class
  - Will have weekly programming assignments: due early Tuesday **mornings**, which you should read as _"Monday night when you go to bed."_
    - Get started on assignments early!
  - Last 4 weeks won't have homeworks, but will do a final group project, which will be an app of your own design. We'll talk more about that as we get closer to it.
  - **[[Questions?]]**

- Assignment **Details** are on GitHub, written into the README of each repo.
  - In order to get access to this stuff, you'll need to (a) sign up for GitHub and (b) let your TA know your GitHub username so we can add you to the org. You should do that ASAP (like during break or immediately after class).

- Policies are on linked wiki page. Things to highlight:
  - Participation: let's try and keep this interesting! Come to class ready to code along with examples; ask lots of questions; volunteer ideas; etc. Take responsibility for your own learning!
  - To help accommodate schedules, you each have two "late days" you can spend on assignments. Think of them as a "extensions with no questions asked". To use one, submit the assignment and include a comment announcing that you're using it.
    - Intended to be used for those weeks when stuff just piled up
  - Collaboration: this is Informatics; it's okay to talk to other people and get help! The goal is to get things working, and to learn from whatever resources you can.
    - However, the assignments you turn in should be your own--Gilligan's Island rule is good for making sure you don't accidentally copy off someone else.
    - If you find/adapt examples from the internet, be sure and cite your source! Include a comment with where you got that from, or note it in your submission
    - This is the Academic Integrity issue, which I assume everyone is aware of and familiar with by this stage in your careers?

- **_Questions on administrative/logistic stuff?_**


### Android History [20min]
- Start off with a question: _What is Android?_
  - It's an Operating System! (software that connects hardware to software and provides general servics)
    - Mobile specific!
  - "Android" also refers to "platform" (e.g., devices that use the OS) and the ecosystem that surrounds it.
    - This includes the device manufacturers who use the platform, and the applications that can be built and run on this platform.
- History
  - 2003: founded by "Android Inc" to build mobile OS operating system (similar to what Symbian (Nokia) was doing)
  - 2005: acquired by Google, who was looking to get into mobile
  - 2007: announces [Open Handset Alliance](http://www.openhandsetalliance.com/), group of tech companies working together to develop "open standards"
    - Google; HTC, Samsung, Sony; T-Mobile, Sprint NTT DoCoMo; Broadcom, Nvidia, etc. Now 84 companies.
  - 2008: First Android device (HTC Dream / T-Mobile G1)
    - 528Mhz ARM chip; 256MB memory; 320x480 resolution capacitive touch; slide-out keyboard!
    - fun device!
  - 2010: First Nexus device: Google-developed "flagship" devices
    - Nexus One: 1Gz Scorpion; 512MB memory; 480x800 AMOLED capacitive touch
    - Comparison: iPhone 6S+ (1.85Ghz dual core ARMv8, 2GB RAM, 5.5" @ 1920x1080)
  - 2014: Android One (low-cost for developing countries)
  - 2015: Project Brillo, aiming for embedded devices
- Market share (e.g., http://www.businessinsider.com/iphone-v-android-market-share-2014-5)
  - questions about what is counted... but what we care about is that there are _a lot_ of Android devices out there!
- Version History
  - [Wiki chart](https://en.wikipedia.org/wiki/Android_version_history)
    - Different "versions" are named after desserts. But from developer perspective, care about API version (e.g., what set of _interfaces_ are available).
  - Interactive version: http://www.android.com/history/
  - Usage breakdown at: http://developer.android.com/about/dashboards/index.html
- Upgrading
  - Updates to devices historically through purchasing new devices (every 18m on average in US)
  - Otherwise updates--including security updates--come through carriers. This is a problem from a consumer perspective!
    - But Google looking to change that; moving some services out of OS into separate "App" (Google Play Services).
  - Android OS is "open source"; latest version at https://source.android.com/. Worth actually digging around in this!
- Legal Battles
  - Would be remiss if didn't mention some of the legal issues surrounding Android
  - Biggest is **Oracle v Google**: basics is that Oracle says the _Java API_ is copyright, so because Google uses that API is Android, Google is violating the copyright
    - Claim: the method signatures themselves (and how they work) are protected!
    - CA court decided for Google in 2012 (can't copyright an API!); reversed by Federal Circuit in 2014. Supreme Court refused to hear in 2015.
    - https://www.eff.org/cases/oracle-v-google for more
    - Just recently: Google will use OpenJDK implementation of Java in Android N, instead of their own (http://venturebeat.com/2015/12/29/google-confirms-next-android-version-wont-use-oracles-proprietary-java-apis/)
      - won't impact us now (or much at all), but may solidify differences in Android and Java SE
  - Others as well:
    - Apple v Samsung: "You infringed our intellectual property!"
    - FairSearch v Google: "You're using predatory pricing!"

**BREAK [5min] @11:30 latest**

### Some Android Architecture [10min, if time?]
- diagram: https://source.android.com/devices/
- Runs on linux kernel (drivers/etc. are in native/C code)
- But applications are written using the Java language that we all know and love.
  - plus resources which are specified in XML.
  - **ASIDE**: Java Review tomorrow for lab... what would be useful?
    - Key Advanced Topics: generics, polymorphism, abstract classes/interfaces, some GUI basics

- Pre-Lollipop (5.0): used [Dalvik](https://en.wikipedia.org/wiki/Dalvik_(software)), a virtual machine (similar to the JVM from Java SE)
  - Java code --compile--> JVM bytecode --translated--> DVM bytecode
    - stored in DEX or ODEX files ("[optimized] Dalvik executables")
    - process is called "dexing"
  - For CS people: register-based architecture (not stack-based!)
  - Includes JIT compilation to native code (like the Java HotSpot)
- Post-Lollipop (5.0): uses Android Runtime (ART)
  - compile into native code on installation! ("Ahead of Time" AOT)
  - accepts DEX bytecode for backwards compatibility
  - faster execution, but longer install time
- https://source.android.com/devices/tech/dalvik/
- Android applications are packaged into APK files (basically zip files)
  - Which are then "side-loaded" or cryptographically signed to be uploaded to the App Store.
- Note: application frameworks are pre-DEXed on device; actually compiling against empty stubs!
  - But any other libraries you include are copied into the app code.
- So when building an App...
  1. Generate Java source files (e.g., for resource files, which are in XML)
  2. Compile Java code into JVM bytecode
  3. "dex" the JVM bytecode into Dalvik bytecode
  4. Pack in assets and graphics into an APK
  5. cryptographically sign the APK file to verify it
- Tools basically do all this for us! We'll just write Java (and resource XML) code


### Development Tools [10min]
- Since it's in Java, we can build Android apps on any OS (unlike some other mobile OS)
  - But the emulator sucks on Windows; recommend you use Mac or a physical device
  - Make sure to use the HAXM (Intel acceleration manager!)
- Physical devices are the best--you'll need USB cable to be able to wire your device into your computer.
  - Any device should be fine; don't need cell service (just WiFi mostly)
  - If you don't normally use an Android phone, play around with it a bit to get use to the interaction language. E.g., how to click/swipe/drag/long-click things.
  - Turn on [developer options](http://developer.android.com/tools/device.html)!

##### Software
- Java 7 **SDK**
- [Gradle](https://gradle.org/) or [Apache ANT](http://ant.apache.org/)
  - These are automated build tools--in effect, they let you specify a command that will do a bunch of steps (e.g., compilation, moving, etc) at once.
  - ANT is the old way, but Gradle is the new hotness
  - You'll be using Gradle for first homework (Java review), and we'll be poking at it periodically through the course.
- [Android Studio & SDK](http://developer.android.com/sdk/index.html)
  - Make sure the SDK command-line tools are installed!
    - put `tools` and `platform-tools` on the `PATH`
    - run `adb` to check
- SDK Tools include:
  - `android`: does SDK/AVD (virtual device) work. Basically IDE commands, but from command-line
  - `emulator` runs the emulator. "virtual phone" in your computer, represents a generic device with hardware you can specify. But has some limitations!
  - `adb` "Android Device Bridge"; connection between your computer and the device. Used for console output!
  - all of these are built into IDE, but command-line is a good fall-back!


### Hello World [30min]
In time remaining, lets get an App up and running and see what we actually will be working with!
- Launch Android Studio (may take a few minutes to open)
- Start a new project!
  - use uwnetid in domain
  - note project location! Desktop is good for now
  - Target: this is the "minimum" SDK you support. Going to target Ice Cream Sandwich (4.0.3) for most this class. This is the earliest version we'll support.
    - Note that this is different than the "target SDK", which is the version you tested on! We'll adjust this in a moment, and will test on API 21 (Lollipop).
    - If you're testing on a physical device running older version, you can target that. But we'll grade at the 5.0 target.
- Select an Empty Activity
  - Activities are "Screens" in your application (things the user can do). We'll talk more about this tomorrow
- And boom, have an Android app!

#### Emulator
- We can run our app by clicking the "Play" or "Run" button
  - We will need to define a device, so let's make an emulator!
    - Nexus 5 is a good choice
      - Use Lollipop, Google APIs, x86!
      - Snapshot to speed up loading
    - Specify **hardware keyboard**
    - Want to go in and edit it (`Tools > Android > AVD Manager`) so it accepts keyboard input!
  - Can to unlock, and there is our app!

#### Project Contents
So what does our app look like in code? What do we have?
- `app/` folder contains our application
  - `manifests/` contains the **Android Manifest** files, which is sort of like a "config" file for the app
  - `java/` contains the Java source code for your project
    - And we can find the `MyActivity` file in here
  - `res/` contains resource files used in the app. These are where we're going to put layout/appearance information
- Also have the `Gradle` scripts. There are a lot of these:
  - `build.gradle`: Top-level Gradle build; project-level (for building!)
  - `app/build.gradle`: Gradle build specific to the app **use this one to customize project!**
    - we can change the target SDK here!
  - `proguard-rules.pro`: config for release version (minization, obfuscation, etc).
  - `gradle.properties`: Gradle-specific build settings, shared
  - `local.properties`: settings local to this machine only
  - `settings.gradle`: Gradle-specific build settings, shared
  - ANT would give:
    - `build.xml`: Ant build script integrated with Android SDK
    - `build.properties`: settings used for build across all machines
    - `local.properties`: settings local to this machine only
    - We're using Gradle, but be aware of ANT stuff for legacy purposes
- `res` has resource files. These are **XML** files that specify details of the app--such as layout.
  - `res/drawable/` : graphics (PNG, JPEG, etc)
  - `res/layout/` : UI XML layout files
  - `res/mipmap/` : launcher icon files
      - fun fact: MIP comes from "multum in parvo", latin for "much in little". Map cause image mapping
  - `res/values/` : general constants
  - See also: http://developer.android.com/guide/topics/resources/available-resources.html

Let's look at what the application does:
- We start with the **Activity** Java source
  - We extend [`Activity`](http://developer.android.com/reference/android/app/Activity.html) (actually a subclass that supports Material Design stuff), making our own customizations. This is a lot like extending `JFrame`!
- Override `onCreate()` method that is called when the Activity starts (more on lifecycle tomorrow).
  - Call super, and then `setContentView` to specify what the content of our activity is.
  - Passing in a value from something called `R`. What _type_ of thing is this? (It's a class!)
  - `R` is a class that is **generated at compiletime** and contains constants that are defined by the "resource" files!
    - Those files are converted into Java variables, which we access through the `R` class.
- `R.layout` refers to the "layouts" resource, so can go there.
  - Can open then up in "design" view. This lets you use GUIs to lay out your application (like a powerpoint slide). But frowned upon--much cleaner/nicer to write it out in code.
    - Same difference between writing your own HTML and using FrontPage or DreamWeaver or Wix
  - XML: eXtensible Markup Language.
    - tags, attributes, values; nesting
  - Defines a layout, and inside that is a TextView (part of text), which has a value--text!
    - Can change that, and re-run the app!
  - If time: can also define this in `values/strings` (e.g., as a constant, refer to as `@string/message`)
- We'll talk about the layout and using these resources _A LOT_ more next week.
- If time: set an icon! (`File > New > Image Asset`)

### Conclusion
That gets us started! Tomorrow we'll do some Java review and go into more details about these Activity things.
- (if time, can poke around IDE)

#### Action Items:
* Sign up for GitHub and let us know your username!
* Get started on Homework 1 (warmup)!
  - If you get done early and can dive into Hwk 2, the better :p
