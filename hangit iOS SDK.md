
<h1>Hangit iOS SDK</h1>

<h2>Overview</h2>

You can use the Hangit platform to leverage location awareness for your iOS application.  The Hangit SDK framework performs location monitoring and records user location positioning information for delivery to the Hangit service.  The SDK also delivers location-based alert notifications to your app in a timely fashion, and enables your app to consume Hangit campaign events and offers: JSON formatted rich messages that provide the user with timely engagement and redeemable coupons.  Users click through to deep links back to your app.

![enter image description here](https://lh3.googleusercontent.com/-3v0YFIrrfVY/VUEys0RemPI/AAAAAAAAA3g/TSzazAEXpT8/s0/hangit.png "hangit.png")

A typical Hangit SDK implementation includes the following:

1. Establishes a session between your app and the SDK, using custom sub-class `ViewController.h*`
2. Enables the Hangit location monitoring service for your app and SDK
3. Enables the app to consume raw JSON campaigns and offers.
*required.


<h2> Getting Started</h2>

>**Latest Version:** The latest version of the Hangit iOS SDK Framework is 1.1.3 and was released April, 22nd 2015.

<h3>Installing the iOS SDK Framework</h3>

In this section we’ll run through the steps to add our Framework to your Xcode project.  If this is your first time out with Xcode and iOS App development, then we recommend you read Apple’s [introductory guide](https://developer.apple.com/library/ios/referencelibrary/GettingStarted/RoadMapiOS/index.html).

>**Warning:** You will not be able to monitor changes for CoreMotion using the Simulator, you will need to test and run your project on a real device.

There are two possible paths for downloading and installing the Hangit SDK framework for iOS.

 - Clone the Hangit SDK Framework, or
 - Use CocoaPods

<h4>Clone the Hangit SDK Framework</h4>

 - Clone the git repo - git clone https://github.com/hangit/iOS_SDK.git
 - Change to the project folder - cd iOS_SDK/
 - Run the Test project

To add the framework and bundle to your own project goto the section ["Add the SDK Framework files to your project"](https://stackedit.io/editor#add-the-sdk-framework-files-to-your-project)

<h4>CocoaPods</h4>

 - Create or edit your Podfile
 - Add this to your Podfile: `pod 'Hangit', '~> 1.1.3'`
 - Run pod install
 - Open the new CocoaPods workspace for your project and jump to [Session Establishment](https://stackedit.io/editor#session-establishment)


<h4>Add the SDK Framework files to your project</h4>

 - Create a new project in Xcode and give it a name.
 - Create an empty folder in your project directory.
 - Copy the resource bundle **HangitResources.bundle** to your new folder.
 - Choose **File > Add Files** to Your ProjectName.
 - Select your new folder containing the SDK files
 - Drag the Hangit.framework file to your Embedded Binaries section under **General** for your projects target.


<h4>Link the SDK files to your project</h4>

In XCode 6, the SDK will be linked automatically to your project.

If you need to manually link the files, or just check that they are there, you can use the following procedure.

>**Note:** Optional Step

 - In the project navigator, select your project.
 - Click **Build Phases -> Link Binary With Libraries.**
 - Check that Hangit.framework is already added, 

if it isn’t:  

 - Click the **Add button (+).**
 - Select Hangit.framework 
 - Click **Add.**

Your project now references the Hangit SDK Framework and your installation is complete.


----------

<h3>iOS 8 Compatibility</h3>

For your app to work correctly with iOS 8 you need to add a new key to your project's plist file, and you need to turn on **Background Modes** in Project **Capabilities**

<h4>plist</h4>

To add a new key to your project’s plist file.

 - In the project navigator, select your project.
 - Select your projects Info.plist file

```
<key>NSLocationAlwaysUsageDescription</key>
<string>Uses background location</string>
```

>The string can be empty, or defined by you the developer, the content is not important.

<h4>Project Capabilities</h4>

To properly setup Project capabilities

 - In the project navigator, select your project.
 - Select your projects **Target** and choose **Capabilities**
 - Turn on Background Modes.
 - Select **"Location updates"** and **"Background fetch"**


----------


<h2>Session Establishment</h2>

As part of establishing a session between your app and the SDK, you will enable your app to receive push notifications and Hangit offers.  You will also specify the hangit API key to enable communication between the app and the Hangit service.

To setup this behaviour you will create a custom sub-class of `UIViewController` called `SessionManagerDelegate`  In this tutorial we will simply re-use the sub-class `ViewController.h` for this purpose.

You will need to implement the `sessionManager` method of the custom sub-class as listed below.  You will also perform additional initialization of this method using the `viewDidLoad()` function.

Object methods:

```objective-c
#import <HangitSDK/HangitSDK.h>

@interface ViewController : UIViewController <SessionManagerDelegate>

/* Hangit SessionManager */
@property (nonatomic, strong) SessionManager *sessionManager;

/* Hangit MapManager */
@property (nonatomic, strong) MapManager *mapManager;

/* Hangit sessionKey Property */
@property (nonatomic, retain) NSString * sessionKey;
```

Declare `SessionManager` inside the [viewDidLoad](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIViewController_Class/) function.  This enables the app to receive push notifications, and present offers to users, as indicated in the code below.  Note that `startSessionUsingLocation:@"YOURAPIKEY` enables your app to communicate with the hangit service, and requires your API authentication key.

```objective-c
self.sessionManager = [SessionManager sharedInstance];

self.sessionManager.delegate = self;

//Push Notification To Users
self.sessionManager.presentNotifications = YES;

//Present Offers To Users
self.sessionManager.presentOfferView = YES;

self.sessionKey = [_sessionManager startSessionUsingLocation:@"YOURAPIKEY"];
```

<h2>Receiving Location Updates</h2>

You can setup your app to consume Location updates from the Hangit SDK.  The Hangit SDK monitors for changes in device location and sends notification to your app.  You can then consume these notifications for your app's location-based needs.

To receive location updates from the Hangit SDK the following is required:

 - include the SDK framework
 - implement the notification method and callback method

<h3>SDK Framework</h3>

Import the framework to your `ViewController.m` for simplicity, or to any custom sub-class that you define for [UIViewController](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIViewController_Class/#//apple_ref/occ/instm/UIViewController/viewDidLoad).

Import the framework using the following code:

```
#import <HangitSDK/HangitSDK.h>
```

<h3>Implementing the Methods</h3>

For the purpose of example, we will implement methods for the `ViewController.m` sub-class.  You can create your own custom sub-class if you like.

You will implement an Observer method `NSNotificationCenter` that will consume location update notification received by the Hangit SDK, as illustrated below:

```objective-c
[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(locationNotification:) name:@"hangitLocationNotification" object:nil];

- (void) locationNotification:(NSNotification *)notification{

    if ([notification object]) {
        CLLocation * location = [[notification object] objectForKey:@"Location"];
        NSLog(@"location callback: %@", location);
    }
}
```

<h3>Callback Method</h3>

You will implement a callback method `didreceiveNotification` in your app's `AppDelegate.m` class, to receive the callback from your app that a Hangit location notification has been consumed.  See the sample below for the implementation:

```objective-c
/* Hangit AppDelegate NotificaitonManager Requirement */

- (void)application:(UIApplication *)application didReceiveLocalNotification:(UILocalNotification *)notification {

    [[NSNotificationCenter defaultCenter] postNotificationName:@"hangitNotificationReceived" object:notification];

}
```

<h2>Consuming Hangit Campaigns and Offers</h2>

You can setup your app to consume Hangit campaign events and offers via the SDK.

Hangit campaigns include events tied to specific location data that can include rich messages to the user and deep links back to your apps.  Hangit campaigns are provisioned in the Hangit portal.

Hangit offers are coupons which can be presented to users when they enter the redemption area.  The redeemed coupons are tracked by the Hangit service.  The Hangit offers are provisioned in the Hangit portal. 

The campaigns and offers are delivered to the app as raw JSON formatted as NSMutable Arrays.

To consume Hangit campaigns and offers, the following is required:

 - Add a `dataService` sub-object to your `ViewController.m` or other custom sub-class you define
 - Implement the callback method
 
 <h3>Hangit Campaigns</h3>

To setup your app to consume offers, add the `dataservice` sub-object to `ViewController.m` and implement the callback method as below:

```objective-c
CLLocation * myLocation = [[CLLocation alloc] initWithLatitude:28.550 longitude:-81.400];

[[DataService instance] getLocationsWithLocation:myLocation sessionKey:@"1234567890" completion:^(NSMutableArray *targetsArray, NSError *error) {

    NSLog(@"%@", targetsArray);

}];
```

<h3>Hangit Offers</h3>

To setup your app to consume offers, add the `dataservice` sub-object to `ViewController.m` and implement the callback method as below:

```objective-c
CLLocation * myLocation = [[CLLocation alloc] initWithLatitude:28.550 longitude:-81.400];

[[DataService instance] getOffersWithLocation:myLocation sessionKey:@"1234567890" completion:^(NSMutableArray *offersArray, NSError *error) {

    DLog(@"%@", offersArray);

}];
```

<h2>Hangit Offers Map</h2>

You can implement the Hangit offers map and display it on yoru View Controller, to enable users to see a map with all the hangit offers nearby.

To place the Hangit Offers map on your View Controller, implement the `mpModule` mthod as part of the `ViewController.m` sub class, or other custom sub-class.   The implementation is described below:

```objective-c
self.mapManager = [[MapManager alloc] initWithNibName:@"MapManager"
        bundle:[NSBundle bundleWithPath:[[NSBundle mainBundle]
        pathForResource:@"HangitResources" ofType:@"bundle"]]];

/* 1. Add the map to the current view and define the frame parameters */

self.mapManager.view.frame = CGRectMake(0,0,320,200);

self.mapManager.view.autoresizingMask = UIViewAutoresizingFlexibleWidth | UIViewAutoresizingFlexibleHeight;

[self.view addSubview:map.view];
```

<h2>Hangit  SDK Settings View</h2>

You can display the Hangit SDK Settings on your View Controller.  You can decide what size you would like the table view frame rect to be, in order to fit with the design of your app.

You will need to implement the `SettingsController` method in your custom sub-class of [UIViewController](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIViewController_Class/#//apple_ref/occ/instm/UIViewController/viewDidLoad)

The implementation is described below:

```objective-c
@property (nonatomic, strong) SettingsController * settingsController;
```

```objective-c
self.settingsController = [[SettingsController alloc] initWithNibName:@"SettingsController"
    bundle:[NSBundle bundleWithPath:[[NSBundle mainBundle]
    pathForResource:@"HangitResources" ofType:@"bundle"]]];

/* 1. Add the UITableView to the current view and define the frame parameters */

self.settingsController.view.frame = CGRectMake(0,0,320,200);

self.settingsController.view.autoresizingMask = UIViewAutoresizingFlexibleWidth | UIViewAutoresizingFlexibleHeight;

[self.view addSubview:self.settingsController.view];
```

