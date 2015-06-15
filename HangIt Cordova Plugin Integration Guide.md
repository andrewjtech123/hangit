

#HangIt Cordova Plugin Integration Guide

##Overview

Cordova is a cross-platform developer platform, which enables web developers to easily create cross-platform apps using HTML, Javascript and CSS.  

You can integrate the HangIt SDK with a Cordova by implementing the HangIt Cordova Plugin.  Once the plugin is added, you will follow a [platform-centered](http://cordova.apache.org/docs/en/5.0.0/guide_overview_index.md.html#Overview) Cordova workflow, in order to combine native Android components with web-based Cordova components in the app.

-add diagram

##Getting Started

The following steps are required to successfully create a Cordova app that will integrate the HangIt plugin.  the plugin enables HangIt Location event triggers to your app, as well as  optional local push notifications to the app user.

* Cordova CLI and Plugman plugin manager installation
* Android SDK (min. version 12)
* create the app project and directory for your HangIt app
* Add Android platform to the app
* add sdk key to plugin.java
* add the plugin to your project
* configure gradle build dependencies
* configure android manifest.xml with permissions, services, and activities settings

##Cordova CLI and Plugman

-brief description with web links

-mention what plugins enable again

##Adding Android Platform

-add background

##Hangit Plugin

-review location

-adding the plugin

-configuring the sdk key

##App Settings

-`receivedEvent` function

-build.gradle

##Manifest.xml

###example

```xml
<?xml version='1.0' encoding='utf-8'?>
<manifest android:hardwareAccelerated="true" android:versionCode="1" android:versionName="0.0.1" package="io.cordova.hellocordova" xmlns:android="http://schemas.android.com/apk/res/android">
    <supports-screens android:anyDensity="true" android:largeScreens="true" android:normalScreens="true" android:resizeable="true" android:smallScreens="true" android:xlargeScreens="true" />
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.READ_PHONE_STATE" />
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="com.google.android.providers.gsf.permission.READ_GSERVICES" />
    <uses-permission android:name="com.google.android.gms.permission.ACTIVITY_RECOGNITION" />
    <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />
    <meta-data android:name="com.google.android.gms.version" android:value="@integer/google_play_services_version" />
    <application android:hardwareAccelerated="true" android:icon="@drawable/icon" android:label="@string/app_name" android:supportsRtl="true">
        <activity android:configChanges="orientation|keyboardHidden|keyboard|screenSize|locale" android:label="@string/activity_name" android:launchMode="singleTop" android:name="MainActivity" android:theme="@android:style/Theme.Black.NoTitleBar" android:windowSoftInputMode="adjustResize">
            <intent-filter android:label="@string/launcher_name">
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
        <service android:name="com.hangit.android.hangit_sdk.ServiceHangITLocation">
            <meta-data android:name="com.hangit.android.hangit_sdk.notification_icon" android:value="ic_launcher" />
            <meta-data android:name="com.hangit.android.hangit_sdk.notification_back_icon" android:value="ic_launcher" />
            <meta-data android:name="com.hangit.android.hangit_sdk.notification_activity" android:value="com.hangit.android.hangit_sdk.UIWebViewActivity" />
            <meta-data android:name="com.hangit.android.hangit_sdk.notification_back_activity" android:value="com.hangit.android.hangit_sdk.NotificationBackActivity" />
        </service>
        <service android:name="com.hangit.android.hangit_sdk.ServiceActivityRecognition" />
        <receiver android:name="com.hangit.android.hangit_sdk.ReceiverBootup">
            <intent-filter>
                <action android:name="android.intent.action.BOOT_COMPLETED" />
                <action android:name="android.intent.action.QUICKBOOT_POWERON" />
            </intent-filter>
        </receiver>
        <activity android:name="com.hangit.android.hangit_sdk.UIWebViewActivity">
        </activity>
        <activity android:name="com.hangit.android.hangit_sdk.NotificationBackActivity">
        </activity>
    </application>
    <uses-sdk android:minSdkVersion="12" android:targetSdkVersion="22" />
</manifest>
```



> Written with [StackEdit](https://stackedit.io/).