# Adaptive Interface Capabilities
## Designing for Different Head Units
Since each car manufacturer has different user interface style guidelines, the number of lines of text, soft and hard buttons, and images supported will vary between different types of head units. When the app first connects to the SDL Core, a `RegisterAppInterface` RPC will be sent by the SDL Core containing the `displayCapability`, `buttonCapabilites`, etc., properties. You can use this information to determine how to lay out the user interface. 

You may access these properties on the `SDLManager.systemCapabilityManager` instance as of SDL iOS library 6.0. More advanced capabilities, such as the `SDLRemoteControlCapability` must be updated through the `systemCapabilityManager`.

## System Capability Manager Properties
The `SystemCapabilityManager` is a new feature available as of version 6.0. If using previous versions of the library,  you can find most of the `SystemCapabilityManager` properties in the `SDLRegisterAppInterfaceResponse` object. You will have to manually extract the desired capability from the `SDLManager.registerResponse` property. 

| Parameters  |  Description | Notes |
| ------------- | ------------- |------------- |
| displayCapabilities | Information about the Sync display. This includes information about available templates, whether or not graphics are supported, and a list of all text fields and the max number of characters allowed in each text field. | Check SDLDisplayCapabilities.h for more information |
| buttonCapabilities | A list of available buttons and whether the buttons support long, short and up-down presses. | Check SDLButtonCapabilities.h for more information |
| softButtonCapabilities | A list of available soft buttons and whether the button support images. Also information about whether the button supports long, short and up-down presses. | Check SDLSoftButtonCapabilities.h for more information |
| presetBankCapabilities | If returned, the platform supports custom on-screen presets. | Check SDLPresetBankCapabilities.h for more information |
| hmiZoneCapabilities | Specifies HMI Zones in the vehicle. There may be a HMI available for back seat passengers as well as front seat passengers. | Check SDLHMIZoneCapabilities.h for more information |
| speechCapabilities | Contains information about TTS capabilities on the SDL platform. Platforms may support text, SAPI phonemes, LH PLUS phonemes, pre-recorded speech, and silence. | Check SDLSpeechCapabilities.h for more information |
| prerecordedSpeechCapabilities | A list of pre-recorded sounds you can use in your app. Sounds may include a help, initial, listen, positive, or a negative jingle. | Check SDLPrerecordedSpeech.h for more information |
| vrCapability | The voice-recognition capabilities of the connected SDL platform. The platform may be able to recognize spoken text in the current language. | Check SDLVRCapabilities.h for more information |
| audioPassThruCapabilities | Describes the sampling rate, bits per sample, and audio types available. | Check SDLAudioPassThruCapabilities.h for more information|
| hmiCapabilities | Returns whether or not the app can support built-in navigation and phone calls. | Check SDLHMICapabilities.h for more information |
| appServicesCapabilities | Describes the capabilities of app services including what service types are supported and the current state of services | Check SDLAppServicesCapabilities.h for more information |
| navigationCapability | Describes the built-in vehicle navigation system's APIs | Check SDLNavigationCapability.h for more information |
| phoneCapability | Describes the built-in phone calling capabilities of the IVI system. | Check SDLPhoneCapability.h for more information |
| videoStreamingCapability | Describes the abilities of the head unit to video stream projection applications | Check SDLVideoStreamingCapability.h for more information |
| remoteControlCapability | Describes the abilities of an app to control built-in aspects of the IVI system | Check SDLRemoteControlCapability.h for more information |

### The Register App Interface RPC
The `RegisterAppInterface` response contains information about the display type, the type of images supported, the number of text fields supported, the HMI display language, and a lot of other useful properties. The table below has a list of properties beyond those available on the `SystemCapabilityManager` returned by the `RegisterAppInterface` response. Each property is optional, so you may not get information for all the parameters in the table.

| Parameters  |  Description | Notes |
| ------------- | ------------- |------------- |
| syncMsgVersion | Specifies the version number of the SDL V4 interface. This is used by both the application and SDL to declare what interface version each is using. | Check SDLSyncMsgVersion.h for more information |
| language | The currently active voice-recognition and text-to-speech language on Sync. | Check SDLLanguage.h for more information |
| vehicleType | The make, model, year, and the trim of the vehicle. | Check SDLVehicleType.h for more information |
| supportedDiagModes | Specifies the white-list of supported diagnostic modes (0x00-0xFF) capable for DiagnosticMessage requests. If a mode outside this list is requested, it will be rejected. | Check SDLDiagnosticMessage.h for more information |
| sdlVersion | The SmartDeviceLink version | String |
| systemSoftwareVersion | The software version of the system that implements the SmartDeviceLink core | String |

