# Introduction

@![iOS, android]
Mobile Navigation allows map partners to bring their applications into the car and display their maps and turn by turn easily for the user. This feature has a different behavior on the head unit than normal applications. The main differences are:

* Navigation Apps don't use base screen templates. Their main view is the video stream sent from the device.
* Navigation Apps can send audio via a binary stream. This will attenuate the current audio source and should be used for navigation commands.
* Navigation Apps can receive touch events from the video stream.
!@

@![android]
!!! Note In order to use SDL's Mobile Navigation feature, the app must have a minimum requirement of Android 4.4 (SDK 19). This is due to using Android's provided video encoder. !!!
!@

@![iOS, android]
## Connecting an app
The basic connection is similar for all apps. Please follow the [Integration Basics](Getting Started/Integration Basics) guide for more information.
!@

@![iOS]
The first difference for a navigation app is the `appHMIType` of `SDLAppHMITypeNavigation` that has to be set in the `SDLLifecycleConfiguration`. Navigation apps are also non-media apps.
!@

@![android]
The first difference for a navigation app is the `appHMIType` of `SDLAppHMITypeNavigation` that has to be set in the `SDLManager`. Navigation apps are also non-media apps.
!@

@![iOS]
The second difference is that a `SDLStreamingMediaConfiguration` must be created and passed to the `SDLConfiguration`. A property called `securityManagers` must be set if connecting to a version of Core that requires secure video & audio streaming. This property requires an array of classes of Security Managers, which will conform to the `SDLSecurityType` protocol. These security libraries are provided by the OEMs themselves, and will only work for that OEM. There is not a general catch-all security library.
@!

@![android]
The second difference is the requirement to call the `setSdlSecurity(List<Class<? extends SdlSecurityBase>> secList)` method from the `SdlManager.Builder ` if connecting to an implementation of Core that requires secure video & audio streaming. This method requires an array of Security Managers, which will extend the `SdlSecurityBase` class. These security libraries are provided by the OEMs themselves, and will only work for that OEM. There is not a general catch-all security library.
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

List<? extends SdlSecurityBase> securityManagers = new ArrayList();
securityManagers.add(OEMSecurityManager1.class);
securityManagers.add(OEMSecurityManager1.class);
builder.setSdlSecurity(securityManagers);

MultiplexTransportConfig mtc = new MultiplexTransportConfig(this, APP_ID, MultiplexTransportConfig.FLAG_MULTI_SECURITY_OFF);
mtc.setRequiresHighBandwidth(true);
builder.setTransportType(transport);

sdlManager = builder.build();
sdlManager.start();
```
!@

@![iOS, android]
!!! IMPORTANT
When compiling, you must make sure to include all possible OEM security managers that you wish to support.
!!!

@![iOS]
## Keyboard Input
To present a keyboard (such as for searching for navigation destinations), you should use the `SDLScreenManager`'s keyboard presentation feature. For more information, see the [Popup Menus and Keyboards](Displaying a User Interface/Popup Menus and Keyboards) guide.
!@

@![javaEE, javaSE]

!!! NOTE
This feature is only available on Android apps. Currently, JavaSE (embedded) and JavaEE (cloud) apps don't support that.
!!!

!@
