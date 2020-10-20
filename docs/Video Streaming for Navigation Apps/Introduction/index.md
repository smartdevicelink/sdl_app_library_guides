# Introduction
Mobile navigation allows map partners to easily display their maps as well as present visual and audio turn-by-turn prompts on the head unit.

 Navigation apps have different behavior on the head unit than normal applications. The main differences are:
 
* Navigation apps don't use base screen templates. Their main view is the video stream sent from the device.
* Navigation apps can send audio via a binary stream. This will attenuate the current audio source and should be used for navigation commands.
* Navigation apps can receive touch events from the video stream.

@![android]
!!! NOTE
In order to use SDL's Mobile Navigation feature, the app must have a minimum requirement of Android 4.4 (SDK 19). This is due to using Android's provided video encoder.
!!!
!@

## Configuring a Navigation App
The basic connection setup is similar for all apps. Please follow the @![iOS][iOS Integration Basics](Getting Started/iOS Integration Basics)!@@![android,javaEE,javaSE][Java Suite Integration Basics](Getting Started/Java Suite Integration Basics)!@ guide for more information.

In order to create a navigation app an @![iOS]`appType`!@@![android,javaSE,javaEE]`appHMIType`!@ of @![iOS]`SDLAppHMITypeNavigation`!@@![android,javaSE,javaEE]`NAVIGATION`!@ must be set in the @![iOS]`SDLManager`'s `SDLLifecycleConfiguration`!@@![android,javaSE,javaEE]`SdlManager`'s `Builder`!@.

@![iOS]The second difference is that a `SDLStreamingMediaConfiguration` must be created and passed to the `SDLConfiguration`. A property called `securityManagers` must be set if connecting to a version of Core that requires secure video and audio streaming. This property requires an array of classes of security managers, which will conform to the `SDLSecurityType` protocol.!@ @![android] The second difference is the ability to call the `setSdlSecurity(List<Class<? extends SdlSecurityBase>> secList)` method from the `SdlManager.Builder` if connecting to an implementation of Core that requires secure video and audio streaming. This method requires an array of security libraries, which will extend the `SdlSecurityBase` class.!@ These security libraries are provided by the OEMs themselves, and will only work for that OEM. There is no general catch-all security library.

@![iOS]
##### Objective-C
```objc
SDLLifecycleConfiguration* lifecycleConfig = [SDLLifecycleConfiguration defaultConfigurationWithAppName:@"<#App Name#>" fullAppId:@"<#App Id#>"];
lifecycleConfig.appType = SDLAppHMITypeNavigation;

SDLEncryptionConfiguration *encryptionConfig = [[SDLEncryptionConfiguration alloc] initWithSecurityManagers:@[OEMSecurityManager.self] delegate:self];
SDLStreamingMediaConfiguration *streamingConfig = [SDLStreamingMediaConfiguration secureConfiguration];
SDLConfiguration *config = [[SDLConfiguration alloc] initWithLifecycle:lifecycleConfig lockScreen:[SDLLockScreenConfiguration enabledConfiguration] logging:[SDLLogConfiguration defaultConfiguration] streamingMedia:streamingConfig fileManager:[SDLFileManagerConfiguration defaultConfiguration] encryption:encryptionConfig];
```

##### Swift
```swift
let lifecycleConfig = SDLLifecycleConfiguration(appName: "<#App Name#>", fullAppId: "<#App Id#>")
lifecycleConfig.appType = .navigation

let encryptionConfig = SDLEncryptionConfiguration(securityManagers: [OEMSecurityManager.self], delegate: self)
let streamingConfig = SDLStreamingMediaConfiguration.secure()
let config = SDLConfiguration(lifecycle: lifecycleConfig, lockScreen: .enabled(), logging: .default(), streamingMedia: streamingConfig, fileManager: .default(), encryption: encryptionConfig)
```
!@

@![android]
```java
SdlManager.Builder builder = new SdlManager.Builder(this, APP_ID, APP_NAME, listener);

Vector<AppHMIType> hmiTypes = new Vector<AppHMIType>();
hmiTypes.add(AppHMIType.NAVIGATION);
builder.setAppTypes(hmiTypes);

// Add security managers if Core requires secure video & audio streaming
List<Class<? extends SdlSecurityBase>> secList = new ArrayList<>();
secList.add(OEMSdlSecurity.class);
builder.setSdlSecurity(secList, serviceEncryptionListener);

MultiplexTransportConfig mtc = new MultiplexTransportConfig(this, APP_ID, MultiplexTransportConfig.FLAG_MULTI_SECURITY_OFF);
mtc.setRequiresHighBandwidth(true);
builder.setTransportType(mtc);

sdlManager = builder.build();
sdlManager.start();
```
!@

!!! MUST
When compiling your app for production, make sure to include all possible OEM security managers that you wish to support.
!!!

@![iOS]
### Preventing Device Sleep
When building a navigation app, you should ensure that the device never sleeps while your app is in the foreground of the device and is in an HMI level other than `NONE`. If your device sleeps, it will be unable to stream video data. To do so, implement the following `SDLManagerDelegate` method.

##### Objective-C
```objc
- (void)hmiLevel:(SDLHMILevel)oldLevel didChangeToLevel:(SDLHMILevel)newLevel {
    if (![newLevel isEqualToEnum:SDLHMILevelNone]) {
        [UIApplication sharedApplication].idleTimerDisabled = YES;
    } else {
        [UIApplication sharedApplication].idleTimerDisabled = NO;
    }
}
```

##### Swift
```swift
func hmiLevel(_ oldLevel: SDLHMILevel, didChangeToLevel newLevel: SDLHMILevel) {
    if newLevel != .none {
        UIApplication.shared.isIdleTimerDisabled = true
    } else {
        UIApplication.shared.isIdleTimerDisabled = false
    }
}
```
!@

## Keyboard Input
To present a keyboard (such as for searching for navigation destinations), you should use the @![iOS]`SDLScreenManager`!@@![android]`ScreenManager`!@'s keyboard presentation feature. For more information, see the [Popup Menus and Keyboards](Displaying a User Interface/Popup Menus and Keyboards) guide.

## Navigation Subscription Buttons
Head units supporting RPC v6.0+ may support navigation-specific subscription buttons for the navigation template. These subscription buttons allow your user to manipulate the map using hard buttons located on car's center console or steering wheel. It is important to support these subscription buttons in order to provide your user with the expected in-car navigation user experience. This is especially true on head units that don't support touch input as there will be no other way for your user to manipulate the map. See [Template Subscription Buttons](Displaying a User Interface/Template Subscription Buttons) for a list of these navigation buttons.