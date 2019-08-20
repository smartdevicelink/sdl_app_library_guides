# Can I Integrate SDL into a React Native App?

## Getting Started
SDL works in a React Native application. Please follow [this guide](https://facebook.github.io/react-native/docs/getting-started) for how to create a new React Native application if you need one.

React Native does support Native Modules, however you will need to follow [this guide](https://facebook.github.io/react-native/docs/native-modules-ios) to integrate the the SDL library into your application.

This guide is not meant to walk you though how to make a React Native app but help you integrate SDL into a React Native application. We will leave the rest up to you to figure out. We will show you a basic example of how to communicate between Javascript code and native code. For more advanced features please refer to the React Native documentation linked above.

!!! NOTE
You must make sure you have [Native Modules](https://facebook.github.io/react-native/docs/native-modules-setup) installed as a dependency in order to use 3rd party APIs in a React Native application. If this is not done your app will not work with SmartDeviceLink.
!!!

!!! NOTE
Native API methods are not exposed to Javascript. Follow the guides for more in depth explanations on how to export the methods you wish to expose to your application.
!!!

!!! NOTE
Make sure you have followed the [Getting Started](https://smartdevicelink.com/en/guides/iOS/getting-started/installation/) and have `SmartDeviceLink` installed on the native side. 
!!!

## Integerating Basics
Native API methods are not exposed automatically to javascript. This means you must expose methods you wish to use to use.  To get started you must implement `RCTBridgeModule` protocol. Please follow [Integrating Basics](https://smartdevicelink.com/en/guides/iOS/getting-started/integration-basics/) for the basic setup. This is the necessary starting point in order to contunie with this example. Please make sure you also set up a simple UI with buttons and some textfields.

## Creating the RCTBridge
To create a native module you must implement the `RCTBridgeModule` protocol. Update ProxyManager to include `RCTBridgeModule`.

!!! NOTE
Swift will have a few more steps in order to achieve this since swift does not support macros.
!!!

##### Objective-C
###### ProxyManager.h
```objc
#import <React/RCTBridgeModule.h>
@interface ProxyManager : NSObject <RCTBridgeModule>
<#...#>
@end
```
###### ProxyManager.h
A  `RCT_EXPORT_MODULE()` macro must be added to the implementation file.
```objc
@implementation ProxyManager
RCT_EXPORT_MODULE();
<#...#>
@end
```
##### Swift
First you must add  `#import "React/RCTBridgeModule.h"` to the `Bridging Header` before you move forward. When creating a swift appication xcode ask if it should create a header file for you. You must include this bridging header for your React Native app to work. You can create this file manually if you choose to do so.

```swift
@objc(ProxyManager)
class ProxyManager: NSObject
<#...#>
```
To expose the Swift class to React Native you must create an Objective-C file in order to use React Native macros.

```objc
// ProxyManager.m
#import "React/RCTBridgeModule.h"

@interface RCT_EXTERN_MODULE(ProxyManager, NSObject)
@end
```
## Exposing Methods
Since it is recommended that the `ProxyManager` is used for inital set up only we suggest creating a new class that exposes your methods and post a notification from the `ProxyManager` class.
Posting a notification to a class that extends `RCTEventEmitter` is the preferred way to send events to JavaScript. When extending `RCTEventEmitter` you must include the `supportedEvents` method.

You may choose to post the notificaiton however you like but in this example we will use a SoftButton to do so. For this example the `appType` used in setup is `SDLAppHMITypeDefault`.

##### Objective-C
Inside the `ProxyManager` add a SoftButton to the HMI. Inside the handler post the notification and pass along a refrence to the `sdlManager` in order to update the UI. You may choose how to keep a refrence to the `sdlManager` object however you like.

```objc
SDLSoftButtonObject *softButton = [[SDLSoftButtonObject alloc] initWithName:@"Button" state:[[SDLSoftButtonState alloc] initWithStateName:@"State 1" text:@"Weather" artwork:nil] handler:^(SDLOnButtonPress * _Nullable buttonPress, SDLOnButtonEvent * _Nullable buttonEvent) {
    if (buttonPress == nil) { return; }

    NSDictionary *managers = @{@"sdlManager" : self.sdlManager};
    [[NSNotificationCenter defaultCenter] postNotificationName:@"ReactNotificationGetWeather" object:nil userInfo:managers];
}];
```
Add a listener for the notificaiton inside the new class and all required methods for sending an event.

###### SDLEventEmitter.h
```objc
#import <React/RCTEventEmitter.h>
#import <React/RCTBridgeModule.h>
#import <Foundation/Foundation.h>

NS_ASSUME_NONNULL_BEGIN

@interface SDLEventEmitter : RCTEventEmitter

@end

NS_ASSUME_NONNULL_END
```
###### SDLEventEmitter.m
```objc
#import "SDLExposeMethods.h"
#import "ProxyManager.h"
#import <React/RCTConvert.h>
#import <SmartDeviceLink/SmartDeviceLink.h>

@implementation SDLEventEmitter

RCT_EXPORT_MODULE()

- (instancetype)init {
    self = [super init];
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(getWeatherNotification:) name:@"ReactNotificationGetWeather" object:nil];

    return self;
}

- (NSArray<NSString *> *)supportedEvents {
    return @[@"GetWeather"];
}

- (void)getWeatherNotification:(NSNotification *)notification {
    if(self.sdlManager == nil) {
        self.sdlManager = notification.userInfo[@"sdlManager"];
    }
    [self sendEventWithName:@"GetWeather" body:@{@"type": @"weatherType"}];
}

@end

```
##### Swift
```swift
@objc(SDLEventEmitter)
class SDLEventEmitter: RCTEventEmitter {

override init() {
    NotificationCenter.default.addObserver(self, selector: #selector(getWeatherNotification(_:)), name: "ReactNotificationGetWeather", object: nil)
    super.init()
}

@objc func getWeatherNotification(notification: Notification) {
    if self.sdlManger == nil {
        self.sdlManager = notification.userInfo["sdlManager"]
    }

    sendEvent(withName: "GetWeather", body: ["type": "weatherType"])
}

override func supportedEvents() -> [String]! {
    return ["GetWeather"]
}

}
```
Make sure you add `#import "React/RCTEventEmitter.h"` to the apps bridging header before moving forward.

Now you need to create the Objective-C bridging class for `SDLEventEmitter` and add the proper `RCT_EXTERN_METHOD` wrapper

```objc
#import "React/RCTBridgeModule.h"
#import "React/RCTEventEmitter.h"
@interface RCT_EXTERN_MODULE(SDLEventEmitter, RCTEventEmitter)

RCT_EXTERN_METHOD(weather:(weather: (id)dict))

@end

```

The above example will then call into JavaScript with an event type `GetWeather`.  You will need to create a  `NativeEventEmitter` with your EventEmitter module and add a listener.

```javascript
import { NativeEventEmitter, NativeModules } from 'react-native';
const  { SDLEventEmitter } = NativeModules;

const testEventEmitter = new NativeEventEmitter(SDLEventEmitter);

// Build a listener to listen for events
const testWeather = testEventEmitter.addListener(
    'GetWeather',
        () => SDLEventEmitter.weather({
            "weather": {
                "low": "77",
                "high": "87",
                "currentTemp": "82",
                "rain": "50%"
            }
        }   
    )
)
```
The last step is to wrap the method you wish to expose inside a `RCT_EXPORT_METHOD` for Objective-C and `RCT_EXTERN_METHOD`  for Swift. 
```objc
RCT_EXPORT_METHOD(weather:(NSDictionary *)dict) {
[self.sdlManager.screenManager beginUpdates];

self.sdlManager.screenManager.textField1 = [NSString stringWithFormat:@"Low: %@ ºF", [RCTConvert NSString:dict[@"weather"][@"low"]]];
self.sdlManager.screenManager.textField2 = [NSString stringWithFormat:@"High: %@ ºF", [RCTConvert NSString:dict[@"weather"][@"high"]]];

[self.sdlManager.screenManager endUpdatesWithCompletionHandler:^(NSError * _Nullable error) {
    if (error != nil) {
        <#Error#>
    } else {
        <#Success#>
    }
}];
@end
}
```
##### Swift
Add the following method to `SDLEventEmitter.swift`
```swift
@objc func weather(_ dict: NSDictionary) {
self.sdlManager.screenManager.beginUpdates()
let weather = dict["weather"]! as! NSDictionary
self.sdlManager.screenManager.textField1 = "Low: \(weather["low"]!) ºF")"
self.sdlManager.screenManager.textField2 = "High: \(weather["high"]!) ºF")"
self.sdlManager.screenManager.endUpdates()
}
```
