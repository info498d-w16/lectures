## 01-21 Data Storage

### Admin [10min]
- Homework check-in
  - Get started on making a basic Fragment appear in your application? Maybe two? (resource work!)
  - Use lecture code as reference/demo; we're basically doing half the work in class
- Fork and clone the repo for today! (Continue working off the movie example from yesterday, so we have things started).


## Menu Buttons [10min]
I've added one change to the movies: the ability to view a `FavoritesFragment`. But we need some way to get to this fragment. I decided a good way was to use an [Options Menu](http://developer.android.com/guide/topics/ui/menus.html).
- This is basically stuff we can put in the ActionBar!

How did we set up the menu?
- Made a `menu` resource (which contains "items", which are entries in the menu)
- In the Activity, override the `onCreateOptionsMenu` event so that it inflates my menu
- And then override `onOptionsItemSelected` to be able to respond to when those items are selected!
  - And call "transaction" methods that swap the fragments!
  - Note I'm having issues with the back button and Fragments I didn't have time to trace

This is a _very_ simple menu; we'll do more with menus next week!


## Dialogs [25min]
First thing for today: I want to finish up one topic that we keep running out of time on (and involves Fragments!): [___Dialogs___](http://developer.android.com/guide/topics/ui/dialogs.html). A dialog is a "popup" modal (doesn't fill the screen) that either asks the user to make a decision or provides some additional information. Think like the `alert()` in JavaScript.

There is a base `Dialog` class, but mostly we use pre-defined versions:
- [`AlertDialog`](http://developer.android.com/reference/android/app/AlertDialog.html) is the most common version: a simple message with buttons you can respond to. Like `JOptionPane` if you've used that.
  ```java
  // 1. Instantiate an AlertDialog.Builder with its constructor
  AlertDialog.Builder builder = new AlertDialog.Builder(getActivity());
  builder.setTitle("Alert!")
         .setMessage("Danger Will Robinson!");
  builder.setPositiveButton(R.string.ok, new DialogInterface.OnClickListener() {
    public void onClick(DialogInterface dialog, int id) {
      // User clicked OK button
    }
  });
  AlertDialog dialog = builder.create();
  dialog.show();
  ```
  - The `AlertDialog.Builder` class lets you create them. Then you can specify titles, messages, etc. Allows for easy customization of an Alert.

[[Practice API reading: can you add X functionality to our little alert?]]
  - specify what happens when that cancel button is pressed?

Other built-in alerts:
- [`DatePickerDialog`](http://developer.android.com/reference/android/app/DatePickerDialog.html) and [`TimePickerDialog`](http://developer.android.com/reference/android/app/TimePickerDialog.html) have pre-defined UI for picking things
  ```java
  TimePickerDialog dialog = new TimePickerDialog(getActivity(), new TimePickerDialog.OnTimeSetListener(){
      @Override
      public void onTimeSet(TimePicker view, int hourOfDay, int minute) {
          //...handle time picking...
      }
  }, 11, 50, false);
  dialog.show();
  ```

But the real workhorse, the one we care about, is [`DialogFragment`](http://developer.android.com/reference/android/app/DialogFragment.html). This acts as the "super class" for Dialogs that we'll be working with: by working with Fragments for dialogs, they can fit into the application's lifecycle and handle those events correctly.
- Basically, this can be a floating container for anything else you want to include!
- We subclass this in order to create custom dialogs. Can treat them as fragments (with `onCreateView()`, etc.)... but more common to just override `onCreateDialog()` and use that to "launch" the dialog.
  - Usually create an `AlertDialog` inside here which is then returned. So that you're still building on that, but you can encapsulate it all in your custom `DialogFragment`. A way of making a `createMyDialog()` method, but inside a class.
  - Examples: [here](http://developer.android.com/guide/topics/ui/dialogs.html#DialogFragment), [here](http://developer.android.com/reference/android/app/DialogFragment.html#AlertDialog), etc.

The other really neat trick is that a `DialogFragment` is just a Fragment... so can be used in all the same ways! So if you make your Fragment extend `DialogFragment` instead (and provide the `newInstance()` method, as in examples), you can call that method and then call `.show()` on the resulting Fragment to make it appear as a Dialog... while still being able to use a Transaction to embed it in a layout! See [Dialog or Embed](http://developer.android.com/reference/android/app/DialogFragment.html#DialogOrEmbed) for details.

...the truth is that Dialogs are not really that commonly used in Android (compared to other GUI systems). More likely to just change the Fragment/screen. And 80% of Dialogs that are used are AlertDialogs. But worth being aware of!

### Toasts [10min]
However, there is one other kind of "pop-up" message that is just awesome--particularly for debugging (when you want to check some output/interaction without Logging it), but also for user interaction

A simple, quick way of giving some short visual feedback is to use what is called a [Toast](http://developer.android.com/guide/topics/ui/notifiers/toasts.html). This is a tiny little textbox that pops up at the bottom of the screen for a moment.
- Toast because it pops up :p

Toasts are pretty simple to implement, as with the following example (from the docs):
```java
Context context = getApplicationContex();
String text = "Hello toast!";
int duration = Toast.LENGTH_SHORT;

Toast toast = Toast.makeText(context, text, duration);
toast.show();
```
- But since `this` Activity _is_ a context, and we can just use the Toast anonymous, we can shorten this to a one-liner:
  ```java
  Toast.makeText(this, "Hello world", Toast.LENGTH_SHORT);
  ```
  - Boom, a quick visual alert method we can use for proof-of-concept stuff!
    - And of course you can customize this as well (specifying your own View to toast), though most common to leave it with default styling.
- Note that this uses a static `makeToast()` method, rather than a constructor. This is an example of a Factory method--a design pattern we'll see a lot. Similar to what the `newInstance()` method in Dialog was doing!
- Toasts are intended to be ways to interact with the user (e.g., giving them quick feedback), but can possibly be useful for testing too, since can just watch the screen! Though in the end, Logcat is going to be your best bet for getting deailed information (and catching runtime errors).



## Data Storage [5min]
The other big important topic for the day is **Data Storage**, by which we mean _persistant_ data storage. We've already used models to "keep track of" data in memory, but we're interested in keeping track of information between Activity runs. _Why would we want to do that?_
- Have you ever stored data between executions in an program/application you've written? How did you do it? (brainstorm)

Android has a small pile of different ways to store data. We'll talk about a couple of them in detail.

0. Well for one, we mentioned that we can use the `Bundle` to save Activity state when destroyed, but we're talking about saving data when the _Application_ is closed (also, `onStateInstanceState()` isn't 100% reliable depending on the lifecycle...)
1. Network/cloud
  - We can send data to the cloud over a network connection! We've made network requests before; we can change that from a `GET` request to a `PUSH` request in order to send some data to a server, for that server to store. We can then query that server's API, just like we do with the Movies.
  - Cloud storage services like [Parse.com](http://parse.com) even have [Android APIs](https://parse.com/docs/android/guide) that you can use. They provide a set of Java classes you can include in your application, which does all the "background thread" work for your interaction! Very slick!
2. File system (write to file)
  - SharedPreferrences is another option (basically a shared XML file)
4. SQL Database


## SQLite Database [forever...]
We'll come back to the other options, but let's start with the SQL Database (since that's the one to use in your homewok---though arguably network is more appropriate for this)

SQLite is an incredibly simple relational database, so much so that it's often included in CRUD systems for testing/demos. Because it's so small and lightweight, it's perfect for performance-limited devices like mobile phones!
- Fun fact: SQLite is really just a single file that structures its content intelligently and supports rapid querying and access.

Android saves the database in a private disk space associated with the application, so each app has its own database file.

Android (and Java, really) [give us an API](http://developer.android.com/training/basics/data-storage/databases.html) we can use to query and interact with the SQL database. So let's walk through how to get that set up!

### Schema Contract
We want to make sure that all of our methods and queries interact with the correct table and row names, without having to remember to retype them all. How could we try to _reuse_ this code? Constants!
- But since we have a whole bunch of them, what we'll actually do is make an inner class to contain these constants, which gives us nicer access :) This is called a **contract class**.
  - make it `final`!
  - empty constructor to avoid calling!
  - Best setup: make "database-level" variables constants of this class, and make "table-level" variables into a abstract class within that
    - implement [`BaseColumns`](http://developer.android.com/reference/android/provider/BaseColumns.html) interface to give us an `_ID` primary key, which we're going to need to have for other APIs to work!
- We can also code in some of our SQL queries that we'll be running: in particular, `CREATE` and `DROP`. Again, keep all the database constants in one place!
  ```sql
  CREATE TABLE people (_id INTEGER PRIMARY KEY, title TEXT, year INTEGER, imdbId INTEGER UNIQUE, posterUrl TEXT);
  DROP TABLE IF EXISTS people;
  ```

  ```java
  private static final String SQL_CREATE_ENTRIES =
      "CREATE TABLE " + Entry.TABLE_NAME + " (" +
        Entry._ID + " INTEGER PRIMARY KEY AUTOINCREMENT," +
        Entry.COL_TITLE + "TEXT" + ", "
        Entry.COL_YEAR + "INTEGER"+ ", "
        Entry.COL_IMDB_ID + "INTEGER UNIQUE" + ", " //only one of those!
        Entry.COL_POSTER_URL + "TEXT"
      " )";

  private static final String SQL_DELETE_ENTRIES =
      "DROP TABLE IF EXISTS " + Entry.TABLE_NAME;
  ```


### Connecting to SQLite
So now that we've got stuff setup, we can go ahead and connect to the database! We'll keep doing this in the Database class to keep things organized
- Ideally this would actually be a [Content Provider](http://developer.android.com/guide/topics/providers/content-providers.html), but we're not going to talk about those yet!

In order to set up the database, we're going to make _another_ inner class. This one will extend [`SQLiteOpenHelper`](http://developer.android.com/reference/android/database/sqlite/SQLiteOpenHelper.html), and it does what it says: helps us access and interact with our database.
- In particular, it gives us `getWritableDatabase()`, which we can use to get access to the database!
- We want to override the _constructor_, `onCreate()`, `onUpgrade()`, and `onDowngrade()`
  - `onCreate()` is for when the _database_ is created, and so on.
  - Recommend copying in the code from http://developer.android.com/training/basics/data-storage/databases.html and modifying that!

Note that `onCreate()` gets an [`SQLightDatabase`](http://developer.android.com/reference/android/database/sqlite/SQLiteDatabase.html) object--this is our database that we can call methods on!
- `db.execSQL()` lets us send a raw SQL query (that isn't a `select`--so one that doesn't return anything). We'll do some other calls in a bit

Finally, we can instantiate our `Helper` in order to get the database created and to be able to access it!
- Ideally we should do that in a Background Thread; we'll put it on the main thread ("helper method") for now and see how it goes.
- And because we'll want to access this repeatedly, let's make it a Singleton!
  ```java
  //synchronized so thread-safe--only one thread can call this method at a time
  public static synchronized DatabaseHelper getHelper(Context context) {
    if (instance == null)
      instance = new DatabaseHelper(context);
      return instance;
    }
  ```

### Inserting into the Database
So let's get this up and running: we can just have our FavoritesFragment instantiate the database helper on creation (for now).

To demonstrate that it works, let's also add something to the database!
- You can add new values to by using the `INSERT` SQL statement along with <a href="http://developer.android.com/reference/android/database/sqlite/SQLiteDatabase.html#execSQL(java.lang.String)">`.execSQL()`</a>, but it's usually easier and cleaner to use the <a href="http://developer.android.com/reference/android/database/sqlite/SQLiteDatabase.html#insert(java.lang.String, java.lang.String, android.content.ContentValues)">`.insert()`</a> method. This takes in a [`ContentValues`](http://developer.android.com/reference/android/content/ContentValues.html) object, which is basically a HashMap you can store values in
```java
ContentValues values = new ContentValues();
values.put(Entry.COLUMN_TITLE, title);
values.put(Entry.COLUMN_YEAR, year);
//...

// Insert the new row, returning the primary key value of the new row
long newRowId = db.insert(Entry.TABLE_NAME, null, values);
Log.i(TAG, "Inserted at row "+newRodId);
```

### Testing the database
Great. So how do we know if it worked (other than the positive row number?)
- We can directly explore the SQLite database that is on your device by using `adb` and the `sqlite3` tool. See [this link](http://developer.android.com/tools/help/sqlite3.html) for instruction. Note that you can run these commands directly from Android Studio by using the "Terminal" tab found at the very bottom (just below the Logcat window).
    ```
    $ adb -s emulator-5554 shell
    # sqlite3 /data/data/com.example.google.rss.rssexample/databases/rssitems.db
    SQLite version 3.3.12
    Enter ".help" for instructions
    .... enter commands, then quit...
    # sqlite> .exit
    ```

(If we want to delete and recreate your database, you can run the drop statement we wrote, either from within the sqlite3 tool or as an `execSQL()` call in your Activity).

Finally, let's hook up the `DetailsFragment` so that when I click that button, I connect to and insert the movie data into my database! [[do so]]

### Break! (~5min)

### Reading from the Database
Terminal is lovely, but we'd like to be able to see that list in our UI, right? We'd like to fill in that `FavoritesFragment` ListView.

<!-- Adapted from homework write-up -->
We can read from the database using the <a href="http://developer.android.com/reference/android/database/sqlite/SQLiteDatabase.html#query(boolean, java.lang.String, java.lang.String[], java.lang.String, java.lang.String[], java.lang.String, java.lang.String, java.lang.String, java.lang.String)">`.query()`</a> method. This method takes _a lot_ of parameters, which you basically use to "fill in the blanks" in an SQL query. See the method description for details. Alternatively, you can just write a raw SQL statement (e.g., `SELECT * FROM tablename` to get all of the data from your table), and execute that with the <a href="http://developer.android.com/reference/android/database/sqlite/SQLiteDatabase.html#rawQuery(java.lang.String, java.lang.String[])">`.rawQuery()`</a> method. Note that you should use `?` arguments rather than directly passing in user-entered Strings to avoid [SQL Injection attacks](https://en.wikipedia.org/wiki/SQL_injection).

Either method you use will return a [`Cursor`](http://developer.android.com/reference/android/database/Cursor.html). This is a lot like an Iterator in Java; it keeps track of where you are in a list (e.g., what `i` we'd be on in a loop), and then provides methods that let us fetch values from the object at that spot in the list. You can then call methods to move around the list (e.g., to move to the next item). For example:
```java
cursor.moveToFront(); //move to the first item
String field0 = cursor.getString(0); //get the first field (column you specified) as a String
String name = cursor.getString(cursor.getColumnIndexOrThrow("title")); //get the "title" field as a String
cursor.moveToNext(); //go to the next item
```

The nice thing about `Cursors` though is that they can easily be fed into `AdapterViews` by using a [`CursorAdapter`](http://developer.android.com/reference/android/widget/CursorAdapter.html) (as opposed to the `ArrayAdapter` we've used previously). We'll use a [**`SimpleCursorAdapter`**](http://developer.android.com/reference/android/widget/SimpleCursorAdapter.html), as this abstracts out a lot of the detail and makes it almost as easy to use an an `ArrayAdapter`.

- We instantiate a new `SimpleCursorAdapter`, passing it a `Context`, a layout resource to inflate, a `Cursor`, an array of column names to fetch from each entry in the Cursor, and a matching list of view resource ids (which should all be `TextViews`) to assign each column to. Then you can set this adapter just like you did with `ArrayAdapter` in the last assignment.

- **Important Note** One of the constructors for `SimpleCursorAdapter` has been _deprecated_ because it runs on the **UI Thread**. So you'll need to also pass in `CursorAdapter.FLAG_REGISTER_CONTENT_OBSERVER` as the final parameter. Our database work will still all run on the UI Thread, but since we're not doing anything complex or updating the database frequently, things _should_ be okay. The preferred solution is to use a [ContentProvider](http://developer.android.com/guide/topics/providers/content-providers.html) and a [Loader](http://developer.android.com/guide/components/loaders.html), which act as wrappers around `ASyncTask`; however, these are larger, more complex topics that we'll get to later to "fix" our application.
  - It it becomes an issues, we can simply add an `ASyncTask` to our `Database` object, and have the method `execute()` that!

When you add a new item to your database, you'll want to tell the `SimpleCursorAdapter` to update by calling `adapter.notifyDataSetChanged()`. This is where that UI Thread work occurs!
    - Can do this in `onResume()`!

And that gives us a database to work with! Not too bad?


## Other Storage Options [if time]
There are a few other options as well for persisting data.

### SharedPreferrences
If you just have a little bit of simple (primitive) data to save, you can use the [SharedPreferences](http://developer.android.com/guide/topics/data/data-storage.html#pref). This is a way to save key-value pairs of primitives (int, float, String, etc).
- It's called "SharedPreferences", but you don't need to store preferences in it, but you can. We'll talk more about "settings" and preferences next week I think.
- This is essential an XML file... so you save anything you can put in XML easily.
- Can fetch the file from the Context (`.getSharedPreferences()`), and then can put and get values into it the same way we did with a Bundle.
- This can be made `MODE_PRIVATE` (only available to this app), or `MODE_WORLD_READABLE/WRITEABLE` (available to other apps)

### Direct File Access
Android lets us write to the file system in much the same way we can in Java (how many people have done this?)
- You actually use the same set of classes that we used for handling the Network connection: `InputStream`, and its _decorators_ like `InputStreamReader`, `BufferedReader`, `FileReader`, etc.
  - We get access to a `File` object which is an object that represents a `File` on the computer. An abstraction around the filesystem.
    - Then we write data to that file!
    - Note that Files can be **directories**, test that with the `isDirectory()` method! Can then get a list of `Files` inside the directory with `listFiles`.

Android actually has a number of different locations we can store files, and accessing them are all slightly different (see [Saving Files](http://developer.android.com/training/basics/data-storage/files.html). In the end they all will give us a `File` or a `FileInputStream`, we just have to use different methods to figure out _which_ file to write to!

Android devices file locations into two groups: **Internal storage** and **External stroage**. Names come from when devices had built-in memory, as well as external SD cards.
  - [Internal storage](http://developer.android.com/guide/topics/data/data-storage.html#filesInternal)) is always accessible, and by default files saved internally are _only_ accessible to your app. Similarly, when the user uninstalls your app, the internal files are deleted. This is usually the best place for "private" file data.
  - [External storage](http://developer.android.com/guide/topics/data/data-storage.html#filesExternal) is not always accessible, is world-readable, and only gets removed on uninstall if you do it in a specific way

Both of these storage systems also have a "cache" location. A cache is "secret storage", but in computing tends to refer to "temporary storage"
- The Cache is different, in that Android will automatically delete cached files if storage space is getting low... but you really shouldn't rely on it to do that (delete your own cache files when you're done with them!) Basically use this for temporary files, and try to keep them _small_ (less than 1MB)

#### Internal Storage
We can get access to internal storage with:
- `openFileOutput(filename, mode)` which gives us a `FileInputStream` for that file in internal storage
  - modes: `MODE_PRIVATE` will create the file (or replace a file of the same name) and make it private to your application. Other modes available are: `MODE_APPEND` (which adds to the end of the file if it exists instead of erasing), `MODE_WORLD_READABLE`, and `MODE_WORLD_WRITEABLE`.
- `createTempFile()` will create a file in the _cache_ storage, and return that file

- Alternatively:
  - `getFilesDir()` gives us a directory `File` for the internal storage, into which we can create a new `File`
  - `getCacheDir()` gives us a directory `File` for the _cache_ internal storage, into which we can create a new `File`

See [Using the Internal Storage](http://developer.android.com/guide/topics/data/data-storage.html#filesInternal) for more details.

((We can demo this, and check it with `adb shell` and `cat`))

#### External Storage
To save in external storage, we need to ask for permission!
```xml
<manifest ...>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    ...
</manifest>
```
- This is a __dangerous__ permission; in Android 6.0 (API 23) we'd need to ask for permissions at run time!

We can then use similar methods as the internal storage to get access to the correct folder, and then use the same File IO methods to read/write. See [Using the External Storage](http://developer.android.com/guide/topics/data/data-storage.html#filesInternal) for details.


... I apologize for the scattered and incomplete nature of today's notes!
