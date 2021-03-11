# Adaptive Interface Capabilities
Since each car manufacturer has different user interface style guidelines, the number of lines of text, soft and hard buttons, and images supported will vary between different types of head units. The system will send information to your app about its capabilities for various user interface elements. You should use this information to create the user interface of your SDL app.

You can access these properties on the @![iOS]`SDLManager.systemCapabilityManager`!@@![android, javaSE, javaEE,javascript]`sdlManager.getSystemCapabilityManager()`!@ instance. 

## System Capability Manager Properties@![javascript]/Methods!@
| Parameters@![javascript]/Methods!@  |  Description | RPC Version |
| ------------- | ------------- | ------------- |
| @![iOS]displays!@@![android, javaSE, javaEE, javascript]SystemCapabilityType.DISPLAYS!@ | Specifies display related information. The primary display will be the first element within the array. Windows within that display are different places that the app could be displayed (such as the main app window and various widget windows). | RPC v6.0+ |
| @![iOS]hmiZoneCapabilities!@@![android, javaSE, javaEE]SystemCapabilityType.HMI_ZONE!@@![javascript]getHmiZoneCapabilities()!@ | Specifies HMI Zones in the vehicle. There may be a HMI available for back seat passengers as well as front seat passengers. | RPC v1.0+ |
| @![iOS]speechCapabilities!@@![android, javaSE, javaEE]SystemCapabilityType.SPEECH!@@![javascript]getSpeechCapabilities()!@ | Contains information about TTS capabilities on the SDL platform. Platforms may support text, SAPI phonemes, LH PLUS phonemes, pre-recorded speech, and silence. | RPC v1.0+ |
| @![iOS]prerecordedSpeechCapabilities!@@![javascript]getPrerecordedSpeechCapabilities()!@ | @![iOS, javascript]A list of pre-recorded sounds you can use in your app. Sounds may include a help, initial, listen, positive, or a negative jingle.!@@![android, javaSE, javaEE, javascript]Currently only available in the SDL_iOS and SDL JavaScript libraries!@ | RPC v3.0+ |
| @![iOS]vrCapability!@@![android, javaSE, javaEE]SystemCapabilityType.VOICE_RECOGNITION!@@![javascript]getVrCapabilities()!@ | The voice-recognition capabilities of the connected SDL platform. The platform may be able to recognize spoken text in the current language. | RPC v1.0+ |
| @![iOS]audioPassThruCapabilities!@@![android, javaSE, javaEE]SystemCapabilityType.AUDIO_PASSTHROUGH!@@![javascript]getAudioPassThruCapabilities()!@ | Describes the sampling rate, bits per sample, and audio types available. | RPC v2.0+ |
| @![iOS]pcmStreamCapabilities!@@![android, javaSE, javaEE]SystemCapabilityType.PCM_STREAMING!@@![javascript]getPcmStreamCapabilities()!@ | Describes different audio type configurations for the audio PCM stream service, e.g. {8kHz,8-bit,PCM}. | RPC v4.1+ |
| @![iOS]hmiCapabilities!@@![android, javaSE, javaEE]SystemCapabilityType.HMI!@@![javascript]getHmiCapabilities()!@ | Returns whether or not the app can support built-in navigation and phone calls. | RPC v3.0+ |
| @![iOS]appServicesCapabilities!@@![android, javaSE, javaEE, javascript]SystemCapabilityType.APP_SERVICES!@ | Describes the capabilities of app services including what service types are supported and the current state of services. | RPC v5.1+ |
| @![iOS]navigationCapability!@@![android, javaSE, javaEE, javascript]SystemCapabilityType.NAVIGATION!@ | Describes the built-in vehicle navigation system's APIs. | RPC v4.5+ |
| @![iOS]phoneCapability!@@![android, javaSE, javaEE, javascript]SystemCapabilityType.PHONE_CALL!@ | Describes the built-in phone calling capabilities of the IVI system. | RPC v4.5+ |
| @![iOS]videoStreamingCapability!@@![android, javaSE, javaEE, javascript]SystemCapabilityType.VIDEO_STREAMING!@ | Describes the abilities of the head unit to video stream projection applications. | RPC v4.5+ |
| @![iOS]remoteControlCapability!@@![android, javaSE, javaEE, javascript]SystemCapabilityType.REMOTE_CONTROL!@ | Describes the abilities of an app to control built-in aspects of the IVI system. | RPC v4.5+ |
| @![iOS]seatLocationCapability!@@![android, javaSE, javaEE, javascript]SystemCapabilityType.SEAT_LOCATION!@| Describes the positioning of each seat in a vehicle| RPC v6.0+ |
    
