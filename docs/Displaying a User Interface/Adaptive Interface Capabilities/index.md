# Adaptive Interface Capabilities
Since each car manufacturer has different user interface style guidelines, the number of lines of text, soft and hard buttons, and images supported will vary between different types of head units. The system will send information to your app about its capabilities for various user interface elements. You should use this information to create the user interface of your SDL app.

You can access these properties on the @![iOS]`SDLManager.systemCapabilityManager`!@@![android, javaSE, javaEE]`sdlManager.getSystemCapabilityManager`!@ instance. 

## System Capability Manager Properties
| Parameters  |  Description |
| ------------- | ------------- |
| @![iOS]displays!@@![android, javaSE, javaEE]SystemCapabilityType.DISPLAYS!@ | Specifies display related information. The primary display will be the first element within the array. Windows within that display are different places that the app could be displayed (such as the main app window and various widget windows). |
| @![iOS]hmiZoneCapabilities!@@![android, javaSE, javaEE]SystemCapabilityType.HMI_ZONE!@ | Specifies HMI Zones in the vehicle. There may be a HMI available for back seat passengers as well as front seat passengers. |
| @![iOS]speechCapabilities!@@![android, javaSE, javaEE]SystemCapabilityType.SPEECH!@ | Contains information about TTS capabilities on the SDL platform. Platforms may support text, SAPI phonemes, LH PLUS phonemes, pre-recorded speech, and silence. |
| prerecordedSpeechCapabilities | @![iOS]A list of pre-recorded sounds you can use in your app. Sounds may include a help, initial, listen, positive, or a negative jingle.!@@![android, javaSE, javaEE]Currently only available in the SDL_iOS library!@ |
| @![iOS]vrCapability!@@![android, javaSE, javaEE]SystemCapabilityType.VOICE_RECOGNITION!@ | The voice-recognition capabilities of the connected SDL platform. The platform may be able to recognize spoken text in the current language. |
| @![iOS]audioPassThruCapabilities!@@![android, javaSE, javaEE]SystemCapabilityType.AUDIO_PASSTHROUGH!@ | Describes the sampling rate, bits per sample, and audio types available. |
| @![iOS]pcmStreamCapabilities!@@![android, javaSE, javaEE]SystemCapabilityType.PCM_STREAMING!@ | Describes different audio type configurations for the audio PCM stream service, e.g. {8kHz,8-bit,PCM}. |
| @![iOS]hmiCapabilities!@@![android, javaSE, javaEE]SystemCapabilityType.HMI!@ | Returns whether or not the app can support built-in navigation and phone calls. |
| @![iOS]appServicesCapabilities!@@![android, javaSE, javaEE]SystemCapabilityType.APP_SERVICES!@ | Describes the capabilities of app services including what service types are supported and the current state of services. |
| @![iOS]navigationCapability!@@![android, javaSE, javaEE]SystemCapabilityType.NAVIGATION!@ | Describes the built-in vehicle navigation system's APIs. |
| @![iOS]phoneCapability!@@![android, javaSE, javaEE]SystemCapabilityType.PHONE_CALL!@ | Describes the built-in phone calling capabilities of the IVI system. |
| @![iOS]videoStreamingCapability!@@![android, javaSE, javaEE]SystemCapabilityType.VIDEO_STREAMING!@ | Describes the abilities of the head unit to video stream projection applications. |
| @![iOS]remoteControlCapability!@@![android, javaSE, javaEE]SystemCapabilityType.REMOTE_CONTROL!@ | Describes the abilities of an app to control built-in aspects of the IVI system. |

### Deprecated Properties
The following properties are deprecated on SDL @![iOS]iOS 6.4!@@![android, javaSE, javaEE]Android 4.10!@ because as of RPC v6.0 they are deprecated. However, these properties will still be filled with information. When connected on RPC <6.0, the information will be exactly the same as what is returned in the `RegisterAppInterfaceResponse` and `SetDisplayLayoutResponse`. However, if connected on RPC >6.0, the information will be converted from the newer-style display information, which means that some information will not be available.

