# Adaptive Interface Capabilities
## Designing for Different Head Units
Since each car manufacturer has different user interface style guidelines, the number of lines of text, soft and hard buttons, and images supported will vary between different types of head units. The system will send information to your app about its capabilities for various user interface elements. You should use this information to create the user interface of your SDL app.

You may access these properties on the @![iOS]`SDLManager.systemCapabilityManager`!@@![android, javaSE, javaEE]`SdlManager.systemCapabilityManager`!@ instance.

## System Capability Manager Properties
| Parameters  |  Description | Notes |
| ------------- | ------------- |------------- |
| @![iOS]displays!@@![android, javaSE, javaEE]`// TODO`!@ | Specifies display related information. The primary display will be the first element within the array. Windows within that display are different places that the app could be displayed (such as the main app window and various widget windows). | Check @![iOS]SDLDisplayCapability.h!@@![android, javaSE, javaEE]DisplayCapability.java!@ |
| @![iOS]hmiZoneCapabilities!@@![android, javaSE, javaEE]SystemCapabilityType.HMI_ZONE!@ | Specifies HMI Zones in the vehicle. There may be a HMI available for back seat passengers as well as front seat passengers. | Check @![iOS]SDLHMIZoneCapabilities.h!@@![android, javaSE, javaEE]HmiZoneCapabilities.java!@ |
| @![iOS]speechCapabilities!@@![android, javaSE, javaEE]SystemCapabilityType.SPEECH!@ | Contains information about TTS capabilities on the SDL platform. Platforms may support text, SAPI phonemes, LH PLUS phonemes, pre-recorded speech, and silence. | Check @![iOS]SDLSpeechCapabilities.h!@@![android, javaSE, javaEE]SpeechCapabilities.java!@ for more information |
| prerecordedSpeechCapabilities | @![iOS]A list of pre-recorded sounds you can use in your app. Sounds may include a help, initial, listen, positive, or a negative jingle.!@@![android, javaSE, javaEE]Currently only available in the SDL_iOS library!@ | @![iOS]Check SDLPrerecordedSpeech.h for more information!@@![android, javaSE, javaEE]currently only available in the SDL_iOS library!@ |
| @![iOS]vrCapability!@@![android, javaSE, javaEE]SystemCapabilityType.VOICE_RECOGNITION!@ | The voice-recognition capabilities of the connected SDL platform. The platform may be able to recognize spoken text in the current language. | Check @![iOS]SDLVRCapabilities.h!@@![android, javaSE, javaEE]VrCapabilities.java!@ for more information |
| @![iOS]audioPassThruCapabilities!@@![android, javaSE, javaEE]SystemCapabilityType.AUDIO_PASSTHROUGH!@ | Describes the sampling rate, bits per sample, and audio types available. | Check @![iOS]SDLAudioPassThruCapabilities.h!@@![android, javaSE, javaEE]AudioPassThruCapabilities.java!@ for more information|
| @![iOS]pcmStreamCapabilities!@@![android, javaSE, javaEE]SystemCapabilityType.PCM_STREAMING!@ | Describes different audio type configurations for the audio PCM stream service, e.g. {8kHz,8-bit,PCM}. | Check @![iOS]SDLAudioPassThruCapabilities.h!@@![android, javaSE, javaEE]AudioPassThruCapabilities.java!@ for more information|
| @![iOS]hmiCapabilities!@@![android, javaSE, javaEE]SystemCapabilityType.HMI!@ | Returns whether or not the app can support built-in navigation and phone calls. | Check @![iOS]SDLHMICapabilities.h!@@![android, javaSE, javaEE]HMICapabilities.java!@ for more information |
| @![iOS]appServicesCapabilities!@@![android, javaSE, javaEE]SystemCapabilityType.APP_SERVICES!@ | Describes the capabilities of app services including what service types are supported and the current state of services. | Check @![iOS]SDLAppServicesCapabilities.h!@@![android, javaSE, javaEE]AppServicesCapabilities.java!@ for more information |
| @![iOS]navigationCapability!@@![android, javaSE, javaEE]SystemCapabilityType.NAVIGATION!@ | Describes the built-in vehicle navigation system's APIs. | Check @![iOS]SDLNavigationCapability.h!@@![android, javaSE, javaEE]NavigationCapability.java!@ for more information |
| @![iOS]phoneCapability!@@![android, javaSE, javaEE]SystemCapabilityType.PHONE_CALL!@ | Describes the built-in phone calling capabilities of the IVI system. | Check @![iOS]SDLPhoneCapability.h!@@![android, javaSE, javaEE]PhoneCapability.java!@ for more information |
| @![iOS]videoStreamingCapability!@@![android, javaSE, javaEE]SystemCapabilityType.VIDEO_STREAMING!@ | Describes the abilities of the head unit to video stream projection applications. | Check @![iOS]SDLVideoStreamingCapability.h!@@![android, javaSE, javaEE]VideoStreamingCapability.java!@ for more information |
| @![iOS]remoteControlCapability!@@![android, javaSE, javaEE]SystemCapabilityType.REMOTE_CONTROL!@ | Describes the abilities of an app to control built-in aspects of the IVI system. | Check @![iOS]SDLRemoteControlCapabilities.h!@@![android, javaSE, javaEE]RemoteControlCapabilities.java!@ for more information |

