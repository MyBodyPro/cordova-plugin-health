# Cordova Health Plugin

![cordova-plugin-health](https://badgers.space/badge/npm/cordova-plugin-health/cyan) 
![MIT license](https://badgers.space/badge/license/mit/cyan)  


A plugin that abstracts fitness and health repositories like Apple HealthKit or Google Health Connect.

This work is based on [cordova healthkit plugin](https://github.com/Telerik-Verified-Plugins/HealthKit). This plugin is kept up to date and requires a recent version of cordova (12 and on) as well as recent iOS and Android SDKs.
For bugs and improvements use the [issues tracker](https://github.com/dariosalvi78/cordova-plugin-health/issues).
For general question or small issue, please use the [gitter channel](https://gitter.im/cordova-plugin-health/Lobby).

## Warning aboutthe new version (3+)

This is a complete rewrite of the Android version of the plugin to support the new [HealthConnect API](https://developer.android.com/health-and-fitness/guides/health-connect). Google Fit APIs are deprecated and [will be made obsolete in 2024](https://developer.android.com/health-and-fitness/guides/health-connect/migrate/comparison-guide#turn-down-fit-android).

Google Fit is no longer supported by this plugin. If, for any masochistic reason, you want to use Google Fit, you need to use an older version of this pluign (2.1.1).

Please be remindful that ***THIS IS STILL A WORK-IN-PROGRESS***. While all functionalities listed here are implemented and working, there are several data types that were supported in older versions and are not supported in this version *YET*.
If you need support for a given data type, please check if it is already implemented in iOS, then either add support in Java and send a pull request, or add an issue and I will prioritize it.

## Installation

In Cordova:

```
cordova plugin add cordova-plugin-health --variable HEALTH_READ_PERMISSION='App needs read access' --variable HEALTH_WRITE_PERMISSION='App needs write access'
```

`HEALTH_READ_PERMISSION` and `HEALTH_WRITE_PERMISSION` are shown when the app tries to grant access to data in HealthKit.

## iOS requirements

* Make sure your app id has the 'HealthKit' entitlement when this plugin is installed (see iOS dev center).
* Also, make sure your app and App Store description comply with the [Apple review guidelines](https://developer.apple.com/app-store/review/guidelines/#healthkit).
* There are [two keys](https://developer.apple.com/library/content/documentation/General/Reference/InfoPlistKeyReference/Articles/CocoaKeys.html#//apple_ref/doc/uid/TP40009251-SW48) to be added to the info.plist file: `NSHealthShareUsageDescription` and `NSHealthUpdateUsageDescription`. These are assigned with a default string by the plugin, but you may want to contextualise them for your app.

## Android requirements

* HealthConnect is made standard on (Google versions of) Android [from version 14 (API level 34)](https://developer.android.com/health-and-fitness/guides/health-connect/develop/get-started#step-1). On older versions of Android, the user has to install the Health Connect app from the Play Store.
* Health Connect SDK supports Android 8 (API level 26) or higher, while the Health Connect app is only compatible with Android 9 (API level 28) or higher see [this](https://developer.android.com/health-and-fitness/guides/health-connect/develop/get-started#step-2).
* Health Connect SDK requires targeting Android API level 34. The current cordova-android package (12.0.1) targets version 33 and uses a version of Gradle that is incompatible with API level 34, so this plugin implements a temporary workaround.
The workaround consists in fixing gradle version (to 8.4), gradle plugin (to 8.1.1), target SDK version (to 34) and minimum SDK version (to 26), see plugin.xml. Additionally, there are issues with some kotlin depenendencies which are fixed in `src/android/build-extras.gradle`. All these hacks will hopefully be removed with future versions of the cordova-android platform.
* Download a recent version of gradle (8.4 or later).
* If you use Android Studio, download at least version Hedgehog.
* Be aware that Health Connect requires the user to have screen lock enabled with a PIN, pattern, or password.
* When publishing the app, you need to comply to [these requests from Google](https://developer.android.com/health-and-fitness/guides/health-connect/publish/request-access).
* This plugin uses AndroidX. You may need to [activate AndroidX](https://cordova.apache.org/announcements/2020/06/29/cordova-android-9.0.0.html) in the Android platform and make sure all other plugins you use are AndroidX compatible.

### Permissions in AndroidManifest.xml

Health Connect requires that each data type accessed is listed as permission in the AndroidManifest.xml file. This plugin will add permissions for *ALL SUPPORTED* data types, both read and write. However, this is a LONG LIST and it may be problematic when submitting to the Play Store. **It is strongly suggested to review the list and remove those that are actually not needed**. See [this](https://developer.android.com/health-and-fitness/guides/health-connect/plan/data-types) to understand which permissions you can remove and which ones you should keep, depending on the data that you actually need.

### Mandatory Privacy Policy on Android

A Privacy Policy must be present on Android in order for the app to be approved for distribution. The plugin includes a simple webview, with no JS activated, to show the Privacy Policy when requested. The Privacy Policy must be formatted as an HTML page (no JS) and placed as a file with name: `privacypolicy.html` under the `www` folder of the project (in other words, the webview loads the following URL: `file:///android_asset/www/privacypolicy.html`). It is possible to change that URL using the function setPrivacyPolicyURL(). To test the Privacy Policy view, you can call launchPrivacyPolicy().

### Manual setup in Capacitor (Ionic)

Capacitor does not automatically include all changes to AndroidManifest.xml or gradle files from plugin.xml. This is a short guide to do this manually. Based on plugin v3.0.0 and @capacitor/android v5.5.1. Future versions may be different.

1. add the Privacy Policy activity to AndroidManifest.xml, inside <application></application>:
```xml
      <!-- For supported versions through Android 13, create an activity to show the rationale
       of Health Connect permissions once users click the privacy policy link. -->
      <activity
        android:name="org.apache.cordova.health.PermissionsRationaleActivity"
        android:exported="true">
        <intent-filter>
          <action android:name="androidx.health.ACTION_SHOW_PERMISSIONS_RATIONALE" />
        </intent-filter>
      </activity>

      <!-- For versions starting Android 14, create an activity alias to show the rationale
       of Health Connect permissions once users click the privacy policy link. -->
      <activity-alias
        android:name="ViewPermissionUsageActivity"
        android:exported="true"
        android:targetActivity="org.apache.cordova.health.PermissionsRationaleActivity"
        android:permission="android.permission.START_VIEW_PERMISSION_USAGE">
        <intent-filter>
          <action android:name="android.intent.action.VIEW_PERMISSION_USAGE" />
          <category android:name="android.intent.category.HEALTH_PERMISSIONS" />
        </intent-filter>
      </activity-alias>
```
2. add the possibility to query for the presence of Health Connect to AndroidManifest.xml, inside the root tag:
```xml
      <!-- Check if Health Connect is installed -->
      <queries>
        <package android:name="com.google.android.apps.healthdata" />
      </queries>
```
3. add permissions to AndroidManifest.xml , inside the root tag. This depends on the actual data types you want to access. See [this](https://developer.android.com/health-and-fitness/guides/health-connect/plan/data-types) for a list.
```xml
      <uses-permission android:name="android.permission.health.READ_STEPS" />
      <uses-permission android:name="android.permission.health.WRITE_STEPS" />
```
4. modify the main build.gradle file and update:
```gradle
classpath 'com.android.tools.build:gradle:8.1.1'
```
5. modify the variables.gradle file, particularly:
```gradle
minSdkVersion = 26
targetSdkVersion = 34
compileSdkVersion = 34
```

## Supported data types

These are currently supported in both Android and iOS. Please notice that older versions of this plugin included more data types, but with Google Fit, not Health Connect. Support for previously supported data types has not been removed on iOS, it's simply not listed here. The plan is to complete the porting of all previously supported data types from Google Fit to Health Connect, just be patient, or give us a hand.

| Data type       | Unit  |    HealthKit equivalent                       |  Health Connect equivalent               |
|-----------------|-------|-----------------------------------------------|------------------------------------------|
| steps           | count | HKQuantityTypeIdentifierStepCount             |   StepsRecord                            |
| weight          | kg    | HKQuantityTypeIdentifierBodyMass              |   Weight                                 |
| fat_percentage  | %     | HKQuantityTypeIdentifierBodyFatPercentage     |   BodyFatRecord                          |
| activity        | activityType | HKWorkoutTypeIdentifier                |   ExerciseSessionRecord                  |
| calories.active | kcal  | HKQuantityTypeIdentifierActiveEnergyBurned    | ActiveCaloriesBurnedRecord               |
| calories.basal  | kcal  | HKQuantityTypeIdentifierBasalEnergyBurned     | BasalMetabolicRateRecord * time window   |


**Note**: units of measurement are fixed!

Returned objects contain a set of fixed fields:

- startDate: a date indicating when the data point starts
- endDate: a date indicating when the data point ends
- unit: the unit of measurement, as a string
- value: the actual value
- sourceBundleId: the identifier of the app that produced the data
- sourceName: (only on iOS) the name of the app that produced the data (as it appears to the user)
- sourceDevice: (only on Android) the device where the data came from manufacturer and model 
- entryMethod: (only on Android) method of insertion, can be "actively_recorded", "automatically_recorded", "manual_entry" or "unknown"
- id: the unique identifier of that measurement


Example values:

| Data type      | Value                             |
|----------------|-----------------------------------|
| steps          | 34                                |
| weight         | 83.3                              |
| fat_percentage | 0.312                             |
| calories.X     | 245.3                             |
| activity       | "walking"<br />**Notes**: recognized activities and their mappings in Health Connect / HealthKit can be found [here](activities_map.md) <br /> additional calories (in kcal) and distance (in m) can be added if the query has the `includeCalories` and/or `includeDistance` flags set. **Warning** If you want to fetch calories and/or distance permission to access those quantities should be requested. |
| gender         | "male" <br/>**Notes**: only available on iOS |
| date_of_birth  | { day: 3, month: 12, year: 1978 } <br/>**Notes**: currently only available on iOS |
| mindfulness     | 1800 <br/>**Notes**: only available on iOS |


## Methods

### isAvailable()

Tells if either HealthKit of Health Connect are available.

```
cordova.plugins.health.isAvailable(successCallback, errorCallback)
```

- successCallback: if available a true is passed as argument, false otherwise
- errorCallback: called if something went wrong, err contains a textual description of the problem

### setPrivacyPolicyURL() Android only

Sets an alternative privacy policy URL to load. By default it loads `file:///android_asset/www/privacypolicy.html`.

```
cordova.plugins.health.setPrivacyPolicyURL(url, successCallback, errorCallback)
```

- url: URL of the privacy policy
- successCallback: called if screen has been launched
- errorCallback: called if something went wrong


### launchPrivacyPolicy() Android only

Launches the Privacy Policy screen needed by Health Connect. Use it for testing how it appears.

```
cordova.plugins.health.launchPrivacyPolicy(successCallback, errorCallback)
```

- successCallback: screen has been launched
- errorCallback: called if something went wrong

### requestAuthorization()

Requests read and/or write access to a set of data types.
It is recommendable to always explain why the app needs access to the data before asking the user to authorize it.


```
cordova.plugins.requestAuthorization(datatypes, successCallback, errorCallback)
```

- authObj: an object containing data types you want to be granted access to. Example:
```javascript
{
  read : ['steps'],            // Read permission
  write : ['steps', 'weight']  // Write permission
}
```
- successCallback: called if permission process completed, called independently of if the user has granted permissions or not, the argument may indicate if the permissions have been granted (but it is not guaranteed that all have been granted)
- errorCallback: called if something went wrong, the argument contains a textual description of the problem

#### Android quirks

- Be aware that if you want to fetch activities you also have to request permission for 'calories' and 'distance'.


#### iOS quirks

- HealthKit does never reveal if the user has actually granted permission.
- Once the user has allowed (or not allowed) the app, this function will not prompt the user again, but will call the callback immediately. See [this](https://developer.apple.com/documentation/healthkit/hkhealthstore/1614152-requestauthorization) for further explanation.


### isAuthorized() 
Check if the app has authorization to read/write a set of datatypes.

```
cordova.plugins.health.isAuthorized(authObj, successCallback, errorCallback)
```

- authObj: an object containing data types you want to be granted access to. Example:
```javascript
{
  read : ['steps'],            // Read permission
  write : ['steps', 'weight']  // Write permission
}
```
- successCallback: if the argument is true, the app is authorized
- errorCallback: called if something went wrong, the argument contains a textual description of the problem

#### iOS quirks

- This method will only check authorization status for writeable data. Read-only data will always be considered as not authorized.
This is [an intended behaviour of HealthKit](https://developer.apple.com/reference/healthkit/hkhealthstore/1614154-authorizationstatus).



### query()

Gets all the data points of a certain data type within a certain time window.

**Warning:** if the time span is big, it can generate long arrays!

```javascript
cordova.plugins.health.query({
  startDate: new Date(new Date().getTime() - 3 * 24 * 60 * 60 * 1000), // three days ago
  endDate: new Date(), // now
  dataType: 'steps',
  limit: 1000,
  ascending: true,
}, successCallback, errorCallback)
```

- startDate: start date from which to get data
- endDate: end data to which to get the data
- dataType: the data type to be queried (see above)
- limit: optional, sets a maximum number of returned values, default is 1000
- ascending: optional, datapoints are ordered in an descending fashion (from newer to older) se this to true to revert this behaviour
- filterOutUserInput: optional, if true, filters out user-entered activities (iOS only)
- includeCalories: optional, used only for dataType "activity". When querying, for each activity, also the active calories (in kcal) will be added. ***Warning*** the app requires access to calories.active to be granted
- includeDistance: optional, used only for dataType "activity". When querying, for each activity, also the distance, run or cycled, (in m) will be added. ***Warning*** the app requires access to distance to be granted
- successCallback: called if all OK, argument contains the result of the query in the form of an array of: { startDate: Date, endDate: Date, value: xxx, unit: 'xxx', sourceName: 'aaaa', sourceBundleId: 'bbbb' }
- errorCallback: called if something went wrong, argument contains a textual description of the problem

#### iOS quirks

- HealthKit does not calculate active and basal calories - these must be input from an app
- HealthKit does not detect activities automatically - these must be input from an app
- When querying for activities, only events whose startDate and endDate are **both** in the query range will be returned.
- When duration (in seconds) is returned, this may be different than the endTime - startTime and actually more accurate.


#### Android quirks

- Health Connect can read data for up to 30 days prior to the time permission was first granted. If the app is reinstalled, the permission history is lost and you can only query from 30 days before installation. See [note here](https://developer.android.com/health-and-fitness/guides/health-connect/develop/read-data).
- Not all datatypes support start and end timestamps, some, such as weight, only have one timestamp. The plugin will just set both start and end to the same value in those cases.
- Active and basal calories can be automatically calculated by Health Connect.
- calories.basal is returned as an average per day (kcal/day), and is usually stored quite sparsely (it rarely change, but chnages in weight and height trigger a ricalculation).
- Calories and distance for activities are actually queried indipendently, using the timestamps for each returned activity. This may considerably slow down the query if the returned activities are many. Use with care.

### queryAggregated()

Gets aggregated data in a certain time window.
Usually the sum is returned for the given quantity.

```javascript
cordova.plugins.health.queryAggregated({
  startDate: new Date(new Date().getTime() - 3 * 24 * 60 * 60 * 1000), // three days ago
  endDate: new Date(), // now
  dataType: 'steps',
  bucket: 'day'
}, successCallback, errorCallback)
```

- startDate: start date from which to get data
- endDate: end data to which to get the data
- dataType: the data type to be queried (see below for supported data types)
- bucket: if specified, aggregation is grouped an array of "buckets" (windows of time), supported values are: 'hour', 'day', 'week', 'month', 'year'
- filterOutUserInput: optional, filters out user-entered activities if set to true (only works on iOS)
- successCallback: called if all OK, argument contains the result of the query, see below for returned data types. If no buckets is specified, the result is an object. If a bucketing strategy is specified, the result is an array.
- errorCallback: called if something went wrong, argument contains a textual description of the problem


Not all data types are supported for aggregated queries.
The following table shows what types are supported and examples of the returned object:

| Data type       | Example of returned object |
|-----------------|----------------------------|
| steps           | { startDate: Date, endDate: Date, value: 5780, unit: 'count' } |
| calories.active | { startDate: Date, endDate: Date, value: 25698.4, unit: 'kcal' } |
| calories.basal  | { startDate: Date, endDate: Date, value: 3547.3, unit: 'kcal' } |
| activity        | Android: { startDate: Date, endDate: Date, value: 567000, unit: 'ms' } <br /> iOS: { startDate: Date, endDate: Date, value: { still: { duration: 520000 }, walking: { duration: 223000 }}, unit: 'activitySummary' }<br />**Note:** durations are expressed in milliseconds |


#### Quirks

- Health Connect does not currently support grouping by activity type, therefore only the total time for all activities can be returned.
- The start and end dates returned are the date of the first and the last available samples. If no samples are found, start and end may not be set.
- When bucketing, buckets will include the whole hour / day / month / week / year where start and end times fall into. For example, if your start time is 2016-10-21 10:53:34, the first daily bucket will start at 2016-10-21 00:00:00.
- Weeks start on Monday.


#### Android quirks

- Currently, it is not possible to group by activity type in aggregated queries, only the total time for all activities can be returned. See discussion [here](https://stackoverflow.com/questions/77512832/how-to-aggregate-by-exercise-type-in-the-android-health-connect-api/77512845#77512845).


### store()

Stores a data point.

```javascript
cordova.plugins.health.store({
	startDate:  new Date(new Date().getTime() - 3 * 60 * 1000), // three minutes ago
	endDate: new Date(),
	dataType: 'steps',
	value: 180,
}, successCallback, errorCallback)
```

- startDate: start date from which the new data starts
- endDate: end date to which he new data ends
- dataType: the data type
- value: the value, depending on the actual data type
- successCallback: called if all OK, in Android, argument returns the ID of the data point that has been inserted
- errorCallback: called if something went wrong, argument contains a textual description of the problem


#### iOS quirks

- In iOS you cannot store the total calories, you need to specify either basal or active. If you use total calories, the active ones will be stored.
- In iOS distance is assumed to be of type WalkingRunning, if you want to explicitly set it to Cycling you need to add the field `cycling: true`.
- When storing an activity, you can also specify calories (active, in kcal) and/or distance (in meters). For example: `dataType: 'activity', value: 'walking', calories: 20, distance: 520`. Distance is set as DistanceWalkingRunning unless an additional `cycling: true` is added to the object. Be aware that you need permission to write calories and distance first, or the call will fail.
- In iOS you cannot store the total calories, you need to specify either basal or active. If you use total calories, the active ones will be stored.

#### Android quirks

- This operation correponds to an insert, not an update. If you want to update the data point you need to delete it first.
- Not all datatypes support start and end timestamps, some, such as weight, only have one timestamp. The plugin will use the start timestamp to set the actual one.
- In Android you can only store basal rate, that is a power. This is estimated from the kcals provided as an argument, divided by the time between the start and end time. When you query the individual sample, you get the kcal/day back, not the kcal, unless you do an aggregated query.


### delete()

Deletes data points. You can either delete a single data point (using its id, Android only), or a set of them within a time range.

```javascript
cordova.plugins.health.delete({
	startDate:  new Date(new Date().getTime() - 3 * 60 * 1000), // three minutes ago
	endDate: new Date(),
	dataType: 'steps'
}, successCallback, errorCallback)
```

or (Android only):

```javascript
cordova.plugins.health.delete({
	id: '812n12123nd23edj3234'
	dataType: 'steps'
}, successCallback, errorCallback)
```

- startDate: start date from which to delete data
- endDate: end date to which to delete the data
- id: id of the point to be deleted (Android only for now)
- dataType: the data type to be deleted
- successCallback: called if all OK
- errorCallback: called if something went wrong, argument contains a textual description of the problem

#### iOS quirks

- You cannot delete the total calories, you need to specify either basal or active. If you use total calories, the active ones will be delete.
- Distance is assumed to be of type WalkingRunning, if you want to explicitly set it to Cycling you need to add the field `cycling: true`.
- Deleting sleep is not supported at the moment.

#### Android quirks

- Health Connect doesn't allow you to delete data points that were generated by other apps





## External resources

* The official Apple documentation for HealthKit [can be found here](https://developer.apple.com/library/ios/documentation/HealthKit/Reference/HealthKit_Framework/index.html#//apple_ref/doc/uid/TP40014707).
* For functions that require the `unit` attribute, you can find the comprehensive list of possible units from the [Apple Developers documentation](https://developer.apple.com/library/ios/documentation/HealthKit/Reference/HKUnit_Class/index.html#//apple_ref/doc/uid/TP40014727-CH1-SW2).
* [HealthKit constants](https://developer.apple.com/library/ios/documentation/HealthKit/Reference/HealthKit_Constants/index.html), used throughout the code.
* Health Connect [supported data types](https://developer.android.com/reference/kotlin/androidx/health/connect/client/records/package-summary).


## Contributions

Any help is more than welcome!

I don't know Objective C and I am not interested in learning it now, so I would particularly appreciate someone who could give me a hand with the iOS part.
Also, I would love to know from you if the plugin is currently used in any app actually available online.
Just send me an email to my_username at gmail.com.

Thanks!