| Parameters | Description |
| ---------- | ----------- |
| @![iOS]displayCapabilities!@@![android, javaSE, javaEE]SystemCapabilityType.DISPLAY!@ | Information about the HMI display. This includes information about available templates, whether or not graphics are supported, and a list of all text fields and the max number of characters allowed in each text field. |
| @![iOS]buttonCapabilities!@@![android, javaSE, javaEE]SystemCapabilityType.BUTTON!@ | A list of available buttons and whether the buttons support long, short and up-down presses. |
| @![iOS]softButtonCapabilities!@@![android, javaSE, javaEE]SystemCapabilityType.SOFTBUTTON!@ | A list of available soft buttons and whether the button support images. Also, information about whether the button supports long, short and up-down presses. |
| @![iOS]presetBankCapabilities!@@![android, javaSE, javaEE]SystemCapabilityType.PRESET_BANK!@ | If returned, the platform supports custom on-screen presets. |

### Image Specifics
Images may be formatted as PNG, JPEG, or BMP. You can find which image types and resolutions are supported using the system capability manager.

Since the head unit connection is often relatively slow (especially over Bluetooth), you should pay attention to the size of your images to ensure that they are not larger than they need to be. If an image is uploaded that is larger than the supported size, the image will be scaled down by Core.

@![iOS]
##### Objective-C
```objc
SDLImageField *field = self.sdlManager.systemCapabilityManager.defaultMainWindowCapability.imageFields[<#index#>]
SDLImageResolution *resolution = field.imageResolution;
```

##### Swift
```swift
let field = sdlManager.systemCapabilityManager.defaultMainWindowCapability.imageFields[<#index#>]
let resolution = field.imageResolution
```
!@

@![android, javaSE, javaEE]
```java
ImageField field = sdlManager.getSystemCapabilityManager().getDefaultMainWindowCapability().getImageFields().get(<#index#>);
ImageResolution resolution = field.getImageResolution();
```
!@ 

#### Example Image Sizes
Below is a table with example image sizes. Check the `SystemCapabilityManager` for the exact image sizes desired by the system you are connecting to. The connected system should be able to scale down larger sizes, but if the image you are sending is much larger than desired, then performance will be impacted.

| ImageName | Used in RPC | Details | Size | Type |
| -------------- | ---------------- | -------- | --------- | ------- |
| softButtonImage      | Show 					    | Image shown on softbuttons on the base screen | 70x70px | png, jpg, bmp |
| choiceImage          | CreateInteractionChoiceSet | Image shown in the manual part of an performInteraction either big (ICON_ONLY) or small (LIST_ONLY) | 70x70px | png, jpg, bmp |
| choiceSecondaryImage | CreateInteractionChoiceSet | Image shown on the right side of an entry in (LIST_ONLY) performInteraction	| 35x35px | png, jpg, bmp |
| vrHelpItem		   | SetGlobalProperties		| Image shown during voice interaction | 35x35px | png, jpg, bmp |
| menuIcon			   | SetGlobalProperties		| Image shown on the “More…” button | 35x35px | png, jpg, bmp |
| cmdIcon			   | AddCommand				    | Image shown for commands in the "More…" menu | 35x35px | png, jpg, bmp |
| appIcon 			   | SetAppIcon				    | Image shown as Icon in the "Mobile Apps" menu | 70x70px | png, jpg, bmp |
| graphic 			   | Show 					    | Image shown on the basescreen as cover art | 185x185px | png, jpg, bmp |

## Querying for System Capabilities
Most head units provide features that your app can use: making and receiving phone calls, an embedded navigation system, video and audio streaming, as well as supporting app services. To find out if the head unit supports a feature as well as more information about the feature, use the @![iOS]`SDLSystemCapabilityManager`!@@![android, javaSE, javaEE]`SystemCapabilityManager`!@ to query the head unit for the desired capability. If a capability is unavailable, the query will return @![iOS]`nil`!@@![android, javaSE, javaEE]`null`!@.

@![iOS]
##### Objective-C
```objc
[self.sdlManager.systemCapabilityManager updateCapabilityType:SDLSystemCapabilityTypeVideoStreaming completionHandler:^(NSError * _Nullable error, SDLSystemCapabilityManager * _Nonnull systemCapabilityManager) {
    if (error != nil || systemCapabilityManager.videoStreamingCapability == nil) {
        return;
    }
    <#Use the video streaming capability#>
}];
```

