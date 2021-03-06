# Intended Audience

This document, source code, and resulting compiled library (SDK) is intended to be used in conjunction with Marketing Suite.  Use of the SDK is only supported after approval from Marketing Suite Client Developer Relations, and your account manager.

# Integrating CCMP with iOS Mobile SDK

CCMP enables marketers to target mobile devices running native applications for iOS and Android. The iOS platform uses APNS and is written in Swift 3. With push notifications, there are three parties in play: CCMP, APNS, and the user's device with an app installed. The basic flow is as follows for Push Notifications.1. After the app starts, the app contacts APNS and requests a **device token**.1. The **device token** is sent back to the device.1. The **device token** is sent to CCMP along with an **App ID** and **Customer ID**.

1. CCMP registers the device token with the **App ID** and **Customer ID**, and sends back a **Push Registration ID** (PRID).

2. CCMP will then launch campaigns intending to target devices that have been registered with Push Notifications through APNS.

3. APNS pushes out the notifications to the devices.

4. After the user taps on the notification on the device, the app will notify CCMP that the app was opened from a Notification.  

   ​

   To make all this work, associations need to be set up between all three parties. This guide will walk you through these steps.

   ​

## Installation

### CocoaPods

[CocoaPods](http://cocoapods.org) is a dependency manager for Cocoa projects. You can install it with the following command:

```bash
$ gem install cocoapods
```

To use the private podspec repo you must add it to your environment

```bash
pod repo add PrivateRepo https://github.com/Marketing-Suite/podSpec.git
```

To integrate EMSMobileSDK into your Xcode project using CocoaPods, specify it in your `Podfile`:

```ruby
platform :ios, '9.0'

source 'https://github.com/Marketing-Suite/podSpec.git'
source 'https://github.com/CocoaPods/Specs.git'

target '<YourApp>' do
  # Comment the next line if you're not using Swift and don't want to use dynamic frameworks
  use_frameworks!

  pod 'EMSMobileSDK'
end
```

Then, run the following command:

```bash
$ pod install
```

> ####  Remember to build the workspace so that EMSMobileSDK is visible to your code



### Carthage

[Carthage](https://github.com/Carthage/Carthage) is a decentralized dependency manager that builds your dependencies and provides you with binary frameworks.

You can install Carthage with [Homebrew](http://brew.sh/) using the following command:

```bash
$ brew update
$ brew install carthage
```

To integrate EMSMobileSDK into your Xcode project using Carthage, specify it in your `Cartfile`:

```ogdl
git "https://github.com/Marketing-Suite/ios-sdk.git" "master"
```

Run `carthage update` to build the framework and drag the built `EMSMobileSDK.framework and Alamofire.framework` into your Xcode project's "Embedded Binaries".



# Integrate the SDK with an App In XCode

With the project open, you can add the SDK to the App by dragging the EMSMobileSDK.framework package into your application project.  This will add the framework to the Linked Framework and Libraries Build Settings and will add the library as an embedded binary.  For an Objective-C based application, this will also create a bridging header file (EMSMobileSDK-swift.h) that will expose the Swift based libraries to your code.

Now that the framework is available to your app, you need to initialize the SDK by adding the following line to the AppDelegate file.  It is also necessary to add the Notifications settings line.

Swift

```swift
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
	// Override point for customization after application launch.
	EMSMobileSDK.default.Initialize(customerID: 100, appID: "", region:EMSRegions.NorthAmerica, options: launchOptions)        
	return true    
}

func applicationWillEnterForeground(_ application: UIApplication) {
	// Called as part of the transition from the background to the active state; here you can undo many of the changes made on entering the background.
	EMSMobileSDK.default.checkOSNotificationSettings()
}
```

Objective-C

```objective-c
-(BOOL)application:(UIApplication )application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
	// Override point for customization after application launch.    
	[[EMSMobileSDK default] InitializeWithCustomerID:100 appID:@"" region:EMSRegionsNorthAmerica options:launchOptions];    
	return YES;
}

-(void)applicationWillEnterForeground:(UIApplication *)application {
  // Called as part of the transition from the background to the inactive state; here you can undo many of the changes made on entering the background.
  
  [[EMSMobileSDK default] checkOSNotificationSettings];
  }
```

At this point the SDK is ready.  You need to request permissions for user notifications and register for a DeviceToken via the following code.

> Note:  You must enable  your application for Push Notifications in the capabilities section of your project settings.

Swift        

```swift
//In DidFinishLaunching or child ViewController
UNUserNotificationCenter.current().requestAuthorization(options: [.alert, .sound, .badge]) {
(granted, error) in
//Parse errors and track state
}
UIApplication.shared.registerForRemoteNotifications()    

func application(_ application: UIApplication, didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
        // Application Code
        try? EMSMobileSDK.default.Subscribe(deviceToken: deviceToken)
    }
```

Objective-C    

```objective-c
//In DidFinishLaunching or child ViewController
UIUserNotificationSettings *settings = [UIUserNotificationSettings settingsForTypes:(UIUserNotificationTypeAlert 
      | UIUserNotificationTypeBadge 
      | UIUserNotificationTypeSound) categories:nil];
	[application registerUserNotificationSettings:settings];
	[application registerForRemoteNotifications];

- (void)application:(UIApplication )application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken {    
  NSError *error;    
  [[EMSMobileSDK default] SubscribeWithDeviceToken:deviceToken error:&error];
}
```

The call to EMSMobileSDK.default.Subscribe will send the device token to CCMP, subscribing the device to CCMP push campaigns.  CCMP will return a unique identifier called a Push Registration ID (PRID).  This will be saved in NSUserDefaults.  

When a push notification is received, whether from CCMP or your own services, call the following function passing in the userInfo object to allow the notification to be registered with CCMP.

Swift    

```swift
func application(_ application: UIApplication, didReceiveRemoteNotification userInfo: [AnyHashable : Any], fetchCompletionHandler completionHandler: @escaping (UIBackgroundFetchResult) -> Void) {
	try? EMSMobileSDK.default.RemoteNotificationReceived(userInfo: userInfo)    
	}
```

Objective-C

```objective-c
-(void)application:(UIApplication )application didReceiveRemoteNotification:(NSDictionary *)userInfo fetchCompletionHandler:(void (^)(UIBackgroundFetchResult))completionHandler{    
  NSError *error;    
  [[EMSMobileSDK default] RemoteNotificationReceivedWithUserInfo:userInfo error:&error];
}
```



> Note:  If you are using the CCMP Sandbox, you must add App Tranport Security settings for the eccmp.com domain to allow insecure (HTTP) traffic to that domain.  All of the other regions are secured and should not require a setting.  To add ATS to your application, modify your info.plist file and add the following

```xml
	<key>NSAppTransportSecurity</key>
	<dict>
		<key>NSAllowsArbitraryLoads</key>
		<false/>
		<key>NSExceptionDomains</key>
		<dict>
			<key>eccmp.com</key>
			<dict>
				<key>NSIncludesSubdomains</key>
				<true/>
				<key>NSExceptionAllowsInsecureHTTPLoads</key>
				<true/>
			</dict>
		</dict>
	</dict>

```

## Using Deep Link

The SDK offers a method to handle deep links from CCMP, the call to EMSMobileSDK.default.HandleDeepLink will parse the incoming deep link URL and returns the original URL along with the Deep Link Parameter entered on CCMP (if any)

> Note:  You first need to configure the app to handle universal links(Select the project -> Capabilities tab -> Turn on Associated Domains -> Add the domain using "applinks" prefix).

```swift
func application(_ application: UIApplication,
                    continue userActivity: NSUserActivity,
                    restorationHandler: @escaping ([Any]?) -> Void) -> Bool {
       
       let deeplink = EMSMobileSDK.default.HandleDeepLink(continue: userActivity)

       //deeplink.deepLinkParameter - dl parameter from CCMP if any
       //deeplink.deepLinkUrl		- Original Deep link URL
       
       return true
   }
```

Objective-C

```objective-c
- (BOOL)application:(UIApplication *)application continueUserActivity:(nonnull NSUserActivity *)userActivity restorationHandler:(nonnull void (^)(NSArray * _Nullable))restorationHandler {
    if ([[userActivity activityType] isEqualToString:NSUserActivityTypeBrowsingWeb])
    {
        [[EMSMobileSDK default] HandleDeepLinkWithUserActivity:userActivity];
    }
    return YES;
}
```

# EMSMobileSDK Methods and Properties

## Properties

**appID** -- The App ID from CCMP when an application is registered.

**prid** -- The Push Registration ID received from CCMP after the SDK is initialized.

**customerID** -- The Customer ID that gets associated with the PRID. This is supplied by the developer.

**region** -- The Region for CCMP.**deviceToken** -- The Device Token sent to the device from APNS. CCMP uses the device token to uniquely identify the device so that Push Notifications can be sent to the device in a campaign



## Methods

**Initialize**(**customerID**: int, **appID**: String, **region**: EMSRegions, **options**: launchOptions)

The Initialize method initializes the SDK from the application setting up the default values used in calling CCMP.  The method allows for the region to be specified but If no region is specified the SDK defaults to North America.  If the userInfo object contains remote notifications, it will use the PRID stored in NSUserDefaults to register the app open with CCMP.

**checkOSNotificationSettings**

The checkOSNotificationSettings function detects OS push notification settings and reports back to the system in order to opt users in/out.

**appID** -- The Application ID from CCMP.

**customerID** -- The Customer ID that uniquely identifies the user of the application.

**region** -- the CCMP region that the application uses. 

**options** -- The options that were passed into the application on start.  Note, that the options are used to determine if the application was launched as a result of the user clicking on the notification.



**Subscribe**(**deviceToken**: Data)

The Subscribe function registers the deviceToken with CCMP for receiving Push Notifications via CCMP campaigns.

**deviceToken** -- The device token sent to the app by the iOS platform.



**UnSubscribe**()

The UnSubscribe function sends a message to CCMP unsubscribing the device from future push notification campaigns.



**RemoteNotificationReceived**(**userInfo**: [AnyHashable: Any])

This function is called when a remote notification is received.  If the notification is from CCMP the SDK will parse the contents and register the receipt of the message as an open in CCMP.  This method is used when the app is running in the foreground when a push notification is received.  Once the RemoteNotificationReceived function is called, the app developer is free to act upon the notification in any way they see fit.

**userInfo** -- An array of Hashable items sent from APNS which may or may not include data from CCMP.  If there is no CCMP specific data in the array, no processing occurs on the SDK.



**HandleDeepLink**(**userActivity**: NSUserActivity)

The HandleDeepLink function parses the information from the userActivity and returns the original Deep link URL, the Deep link Paramater if any, and finally register the link count on CCMP.

**userActivity** -- Passed-in userActivity.



## Sequence

![EMSMobileSDK](EMSMobileSDK.png)