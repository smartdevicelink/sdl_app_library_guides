# Can I Integrate SDL into a React Native App?
SDL does work and can be integrated into a React Native application. 

!!! NOTE
You must make sure you have [Native Modules](https://facebook.github.io/react-native/docs/native-modules-setup) installed as a dependency in order to use 3rd party APIs in a React Native application. If this is not done your app will not work with SmartDeviceLink. Native API methods are not exposed to JavaScript automatically, this must be done manually by you.
!!!

Please follow [this guide](https://facebook.github.io/react-native/docs/getting-started) for how to create a new React Native application if you need one. Also ensure you have followed the [Getting Started](Getting Started/Installation) and have `SmartDeviceLink` installed on the native side. 

To install SDL into your React Native app, you will need to follow [this guide](https://facebook.github.io/react-native/docs/native-modules-ios) to integrate the SDL library into your application using React Native's Native Modules feature.

## Getting Started
This guide is not meant to walk you through how to make a React Native app but help you integrate SDL into an existing application. We will show you a basic example of how to communicate between your app's JavaScript code and SDL's native Obj-C code. For more advanced features, please refer to the React Native documentation linked above.

## Integration Basics
Native API methods are not exposed automatically to JavaScript. This means you must expose methods you wish to use from SDL to your React Native app. You must implement the `RCTBridgeModule` protocol into a bridge class (see below for an example). Please follow [SmartDeviceLink Integration Basics](Getting Started/Integration Basics) for the basic setup of a native SDL `ProxyManager` class that your bridge code will communicate with. This is the necessary starting point in order to continue with this example. Also set up a simple UI with buttons and some text on the SDL side.

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
Before you move forward, you must add  `#import "React/RCTBridgeModule.h"` to your `Bridging Header`. When creating a Swift application and importing Objective-C code, Xcode should ask if it should create this header file for you. You must include this bridging header for your React Native app to work. You can create this file manually as well. 

```swift
@objc(ProxyManager)

class ProxyManager: NSObject {

<#...#>

}
```

Next, to expose the Swift class to React Native, you must create an Objective-C file in order to use React Native macros.

##### Objective-C
###### ProxyManager.m 
```objc
#import "React/RCTBridgeModule.h"

@interface RCT_EXTERN_MODULE(ProxyManager, NSObject)

@end
```

### Emitting Event Notifications to JavaScript
Inside the `ProxyManger` class post a notification for a particular event you wish to execute. The bridge will observe this event and will call the React Native listener that you will set up later in the documentation below.

##### Objective-C
Inside the `ProxyManager` add a soft button to your SDL HMI. Inside the soft button handler, post the notification and pass along a reference to the `sdlManager` in order to update your React Native UI through the bridge.

```objc
SDLSoftButtonObject *softButton = [[SDLSoftButtonObject alloc] initWithName:@"Button" state:[[SDLSoftButtonState alloc] initWithStateName:@"State 1" text:@"Data" artwork:nil] handler:^(SDLOnButtonPress * _Nullable buttonPress, SDLOnButtonEvent * _Nullable buttonEvent) {
    if (buttonPress == nil) { return; }

    NSDictionary *userInfo = @{@"sdlManager": self.sdlManager};
    [[NSNotificationCenter defaultCenter] postNotificationName:<#Notification Name#> object:nil userInfo:managers];
}];

self.sdlManager.screenManager.softButtonObjects = @[softButton];
```

##### Swift
```swift
let softButton = SDLSoftButtonObject(name: "Button", state: SDLSoftButtonState(stateName: "State", text: "Data", artwork: nil), handler: { (buttonPress, butonEvent) in
    guard buttonPress == nil else { return }
    
    let userInfo = ["sdlManager": self.sdlManager]
    NotificationCenter.default.post(name: NSNotification.Name(rawValue: <#Notification Name#>), object: nil, userInfo: managers)
})

self.sdlManager.screenManager.softButtonObjects = [softButton];
```

#### Create the EventEmitter Bridge Class

Create the class that will be the listener for the notiification you created above. This class will be sending and receiving messages from your JavaScript code (React Native). There are some required method(s) you must include in order to be a React Native `EventEmitter`; these method(s) are in the example below. 

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
    // Subscribe to event notifications sent from ProxyManager
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(getDoActionNotification:) name:<#Notification Name#> object:nil];

    return self;
}

// Required Method 
- (NSArray<NSString *> *)supportedEvents {
    return @[@"DoAction"];
}

// Run this code when the subscribed event notification is received
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
    // Subscribe to event notifications sent from ProxyManager
    NotificationCenter.default.addObserver(self, selector: #selector(doAction(_:)), name: Notification.Name(rawValue: "<#Notification Name#>", object: nil)
    super.init()
}

// Run this code when the subscribed event notification is received
@objc func doAction(_ notification: Notification) {
    if self.sdlManger == nil {
        self.sdlManager = notification.userInfo["sdlManager"]
    }

    sendEvent(withName: "DoAction", body: ["type": "actionType"])
}

// Required Method
override func supportedEvents() -> [String]! {
    return ["DoAction"]
}
}
```

The above example will call into your JavaScript code with an event type `DoAction`. Inside your React Native (JavaScript) code, create a `NativeEventEmitter` object within your `EventEmitter` module and add a listener for the event.

##### JavaScript
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

### Exposing Methods
The last step is to wrap any methods you wish to expose inside `RCT_EXPORT_METHOD` for Objective-C and `RCT_EXTERN_METHOD` for Swift. Inside the `SDLEventEmitter.m` file add the following method.

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
}
```

##### Swift

!!! NOTE
Make sure you add `#import "React/RCTEventEmitter.h"` to the apps bridging header.
!!!

If you're making a React Native application and using native Swift code, you will need to create the Objective-C bridger for the `SDLEventEmitter` class you created above. Wrap the method(s) you wish to expose in a  `RCT_EXTERN_METHOD` macro inside your wrapper class. This wrapper will allow the JavaScript code to talk with your native code.

```objc
#import "React/RCTBridgeModule.h"
#import "React/RCTEventEmitter.h"

@interface RCT_EXTERN_MODULE(SDLEventEmitter, RCTEventEmitter)

RCT_EXTERN_METHOD(eventCall:(eventCall: (id)dict))

@end
```
Add the following method to `SDLEventEmitter.swift`:

```swift
@objc func eventCall(_ dict: NSDictionary) {
    self.sdlManager.screenManager.beginUpdates()
    let data = dict["data"]! as! NSDictionary
    self.sdlManager.screenManager.textField1 = "Low: \(data["low"]!) ºF")"
    self.sdlManager.screenManager.textField2 = "High: \(data["high"]!) ºF")"
    self.sdlManager.screenManager.endUpdates()
}
```