### Deprecated Properties
The following properties are deprecated on SDL @![iOS]iOS 6.4!@@![android, javaSE, javaEE]Android 4.10!@ because as of RPC v6.0 they are deprecated. However, these properties will still be filled with information. When connected on RPC <6.0, the information will be exactly the same as what is returned in the `RegisterAppInterfaceResponse` and `SetDisplayLayoutResponse`. However, if connected on RPC >6.0, the information will be converted from the newer-style display information, which means that some information will not be available.

| @![iOS]displayCapabilities!@@![android, javaSE, javaEE]SystemCapabilityType.DISPLAY!@ | Information about the HMI display. This includes information about available templates, whether or not graphics are supported, and a list of all text fields and the max number of characters allowed in each text field. | Check @![iOS]SDLDisplayCapabilities.h!@@![android, javaSE, javaEE]DisplayCapabilities.java!@ for more information |
| @![iOS]buttonCapabilities!@@![android, javaSE, javaEE]SystemCapabilityType.BUTTON!@ | A list of available buttons and whether the buttons support long, short and up-down presses. | Check @![iOS]SDLButtonCapabilities.h!@@![android, javaSE, javaEE]ButtonCapabilities.java!@ for more information |
| @![iOS]softButtonCapabilities!@@![android, javaSE, javaEE]SystemCapabilityType.SOFTBUTTON!@ | A list of available soft buttons and whether the button support images. Also, information about whether the button supports long, short and up-down presses. | Check @![iOS]SDLSoftButtonCapabilities.h!@@![android, javaSE, javaEE]SoftButtonCapabilities.java!@ for more information |
| @![iOS]presetBankCapabilities!@@![android, javaSE, javaEE]SystemCapabilityType.PRESET_BANK!@ | If returned, the platform supports custom on-screen presets. | Check @![iOS]SDLPresetBankCapabilities.h!@@![android, javaSE, javaEE]PresetBankCapabilities.java!@ for more information |

## The Register App Interface Response (RAIR) RPC
The `RegisterAppInterface` response contains information about the display type, the type of images supported, the number of text fields supported, the HMI display language, and a lot of other useful properties. The table below has a list of properties beyond those available on the system capability manager returned by the `RegisterAppInterface` response. Each property is optional, so you may not get data for all the parameters in the following table.

