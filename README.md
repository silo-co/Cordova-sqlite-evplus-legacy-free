# Cordova/PhoneGap sqlite storage - free enterprise version with internal memory improvements
 
Native interface to sqlite in a Cordova/PhoneGap plugin for Android, iOS, and Windows "Universal" (8.1) with API similar to HTML5/[Web SQL API](http://www.w3.org/TR/webdatabase/).

This version is available under GPL v3 (http://www.gnu.org/licenses/gpl.txt) or commercial license. Contact for commercial license: info@litehelpers.net

This version branch contains the source code for the Android, iOS, and Windows "Universal" (8.1) versions. This version branch does not contain any libraries or source code from www.sqlite.org. This version branch uses the built-in sqlite libraries on Android and iOS.

NOTE (TBD): no Circle CI or Travis CI working in this version branch.

## Status

- Free support is available in [litehelpers / Cordova-sqlite-enterprise-free / issues](https://github.com/litehelpers/Cordova-sqlite-enterprise-free/issues); commercial support is available by contacting: info@litehelpers.net
- Patches patches will *NOT* be accepted on this project due to potential licensing issues.
- Features omitted from this version branch: pre-populated database support, REGEXP extension for iOS, WP(7/8) and Amazon Fire-OS support
- Windows "Universal" for Windows 8.0/8.1(+) and Windows Phone 8.1(+) version is in an alpha state:
  - Issue with UNICODE `\u0000` character (same as `\0`)
  - No background processing (for future consideration)
  - You *may* encounter issues with Cordova CLI due to [CB-8866](https://issues.apache.org/jira/browse/CB-8866); as a workaround you can install using [litehelpers / cordova-windows-nufix](https://github.com/litehelpers/cordova-windows-nufix) and `plugman` as described below.
  - In addition, problems with the Windows "Universal" version have been reported in case of a Cordova project using a Visual Studio template/extension instead of Cordova/PhoneGap CLI or `plugman`
  - Not tested with a Windows 10 (or Windows Phone 10) target; Windows 10 build is not expected to work with Windows Phone
- FTS3, FTS4, and R-Tree support is tested working OK in this version (for all target platforms in this version branch Android/iOS/Windows "Universal")
- Status for the other target platforms:
  - Android: now using [Android-sqlite-connector](https://github.com/liteglue/Android-sqlite-connector) (with sqlite `3.7.17`), with support for FTS3/FTS4 and R-Tree
  - iOS: sqlite `3.8.10.2` embedded
- Android is supported back to SDK 10 (a.k.a. Gingerbread, Android 2.3.3); support for older versions is available upon request.
- API to open the database may be changed somewhat to be more streamlined. Transaction and single-statement query API will NOT be changed.

## Announcements

- This version has the following improvement(s):
  - iOS version can now handle all UNICODE characters, using URI encoding as a workaround for [Cordova bug CB-9435](https://issues.apache.org/jira/browse/CB-9435).
  - Multi-part transactions API (described below)
  - Error result with proper Web SQL `code` member and `sqliteCode` as reported by the SQLite C library (Android/iOS)
  - flat JSON interface between Javascript and native parts
  - *optional*: transaction sql chunking, which can be enabled by changing the `MAX_SQL_CHUNK` value in SQLitePlugin.js
- PhoneGap Build is now supported through the npm package: http://phonegap.com/blog/2015/05/26/npm-plugins-available/
- [MetaMemoryT / websql-promise](https://github.com/MetaMemoryT/websql-promise) now provides a Promises-based interface to both Web SQL and this plugin
- iOS version is now fixed to override the correct pluginInitialize method and should work with recent versions of iOS
- Project has been renamed to prevent confusion with [davibe / Phonegap-SQLitePlugin](https://github.com/davibe/Phonegap-SQLitePlugin) (original version for iOS, with a different API)
- New project location (should redirect)
- The test suite is completely ported to Jasmine (2.2.0) and was used to verify the functionality of the new Windows version
- [SQLCipher](https://www.zetetic.net/sqlcipher/) for Windows (8.1) in addition to Android & iOS is now supported by [litehelpers / Cordova-sqlcipher-adapter](https://github.com/litehelpers/Cordova-sqlcipher-adapter)
- New `openDatabase` and `deleteDatabase` `location` option to select database location (iOS *only*) and disable iCloud backup
- Fixes to work with PouchDB by [@nolanlawson](https://github.com/nolanlawson)

## Highlights

- Drop-in replacement for HTML5/[Web SQL API](http://www.w3.org/TR/webdatabase/), the only change should be `window.openDatabase()` --> `sqlitePlugin.openDatabase()`
- Failure-safe nested transactions with batch processing optimizations
- As described in [this posting](http://brodyspark.blogspot.com/2012/12/cordovaphonegap-sqlite-plugins-offer.html):
  - Keeps sqlite database in a user data location that is known; can be reconfigured (iOS version); and synchronized to iCloud by default (iOS version; can be disabled as described below).
  - No 5MB maximum, more information at: http://www.sqlite.org/limits.html

## Some apps using this plugin

- REDCap Mobile App (part of [Project REDCap Software](http://projectredcap.org/software.php) for data collection)

## Known issues

- A stability issue was reported on the iOS version when in use together with [SockJS](http://sockjs.org/) client such as [pusher-js](https://github.com/pusher/pusher-js) at the same time. The workaround is to call sqlite functions and [SockJS](http://sockjs.org/) client functions in separate ticks (using setTimeout with 0 timeout).
- If a sql statement fails for which there is no error handler or the error handler does not return `false` to signal transaction recovery, the plugin fires the remaining sql callbacks before aborting the transaction.
- In case of an error, the error `code` member is bogus on Android and Windows (fixed for Android in [litehelpers / Cordova-sqlite-enterprise-free](https://github.com/litehelpers/Cordova-sqlite-enterprise-free)).
- Possible crash on Android when using Unicode emoji characters due to [Android bug 81341](https://code.google.com/p/android/issues/detail?id=81341), which _should_ be fixed in Android 6.x
- REGEXP is only supported on iOS, known to be broken on Android (default database implementation) and Windows ("Universal").
- Close database bugs described below.
- When a database is opened and deleted without closing, the iOS version is known to leak resources.
- It is NOT possible to open multiple databases with the same name but in different locations (iOS version).
- Problems reported with PhoneGap Build in the past:
  - PhoneGap Build Hydration.
  - For some reason, PhoneGap Build may fail to build the iOS version unless the name of the app starts with an uppercase and contains no spaces (see [#243](https://github.com/litehelpers/Cordova-sqlite-storage/issues/243); [Wizcorp/phonegap-facebook-plugin#830](https://github.com/Wizcorp/phonegap-facebook-plugin/issues/830); [phonegap/build#431](https://github.com/phonegap/build/issues/431)).

## Other limitations

- The db version, display name, and size parameter values are not supported and will be ignored.
- This plugin will not work before the callback for the "deviceready" event has been fired, as described in **Usage**. (This is consistent with the other Cordova plugins.)
- Will not work in a web worker or iframe since these are not supported by the Cordova framework.
- In-memory database `db=window.sqlitePlugin.openDatabase({name: ":memory:"})` is currently not supported.
- The Android version cannot work with more than 100 open db files (due to the threading model used).
- Fixed with a workaround in this version: ~~UNICODE line separator (`\u2028`) and paragraph separator (`\u2029`) are currently not supported and known to be broken in iOS version due to [Cordova bug CB-9435](https://issues.apache.org/jira/browse/CB-9435).~~
- Blob type is currently not supported and known to be broken on multiple platforms.
- UNICODE `\u0000` (same as `\0`) character not working in Android or Windows/Windows Phone (8.1/XX)
- Case-insensitive matching and other string manipulations on Unicode characters, which is provided by optional ICU integration in the sqlite source and working with recent versions of Android, is not supported for any target platforms.
- iOS version uses a thread pool but with only one thread working at a time due to "synchronized" database access
- Large query result can be slow, also due to JSON implementation
- ATTACH another database file is not supported (due to path specifications, which work differently depending on the target platform)
- User-defined savepoints are not supported and not expected to be compatible with the transaction locking mechanism used by this plugin. In addition, the use of BEGIN/COMMIT/ROLLBACK statements is not supported.
- Problems have been reported when using this plugin with Crosswalk (for Android). A couple of things you can try:
  - Install Crosswalk as a plugin instead of using Crosswalk to create the project.
  - Use `androidDatabaseImplementation: 2` in the openDatabase options as described below.
- Does not work with [axemclion / react-native-cordova-plugin](https://github.com/axemclion/react-native-cordova-plugin) since the `window.sqlitePlugin` object exported (ES5 feature). It is recommended to use [andpor / react-native-sqlite-storage](https://github.com/andpor/react-native-sqlite-storage) for SQLite database access with React Native Android/iOS instead.

## Further testing needed

- Multi-page apps
- Use within [InAppBrowser](http://docs.phonegap.com/en/edge/cordova_inappbrowser_inappbrowser.md.html)
- UNICODE characters not fully tested in the Windows "Universal" (8.1) version
- Use with triggers and JOIN
- TODO add some *more* REGEXP tests
- Integration with JXCore for Cordova (must be built without sqlite(3) built-in)

## Some tips and tricks

- If you run into problems and your code follows the asynchronous HTML5/[Web SQL](http://www.w3.org/TR/webdatabase/) transaction API, you can try opening your database using `window.openDatabase` and see if you get the same problems.

## Common pitfalls

- It is NOT allowed to execute sql statements on a transaction following the HTML5/[Web SQL API](http://www.w3.org/TR/webdatabase/), as described below.
- It is possible to make a Windows Phone 8.1 project using either the `windows` platform or the `wp8` platform. The `windows` platform is highly recommended over `wp8` whenever possible. Also, some plugins only support `windows` and some plugins support only `wp8`.

## Alternatives

### Other versions

- [litehelpers / Cordova-sqlite-storage](https://github.com/litehelpers/Cordova-sqlite-storage) - Cordova sqlite storage plugin with permissive licensing terms, supported for more platforms.
- [litehelpers / Cordova-sqlcipher-adapter](https://github.com/litehelpers/Cordova-sqlcipher-adapter) - supports [SQLCipher](https://www.zetetic.net/sqlcipher/) for Android, iOS, and Windows (8.1)
- Adaptation for React Native Android and iOS: [andpor / react-native-sqlite-storage](https://github.com/andpor/react-native-sqlite-storage)
- Original version for iOS (with a slightly different transaction API): [davibe / Phonegap-SQLitePlugin](https://github.com/davibe/Phonegap-SQLitePlugin)

### Other SQLite adapter projects

- [object-layer / AnySQL](https://github.com/object-layer/anysql) - Unified SQL API over multiple database engines
- [an-rahulpandey / cordova-plugin-dbcopy](https://github.com/an-rahulpandey/cordova-plugin-dbcopy) - (Alternative way to) add support for pre-populated database
- Simpler sqlite plugin with a simpler API: [samikrc / CordovaSQLite](https://github.com/samikrc/CordovaSQLite)
- [EionRobb / phonegap-win8-sqlite](https://github.com/EionRobb/phonegap-win8-sqlite) - WebSQL add-on for Win8/Metro apps (perhaps with a different API), using an old version of the C++ library from [SQLite3-WinRT Component](https://github.com/doo/SQLite3-WinRT) (as referenced by [01org / cordova-win8](https://github.com/01org/cordova-win8))
- [SQLite3-WinRT Component](https://github.com/doo/SQLite3-WinRT) - C++ component that provides a nice SQLite API with promises for WinJS
- [01org / cordova-win8](https://github.com/01org/cordova-win8) - old, unofficial version of Cordova API support for Windows 8 Metro that includes an old version of the C++ [SQLite3-WinRT Component](https://github.com/doo/SQLite3-WinRT)
- [MSOpenTech / cordova-plugin-websql](https://github.com/MSOpenTech/cordova-plugin-websql) - Windows 8(+) and Windows Phone 8(+) WebSQL plugin versions in C#
- [MetaMemoryT / websql-client](https://github.com/MetaMemoryT/websql-client) - provides the same API and connects to [websql-server](https://github.com/MetaMemoryT/websql-server) through WebSockets.

### Alternative solutions

- Another sqlite binding for React-Native (iOS version): [almost/react-native-sqlite](https://github.com/almost/react-native-sqlite)
- Use [NativeScript](https://www.nativescript.org) with its web view and [NathanaelA / nativescript-sqlite](https://github.com/Natha
naelA/nativescript-sqlite) (Android and/or iOS)
- Standard HTML5 [local storage](https://en.wikipedia.org/wiki/Web_storage#localStorage)
- [Realm.io](https://realm.io/)

# Usage

The idea is to emulate the HTML5/[Web SQL API](http://www.w3.org/TR/webdatabase/) as closely as possible. The only major change is to use `window.sqlitePlugin.openDatabase()` (or `sqlitePlugin.openDatabase()`) instead of `window.openDatabase()`. If you see any other major change please report it, it is probably a bug.

**NOTE:** If a sqlite statement in a transaction fails with an error, the error handler *must* return `false` in order to recover the transaction. This is correct according to the HTML5/[Web SQL API](http://www.w3.org/TR/webdatabase/) standard. This is different from the WebKit implementation of Web SQL in Android and iOS which recovers the transaction if a sql error hander returns a non-`true` value.

## Opening a database

There are two options to open a database access object:
- **Recommended:** `var db = window.sqlitePlugin.openDatabase({name: "my.db", location: 1}, successcb, errorcb);`
- **Classical:** `var db = window.sqlitePlugin.openDatabase("myDatabase.db", "1.0", "Demo", -1);`

The `location` option is used to select the database subdirectory location (iOS *only*) with the following choices:
- `0` (default): `Documents` - visible to iTunes and backed up by iCloud
- `1`: `Library` - backed up by iCloud, *NOT* visible to iTunes
- `2`: `Library/LocalDatabase` - *NOT* visible to iTunes and *NOT* backed up by iCloud

**IMPORTANT:** Please wait for the "deviceready" event, as in the following example:

```js
// Wait for Cordova to load
document.addEventListener("deviceready", onDeviceReady, false);

// Cordova is ready
function onDeviceReady() {
  var db = window.sqlitePlugin.openDatabase({name: "my.db"});
  // ...
}
```

The successcb and errorcb callback parameters are optional but can be extremely helpful in case anything goes wrong. For example:

```js
window.sqlitePlugin.openDatabase({name: "my.db"}, function(db) {
  db.transaction(function(tx) {
    // ...
  }, function(err) {
    console.log('Open database ERROR: ' + JSON.stringify(err));
  });
});
```

If any sql statements or transactions are attempted on a database object before the openDatabase result is known, they will be queued and will be aborted in case the database cannot be opened.

**OTHER NOTES:**
- The database file name should include the extension, if desired.
- It is possible to open multiple database access objects for the same database.
- The database access object can be closed as described below.

## SQL transactions

The following types of SQL transactions are supported by this version:
- Single-statement transactions
- Standard asynchronous transactions
- Multi-part transactions

### Single-statement transactions

Sample:

```Javascript
db.executeSql("SELECT LENGTH('tenletters') AS stringlength", [], function (res) {
  console.log('got stringlength: ' + res.rows.item(0).stringlength);
}, function(error) {
  console.log('SELECT error: ' + error.message);
});
```

## Standard asynchronous transactions

Standard asynchronous transactions follow the HTML5/[Web SQL API](http://www.w3.org/TR/webdatabase/) which is very well documented and uses BEGIN and COMMIT or ROLLBACK to keep the transactions failure-safe. Here is a very simple example from the test suite:

```Javascript
db.transaction(function(tx) {
  tx.executeSql("SELECT UPPER('Some US-ASCII text') AS uppertext", [], function(tx, res) {
    console.log("res.rows.item(0).uppertext: " + res.rows.item(0).uppertext);
  }, function(error) {
    console.log('SELECT error: ' + error.message);
  });
}, function(error) {
  console.log('transaction error: ' + error.message);
}, function() {
  console.log('transaction ok');
});
```

In case of a read-only transaction, it is possible to use `readTransaction` which will not use BEGIN, COMMIT, or ROLLBACK:

```Javascript
db.readTransaction(function(tx) {
  tx.executeSql("SELECT UPPER('Some US-ASCII text') AS uppertext", [], function(tx, res) {
    console.log("res.rows.item(0).uppertext: " + res.rows.item(0).uppertext);
  }, function(error) {
    console.log('SELECT error: ' + error.message);
  });
}, function(error) {
  console.log('transaction error: ' + error.message);
}, function() {
  console.log('transaction ok');
});
```

**WARNING:** It is NOT allowed to execute sql statements on a transaction after it has finished. Here is an example from my [Populating Cordova SQLite storage with the JQuery API post](http://www.brodybits.com/cordova/sqlite/api/jquery/2015/10/26/populating-cordova-sqlite-storage-with-the-jquery-api.html):
[post about , for example:

```Javascript
  // BROKEN SAMPLE:
  var db = window.sqlitePlugin.openDatabase({name: "test.db"});
  db.executeSql("DROP TABLE IF EXISTS tt");
  db.executeSql("CREATE TABLE tt (data)");

  db.transaction(function(tx) {
    $.ajax({
      url: 'https://api.github.com/users/litehelpers/repos',
      dataType: 'json',
      success: function(res) {
        console.log('Got AJAX response: ' + JSON.stringify(res));
        $.each(res, function(i, item) {
          console.log('REPO NAME: ' + item.name);
          tx.executeSql("INSERT INTO tt values (?)", JSON.stringify(item.name));
        });
      }
    });
  }, function(e) {
    console.log('Transaction error: ' + e.message);
  }, function() {
    // Check results:
    db.executeSql('SELECT COUNT(*) FROM tt', [], function(res) {
      console.log('Check SELECT result: ' + JSON.stringify(res.rows.item(0)));
    });
  });
```

You can find more details and a step-by-step description how to do this right in the [Populating Cordova SQLite storage with the JQuery API post](http://www.brodybits.com/cordova/sqlite/api/jquery/2015/10/26/populating-cordova-sqlite-storage-with-the-jquery-api.html):

### Multi-part transactions

Sample:

```Javascript
var tx = db.beginTransaction();
tx.executeSql("DROP TABLE IF EXISTS mytable");
tx.executeSql("CREATE TABLE mytable (myfield)");

tx.executeSql("INSERT INTO mytable values(?)", ['test value']);
tx.executeSql("SELECT * from mytable", [], function(tx, res) {
  console.log("Got value: " + res.rows.item(0).myfield);
}, function(e) {
  console.log("Ignore unexpected error callback with message: " + e.message);
  return false;
});

tx.end(function() {
  console.log('Optional success callback');
}, function(e) {
  console.log("Optional error callback with message: " + e.message);
});
```

Sample with abort:

```Javascript
var tx = db.beginTransaction();
tx.executeSql("INSERT INTO mytable values(?)", ['wrong data']);
tx.abort(function() {
  console.log('Optional callback');
});
```

IMPORTANT NOTES:
- In case a `tx.executeSql` call results in an error and it does not have an error callback or the error callback does NOT return `false`, the transaction will be aborted with a ROLLBACK upon the `tx.end` call.
- When a multi-part transaction is started by the `db.beginTransaction` call, all other transactions are blocked until the multi-part transaction is either completed successfully or aborted (with a ROLLBACK).

## Background processing

The threading model depends on which version is used:
- For Android, one background thread per db;
- for iOS, background processing using a very limited thread pool (only one thread working at a time);
- for Windows "Universal" (8.1), no background processing (for future consideration).

# Sample with PRAGMA feature

This is a pretty strong test: first we create a table and add a single entry, then query the count to check if the item was inserted as expected. Note that a new transaction is created in the middle of the first callback.

```js
// Wait for Cordova to load
document.addEventListener("deviceready", onDeviceReady, false);

// Cordova is ready
function onDeviceReady() {
  var db = window.sqlitePlugin.openDatabase({name: "my.db"});

  db.transaction(function(tx) {
    tx.executeSql('DROP TABLE IF EXISTS test_table');
    tx.executeSql('CREATE TABLE IF NOT EXISTS test_table (id integer primary key, data text, data_num integer)');

    // demonstrate PRAGMA:
    db.executeSql("pragma table_info (test_table);", [], function(res) {
      console.log("PRAGMA res: " + JSON.stringify(res));
    });

    tx.executeSql("INSERT INTO test_table (data, data_num) VALUES (?,?)", ["test", 100], function(tx, res) {
      console.log("insertId: " + res.insertId + " -- probably 1");
      console.log("rowsAffected: " + res.rowsAffected + " -- should be 1");

      db.transaction(function(tx) {
        tx.executeSql("select count(id) as cnt from test_table;", [], function(tx, res) {
          console.log("res.rows.length: " + res.rows.length + " -- should be 1");
          console.log("res.rows.item(0).cnt: " + res.rows.item(0).cnt + " -- should be 1");
        });
      });

    }, function(e) {
      console.log("ERROR: " + e.message);
    });
  });
}
```

**NOTE:** PRAGMA statements must be executed in `executeSql()` on the database object (i.e. `db.executeSql()`) and NOT within a transaction.

## Sample with transaction-level nesting

In this case, the same transaction in the first executeSql() callback is being reused to run executeSql() again.

```js
// Wait for Cordova to load
document.addEventListener("deviceready", onDeviceReady, false);

// Cordova is ready
function onDeviceReady() {
  var db = window.sqlitePlugin.openDatabase("Database", "1.0", "Demo", -1);

  db.transaction(function(tx) {
    tx.executeSql('DROP TABLE IF EXISTS test_table');
    tx.executeSql('CREATE TABLE IF NOT EXISTS test_table (id integer primary key, data text, data_num integer)');

    tx.executeSql("INSERT INTO test_table (data, data_num) VALUES (?,?)", ["test", 100], function(tx, res) {
      console.log("insertId: " + res.insertId + " -- probably 1");
      console.log("rowsAffected: " + res.rowsAffected + " -- should be 1");

      tx.executeSql("select count(id) as cnt from test_table;", [], function(tx, res) {
        console.log("res.rows.length: " + res.rows.length + " -- should be 1");
        console.log("res.rows.item(0).cnt: " + res.rows.item(0).cnt + " -- should be 1");
      });

    }, function(e) {
      console.log("ERROR: " + e.message);
    });
  });
}
```

This case will also works with Safari (WebKit), assuming you replace `window.sqlitePlugin.openDatabase` with `window.openDatabase`.

## Close a database object

```js
db.close(successcb, errorcb);
```

It is OK to close the database within a transaction callback but *NOT* within a statement callback. The following example is OK:

```Javascript
db.transaction(function(tx) {
  tx.executeSql("SELECT LENGTH('tenletters') AS stringlength", [], function(tx, res) {
    console.log('got stringlength: ' + res.rows.item(0).stringlength);
  });
}, function(error) {
  // OK to close here:
  console.log('transaction error: ' + error.message);
  db.close();
}, function() {
  // OK to close here:
  console.log('transaction ok');
  db.close(function() {
    console.log('database is closed ok');
  });
});
```

The following example is NOT OK:

```Javascript
// BROKEN:
db.transaction(function(tx) {
  tx.executeSql("SELECT LENGTH('tenletters') AS stringlength", [], function(tx, res) {
    console.log('got stringlength: ' + res.rows.item(0).stringlength);
    // BROKEN - this will trigger the error callback:
    db.close(function() {
      console.log('database is closed ok');
    }, function(error) {
      console.log('ERROR closing database');
    });
  });
});
```

**BUG 1:** It is currently NOT possible to close a database in a `db.executeSql` callback. For example:

```Javascript
// BROKEN DUE TO BUG:
db.executeSql("SELECT LENGTH('tenletters') AS stringlength", [], function (res) {
  var stringlength = res.rows.item(0).stringlength;
  console.log('got stringlength: ' + res.rows.item(0).stringlength);

  // BROKEN - this will trigger the error callback DUE TO BUG:
  db.close(function() {
    console.log('database is closed ok');
  }, function(error) {
    console.log('ERROR closing database');
  });
});
```

**BUG 2:** If multiple database access objects are opened for the same database and one database access object is closed, the database is no longer available for the other database access objects. Possible workarounds:
- It is still possible to open one or more new database access objects on a database that has been closed.
- It *should* be OK not to explicitly close a database handle since database transactions are [ACID](https://en.wikipedia.org/wiki/ACID) compliant and the app's memory resources are cleaned up by the system upon termination.

## Delete a database

```js
window.sqlitePlugin.deleteDatabase({name: "my.db", location: 1}, successcb, errorcb);
```

`location` as described above for `openDatabase` (iOS *only*)

**NOTE:** not implemented for Windows "Universal" (8.1) version.

# Installing

## Windows "Universal" target platform

**PREREQUISITE** in this version branch *ONLY*: install recent version of `sqlite3.[hc]` in `src/external` and make sure the following constants are defined: `#define SQLITE_TEMP_STORE 2` `#define SQLITE_THREADSAFE 2` and for Windows Phone 8.1 *only*: `#define SQLITE_WIN32_FILEMAPPING_API 1`

**IMPORTANT:** There are issues supporing certain Windows target platforms due to [CB-8866](https://issues.apache.org/jira/browse/CB-8866):
- When using Visual Studio, the default target ("Mixed Platforms") will not work
- Problems have been with the Windows "Universal" version case of a Cordova project using a Visual Studio template/extension instead of Cordova/PhoneGap CLI or `plugman`
As an alternative, which will support the ("Mixed Platforms") target, you can use `plugman` instead with [litehelpers / cordova-windows-nufix](https://github.com/litehelpers/cordova-windows-nufix), as described here.

### Using plugman to support "Mixed Platforms"

- make sure you have the latest version of `plugman` installed: `npm install -g plugman`
- Download the [cordova-windows-nufix 3.9.0-nufixpre-01 zipball](https://github.com/litehelpers/cordova-windows-nufix/archive/3.9.0-nufixpre-01.zip) (or you can clone [litehelpers / cordova-windows-nufix](https://github.com/litehelpers/cordova-windows-nufix) instead)
- Create your Windows "Universal" (8.1) project using [litehelpers / cordova-windows-nufix](https://github.com/litehelpers/cordova-windows-nufix):
  - `path.to.cordova-windows-nufix/bin/create.bat your_app_path your.app.id YourAppName`
- `cd your_app_path` and install plugin using `plugman`:
  - `plugman install --platform windows --project . --plugin https://github.com/litehelpers/Cordova-sqlite-enterprise-free`
- Put your sql program in your project `www` (don't forget to reference it from `www\index.html` and wait for `deviceready` event)

Then your project in `CordovaApp.sln` should work with "Mixed Platforms" on both Windows 8.1 and Windows Phone 8.1.

## Easy install with Cordova CLI tool

    npm install -g cordova # if you don't have cordova
    cordova create MyProjectFolder com.my.project MyProject && cd MyProjectFolder # if you are just starting
    cordova plugin add https://github.com/litehelpers/Cordova-sqlite-enterprise-free
 
You can find more details at [this writeup](http://iphonedevlog.wordpress.com/2014/04/07/installing-chris-brodys-sqlite-database-with-cordova-cli-android/).

**WARNING:** for Windows target platform please read the section below.

**IMPORTANT:** sometimes you have to update the version for a platform before you can build, like: `cordova prepare ios`

**NOTE:** If you cannot build for a platform after `cordova prepare`, you may have to remove the platform and add it again, such as:

    cordova platform rm ios
    cordova platform add ios

## Easy install with plugman tool

```shell
plugman install --platform MYPLATFORM --project path.to.my.project.folder --plugin https://github.com/litehelpers/Cordova-sqlite-enterprise-free
```

where MYPLATFORM is `android`, `ios`, or `windows`.

A posting how to get started developing on Windows host without the Cordova CLI tool (for Android target only) is available [here](http://brodybits.blogspot.com/2015/03/trying-cordova-for-android-on-windows-without-cordova-cli.html).

## Source tree

- `SQLitePlugin.coffee.md`: platform-independent (Literate coffee-script, can be read by recent coffee-script compiler)
- `www`: `SQLitePlugin.js` platform-independent Javascript as generated from `SQLitePlugin.coffee.md` (and checked in!)
- `src`: platform-specific source code:
   - `external` - placeholder used to import `sqlite-connector.jar`, `sqlite-native-driver.jar` (needed for Android version), and `sqlite3.[hc]` (needed to build Windows "Universal" (8.1) version) in this version branch
   - `android` - Java plugin code for Android
   - `ios` - Objective-C plugin code for iOS;
   - `windows` - Javascript proxy code and SQLite3-WinRT project for Windows "Universal" (8.1);
- `spec`: test suite using Jasmine (2.2.0), ported from QUnit `test-www` test suite, working on all platforms
- `tests`: very simple Jasmine test suite that is run on Circle CI (Android version) and Travis CI (iOS version)
- `Lawnchair-adapter`: Lawnchair adaptor, based on the version from the Lawnchair repository, with the basic Lawnchair test suite in `test-www` subdirectory

## Manual installation - Android version

These installation instructions are based on the Android example project from Cordova/PhoneGap 2.7.0, using the `lib/android/example` subdirectory from the PhoneGap 2.7 zipball.

 - Install `SQLitePlugin.js` from `www` into `assets/www`
 - Install `SQLitePlugin.java` from `src/android/io/liteglue` into `src/io/liteglue` subdirectory
 - Install the `libs` subtree from `common` with `sqlite-connector.jar` and `sqlite-native-driver.jar` into your Android project
 - Add the plugin element `<plugin name="SQLitePlugin" value="io.liteglue.SQLitePlugin"/>` to `res/xml/config.xml`

Sample change to `res/xml/config.xml` for Cordova/PhoneGap 2.x:

```diff
--- config.xml.orig	2015-04-14 14:03:05.000000000 +0200
+++ res/xml/config.xml	2015-04-14 14:08:08.000000000 +0200
@@ -36,6 +36,7 @@
     <preference name="useBrowserHistory" value="true" />
     <preference name="exit-on-suspend" value="false" />
 <plugins>
+    <plugin name="SQLitePlugin" value="io.liteglue.SQLitePlugin"/>
     <plugin name="App" value="org.apache.cordova.App"/>
     <plugin name="Geolocation" value="org.apache.cordova.GeoBroker"/>
     <plugin name="Device" value="org.apache.cordova.Device"/>
```

Before building for the first time, you have to update the project with the desired version of the Android SDK with a command like:

    android update project --path $(pwd) --target android-19

(assuming Android SDK 19, use the correct desired Android SDK number here)

**NOTE:** using this plugin on Cordova pre-3.0 requires the following changes to `SQLitePlugin.java`:

```diff
diff -u Cordova-sqlite-storage/src/android/io/liteglue/SQLitePlugin.java src/io/liteglue/SQLitePlugin.java
--- Cordova-sqlite-storage/src/android/io/liteglue/SQLitePlugin.java	2015-04-14 14:05:01.000000000 +0200
+++ src/io/liteglue/SQLitePlugin.java	2015-04-14 14:10:44.000000000 +0200
@@ -22,8 +22,8 @@
 import java.util.regex.Matcher;
 import java.util.regex.Pattern;
 
-import org.apache.cordova.CallbackContext;
-import org.apache.cordova.CordovaPlugin;
+import org.apache.cordova.api.CallbackContext;
+import org.apache.cordova.api.CordovaPlugin;
 
 import org.json.JSONArray;
 import org.json.JSONException;
```

## Manual installation - iOS version

### SQLite library

In the Project "Build Phases" tab, select the _first_ "Link Binary with Libraries" dropdown menu and add the library `libsqlite3.dylib` or `libsqlite3.0.dylib`.

**NOTE:** In the "Build Phases" there can be multiple "Link Binary with Libraries" dropdown menus. Please select the first one otherwise it will not work.

### SQLite Plugin

- Copy `SQLitePlugin.[hm]` from `src/ios` into your project Plugins folder and add them in XCode (I always just have "Create references" as the option selected).
- Copy `SQLitePlugin.js` from `www` into your project `www` folder
- Enable the SQLitePlugin in `config.xml`

Sample change to `config.xml` for Cordova/PhoneGap 2.x:

```diff
--- config.xml.old	2013-05-17 13:18:39.000000000 +0200
+++ config.xml	2013-05-17 13:18:49.000000000 +0200
@@ -39,6 +39,7 @@
     <content src="index.html" />
 
     <plugins>
+        <plugin name="SQLitePlugin" value="SQLitePlugin" />
         <plugin name="Device" value="CDVDevice" />
         <plugin name="Logger" value="CDVLogger" />
         <plugin name="Compass" value="CDVLocation" />
```

## Manual installation - Windows "Universal" (8.1) version

Described above.

## Quick installation test

Assuming your app has a recent template as used by the Cordova create script, add the following code to the `onDeviceReady` function, after `app.receivedEvent('deviceready');`:

```Javascript
  window.sqlitePlugin.openDatabase({ name: 'hello-world.db' }, function (db) {
    db.executeSql("select length('tenletters') as stringlength", [], function (res) {
      var stringlength = res.rows.item(0).stringlength;
      console.log('got stringlength: ' + stringlength);
      document.getElementById('deviceready').querySelector('.received').innerHTML = 'stringlength: ' + stringlength;
   });
  });
```

### Old installation test

Make a change like this to index.html (or use the sample code) verify proper installation:

```diff
--- index.html.old	2012-08-04 14:40:07.000000000 +0200
+++ assets/www/index.html	2012-08-04 14:36:05.000000000 +0200
@@ -24,7 +24,35 @@
     <title>PhoneGap</title>
       <link rel="stylesheet" href="master.css" type="text/css" media="screen" title="no title">
       <script type="text/javascript" charset="utf-8" src="cordova-2.0.0.js"></script>
-      <script type="text/javascript" charset="utf-8" src="main.js"></script>
+      <script type="text/javascript" charset="utf-8" src="SQLitePlugin.js"></script>
+
+
+      <script type="text/javascript" charset="utf-8">
+      document.addEventListener("deviceready", onDeviceReady, false);
+      function onDeviceReady() {
+        var db = window.sqlitePlugin.openDatabase("Database", "1.0", "Demo", -1);
+
+        db.transaction(function(tx) {
+          tx.executeSql('DROP TABLE IF EXISTS test_table');
+          tx.executeSql('CREATE TABLE IF NOT EXISTS test_table (id integer primary key, data text, data_num integer)');
+
+          tx.executeSql("INSERT INTO test_table (data, data_num) VALUES (?,?)", ["test", 100], function(tx, res) {
+          console.log("insertId: " + res.insertId + " -- probably 1"); // check #18/#38 is fixed
+          alert("insertId: " + res.insertId + " -- should be valid");
+
+            db.transaction(function(tx) {
+              tx.executeSql("SELECT data_num from test_table;", [], function(tx, res) {
+                console.log("res.rows.length: " + res.rows.length + " -- should be 1");
+                alert("res.rows.item(0).data_num: " + res.rows.item(0).data_num + " -- should be 100");
+              });
+            });
+
+          }, function(e) {
+            console.log("ERROR: " + e.message);
+          });
+        });
+      }
+      </script>
 
   </head>
   <body onload="init();" id="stage" class="theme">
```

# Common traps & pitfalls

- The plugin class name starts with "SQL" in capital letters, but in Javascript the `sqlitePlugin` object name starts with "sql" in small letters.
- Attempting to open a database before receiving the "deviceready" event callback.

# Support

## Reporting issues

If you have an issue with the plugin please check the following first:
- You are using the latest version of the Plugin Javascript & platform-specific Java or Objective-C source from this repository.
- You have installed the Javascript & platform-specific Java or Objective-C correctly.
- You have included the correct version of the cordova Javascript and SQLitePlugin.js and got the path right.
- You have registered the plugin properly in `config.xml`.

If you still cannot get something to work, please create a fresh, clean Cordova project, add this plugin according to the instructions above, and try a simple test program.

If you continue to see the issue in a new, clean Cordova project:
- Make the simplest test program you can to demonstrate the issue, including the following characteristics:
  - it completely self-contained, i.e. it is using no extra libraries beyond cordova & SQLitePlugin.js;
  - if the issue is with *adding* data to a table, that the test program includes the statements you used to open the database and create the table;
  - if the issue is with *retrieving* data from a table, that the test program includes the statements you used to open the database, create the table, and enter the data you are trying to retrieve.

Then you can [raise the new issue](https://github.com/litehelpers/Cordova-sqlite-enterprise-free/issues/new).

## Commercial support

You may contact info@litehelpers.net

# Unit tests

Unit testing is done in `spec`.

## running tests from shell

To run the tests from \*nix shell, simply do either:

    ./bin/test.sh ios

or for Android:

    ./bin/test.sh android

To run then from a windows powershell do either

    .\bin\test.ps1 android

or for Windows (8.1):

    .\bin\test.ps1 windows

# Adapters

## Lawnchair Adapter

### Common adapter

Please look at the `Lawnchair-adapter` tree that contains a common adapter, which should also work with the Android version, along with a test-www directory.

### Included files

Include the following Javascript files in your HTML:

- `cordova.js` (don't forget!)
- `lawnchair.js` (you provide)
- `SQLitePlugin.js` (in case of Cordova pre-3.0)
- `Lawnchair-sqlitePlugin.js` (must come after `SQLitePlugin.js` in case of Cordova pre-3.0)

### Sample

The `name` option determines the sqlite database filename, *with no extension automatically added*. Optionally, you can change the db filename using the `db` option.

In this example, you would be using/creating a database with filename `kvstore`:

```Javascript
kvstore = new Lawnchair({name: "kvstore"}, function() {
  // do stuff
);
```

Using the `db` option you can specify the filename with the desired extension and be able to create multiple stores in the same database file. (There will be one table per store.)

```Javascript
recipes = new Lawnchair({db: "cookbook", name: "recipes", ...}, myCallback());
ingredients = new Lawnchair({db: "cookbook", name: "ingredients", ...}, myCallback());
```

**KNOWN ISSUE:** the new db options are *not* supported by the Lawnchair adapter. The workaround is to first open the database file using `sqlitePlugin.openDatabase()`.

## PouchDB

The adapter is now part of [PouchDB](http://pouchdb.com/) thanks to [@nolanlawson](https://github.com/nolanlawson), see [PouchDB FAQ](http://pouchdb.com/faq.html).

**NOTE:** The PouchDB adapter has not been tested with the Android version which is using the new [Android-sqlite-connector](https://github.com/liteglue/Android-sqlite-connector).

# Contributing

## Community

- Testimonials of apps that are using this plugin would be especially helpful.
- Reporting issues at [litehelpers / Cordova-sqlite-enterprise-free / issues](https://github.com/litehelpers/Cordova-sqlite-enterprise-free/issues) can help improve the quality of this plugin.

**NOTE:** As stated above, patches will *NOT* be accepted on this project due to potential licensing issues. Issues with reproduction scenarios will help maintain and improve the quality of this plugin for future users.

# External sources

- https://github.com/liteglue/Android-sqlite-connector
- https://github.com/liteglue/Android-sqlite-native-driver

# Major branches

TBD fix for this version:

- `common-src` - source for Android ~~(*not* using [Android-sqlite-connector](https://github.com/liteglue/Android-sqlite-connector))~~, iOS, Windows (8.1), ~~and Amazon Fire-OS~~ versions (shared with [litehelpers / Cordova-sqlcipher-adapter](https://github.com/litehelpers/Cordova-sqlcipher-adapter))
- `evfree-rc` - pre-release of free enterprise version for all supported platforms, including library dependencies for Android and Windows "Universal" (8.1/XX)
- [FUTURE TBD] ~~`master` - version for release, to be included in PhoneGap build.~~

## Contact

info@litehelpers.net