### Deprecated Properties
The following properties are deprecated on SDL @![iOS]iOS 6.4!@@![android, javaSE, javaEE]Android 4.10!@@![javascript]JavaScript 1.0!@ because as of RPC v6.0 they are deprecated. However, these properties will still be filled with information. When connected on RPC <6.0, the information will be exactly the same as what is returned in the `RegisterAppInterfaceResponse` and `SetDisplayLayoutResponse`. However, if connected on RPC >6.0, the information will be converted from the newer-style display information, which means that some information will not be available.

| Parameters | Description |
| ---------- | ----------- |
| @![iOS]displayCapabilities!@@![android, javaSE, javaEE]SystemCapabilityType.DISPLAY!@@![javascript]getDisplayCapabilities()!@ | Information about the HMI display. This includes information about available templates, whether or not graphics are supported, and a list of all text fields and the max number of characters allowed in each text field. |
| @![iOS]buttonCapabilities!@@![android, javaSE, javaEE]SystemCapabilityType.BUTTON!@@![javascript]getDefaultMainWindowCapability().getButtonCapabilities()!@ | A list of available buttons and whether the buttons support long, short and up-down presses. |
| @![iOS]softButtonCapabilities!@@![android, javaSE, javaEE]SystemCapabilityType.SOFTBUTTON!@@![javascript]getDefaultMainWindowCapability().getSoftButtonCapabilities()!@ | A list of available soft buttons and whether the button support images. Also, information about whether the button supports long, short and up-down presses. |
| @![iOS]presetBankCapabilities!@@![android, javaSE, javaEE]SystemCapabilityType.PRESET_BANK!@@![javascript]getPresetBankCapabilities()!@ | If returned, the platform supports custom on-screen presets. |

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
ImageField field = sdlManager.getSystemCapabilityManager().getDefaultMainWindowCapability().getImageFields().get(index);
ImageResolution resolution = field.getImageResolution();
```
!@ 

@![javascript]
```js
const field = sdlManager.getSystemCapabilityManager().getDefaultMainWindowCapability().getImageFields()['<#index#>'];
const resolution = field.getImageResolution();
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
| graphic 			   | Show 					    | Image shown on the base screen as cover art | 185x185px | png, jpg, bmp |

## Querying and Subscribing System Capabilities
Capabilities that can be updated can be queried and subscribed to using the @![iOS]`SDLSystemCapabilityManager`!@@![android, javaSE, javaEE, javascript]`SystemCapabilityManager`!@.

### Determining Support for System Capabilities
You should check if the head unit supports your desired capability before subscribing to or updating the capability.

@![iOS]
##### Objective-C
```objc
BOOL navigationSupported = [self.sdlManager.systemCapabilityManager isCapabilitySupported:SDLSystemCapabilityTypeNavigation];
```

##### Swift
```swift
let navigationSupported = sdlManager.systemCapabilityManager.isCapabilitySupported(type: .navigation)
```
!@

@![android, javaSE, javaEE]
```java
boolean navigationSupported = sdlManager.getSystemCapabilityManager().isCapabilitySupported(SystemCapabilityType.NAVIGATION);
```
!@

@![javascript]
```js
const navigationSupported = sdlManager.getSystemCapabilityManager().isCapabilitySupported(SDL.rpc.enums.SystemCapabilityType.NAVIGATION);
```
!@

