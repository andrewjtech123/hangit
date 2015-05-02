
<h1>Hangit iOS SDK</h1>

- [Hangit iOS SDK](#)
	- [Overview](#)
	- [ Getting Started](#)
	- [Session Establishment](#)
	- [Receiving Location Updates](#)
		- [SDK Framework](#)
		- [Implementing the Methods](#)
		- [Callback Method](#)
	- [Consuming Hangit Campaigns and Offers](#)
		- [Hangit Offers](#)
	- [Hangit Offers Map](#)
	- [Hangit  SDK Settings View](#)

<h2>Overview</h2>

You can use the Hangit platform to leverage location awareness for your iOS application.  The Hangit SDK framework performs location monitoring and records user location positioning information for delivery to the Hangit service.  The SDK also delivers location-based alert notifications to your app in a timely fashion, and enables your app to consume Hangit campaign events: JSON formatted rich messages that provide the user with timely offers and redeemable coupons.  Users click through to deep links back to your app.

![enter image description here](https://lh3.googleusercontent.com/-3v0YFIrrfVY/VUEys0RemPI/AAAAAAAAA3g/TSzazAEXpT8/s0/hangit.png "hangit.png")

A typical Hangit SDK implementation includes the following:

1. Establishes a session between your app and the SDK, using custom sub-class ViewController.h*
2. Enables the Hangit location monitoring service for you app and SDK
3. Enables the app to consume raw JSON campaign data to display "rich" messages to the user.
4. Enables the app to consume raw JSON hangit campaign offers.
*required.


<h2> Getting Started</h2>


<h2>Session Establishment</h2>

As part of establishing a session between your app and the SDK, you will enable your app to receive push notifications and hangit offers.  You will also specify the hangit API key to enable communication between the app and the hangit service.

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

You can setup your app to consume Location updates from the Hangit SDK.  The Hangit SDK monitors for changes in device location and sends notification to your app.  You can then consume these notifications for your app's location-based needs, or you can use the notifications to call the Hangit service for Hangit campaigns and offers.

-insert diagram

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

```
self.settingsController = [[SettingsController alloc] initWithNibName:@"SettingsController"
    bundle:[NSBundle bundleWithPath:[[NSBundle mainBundle]
    pathForResource:@"HangitResources" ofType:@"bundle"]]];

/* 1. Add the UITableView to the current view and define the frame parameters */

self.settingsController.view.frame = CGRectMake(0,0,320,200);

self.settingsController.view.autoresizingMask = UIViewAutoresizingFlexibleWidth | UIViewAutoresizingFlexibleHeight;

[self.view addSubview:self.settingsController.view];
```

