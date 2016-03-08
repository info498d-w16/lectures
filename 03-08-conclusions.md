## 03-08 Conclusions

### Admin & Presentations
Today is the last day of class (because Thursday are presentations!)

Thursday's presentation logistics:
- Lab time is work time; can put the final touches on your project (e.g., make sure it deploys and stuff works; typo/icon fixes, etc). This is *not* the time to be adding features!
- This is also when you should **submit your project to Canvas** (it's technically due at the end of lab!)
  - In addition, be sure and have everyone take 3 minutes to fill out the ___peer evaluation survey___ linked from the project write-up (or on catalyst, etc). This lets me know if there were any issues in the groups, and a way I can give extra credit to people who went above-and-beyond. **YOU MUST DO THIS**

Around 10:30 we'll start [presentations][project writeup]: Rather than having everyone sit and squint at a mobile phone held at the front of the room, will do this as a poster session:
- Each group will man a "station" here. 1-2 people should be at the station at all times, but others can wander around and see the projects (make sure you switch periodically!)
  - You might not be able to see everything, since expect this to run in 8-10 minute "chunks", but can at least check out most the apps!
- At your station, should be have app running on two different (physical) devices: one you can show off, and one that the demo-ees can look at.
- At your station, be prepared to give a 5-minute "pitch": in other words you explain your app for 5 minutes. Walk the visitor through your program, explain how it works, show it off, etc.
  - Be ready to answer any questions!

I've also got some industry guests coming to visit. Guests are intended to let you get some feedback on your work that _isn't_ from me and my academic perspective, and to give these demos a bit more weight. Good practice pitching project!
  - Peri Hartman, head of Perinote LLC (a startup that develops the "Perinote" task/note organizer app),
  - Chris Risner, "evangelist" at Microsoft (focuses on mobile and web offerings)

Plan is we'll cycle around and let you show off your app (a lot!).
- I often do a kind of "peer review" form for people to fill out, but I'm going to trust that everyone will show up and participate in this process... right?

Any questions about the day?

Two things to do today: want to talk _briefly_ about "releasing" apps, and then want to leave you with a couple concluding thoughts

### Deploying to Play Store
There are a couple of steps to [releasing your app][publishing] and publishing it [on the Play store](http://developer.android.com/distribute/googleplay/start.html). Luckily, as with most things in Android, the steps are pretty well documented. But we'll walk through them cause we have the day!

The probably least intuitive piece is [signing your app](http://developer.android.com/tools/publishing/app-signing.html). All Android apps need to be [**cryptographically signed**](https://en.wikipedia.org/wiki/Digital_signature) in order to be installed
- Effectively this is "signing your work": it says that you are verifying that yes, _you_ built this app---and you built it in this way (it hasn't been altered or corrupted)
- Note that Android Apps are self-signed, meaning its not a strong security measure (any malicious actor can sign an app, and the signatures themselves are not too hard to steal or fake). But it does allow Android/Google to _mostly_ identify the author of an app.

We've been doing development in ___debug mode___, meaning that's how we've signed our apps. This uses a temporary, debug certificate that has a known password (signature) so you don't need to retype it every time. It's part of that issue we had getting a shared Maps key to work across multiple computers!
- If we want to buld for release, we need to build and sign our app in ___release mode___. This involves generating a certificate (think: generating a password) and using it to sign our built app.

Android Studio makes it pretty easy to sign your app in release mode:
- Select `Build > Generate Signed APK`
- `Create New` to create a new keystore
- Fill in the details

And that's it, really! This will generate an `.apk` file that can be distributed and installed on any device.
- Note that you can generate this `.apk` as part of the build process under Settings; see docs for details

There are a number of other steps you'll want to keep in mind as you prepare for release
1. Make sure you have a suitable Application icon! This is important. We've made these through Android Studio
- You'll want to pick a good package-name: once you've released your app, you can't change it (other than releasing a different app)
- Turn off logging and debugging! This slows down the app and eats up the log file; so removing these is a good idea
- Review all of your manifest and gradle settings to make sure everything is up to date

See [the docs](http://developer.android.com/tools/publishing/preparing.html#publishing-configure) for more details.

Publishing on the Play Store is its own separate beast; luckily, there is a [Launch Checklist][store checklist] that makes following the steps easy.
- Note that you will need to sign up for a developer account at https://play.google.com/apps/publish/. Registration costs $25, which is why we aren't requiring it.
- The full details I leave to you to read up on: you should be comfortable following the Android docs by now!

Any other questions about releasing apps (I haven't done this, but can give you my best understanding. Min may be able to chime in more).

### Conclusions
So a couple of concluding thoughts before we wrap up:

In this class we've studied how to write programs using the Android framework, allowing us to develop programs for mobile devices
- This framework is designed to let us deal with the restrictions and limitations of mobile platforms: limited resources and memory, different modes of interaction, and a heterogenous set of devices (different screen sizes, processors, etc.). The framework is structured the way it is to try and support these limitations (e.g., minimizing memory usage)
- At the same time, mobile devices do provide some unique capabilities: sensors for location and motion, unique interaction modes like with wearables, etc. And again, the framework is intended to make harnessing these capabilities as easy as possible.

While we don't really have a summary of the course trajectory (since just learning the API), I did want to flag a few key take-aways I hope you get from doing mobile development:

___Develop for Flexibility___
As I said, Android in particular runs across a wide variety of different devices (from low-end to high-end, from old to new). And the framework is used for increasingly large number of platforms (including wearables, auto, etc). So you want to design your applications to be as _flexible_ as possible.
- We've looked at a number of ways to support flexibility (resource configurations, permissions/feature checking, compatibility libraries, etc). These are a core part of the framework _because_ of the variety in the platform... but it's also something you should keep in mind with your own designs.
- Don't assume that phones will be a certain size or used in a certain way. Letting the user solve their problem in a variety of ways!


___Develop for Universal Usability___
Android devices _and users_ may have limitations that you and your device don't:
- Device limitations like small screens, slow processors, etc
- Context limitations like lack of data access or GPS signal
- User limitations (physical disabilities, etc)

As you're developing your apps, you want to make sure to support users and devices with these limitations. This wlll make the apps work better for _everyone_
- using relative measurements like font sizes and layouts to work across less-capable devices
- keep resource and data usage low (makes it run faster for everyone!)
- support accessibility systems like Talkback; consider how to integrate other modalities like voice and motion
- make sure your app "gracefully degrades" if features like GPS or sensors aren't available


___Design for New Experiences___
Finally, when building mobile apps think about what makes these apps unique to _mobile_. As you've seen in 343: the web is capable of a lot of functionality, and mobile phones have full-featured browsers. So you'd want to think about why this should be a mobile device (and only work on mobile platforms) rather than a web site (and work everywhere).
- Some of it is speed and ease of use and how that fits into a user's interaction (e.g., open up the Twitter app rather that going to Twitter)
- Lots of capabilities unique to phones to harness: location sensing, touch and motion, etc.

Challenge you to think through these issues to make sure you're developing something that's useful and _actually solves a problem in the world_. And that goes beyond this class!


### Evaluations
Okay, last piece: this is the first run of this course, so I'd like to get some feedback on what went well and what could go better (e.g., in next quarter's section). There are formal evaluations (will get to those in a minute), but wanted to open up chance for discussion
- What went well? What could go better? Anything you expected to learn/do but didn't?

Specifically:
- Pacing
  - particularly Java: what language pieces should I make sure students are on top of? Which we should have reviewed?
  - some ideas with content (e.g., SQL be pushed way back)
  - And during lecture
    - is in-class coding useful? What could we do better?
- Assignments (in general/specific)
  - Other neat projects to work on instead?
  - Motion "game": more specific (e.g., ball maze) rather than open-ended?
    - non-touch keyboard?
  - Todo list instead of self-tracker?

Finally: course evaluations should be online (you got an email about that this morning). _Please fill these out!!_ This is important to the administrative side of the school, and does a lot to let us know what is working and what we should work on.
- You have some time left in class: take 5 minutes and fill out the evaluation **today**. Right now! You could wait if my presence will have undo influence of course, but please don't ignore this!

Any other questions? Then rest of the period is to work on projects; I'll hang out and help/answer questions as needed.

Thanks so much for a great class!!

#### References
[project writeup]: https://canvas.uw.edu/courses/1023396/pages/project
[project writeup][project writeup]
[publishing]: http://developer.android.com/tools/publishing/preparing.html
[publishing][publishing]
[store checklist]: http://developer.android.com/distribute/tools/launch-checklist.html
[store checklist][store checklist]
