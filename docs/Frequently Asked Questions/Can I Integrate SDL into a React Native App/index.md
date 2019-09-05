# Can I Integrate SDL into a React Native App?
SDL does work and can be integrated into a React Native application. 

!!! NOTE
You must make sure you have [Native Modules](https://facebook.github.io/react-native/docs/native-modules-setup) installed as a dependency in order to use 3rd party APIs in a React Native application. If this is not done your app will not work with SmartDeviceLink. Native API methods are not exposed to Javascript automatically, this must be done manually by you.
!!!

Please follow [this guide](https://facebook.github.io/react-native/docs/getting-started) for how to create a new React Native application if you need one. Also ensure you have followed the [Getting Started](Getting Started/Installation) and have `SmartDeviceLink` installed on the native side. 

To install SDL into your React Native app, you will need to follow [this guide](https://facebook.github.io/react-native/docs/native-modules-ios) to integrate the SDL library into your application using React Native's Native Modules feature.

## Getting Started
This guide is not meant to walk you through how to make a React Native app but help you integrate SDL into an existing application. We will show you a basic example of how to communicate between your app's JavaScript code and SDL's native Obj-C code. For more advanced features, please refer to the React Native documentation linked above.

## Integration Basics
Native API methods are not exposed automatically to Javascript. This means you must expose methods you wish to use from SDL to your React Native app. You must implement the `RCTBridgeModule` protocol into a bridge class (see below for an example). Please follow [Integrating Basics](Getting Started/Integration Basics) for the basic setup of a native SDL `ProxyManager` class that your bridge code will call into. This is the necessary starting point in order to continue with this example. Please make sure you also set up a simple UI with buttons and some textfields on the SDL side.

### Creating the RCTBridge
To create a native module you must implement the `RCTBridgeModule` protocol. Update your `ProxyManager` to include `RCTBridgeModule`.

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
First, you must add  `#import "React/RCTBridgeModule.h"` to your `Bridging Header` before you move forward. When creating a Swift application and importing Objective-C code, Xcode should ask if it should create this header file for you. You must include this bridging header for your React Native app to work. You can create this file manually as well. 

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
We suggest creating a new class that exposes your methods and post notification(s) from the `ProxyManager` class.

##### Objective-C
Inside the `ProxyManager` add a SoftButton to your SDL HMI. Inside the handler post the notification and pass along a reference to the `sdlManager` in order to update the UI. You may choose how to keep a reference to the `sdlManager` object however you like.

```objc
SDLSoftButtonObject *softButton = [[SDLSoftButtonObject alloc] initWithName:@"Button" state:[[SDLSoftButtonState alloc] initWithStateName:@"State 1" text:@"Data" artwork:nil] handler:^(SDLOnButtonPress * _Nullable buttonPress, SDLOnButtonEvent * _Nullable buttonEvent) {
    if (buttonPress == nil) { return; }

    NSDictionary *userInfo = @{@"sdlManager" : self.sdlManager};
    [[NSNotificationCenter defaultCenter] postNotificationName:<#Notification Name#> object:nil userInfo:managers];
}];
self.sdlManager.screenManager.softButtonObjects = @[softButton];
```

##### Swift
```swift
let softButton = SDLSoftButtonObject(name: "Button", state: SDLSoftButtonState(stateName: "State", text: "Data", artwork: nil), handler: { (buttonPress, butonEvent) in
    guard buttonPress == nil else { return }
    
    let userInfo = ["sdlManager":self.sdlManager]
    NotificationCenter.default.post(name: NSNotification.Name(rawValue: <#Notification Name#>), object: nil, userInfo: managers)
})
self.sdlManager.screenManager.softButtonObjects = [softButton];
```
### Create the EventEmitter Class

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
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(getDoActionNotification:) name:<#Notification Name#> object:nil];

    return self;
}

- (NSArray<NSString *> *)supportedEvents {
    return @[@"DoAction"];
}

- (void)getDoActionNotification:(NSNotification *)notification {
    if(self.sdlManager == nil) {
        self.sdlManager = notification.userInfo[@"sdlManager"];
    }
    [self sendEventWithName:@"DoAction" body:@{@"type": @"actionType"}];
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

    sendEvent(withName: "DoAction", body: ["type": "actionType"])
}

override func supportedEvents() -> [String]! {
    return ["DoAction"]
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

RCT_EXTERN_METHOD(eventCall:(eventCall: (id)dict))

@end
```

The above example will then call into JavaScript with an event type `DoAction`. Inside your React Native(Javascript) code create a `NativeEventEmitter` object  within your `EventEmitter` module and add a listener for the event.

```javascript
import { NativeEventEmitter, NativeModules } from 'react-native';
const  { SDLEventEmitter } = NativeModules;

const testEventEmitter = new NativeEventEmitter(SDLEventEmitter);

// Build a listener to listen for events
const testData = testEventEmitter.addListener(
    'DoAction',
        () => SDLEventEmitter.eventCall({
            "data": {
                "low": "77",
                "high": "87",
                "currentTemp": "82",
                "rain": "50%"
            }
        }   
    )
)
```

The last step is to wrap the method you wish to expose inside an `RCT_EXPORT_METHOD` for Objective-C and `RCT_EXTERN_METHOD` for Swift. 

##### Objective-C
```objc
RCT_EXPORT_METHOD(eventCall:(NSDictionary *)dict) {
[self.sdlManager.screenManager beginUpdates];

self.sdlManager.screenManager.textField1 = [NSString stringWithFormat:@"Low: %@ ºF", [RCTConvert NSString:dict[@"data"][@"low"]]];
self.sdlManager.screenManager.textField2 = [NSString stringWithFormat:@"High: %@ ºF", [RCTConvert NSString:dict[@"data"][@"high"]]];

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
@objc func eventCall(_ dict: NSDictionary) {
    self.sdlManager.screenManager.beginUpdates()
    let data = dict["data"]! as! NSDictionary
    self.sdlManager.screenManager.textField1 = "Low: \(data["low"]!) ºF")"
    self.sdlManager.screenManager.textField2 = "High: \(data["high"]!) ºF")"
    self.sdlManager.screenManager.endUpdates()
}
```