##### Swift
```swift
sdlManager.systemCapabilityManager.updateCapabilityType(.videoStreaming) { (error, manager) in
    guard error == nil, let videoStreamingCapability = manager.videoStreamingCapability else {
        return
    }
    <#Use the video streaming capability#>
}
```
!@

@![android, javaSE, javaEE]
```java
sdlManager.getSystemCapabilityManager().getCapability(SystemCapabilityType.APP_SERVICES, new OnSystemCapabilityListener() {
    @Override
    public void onCapabilityRetrieved(Object capability) {
        AppServicesCapabilities servicesCapabilities = (AppServicesCapabilities) capability;
    }

    @Override
    public void onError(String info) {
        <# Handle Error #>
    }
});
```
!@

### Subscribing to System Capabilities
In addition getting the current system capabilities, it is also possible to subscribe for updates when the head unit capabilities change. @![iOS]To get these notifications you must register using a `subscribeToCapabilityType:` method.!@@![android, javaSE, javaEE]Since this information must be queried from Core you must implement the `OnSystemCapabilityListener`.!@ This feature is only available RPC v5.1 or greater connections (except for DISPLAYS, which is backward compatible to RPC v1.0).

@![iOS]
#### Checking if the Head Unit Supports Subscriptions
##### Objective-C
```objc
BOOL supportsSubscriptions = self.sdlManager.systemCapabilityManager.supportsSubscriptions;
```
##### Swift
```swift
let supportsSubscriptions = sdlManager.systemCapabilityManager.supportsSubscriptions;
```
!@

#### Subscribe to a Capability
@![iOS]
##### Objective-C
```objc
// Subscribing to a capability via a selector callback. `success` will be `NO` if the subscription fails.
BOOL success = [self.sdlManager.systemCapabilityManager subscribeToCapabilityType:SDLSystemCapabilityTypeNavigation withObserver:self selector:@selector(navigationCapabilitySelectorCallback:)];

// This can either have one or zero parameters. If one parameter it must be of type `SDLSystemCapability`. See the [reference documentation](https://smartdevicelink.com/en/docs/iOS/master/Classes/SDLSystemCapabilityManager/) for more details.
- (void)navigationCapabilitySelectorCallback:(SDLSystemCapability *)capability {
    SDLNavigationCapability *navCapability = capability.navigationCapability;

    <#Use the capability#>
}

// Subscribing to a capability via a block callback. `subscribeToken` will be `nil` if the subscription failed. Pass `subscribeToken` to the observer parameter of `unsubscribeFromCapabilityType:withObserver:` to unsubscribe the block.
id subscribeToken = [self subscribeToCapabilityType:SDLSystemCapabilityTypeNavigation usingBlock:^(SDLSystemCapability * _Nonnull capability) {
    SDLNavigationCapability *navCapability = capability.navigationCapability;
    <#Use the capability#>
}];
```

##### Swift
```swift
// Subscribing to a capability via a selector callback
sdlManager.systemCapabilityManager.subscribe(toCapabilityType: .navigation, withObserver: self, selector: #selector(navigationCapabilitySelectorCallback(_:)))

@objc private func navigationCapabilitySelectorCallback(_ capability: SDLSystemCapability) {
    let navCapability = capability.navigationCapability;

    <#Use the capability#>
}

// Subscribing to a capability via a block callback
sdlManager.systemCapabilityManager.subscribe(toCapabilityType: .navigation) { (capability) in
    let navCapability = capability.navigationCapability;
    <#Use the capability#>
}
```
!@
@![android, javaSE, javaEE]
```java
sdlManager.getSystemCapabilityManager().addOnSystemCapabilityListener(SystemCapabilityType.APP_SERVICES, new OnSystemCapabilityListener() {
    @Override
    public void onCapabilityRetrieved(Object capability) {
        AppServicesCapabilities servicesCapabilities = (AppServicesCapabilities) capability;
    }

    @Override
    public void onError(String info) {
        <# Handle Error #>
    }
});

GetSystemCapability getSystemCapability = new GetSystemCapability();
getSystemCapability.setSystemCapabilityType(SystemCapabilityType.APP_SERVICES);
getSystemCapability.setSubscribe(true);
sdlManager.sendRPC(getSystemCapability);
```
!@