| Parameters  |  Description | Notes |
| ------------- | ------------- |------------- |
| language | The currently active VR+TTS language on the module. | Check @![iOS]SDLLanguage.h!@@![android, javaSE, javaEE]Language.java!@ for more information |
| vehicleType | The make, model, year, and the trim of the vehicle. | Check @![iOS]SDLVehicleType.h!@@![android, javaSE, javaEE]VehicleType.java!@ for more information |
| supportedDiagModes | Specifies the white-list of supported diagnostic modes (0x00-0xFF) capable for DiagnosticMessage requests. If a mode outside this list is requested, it will be rejected. | Check @![iOS]SDLDiagnosticMessage.h!@@![android, javaSE, javaEE]DiagnosticMessage.java!@ for more information |
| sdlVersion | The version of SmartDeviceLink core running on the connected head unit. | String |
| systemSoftwareVersion | The software version of the connected system that implements SmartDeviceLink core. | String |

### Image Specifics
#### Image File Types
Images may be formatted as PNG, JPEG, or BMP. You can find which image types are supported in @![iOS]`SDLManager.systemCapabilityManager.defaultMainWindowCapability.imageTypeSupported`!@@![android, javaSE, javaEE]`// TODO`!@.

#### Image Sizes
If an image is uploaded that is larger than the supported size, that image will be scaled down by Core. All image sizes are available from the @![iOS]`SDLManager.systemCapabilityManager.defaultMainWindowCapability.imageFields[<#item#>].imageResolution`!@@![android, javaSE, javaEE]`// TODO`!@ property once the manager has started successfully.

##### Example Image Sizes
Below is a table with example image sizes. Check the `SystemCapabilityManager` for the exact image sizes desired by the system you are connecting to. The connected system should be able to scale down larger sizes, but if the image you are sending is much larger than desired, then performance will be impacted.

| ImageName | Used in RPC | Details | Height | Width | Type |
|:--------------|:----------------|:--------|:---------|:-------|:-------|
softButtonImage		 | Show 					  | Image shown on softbuttons on the base screen | 70px | 70px | png, jpg, bmp
choiceImage 		 | CreateInteractionChoiceSet | Image shown in the manual part of an performInteraction either big (ICON_ONLY) or small (LIST_ONLY) | 70px | 70px | png, jpg, bmp
choiceSecondaryImage | CreateInteractionChoiceSet | Image shown on the right side of an entry in (LIST_ONLY) performInteraction	| 35px | 35px | png, jpg, bmp
vrHelpItem			 | SetGlobalProperties		  | Image shown during voice interaction | 35px | 35px | png, jpg, bmp
menuIcon			 | SetGlobalProperties		  | Image shown on the “More…” button | 35px | 35px | png, jpg, bmp
cmdIcon				 | AddCommand				  | Image shown for commands in the "More…" menu | 35px | 35px | png, jpg, bmp
appIcon 			 | SetAppIcon				  | Image shown as Icon in the "Mobile Apps" menu | 70px | 70px | png, jpg, bmp
graphic 			 | Show 					  | Image shown on the basescreen as cover art | 185px | 185px | png, jpg, bmp

## System Capabilities
Most head units provide features that your app can use: making and receiving phone calls, an embedded navigation system, video and audio streaming, as well as supporting app services. To find out if the head unit supports a feature as well as more information about the feature, use the @![iOS]`SDLSystemCapabilityManager`!@@![android, javaSE, javaEE]`SystemCapabilityManager`!@ to query the head unit for the desired capability. Querying for capabilities is only available on head units supporting RPC v4.5 or greater (except for DISPLAYS, which is backward compatible to SDL 1.0); if connecting to older head units, the query will return @![iOS]`nil`!@@![android, javaSE, javaEE]`null`!@.

### Querying for System Capabilities
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

### Subscribing to System Capability Updates
In addition getting the current system capabilities, it is also possible to subscribe for updates when the head unit capabilities change. @![iOS]To get these notifications you must register using a `subscribeToCapabilityType:` method.!@@![android, javaSE, javaEE]Since this information must be queried from Core you must implement the `OnSystemCapabilityListener`.!@ This feature is only available on head units supporting RPC v.5.1 or greater (except for DISPLAYS, which is backward compatible to RPC v1.0).

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
