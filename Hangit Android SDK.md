

<h1>Hangit Android SDK</h1>

<h2>Overview</h2>

You can use the Hangit Android SDK to create an app that will receive Hangit event notifications and display them to the user.  The SDK monitors the user device location and can trigger an event notification when the user enters the pre-defined redemption area (provisioned in the Hangit portal).

A successful implementation of the Hangit Android SDK requires the following:

 - Configure your Android manifest.xml to set permissions and references to Hangit SDK services (identify which activity to launch when Hangit location notification event is received by the app.)
 - Set the Hangit SDK API key
 - Setup the SDK to run in the background of your app.
	 - sets [Activity](http://developer.android.com/reference/android/app/Activity.html) that will start the background service
	 - configure Activity listed in Android manifest that will accept the intent from the event notification and its related set of data points (describes campaign event, or offer).



<h2>Getting Started</h2>

>**Note:** For more information and sample code, please refer to our sample application at https://github.com/hangit/Android_SDK_2.

<h3>Installing</h3>

>**Info:** Sample code can be found in our repository https://github.com/hangit/Android_SDK_2

To install the sdk framework

 - Add a reference to the project
 - Add the hangit_sdk library to your project's **/libs** folder
 - Add a dependancy for [Google Play Services](https://play.google.com/store?hl=en)

**Adding the Reference:**

To add the reference

 - Add a flatDir repository reference to the build.gradle file in the Android application.

```objective-c
Build.Gradle repositories
repositories {
    flatDir {
        dirs 'libs'
    }
}
```

**Adding the hangit_sdk Libray:**

 - Downlad the hangit_sdk.aar file. The hangit_sdk.aar file is located in the HangIt repository at https://github.com/hangit/Android_SDK_2/tree/master/PublisherExample/app/libs .
 - Copy the `hang_sdk.aar` file to your project's **/libs** folder.

**Adding the dependancy:**

Without Google play services the `AndroidManifest.xml` file will throw a compilation error on meta-data com.google.android.gms.version.

Use the code below to add the dependancy

```objective-c
Build.Gradle dependencies
dependencies {
    compile(name:'hangit_sdk', ext:'aar')
    compile 'com.google.android.gms:play-services:6.5.87'
}
```



<h2>Setting Up the Android Manifest</h2>

You will configure the [manifest](http://developer.android.com/guide/topics/manifest/manifest-intro.html) to set appropriate permissions and declare the services required by your app and the SDK.

Set the permissions 

```objective-c
AndroidManifest.xml <manifest>
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.READ_PHONE_STATE" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="com.google.android.providers.gsf.permission.READ_GSERVICES"/>
<uses-permission android:name="com.google.android.gms.permission.ACTIVITY_RECOGNITION" />
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
<meta-data android:name="com.google.android.gms.version" android:value="@integer/google_play_services_version" />
```

Add references to the hangit_sdk services within the node.

```objective-c
AndroidManifest.xml Application
<service android:name="com.hangit.android.hangit_sdk.ServiceHangITLocation">
    <meta-data android:name="com.hangit.android.hangit_sdk.notification_icon" android:value="ic_launcher"/>
    <meta-data android:name="com.hangit.android.hangit_sdk.notification_activity" android:value="com.hangit.android.hangit_sdk.UIWebViewActivity"/>
</service>
<service android:name="com.hangit.android.hangit_sdk.ServiceActivityRecognition" />
<receiver android:name="com.hangit.android.hangit_sdk.ReceiverBootup" >
    <intent-filter>
        <action android:name="android.intent.action.BOOT_COMPLETED" />
        <action android:name="android.intent.action.QUICKBOOT_POWERON" />
    </intent-filter>
</receiver>
```

<h3>Definition of Services</h3>

Service     | Definition
----------- | ---
`ServiceHangITLocation` | Interacts with the Android device to obtain locations updates via GPS, wifi, and network.
`Meta-data android:name=com.hangit.android.hangit_sdk.notification_icon`   | This meta data value identifies which icon to show on location message notification.
`Meta-data android:name=com.hangit.android.hangit_sdk.notification_activity`     | This meta data value identifies which activity to launch when the location message notification is received.
`ServiceActivityRecognition`          |Interacts with the Android device's activity messages for location accuracy.
`ReceiverBootup` | Allows our services to start when the device boots up.


<h3>Add the SDK API Key</h3> 

>**Info:** To obtain a HangIt SDK key, go to http://portal.hangit.com

Add the hangIt SDK key to a reference file. In the provided sample could the SDK key is stored in the [strings.xml](http://developer.android.com/guide/topics/resources/string-resource.html) file.

```objective-c
String.xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="hangit_sdk_key">[YOUR API KEY FROM HANGIT PORTAL]</string>
</resources>
```

<h2>Running hangit_sdk client in the background of your application</h2>

To configure the sdk client to run in the background of your app. the following is required:

 - Configure the method calls on the Activity that will be used to start the SDK background service (`addEventListener()` )
 - set the method that will start the background SDK services (`hangItClient.initialize()` )
 - Create an Activity class that will accept the intent (and associated data points) from the SDK service previously listed in the manifest (`Meta-data android:name=com.hangit.android.hangit_sdk.notification_activity`)
 - 

<h3>Starting the Background Service</h3>

Add the following code snippet to the `onCreate` method of the Activity intended to start the HangIT background services.  This will add an EventListener to listen for Hangit events from the SDK.

```objective-c
MainActivity onCreate
ManagerGeneral.getHangITClient().addEventListener(this);
```
The `hangItClient.initialize()` method is required to start the hangItClient background services. Add the following code snippet after the configuration method calls in the `onCreate` method of the Activity intended to start HangIT background services.

```objective-c
MainActivity onCreate
ManagerGeneral.getHangITClient().initialize(this, getString(R.string.hangit_sdk_key));
```

<h3>Creating the Activity Class</h3>

After an offer is received and the hangItClient sends a notification to the device, the notification will open the activity configured in Android Manifest at `"Meta-data android:name=com.hangit.android.hangit_sdk.notification_activity"` under the `<service android:name="com.hangit.android.hangit_sdk.ServiceHangITLocation">` node. The hangItClient will pass key values on an intent to the `NotificationActivityClass.` The code below outlines a sample Activity class accepting the intent with a list of data points available.

```objective-c
OfferNotificationActivity.java
public class OfferNotificationActivity extends Activity {
        private static final String KEY_PLACE_ID = "com.hangit.android.hangit_sdk.KEY_OFFER_ID";
        private static final String KEY_CENTROID_LAT = "com.hangit.android.hangit_sdk.KEY_CENTROID_LAT";
        private static final String KEY_CENTROID_LONG = "com.hangit.android.hangit_sdk.KEY_CENTROID_LONG";
        private static final String KEY_CAMPAIGN_NAME = "com.hangit.android.hangit_sdk.KEY_CAMPAIGN_NAME";
        private static final String KEY_EVENT_NAME = "com.hangit.android.hangit_sdk.KEY_EVENT_NAME";
        private static final String KEY_EVENT_ID = "com.hangit.android.hangit_sdk.KEY_EVENT_ID";
        private static final String KEY_CENTROID_RADIUS = "com.hangit.android.hangit_sdk.KEY_CENTROID_RADIUS";
        private static final String KEY_ALERT_TEXT = "com.hangit.android.hangit_sdk.KEY_ALERT_TEXT";
        private static final String KEY_URL = "com.hangit.android.hangit_sdk.KEY_URL";
        private static final String TAG = "OfferNotificationActivity";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        setContentView(R.layout.activity_offer_notification);

        Intent intent = getIntent();

        int placeId = intent.getIntExtra(KEY_PLACE_ID, 0);
        double lat = intent.getDoubleExtra(KEY_CENTROID_LAT, 0);
        double lng = intent.getDoubleExtra(KEY_CENTROID_LONG, 0);
        String campaignName = intent.getStringExtra(KEY_CAMPAIGN_NAME);
        String eventName = intent.getStringExtra(KEY_EVENT_NAME);
        int eventId = intent.getIntExtra(KEY_EVENT_ID, 0);
        double radius = intent.getDoubleExtra(KEY_CENTROID_RADIUS, 0);
        String alertText = intent.getStringExtra(KEY_ALERT_TEXT);
        String url = intent.getStringExtra(KEY_URL);
    }
}
```

<


> Written with [StackEdit](https://stackedit.io/).