### System Capabilities
Most head units provide features that your app can use: making and receiving phone calls, an embedded navigation system, video and audio streaming, as well as supporting app services. To find out if the head unit supports a feature as well as more information about the feature, use the `SystemCapabilityManager` to query the head unit for the desired capability. Querying for capabilities is only availble on head units supporting v.4.5 or greater; if connecting to older head units, the query will return `nil` even if the head unit may support the capabilty.

##### Objective-C
```objc
[sdlManager.systemCapabilityManager updateCapabilityType:SDLSystemCapabilityTypeVideoStreaming completionHandler:^(NSError * _Nullable error, SDLSystemCapabilityManager * _Nonnull systemCapabilityManager) {
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

#### Subscribing to Updates to System Capabilities
In addition getting the current system capbilities it is also possible to register to get updates when the head unit capabilities change. To get these notifications you must register for the `SDLDidReceiveSystemCapabilityUpdatedNotification` notification. This feature is only availble on head units supporting v.5.2 or greater.

##### Objective-C
```objc
[NSNotificationCenter.defaultCenter addObserver:self selector:@selector(systemCapabilityUpdatedNotification:) name:SDLDidReceiveSystemCapabilityUpdatedNotification object:nil];

- (void)systemCapabilityUpdatedNotification:(SDLRPCNotificationNotification *)notification {
    SDLOnSystemCapabilityUpdated *capability = (SDLOnSystemCapabilityUpdated *)notification.notification;
    SDLSystemCapability *systemCapability = capability.systemCapability;
    <#Use the system capability#>
}
```

##### Swift
```swift
NotificationCenter.default.addObserver(self, selector: #selector(systemCapabilityUpdatedNotification(_:)), name: .SDLDidReceiveSystemCapabilityUpdated, object: nil)

@objc func systemCapabilityUpdatedNotification(_ notification: SDLRPCNotificationNotification) {
    guard let capability = notification.notification as? SDLOnSystemCapabilityUpdated else {
        return
    }

    let systemCapability = capability.systemCapability
    <#Use the system capability#>
}
```


## Image Specifics
### Image File Type
Images may be formatted as PNG, JPEG, or BMP. Check the `RegisterAppInterfaceResponse.displayCapability` properties to find out what image formats the head unit supports.

### Image Sizes
If an image is uploaded that is larger than the supported size, that image will be scaled down to accommodate. All image sizes are available from the `SDLManager`'s `registerResponse` property once in the completion handler for `startWithReadyHandler`.

| ImageName | Used in RPC | Details | Height | Width | Type |
|:--------------|:----------------|:--------|:---------|:-------|:-------|
softButtonImage		 | Show 					  | Will be shown on softbuttons on the base screen														  | 70px         | 70px   | png, jpg, bmp
choiceImage 		 | CreateInteractionChoiceSet | Will be shown in the manual part of an performInteraction either big (ICON_ONLY) or small (LIST_ONLY) | 70px         | 70px   | png, jpg, bmp
choiceSecondaryImage | CreateInteractionChoiceSet | Will be shown on the right side of an entry in (LIST_ONLY) performInteraction						  | 35px 		 | 35px   | png, jpg, bmp
vrHelpItem			 | SetGlobalProperties		  | Will be shown during voice interaction 																  | 35px 		 | 35px   | png, jpg, bmp
menuIcon			 | SetGlobalProperties		  | Will be shown on the “More…” button 																  | 35px 		 | 35px   | png, jpg, bmp
cmdIcon				 | AddCommand				  | Will be shown for commands in the "More…" menu 														  | 35px 		 | 35px   | png, jpg, bmp
appIcon 			 | SetAppIcon				  | Will be shown as Icon in the "Mobile Apps" menu 													  | 70px 		 | 70px   | png, jpg, bmp
graphic 			 | Show 					  | Will be shown on the basescreen as cover art 														  | 185px 		 | 185px  | png, jpg, bmp