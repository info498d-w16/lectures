## 02-02 Permissions and Files

### Admin [5min]
- Homework check-in
  - Did the last one work out any better?
  - Current one is online, smaller in scope (though still that last "extra" piece that is probably too big, but isn't as bad as it seems!)
    - Making a location-based drawer... and being able to share the drawings! And go draw something!
    - _Not_ a partner assignment!
- {{Guest talk debrief}}
- We're just going to keep working with yesterday's repo, to add in a few details.


## Permissions
This week we're dealing with location, which as we mentioned is sensitive information that you need permission to access. So let's talk about how Android handles [permissions](http://developer.android.com/guide/topics/security/permissions.html) in a bit more detail:
- The biggest part of Android's design is the idea of [_sandboxing_](https://en.wikipedia.org/wiki/Sandbox_(computer_security)): each application gets its own "sandbox" to play in (where all the toys are kept), but isn't able to go outside the box. It's not a total 100% sandbox, but things outside the sandbox are things that would be impactful to the user (like network or file access, etc).
  - In order to get outside the sandbox you need to explicitly request permission, as we've done before.

- This also happens at a package level, where packages (applications) are isolated from packages _from other developers_; you can use certificate signing (which occurs as part of our build process automatically) to mark two packages as from the same developer if we want them to interact.

- Additionally, Android's underlying OS is Linux-based, so it actually uses Linux's permission system under the hood (with user and group ids that grant access to particular files or processes).

We ask for permission by declaring usage explicitly in the `Manifest`. These permissions we can access are divided into two categories: [normal and dangerous](http://developer.android.com/guide/topics/security/permissions.html#normal-dangerous).
- [**Normal permissions**][normal-list] are those that may impact the user (so require permission), but don't pose any serious risk. They are granted by the user at _install time_; if the user chooses to install the app, permission is granted to that app. See the list for examples.
- **Dangerous permissions**, on the other hand, have the risk of violating privacy concerns, or otherwise messing with the user's device or other apps. These _also_ need to be granted at install time. But _IN ADDITION_, in the latest version of Android (6.0 Marshmallow, API 23), users _additionally_ need to grant permission access at runtime, when you try to actually invoke the "permitted" action.
  - The user grants permission via a pop-up. Note that permissions are granted in "groups", so if the user agrees to give you `RECEIVE_SMS` permission, you get `SEND_SMS` permission as well. See [the list](http://developer.android.com/guide/topics/security/permissions.html#permission-groups).
  - As I read the docs, when you grant permission at runtime that permission stays granted as long as the app is around (at least until it's closed, but I think it becomes tied with the until you uninstall it). The big difference here is that the user can choose to _revoke_ or _deny_ privileges at any time... so you always have to check if the user has granted the privileges or not (even if they won't get nagged about it).

Let's walk through how we grant permissions at run-time, by modifying our `LocationActivity`.
- For setup: we'll need to change our target SDK to 23, AND make sure we're running 6.0 on the emulator (because this only works if the OS supports it AND you've targeted that high!)
  - If you're working off a hardware phone... well you may not be able to totally reproduce this demo :/

First we _still_ need to request permission in the `Manifest`; if we haven't announced that we might ask for permission, we won't be allowed to ask in the future.

Before we do something that requires permissions we can check that we have permission with:
```java
int permissionCheck = ContextCompat.checkSelfPermission(activity, Manifest.permission.PERMISSION_NAME);
```
- This is basically a look-up function. It will return either `PackageManager.PERMISSION_GRANTED` or `PackageManager.PERMISSION_DENIED`

If permission is granted, great! We can go about our business. But if it's not, then we have to ask for it. We do this by calling
```java
ActivityCompat.requestPermissions(activity, new String[]{Manifest.permission.PERMISSION_NAME}, REQUEST_CODE);
```
This method takes a context and then an array of permissions that we access to. We're also going to give it a request code (an `int`), which we can use to identify that request in a callback (just like we've done when sending an Intent for a result... is this pattern looking familiar)?
- This is a lot like sending an Intent, if the Intent were to the permission system.

And then we need that callback:
```java
public void onRequestPermissionsResult(int requestCode, String permissions[], int[] grantResults) {
    switch (requestCode) {
        case REQUEST_CODE: {
          if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
            //have permission! Do stuff!
          }
        }
    }
}
```

We check which request we're hearing, what permissions were granted (if any; they can piece-wise grant permissions!), and then we can react if everything is good...like by finally requesting location.
- So in the end, our location system has like 3 levels of callback: to start the API, to request permission, to request the location, and THEN to actually get a location! (Lots of whew)

Note that if they deny us permission once, we might want to try and explain _why_ we're asking permission (see [best practices](http://developer.android.com/training/permissions/best-practices.html)). Google offers a utility method (`ActivityCompat.shouldShowRequestPermissionRationale()`) which we can use to check if they've denied us once. And if that's true, we might show a Dialog or something to explain ourselves--and if they OK that dialog, then we can ask again.


## File System Access
Permissions are significantly there to control access to the file system, so we're going to take a chance to talk about [working with files](http://developer.android.com/training/basics/data-storage/files.html) in Android. This is a form of [Data Storage](http://developer.android.com/guide/topics/data/data-storage.html) we can use, in addition to the `SQLiteDatabase` and `SharedPreferences`.

Android devices file locations into two groups: **Internal storage** and **External stroage**. Names come from when devices had built-in memory, as well as external SD cards.
  - [Internal storage](http://developer.android.com/guide/topics/data/data-storage.html#filesInternal)) is always accessible, and by default files saved internally are _only_ accessible to your app. Similarly, when the user uninstalls your app, the internal files are deleted. This is usually the best place for "private" file data.
  - [External storage](http://developer.android.com/guide/topics/data/data-storage.html#filesExternal) is not always accessible, is world-readable, and only gets removed on uninstall if you do it in a specific way (that we'll talk about!)
  - [When do we use each?][internal-v-external]
    - I kept bouncing on where to save the file for the homework, so got it working with internal :p

Both of these storage systems also have a "cache" location. A cache is "secret storage", but in computing tends to refer to "temporary storage"
- The Cache is different; in that Android will automatically delete cached files if storage space is getting low... but you really shouldn't rely on it to do that (delete your own cache files when you're done with them!) Basically use this for temporary files, and try to keep them _small_ (less than 1MB)

All of these systems involve working with the [`File`](http://developer.android.com/reference/java/io/File.html) class. This represents a "file" (or a "directory") object.
- We can instantiate a `File` by passing it a directory (which is another file) and a filename (String). Note that instantiating the file will create it if it doesn't exist.
- Note that we can test if a `File` is a file with `.isDirectory()`, and we can create new directories by taking a file and calling `.mkdir()` on it. Can get a list of `Files` inside the directory with `listFiles()`.
- The difference between internal and external storage, _in practice_, basically involves which directory you save the file in!

### External Storage
We'll do [External Storage](http://developer.android.com/guide/topics/data/data-storage.html#filesExternal) as a demo, since the code is sort of a super-set of what you can do.
- First we're going to need permission! `WRITE_EXTERNAL_STORAGE`
  - This is a _dangerous_ permission, so we'd need to ask for permission again... or we can set our target SDK down a notch for testing :p
- Then we need to do is check if external storage is available (e.g., that the SD card is mounted). We do this with a check:
```java
public boolean isExternalStorageWritable() {
    String state = Environment.getExternalStorageState();
    if (Environment.MEDIA_MOUNTED.equals(state)) {
        return true;
    }
    return false;
}
```

Then we need to pick what directory to save the files in. With external storage, we have two options:
- We can save the file _publicly_. We use `getExternalStoragePublicDirectory()` to access this, passing it what [type][external-dir-types] of directory we want (e.g., `DIRECTORY_MUSIC`, `DIRECTORY_PICTURES`, `DIRECTORY_DOWNLOADS` etc). This basically drops files into the same folders that everything else is accessing, great for shared data.
- Alternatively since API 18, we save the file _privately_, but still on external storage (these are world-readable, but are hidden from the user as media, so don't look like public files). We do this with the `getExternalFilesDir()` method, again passing it a _type_ (since we're basically making our own version of the public folders). We can also use `null` for the type!
  - Since API 19 (4.4 KitKat), you don't need permission to write to private external storage. So can only get permission for versions lower than that:
  ```xml
  <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"
                   android:maxSdkVersion="18" />
  ```
  - If we have time, we can look at what this creates on the file system by logging in with `adb -s emulator-5554 shell`, if `adb` is on path...

We then can write to the file by creating a `FileOutputStream` object (or a `FileInputStream` for reading). We just pass this the `File` to write to.
- We write `bytes` to the stream... but can write a String by calling `myString.getBytes()`.
- **remember to `.close()` the stream when done!**
- We can also use the same decorators as in Java (e.g., `BufferedReader(FileReader(file))`) if we want those capabilities.


### Internal Storage & Cache
[Internal storage](http://developer.android.com/guide/topics/data/data-storage.html#filesInternal) works pretty much the same way. Remember that internal storage is always _private_ to the app.
- We don't need write permission for internal storage!
- For internal storage, we can use `getFilesDir()` to get the files directory (just like we did with external)
- But _alternatively_, we can use `context#openFileOutput()` (or `context#openFileInput()`) and pass it the _name_ of the file to open. This gives us back the `Stream` object for that file in the `filesDir()`, without us needing to do any work (cutting out the middle-man!)
  - takes a second parameter: `MODE_PRIVATE` will create the file (or replace a file of the same name) and make it private to your application. Other modes available are: `MODE_APPEND` (which adds to the end of the file if it exists instead of erasing), `MODE_WORLD_READABLE`, and `MODE_WORLD_WRITEABLE`.

Can access internal cache with `getCacheDir()` (and same process), or external cache with `getExternalCacheDir()` (but usually use internal cache).

And again, same process for reading and writing, once you have the file.


## Sharing Files
Once we have these files, we can share them with other apps! How do we communicate with other apps? `Intents`!
- Let's craft an _explicit intent_ for `ACTION_SEND`. We'll set our type as `text/plain` (cause we're sending a text file)
- We can then attach the file as an extra---specifically `EXTRA_STREAM`. The extra won't actually be the `File`, but a `Uri` (think: the "url" or location of the file).
  - get the Uri with `Uri.fromFile(File)`.
- Since multiple activities may support this action, we can then wrap the intent in a "chooser" to force the user to pick which Activity to use:
```java
Intent chooser = Intent.createChooser(shareIntent, "Share File");
//check that there is at least one option
if (shareIntent.resolveActivity(getPackageManager()) != null) {
    startActivity(chooser);
}
```
- (this should work with email, unless I need to set it up, in which case someone else can demo it :p)

What happens if we try and share an Internal file? You'll get an error, because the other (email) app doesn't have permission to read that file!

There is a way around this though, and it's by using a `ContentProvider`. We keep mentioning these (we used on when accessing the SMS list). What is a `ContentProvider`?
- A `ContentProvider` provides an abstraction around a set of data (think a database, but can also imaging the file system as a "database", with keys as filenames and values as files!), in a way that allows it to be interacted with _like_ a database. For example, we can `query()` it (just like a database), and it will do the work of returning us usable values (even if it isn't actually a database underneath!)
  - Like imagine if we wanted to treat an XML file as a "database"; the `ContentProvider` would do the work of converting the XML data into structured responses like we'd get from an SQL query.
  - Or more to the point, by converting a set of `Files` into a set of readable contents (e.g., accessible with the `content://` protocol) that can be used and returned and understood by other apps! This is the specific work of a [`FileProvider`](http://developer.android.com/reference/android/support/v4/content/FileProvider.html).
- We'll talk about these in more detail (like how to create them) later, but want to give you another taste of how they work for files in particular (in part because I got it working, and was going to be relevant for homework).

Setting up a `FileProvider` is luckily not too complex, though it has a couple of steps. You will need to declare the `<provider>` inside you Manifest (see the [guide link](http://developer.android.com/training/secure-file-sharing/setup-sharing.html) for an example).
```xml
<provider
    android:name="android.support.v4.content.FileProvider"
    android:authorities="edu.uw.mapdemo.fileprovider"
    android:exported="false"
    android:grantUriPermissions="true">
    <meta-data
        android:name="android.support.FILE_PROVIDER_PATHS"
        android:resource="@xml/fileprovider" />
</provider>
```
The attributes you will need to specify are:
- `android:authority` should be your package name followed by `.fileprovider` (e.g., `edu.uw.myapp.fileprovider`). This says what source/domain is granting permission for others to use the file.
- The child `<meta-data>` tag includes an `androd:resource` attribute that should point to an XML resource, of type `xml` (the same as used for your SharedPreferences). _You will need to create this file!_ The contents of this file will be a list of what _subdirectories_ you want the `FileProvider` to be able to provide. It will look something like:
```xml
<?xml version="1.0" encoding="utf-8"?>
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <files-path name="my_maps" path="maps/" />
</paths>
```
The `<files-path>` entry refers to a subdirectory inside the Internal Storage files (the same place that `.getFilesDir()` points to), with the `path` specifying the name of the subdirectory (see why we made one called `maps/`?)

Once you have the provider specified, you can use it to get a `Uri` to the "shared" version of the file using:
```java
Uri fileUri = FileProvider.getUriForFile(context, "edu.uw.myapp.fileprovider", fileToShare);
```
(note that the second parameter is the "authority" you specified in your `<provider>` in the Manifest). You can then use this `Uri` as the `EXTRA_STREAM` extra in the Intent that you want to share!


And that about covers it!


<!-- References -->
[normal-list]: http://developer.android.com/guide/topics/security/normal-permissions.html
[external-dir-types]: http://developer.android.com/reference/android/os/Environment.html#lfields
[internal-v-external]: http://developer.android.com/training/basics/data-storage/files.html#InternalVsExternalStorage
