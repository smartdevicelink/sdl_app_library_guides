# Introduction
Mobile Navigation allows map partners to bring their applications into the car and display their maps and turn by turn easily for the user. This feature has a different behavior on the head unit than normal applications. The main differences are:

* Navigation Apps don't use base screen templates. Their main view is the video stream sent from the device.
* Navigation Apps can send audio via a binary stream. This will attenuate the current audio source and should be used for navigation commands.
* Navigation Apps can receive touch events from the video stream.

@![android]
!!! NOTE
In order to use SDL's Mobile Navigation feature, the app must have a minimum requirement of Android 4.4 (SDK 19). This is due to using Android's provided video encoder.
!!!
!@

## Connecting an app
The basic connection is similar for all apps. Please follow the [Integration Basics](Getting Started/Integration Basics) guide for more information.

The first difference for a navigation app is the `appHMIType` of @![iOS]`SDLAppHMITypeNavigation`!@@![android,javaSE,javaEE]`NAVIGATION`!@ that has to be set in the @![iOS]`SDLManager`'s `SDLLifecycleConfiguration`!@@![android,javaSE,javaEE]`SdlManager`'s `Builder`!@.

@![iOS]
The second difference is that a `SDLStreamingMediaConfiguration` must be created and passed to the `SDLConfiguration`. A property called securityManagers must be set if connecting to a version of Core that requires secure video & audio streaming. This property requires an array of classes of Security Managers, which will conform to the `SDLSecurityType` protocol. These security libraries are provided by the OEMs themselves, and will only work for that OEM. There is not a general catch-all security library.
!@

@![android]
The second difference is the ability to call the `setSdlSecurity(List<Class<? extends SdlSecurityBase>> secList)` method from the `SdlManager.Builder` if connecting to an implementation of Core that requires secure video & audio streaming. This method requires an array of security libraries, which will extend the `SdlSecurityBase` class. These security libraries are provided by the OEMs themselves, and will only work for that OEM. There is not a general catch-all security library.
!@

@![iOS]
##### Objective-C
```objc
SDLLifecycleConfiguration* lifecycleConfig = [SDLLifecycleConfiguration defaultConfigurationWithAppName:@"<#App Name#>" fullAppId:@"<#App Id#>"];
lifecycleConfig.appType = SDLAppHMITypeNavigation;

SDLStreamingMediaConfiguration *streamingConfig = [SDLStreamingMediaConfiguration secureConfigurationWithSecurityManagers:@[OEMSecurityManager.class]];
SDLConfiguration *config = [SDLConfiguration configurationWithLifecycle:lifecycleConfig lockScreen:[SDLLockScreenConfiguration enabledConfiguration] logging:[SDLLogConfiguration defaultConfiguration] streamingMedia:streamingConfig fileManager:[SDLFileManagerConfiguration defaultConfiguration]];
```

##### Swift
```swift
let lifecycleConfig = SDLLifecycleConfiguration(appName: "<#App Name#>", fullAppId: "<#App Id#>")
lifecycleConfig.appType = .navigation

let streamingConfig = SDLStreamingMediaConfiguration(securityManagers: [OEMSecurityManager.self])
let config = SDLConfiguration(lifecycle: lifecycleConfig, lockScreen: .enabled(), logging: .default(), streamingMedia: streamingConfig, fileManager: .default())
```
!@

@![android]
```java
SdlManager.Builder builder = new SdlManager.Builder(this, APP_ID, APP_NAME, listener);

Vector<AppHMIType> hmiTypes = new Vector<AppHMIType>();
hmiTypes.add(AppHMIType.NAVIGATION);
builder.setAppTypes(hmiTypes);

// Add security managers if Core requires secure video & audio streaming
List<? extends SdlSecurityBase> securityManagers = new ArrayList();
builder.setSdlSecurity(Arrays.asList(OEMSecurityManager1.class, OEMSecurityManager2.class));

MultiplexTransportConfig mtc = new MultiplexTransportConfig(this, APP_ID, MultiplexTransportConfig.FLAG_MULTI_SECURITY_OFF);
mtc.setRequiresHighBandwidth(true);
builder.setTransportType(transport);

sdlManager = builder.build();
sdlManager.start();
```
!@

!!! IMPORTANT
When compiling, you must make sure to include all possible OEM security managers that you wish to support.
!!!

## Keyboard Input
To present a keyboard (such as for searching for navigation destinations), you should use the @![iOS]`SDLScreenManager`'s!@@![android,javaSE,javaEE]`ScreenManager`'s!@ keyboard presentation feature. For more information, see the [Popup Menus and Keyboards](Displaying a User Interface/Popup Menus and Keyboards) guide.
