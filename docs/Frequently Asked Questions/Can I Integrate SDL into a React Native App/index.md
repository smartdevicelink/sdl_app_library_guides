# Can I Integrate SDL into a React Native App?

## Getting Started
SDL does work and can be integrated into a React Native application. 

Please follow [this guide](https://facebook.github.io/react-native/docs/getting-started) for how to create a new React Native application if you need one. Also ensure you have followed the [Getting Started](Getting Started/Installation) and have `SmartDeviceLink` installed on the native side. 

To install SDL into your React Native app, you will need to follow [this guide](https://facebook.github.io/react-native/docs/native-modules-ios) to integrate the SDL library into your application using React Native's Native Modules feature.

This guide is not meant to walk you though how to make a React Native app but help you integrate SDL into an existing application. We will show you a basic example of how to communicate between your app's JavaScript code and SDL's native Obj-C code. For more advanced features, please refer to the React Native documentation linked above.

!!! NOTE
You must make sure you have [Native Modules](https://facebook.github.io/react-native/docs/native-modules-setup) installed as a dependency in order to use 3rd party APIs in a React Native application. If this is not done your app will not work with SmartDeviceLink.
!!!

!!! NOTE
Native API methods are not exposed to Javascript. Follow the guides for more in depth explanations on how to export the methods you wish to expose to your application.
!!!

!!! NOTE
Make sure you have followed the [Getting Started](https://smartdevicelink.com/en/guides/iOS/getting-started/installation/) and have `SmartDeviceLink` installed on the native side. 
!!!

## Integration Basics
Native API methods are not exposed automatically to Javascript. This means you must expose methods you wish to use from SDL to your React Native app. You must implement the `RCTBridgeModule` protocol into a bridge class (see below for an example). Please follow [Integrating Basics](Getting Started/Integration Basics) for the basic setup of a native SDL `ProxyManager` class that your bridge code will call into. This is the necessary starting point in order to continue with this example. Please make sure you also set up a simple UI with buttons and some textfields.

### Creating the RCTBridge
To create a native module you must implement the `RCTBridgeModule` protocol. Update your `ProxyManager` to include `RCTBridgeModule`.

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

###### ProxyManager.m
An `RCT_EXPORT_MODULE()` macro must be added to the implementation file to expose the class to React Native.
```objc
@implementation ProxyManager

RCT_EXPORT_MODULE();
<#...#>

@end
```

##### Swift
First you must add  `#import "React/RCTBridgeModule.h"` to your `Bridging Header` before you move forward. When creating a Swift application and importing Objective-C code, Xcode should ask if it should create this header file for you. You must include this bridging header for your React Native app to work. You can create this file manually as well. 

```swift
@objc(ProxyManager)

class ProxyManager: NSObject {

<#...#>

}
```

Next, to expose the Swift class to React Native you must create an Objective-C file in order to use React Native macros.

##### Objective-C
###### ProxyManager.m 
```objc
#import "React/RCTBridgeModule.h"

@interface RCT_EXTERN_MODULE(ProxyManager, NSObject)

@end
```

### Exposing Methods
Since it is recommended that the `ProxyManager` is used for inital set up only we suggest creating a new class that exposes your methods and post a notification from the `ProxyManager` class.


##### Objective-C
Inside the `ProxyManager` add a SoftButton to the HMI. Inside the handler post the notification and pass along a refrence to the `sdlManager` in order to update the UI. You may choose how to keep a refrence to the `sdlManager` object however you like.

```objc
SDLSoftButtonObject *softButton = [[SDLSoftButtonObject alloc] initWithName:@"Button" state:[[SDLSoftButtonState alloc] initWithStateName:@"State 1" text:@"Weather" artwork:nil] handler:^(SDLOnButtonPress * _Nullable buttonPress, SDLOnButtonEvent * _Nullable buttonEvent) {
    if (buttonPress == nil) { return; }

    NSDictionary *userInfo = @{@"sdlManager" : self.sdlManager};
    [[NSNotificationCenter defaultCenter] postNotificationName:<#Notification Name#> object:nil userInfo:managers];
}];
```

##### Swift
```swift
let softButton = SDLSoftButtonObject(name: "Button", state: SDLSoftButtonState(stateName: "State", text: "Weather", artwork: nil), handler: { (buttonPress, butonEvent) in
    guard buttonPress == nil else { return }
    
    let userInfo = ["sdlManager":self.sdlManager]
    NotificationCenter.default.post(name: NSNotification.Name(rawValue: <#Notification Name#>), object: nil, userInfo: managers)
})

Create the new class and add a listener for the notificaiton and all required methods for sending an event.

##### Objective-C
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
#import "SDLEventEmitter.h"
#import "ProxyManager.h"
#import <React/RCTConvert.h>
#import <SmartDeviceLink/SmartDeviceLink.h>

@implementation SDLEventEmitter

RCT_EXPORT_MODULE()

- (instancetype)init {
    self = [super init];
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(getWeatherNotification:) name:<#Notification Name#> object:nil];

    return self;
}

- (NSArray<NSString *> *)supportedEvents {
    return @[@"DoAction"];
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
    NotificationCenter.default.addObserver(self, selector: #selector(doAction(_:)), name: Notification.Name(rawValue: "<#Notification Name#>", object: nil)
    super.init()
}

@objc func doAction(_ notification: Notification) {
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

!!! NOTE
Make sure you add `#import "React/RCTEventEmitter.h"` to the apps bridging header before moving forward if you're making a Swift application.
!!!

!!! NOTE

If you're using Swift, you will need to create the Objective-C bridging class for `SDLEventEmitter` and add the proper `RCT_EXTERN_METHOD` wrapper.
!!!

```objc
#import "React/RCTBridgeModule.h"
#import "React/RCTEventEmitter.h"
@interface RCT_EXTERN_MODULE(SDLEventEmitter, RCTEventEmitter)

RCT_EXTERN_METHOD(weather:(weather: (id)dict))

@end
```

The above example will then call into JavaScript with an event type `GetWeather`. You will need to create a  `NativeEventEmitter` with your `EventEmitter` module and add a listener.

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