### Manual Querying for System Capabilities
Most head units provide features that your app can use: making and receiving phone calls, an embedded navigation system, video and audio streaming, as well as supporting app services. To pull information about this capability, use the @![iOS]`SDLSystemCapabilityManager`!@@![android, javaSE, javaEE, javascript]`SystemCapabilityManager`!@ to query the head unit for the desired capability. If a capability is unavailable, the query will return @![iOS]`nil`!@@![android, javaSE, javaEE, javascript]`null`!@.

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
        // Handle Error
    }
}, false);
```
!@

@![javascript]
```js
const appServicesCapabilities = await sdlManager.getSystemCapabilityManager().updateCapability(SDL.rpc.enums.SystemCapabilityType.APP_SERVICES);
if (appServicesCapabilities !== null) {
    // Capability retrieved
} else {
    // Handle Error
}
```
!@

### Subscribing to System Capabilities (RPC v5.1+)
In addition to getting the current system capabilities, it is also possible to subscribe for updates when the head unit capabilities change. @![iOS]To get these notifications you must register using a `subscribeToCapabilityType:` method.!@@![android, javaSE, javaEE]Since this information must be queried from Core you must implement the `OnSystemCapabilityListener`.!@

!!! NOTE
If @![iOS]`supportsSubscriptions == NO`!@@![android, javaSE, javaEE]`supportsSubscriptions == false`!@@![javascript]`supportsSubscriptions === false`!@, you can still subscribe to capabilities, however, you must manually poll for new capability updates using @![iOS]`updateCapabilityType:completionHandler:`!@@![android, javaSE, javaEE]`getCapability(type, listener, forceUpdate)` with `forceUpdate` set to `true`!@@![javascript]`getCapability(type)`!@. All subscriptions will be automatically updated when that method returns a new value.

The `DISPLAYS` type can be subscribed on all SDL versions.
!!!

#### Checking if the Head Unit Supports Subscriptions
@![iOS]
##### Objective-C
```objc
BOOL supportsSubscriptions = self.sdlManager.systemCapabilityManager.supportsSubscriptions;
```
##### Swift
```swift
let supportsSubscriptions = sdlManager.systemCapabilityManager.supportsSubscriptions;
```
!@
@![android, javaSE, javaEE]
```java
boolean supportsSubscriptions = sdlManager.getSystemCapabilityManager().supportsSubscriptions();
```
!@
@![javascript]
The `supportsSubscriptions` method currently is not supported by the JavaScript Suite. This will be addressed in a future release.
!@

#### Subscribe to a Capability
@![iOS]
##### Objective-C
```objc
// Subscribing to a capability via a selector callback. `success` will be `NO` if the subscription fails.
BOOL success = [self.sdlManager.systemCapabilityManager subscribeToCapabilityType:SDLSystemCapabilityTypeNavigation withObserver:self selector:@selector(navigationCapabilitySelectorCallback:error:subscribed:)];

// This can either have one or zero parameters. If one parameter it must be of type `SDLSystemCapability`. See the [reference documentation](https://smartdevicelink.com/en/docs/iOS/master/Classes/SDLSystemCapabilityManager/) for more details.
- (void)navigationCapabilitySelectorCallback:(SDLSystemCapability *)capability error:(NSError *)error subscribed:(BOOL)isSubscribed {
    if (error != nil) { return; }
    SDLNavigationCapability *navCapability = capability.navigationCapability;

    <#Use the capability#>
}

// Subscribing to a capability via a block callback. `subscribeToken` will be `nil` if the subscription failed. Pass `subscribeToken` to the observer parameter of `unsubscribeFromCapabilityType:withObserver:` to unsubscribe the block.
id subscribeToken = [self subscribeToCapabilityType:SDLSystemCapabilityTypeNavigation withUpdateHandler:^(SDLSystemCapability *_Nullable capability, BOOL subscribed, NSError *_Nullable error) {
    if (error != nil) { return; }
    SDLNavigationCapability *navCapability = capability.navigationCapability;
    <#Use the capability#>
}];
```

##### Swift
```swift
// Subscribing to a capability via a selector callback
sdlManager.systemCapabilityManager.subscribe(toCapabilityType: .navigation, withObserver: self, selector: #selector(navigationCapabilitySelectorCallback(_:error:subscribed:)))

@objc private func navigationCapabilitySelectorCallback(_ capability: SDLSystemCapability?, error: NSError?, subscribed: Bool) {
    guard error == nil else { return }
    let navCapability = capability.navigationCapability;

    <#Use the capability#>
}

// Subscribing to a capability via a block callback
sdlManager.systemCapabilityManager.subscribe(capabilityType: .navigation) { (capability, subscribed, error) in
    guard error == nil else { return }
    let navCapability = capability.navigationCapability
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
        // Handle Error 
    }
});
```
!@

@![javascript]
```js
sdlManager.getSystemCapabilityManager().addOnSystemCapabilityListener(SDL.rpc.enums.SystemCapabilityType.APP_SERVICES, function (capability) {
    // This listener is now subscribed to AppServicesCapabilities
    if (capability instanceof SDL.rpc.structs.AppServicesCapabilities) {
        // Got an AppServicesCapabilities struct
    }
})
```
!@
