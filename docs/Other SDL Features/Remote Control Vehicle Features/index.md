# Remote Control Vehicle Features
The remote control framework allows apps to control certain modules, such as climate, radio, seat, lights, etc., within a vehicle. Vehicles running RPC v6.0+ may have multiple modules of the same type and each module can be controlled separately depending on seat location and permissions.

!!! Note
Not all head units support this feature. If you are using this feature in your app you will most likely need to request permission from the vehicle manufacturer.
!!!

## Why Use Remote Control?
Consider the following scenarios:

- A radio application wants to use the in-vehicle radio tuner. It needs the functionality to select the radio band (AM/FM/XM/HD/DAB), tune the radio frequency or change the radio station, as well as obtain general radio information for decision making.
- A climate control application needs to turn on the AC, control the air circulation mode, change the fan speed and set the desired cabin temperature.
- A user profile application wants to remember users' favorite settings and apply it later automatically when the users get into the same/another vehicle.

### Remote Control Modules
Currently, the remote control feature supports these modules:

| Remote Control Modules | Version  |
| ---------              | -------  |
| Climate                | SDL v4.5 |
| Radio                  | SDL v4.5 |
| Seat                   | SDL v5.0 |
| Audio                  | SDL v5.0 |
| Light                  | SDL v5.0 |
| HMI Settings           | SDL v5.0 |

The following table lists what control items are in each control module.

#### Climate

| Control Item | RPC Item Name | Value Range | Type | Comments | Version Changes |
| --------------- | ------------ | ------------ | ------------ | ------------ | ------------ |
| Climate Enable | climateEnable | on, off | Get/Set/Notification | Enabled to turn on the climate system, Disabled to turn off the climate system. All other climate items need climate enabled to work. | Since SDL v6.0 |
| Current Cabin Temperature | currentTemperature | N/A | Get/Notification | Read only, value range depends on OEM | Since SDL v4.5 |
| Desired Cabin Temperature | desiredTemperature | N/A | Get/Set/Notification | Value range depends on OEM | Since SDL v4.5 |
| AC Setting | acEnable | on, off | Get/Set/Notification |  | Since SDL v4.5 |
| AC MAX Setting | acMaxEnable | on, off  | Get/Set/Notification |  | Since SDL v4.5 |
| Air Recirculation Setting | circulateAirEnable | on, off  | Get/Set/Notification |  | Since SDL v4.5 |
| Auto AC Mode Setting | autoModeEnable | on, off | Get/Set/Notification |  | Since SDL v4.5 |
| Defrost Zone Setting | defrostZone | front, rear, all, none  | Get/Set/Notification |  | Since SDL v4.5 |
| Dual Mode Setting | dualModeEnable | on, off  | Get/Set/Notification |  | Since SDL v4.5 |
| Fan Speed Setting | fanSpeed | 0%-100% | Get/Set/Notification |  | Since SDL v4.5 |
| Ventilation Mode Setting | ventilationMode | upper, lower, both, none | Get/Set/Notification |  | Since SDL v4.5 |
| Heated Steering Wheel Enabled | heatedSteeringWheelEnable | on, off | Get/Set/Notification | | Since SDL v5.0 |
| Heated Windshield Enabled | heatedWindshieldEnable | on, off | Get/Set/Notification | | Since SDL v5.0 |
| Heated Rear Window Enabled | heatedRearWindowEnable | on, off | Get/Set/Notification | | Since SDL v5.0 |
| Heated Mirrors Enabled | heatedMirrorsEnable | on, off | Get/Set/Notification | | Since SDL v5.0 |

#### Radio

| Control Item | RPC Item Name | Value Range | Type | Comments | Version Changes |
| --------------- | ------------ | ------------ | ------------ | ------------ | ------------ |
| Radio Enabled | radioEnable | true,false  | Get/Set/Notification | Read only, all other radio control items need radio enabled to work | Since SDL v4.5 |
| Radio Band | band | AM,FM,XM  | Get/Set/Notification | | Since SDL v4.5 |
| Radio Frequency | frequencyInteger / frequencyFraction | 0-1710, 0-9 | Get/Set/Notification | Value range depends on band | Since SDL v4.5 |
| Radio RDS Data | rdsData | RdsData struct | Get/Notification | Read only | Since SDL v4.5 |
| Available HD Channels | availableHdChannels | Array size 0-8, values 0-7 | Get/Notification | Read only | Since SDL v6.0, replaces availableHDs |
| Available HD Channels (DEPRECATED) | availableHDs | 1-7 (Deprecated in SDL v.6.0) (1-3 before SDL v.5.0) | Get/Notification | Read only | Since SDL v4.5, updated in v5.0, deprecated in v6.0 |
| Current HD Channel | hdChannel | 0-7 (1-3 before SDL v.5.0) (1-7 between SDL v.5.0-6.0) | Get/Set/Notification |  | Since SDL v4.5, updated in SDL v5.0, updated in SDL v6.0 |
| Radio Signal Strength | signalStrength | 0-100% | Get/Notification | Read only | Since SDL v4.5 |
| Signal Change Threshold | signalStrengthThreshold | 0-100% | Get/Notification | read only | Since SDL v4.5 |
| Radio State | state | Acquiring, acquired, multicast, not_found | Get/Notification | Read only | Since SDL v4.5 |
| SIS Data | sisData | See SisData struct | Get/Notification | Read only | Since SDL v5.0 |

#### Seat

| Control Item | RPC Item Name | Value Range | Type | Comments | Version Changes |
| --------------- | ------------ | ------------ | ------------ | ------------ | ------------ |
| Seat Heating Enabled | heatingEnabled | true, false | Get/Set/Notification | Indicates whether heating is enabled for a seat | Since SDL v5.0 |
| Seat Cooling Enabled | coolingEnabled | true, false | Get/Set/Notification | Indicates whether cooling is enabled for a seat | Since SDL v5.0 |
| Seat Heating level | heatingLevel | 0-100% | Get/Set/Notification | Level of the seat heating | Since SDL v5.0 |
| Seat Cooling level | coolingLevel | 0-100% | Get/Set/Notification | Level of the seat cooling | Since SDL v5.0 |
| Seat Horizontal Positon | horizontalPosition | 0-100% | Get/Set/Notification | Adjust a seat forward/backward, 0 means the nearest position to the steering wheel, 100% means the furthest position from the steering wheel | Since SDL v5.0 |
| Seat Vertical Position | verticalPosition | 0-100% | Get/Set/Notification | Adjust seat height (up or down) in case there is only one actuator for seat height, 0 means the lowest position, 100% means the highest position| Since SDL v5.0 |
| Seat-Front Vertical Position | frontVerticalPosition | 0-100% | Get/Set/Notification | Adjust seat front height (in case there are two actuators for seat height), 0 means the lowest position, 100% means the highest position | Since SDL v5.0 |
| Seat-Back Vertical Position | backVerticalPosition | 0-100% | Get/Set/Notification | Adjust seat back height (in case there are two actuators for seat height), 0 means the lowest position, 100% means the highest position | Since SDL v5.0 |
| Seat Back Tilt Angle | backTiltAngle | 0-100% | Get/Set/Notification | Backrest recline, 0 means the angle that back top is nearest to the steering wheel, 100% means the angle that back top is furthest from the steering wheel | Since SDL v5.0 |
| Head Support Horizontal Positon | headSupportHorizontalPosition | 0-100% | Get/Set/Notification | Adjust head support forward/backward, 0 means the nearest position to the front, 100% means the furthest position from the front | Since SDL v5.0 |
| Head Support Vertical Position | headSupportVerticalPosition | 0-100% | Get/Set/Notification | Adjust head support height (up or down), 0 means the lowest position, 100% means the highest position | Since SDL v5.0 |
| Seat Massaging Enabled | massageEnabled | true, false | Get/Set/Notification | Indicates whether massage is enabled for a seat | Since SDL v5.0 |
| Massage Mode | massageMode | MassageModeData struct | Get/Set/Notification | List of massage mode of each zone | Since SDL v5.0 |
| Massage Cushion Firmness | massageCushionFirmness | MassageCushionFirmness struct | Get/Set/Notification | List of firmness of each massage cushion | Since SDL v5.0 |
| Seat memory | memory | SeatMemoryAction struct | Get/Set/Notification | Seat memory | Since SDL v5.0 |

#### Audio

|Control Item | RPC Item Name | Value Range | Type | Comments | Version Changes |
| --------------- | ------------ | ------------ | ------------ | ------------ | ------------ |
| Audio Volume | volume | 0%-100% | Get/Set/Notification | The audio source volume level | Since SDL v5.0 |
| Audio Source | source | PrimaryAudioSource enum | Get/Set/Notification | Defines one of the available audio sources | Since SDL v5.0 |
| Keep Context | keepContext | true, false | Set only | Controls whether the HMI will keep the current application context or switch to the default media UI/APP associated with the audio source | Since SDL v5.0 |
| Equilizer Settings | equalizerSettings | EqualizerSettings struct | Get/Set/Notification | Defines the list of supported channels (band) and their current/desired settings on HMI | Since SDL v5.0 |

#### Light

| Control Item | RPC Item Name | Value Range | Type | Comments | Version Changes |
| --------------- | ------------ | ------------ | ------------ | ------------ | ------------ |
| Light State | lightState | Array of LightState struct | Get/Set/Notification | | Since SDL v5.0 |

#### HMI Settings

| Control Item | RPC Item Name | Value Range | Type | Comments | Version Changes |
| --------------- | ------------ | ------------ | ------------ | ------------ | ------------ |
| Display Mode | displayMode | Day, Night, Auto | Get/Set/Notification | Current display mode of the HMI display | Since SDL v5.0 |
| Distance Unit | distanceUnit | Miles, Kilometers | Get/Set/Notification | Distance Unit used in the HMI (for maps/tracking distances) | Since SDL v5.0 |
| Temperature Unit | temperatureUnit | Fahrenheit, Celsius | Get/Set/Notification | Temperature Unit used in the HMI (for temperature measuring systems) | Since SDL v5.0 |

### Remote Control Button Presses
The remote control framework also allows mobile applications to send simulated button press events for the following common buttons in the vehicle.

| RC Module | Control Button |
| ------------ | ------------ |
| **Climate** | AC |
|             | AC MAX |
|             | RECIRCULATE |
|             | FAN UP |
|             | FAN DOWN |
|             | TEMPERATURE UP |
|             | TEMPERATURE DOWN |
|             | DEFROST |
|             | DEFROST REAR |
|             | DEFROST MAX |
|             | UPPER VENT |
|             | LOWER VENT |
| **Radio**   | VOLUME UP |
|             | VOLUME DOWN |
|             | EJECT |
|             | SOURCE |
|             | SHUFFLE |
|             | REPEAT |

## Integration
For remote control to work, the head unit must support SDL Core v.4.4 or newer. Also your app's @![iOS]`appType`!@ @![android, javaSE, javaEE]`appHMIType`!@ must include `REMOTE_CONTROL`.

!!! Note
When connected to pre-Core 6.0 systems only the main module of a module type can be controlled. The modules you may control will depend on the OEM.
!!!

### Multiple Modules (RPC v6.0+)
Multiple modules can exist for each module type starting in SDL v6.0. In previous SDL versions, only one module was available for each module type. A new struct `moduleInfo` (on the `XYZControlCapabilities` objects) was created to give developers the information they need to control a specific module and contains a unique `moduleId` for each module. When sending remote control RPCs to a v6.0+, the `moduleId` must be provided to control the desired module. If no `moduleId` is set, the HMI will use the default module of that module type. When connected to pre-6.0 systems, the `moduleInfo` struct will be @![iOS]`nil`!@@![android, javaSE, javaEE]`null`!@.

The permissions to control a module is seat-based. Depending on which seat the user is sitting in they may or may not be able to control certain modules. For example, only the person sitting in the front passenger seat can control the front passenger seat module. Modules may also allow multiple users to access them, while some modules may only allow one user at a time. Access control for a module will depend on the OEM.

### Getting Remote Control Module Information
Prior to using any remote control RPCs, you must check that the head unit has the remote control capability. As you will encounter head units that do *not* support remote control, or head units that do not give your application permission to read and write remote control data, this check is important.

When connected to Core v6.0+, you will want to save this information for future use. The `moduleId` contained within the `moduleInfo` struct on each capability when connected to a v6.0+ system is necessary to control that module. If connected to RPC <6.0, you should still get remote control module data to know what properties are available.

@![iOS]
##### Objective-C
```objc
[self.sdlManager.systemCapabilityManager subscribeToCapabilityType:SDLSystemCapabilityTypeRemoteControl withBlock:^(SDLSystemCapability * _Nonnull capability) {
    if(!capability.remoteControlCapability) { return; }
    <#Save Remote Capabilities#>
}];
```

##### Swift
```swift
sdlManager.systemCapabilityManager.subscribe(toCapabilityType: .remoteControl, with: { capability in
    guard (capability.remoteControlCapability != nil) else { return }
    <#Save Remote Capabilities#>
})

```
!@

@![java,javaEE,javaSE]
```java
// ToDo - Add example
```
!@

#### Getting Module Data Location and Service Areas (RPC 6.0+ Only)
With the saved remote control capabilities struct you can build a UI to display modules to the user by getting the location of the module and the area that it services. This will map to the grid you receive in `Setting the User's Seat` below.

!!! Note
This data is only available when connected to SDL 6.0+ systems. On previous systems, only one module per module type was available, so its location didn't matter. You won't be able to build a custom UI for those cases and should use a generic UI instead.
!!!

@![iOS]
##### Objective-C
```objc
// Get the first climate module's information
SDLClimateControlCapabilities *firstClimateModule = <#Remote Control Capabilities#>.climateControlCapabilities.firstObject;

NSString *climateModuleId = firstClimateModule.moduleInfo.moduleId;
SDLGrid *climateModuleLocation = firstClimateModule.moduleInfo.location;
// And so forth
```

##### Swift
```swift
// Get the first climate module's information
SDLClimateControlCapabilities *firstClimateModule = <#Remote Control Capabilities#>.climateControlCapabilities.first;

let climateModuleId = firstClimateModule.moduleInfo.moduleId;
let climateModuleLocation = firstClimateModule.moduleInfo.location;
// And so forth
```
!@

@![java,javaEE,javaSE]
```java
// ToDo - Add example
```
!@

### Setting The User's Seat (RPC 6.0+ only)
The first step before attempting to take control of any module is to have the user select their seat location. Seat location may affect which modules the user is permitted to control depending on the OEMs rules. The default seat location is `Driver`. Seat location can be updated by setting the `userLocation` property in the @![iOS]`SDLSetGlobalProperties`!@@![android, javaSE, javaEE]`SetGlobalProperties`!@ RPC and sending it.

You may wish to show the user a map or list of all available seats in your app in order to ask them where they are located. This example is only meant to show you how to access the available data, and not how to build your UI/UX. 

An array of seats can be found in the `seatLocationCapability`'s `seat` array. Each `SeatLocation` object within the `seats` array will have a `grid` parameter. The `grid` will tell you the seat placement of that particular seat. This information can be very useful for creating a map or list for users to select from.

@![iOS]
##### Objective-C
```objc
[self.sdlManager.systemCapabilityManager subscribeToCapabilityType:SDLSystemCapabilityTypeSeatLocation withBlock:^(SDLSystemCapability * _Nonnull capability) {
    if (!capability) { return; }
    NSArray<SDLSeatLocation *> *seats = capability.seatLocationCapability.seats;

    <#Save Remote Capabilities#>
}];
```

##### Swift
```swift
sdlManager.systemCapabilityManager.subscribe(toCapabilityType: .seatLocation, with: { capability in
    guard let seats = capability.seatLocationCapability?.seats else { return }

    <#Save Remote Capabilities#>
})
```
!@

@![java,javaEE,javaSE]
```java
// ToDo - Add example
```
!@

The `grid` system starts with the front left corner of the bottom level of the vehicle being `(0,0,0)`. For example, assuming a United States vehicle, a `grid` of `col=0`, `row=0` and `level=0` could be the drivers' seat. A `col=2`, `row=0` and `level=0` could be referring to the front right passenger location, assuming the car has 3 columns. A negative `col` or `row` means it is outside the vehicle. The `colspan` and `rowspan` properties tell you how many rows and columns that module or seat takes up.

![Car](assets/Car.png)

|         | col=0   | col=1   | col=2   |
| ---     | ---     | ---     | ---     |
| row=0   | driver's seat: {col=0, row=0, level=0, colspan=1, rowspan=1, levelspan=1} |   | front passenger's seat : {col=2, row=0, level=0, colspan=1, rowspan=1, levelspan=1} |
| row=1   | rear-left seat : {col=0, row=1, level=0, colspan=1, rowspan=1, levelspan=1} | rear-middle seat :  {col=1, row=1, level=0, colspan=1, rowspan=1, levelspan=1} | rear-right seat : {col=2, row=1, level=0, colspan=1, rowspan=1, levelspan=1} |

As noted above, when the user selects their seat, you can send an @![iOS]`SDLSetGlobalProperties`!@@![android, javaSE, javaEE]`SetGlobalProperties`!@ RPC and set the `userLocation` property in order to set that user's location within the vehicle.

@![iOS]
##### Objective-C
```objc
SDLSetGlobalProperties *seatLocation = [[SDLSetGlobalProperties alloc] init];
seatLocation.userLocation = <#Selected Seat#>;
[self.sdlManager sendRequest:seatLocation withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if(!response.success) { return; }
    <#Seat Data Updated#>
}];
```

##### Swift
```swift
let seatLocation = SDLSetGlobalProperties()
seatLocation.userLocation = <#Selected Seat#>;
sdlManager.send(request: setData, responseHandler: { (request, response, error) in
    guard response?.success.boolValue == true else { return }
    <#Seat Data Updated#>
})
```
!@

@![java,javaEE,javaSE]
```java
// ToDo - Add example
```
!@

### Getting Module Data 
Seat location does not affect the ability to get data from a module. Once you know you have permission to use the remote control feature and you have `moduleId`(s) (when connected to 6.0+ systems), you can retrieve the data for any module. The following code is an example of how to subscribe to the data of a radio module. 

When connected to < 6.0 head units, there can only be one module for each module type (e.g. there can only be one climate module, light module, radio module, etc.), so you will not need to pass a `moduleId`.

#### Subscribing to Module Data
You can either subscribe to module data or receive it one time. If you choose to subscribe to module data you will receive continuous updates on the vehicle data you have subscribed to.

!!! NOTE
Subscribing to the `OnInteriorVehicleData` notification must be done before sending the @![iOS]`SDLGetInteriorVehicleData`!@@![android, javaSE, javaEE]`GetInteriorVehicleData`!@ request.
!!!

@![iOS]
##### Objective-C
```objc
[self.sdlManager subscribeToRPC:SDLDidReceiveInteriorVehicleDataNotification withBlock:^(__kindof SDLRPCMessage * _Nonnull message) {
    SDLRPCNotificationNotification *dataNotification = (SDLRPCNotificationNotification *)note;
    SDLOnInteriorVehicleData *onInteriorVehicleData = (SDLOnInteriorVehicleData *)dataNotification.notification;
    if (onInteriorVehicleData == nil) { return; }

    // This block will now be called whenever vehicle data changes
    // NOTE: If you subscibe to multiple modules, all the data will be sent here. You will have to split it out based on `onInteriorVehicleData.moduleData.moduleType` yourself.
    <#Code#>
}];
```

After you subscribe to the `SDLDidReceiveInteriorVehicleDataNotification` you must also subscribe to the module you wish to receive updates for. Subscribing to a module will send a notification when that particular module is changed.

###### RPC < v6.0
```objc
SDLGetInteriorVehicleData *getInteriorVehicleData = [[SDLGetInteriorVehicleData alloc] initAndSubscribeToModuleType:SDLModuleTypeRadio];
[self.sdlManager sendRequest:getInteriorVehicleData withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    SDLGetInteriorVehicleDataResponse *dataResponse = (SDLGetInteriorVehicleDataResponse *)response;
    // This can now be used to retrieve data
    <#Code#>
}];
```

###### RPC v6.0+
```objc
SDLGetInteriorVehicleData *getInteriorVehicleData = [[SDLGetInteriorVehicleData alloc] initAndSubscribeToModuleType:SDLModuleTypeRadio moduleId:@"<#ModuleID#>"];
[self.sdlManager sendRequest:getInteriorVehicleData withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    SDLGetInteriorVehicleDataResponse *dataResponse = (SDLGetInteriorVehicleDataResponse *)response;
    // This can now be used to retrieve data
    <#Code#>
}];
```

##### Swift
```swift
sdlManager.subscribe(to: .SDLDidReceiveInteriorVehicleData) { (message) in
    let onInteriorVehicleData = message as? SDLOnInteriorVehicleData else { return }

    // This block will now be called whenever vehicle data changes
    // NOTE: If you subscribe to multiple modules, all the data will be sent here. You will have to split it out based on `onInteriorVehicleData.moduleData.moduleType` yourself.
    <#Code#>
}
```

After you subscribe to the `.SDLDidReceiveInteriorVehicleData` you must also subscribe to the module you wish to receive updates for. Subscribing to a module will send a notification when that particular module is changed.

###### RPC < v6.0
```swift
let getInteriorVehicleData = SDLGetInteriorVehicleData(andSubscribeToModuleType: .radio)
sdlManager.send(request: getInteriorVehicleData) { (req, res, err) in
    guard let response = res as? SDLGetInteriorVehicleDataResponse else { return }
    // This can now be used to retrieve data
    <#Code#>
}
```

###### RPC v6.0+
```swift
let getInteriorVehicleData = SDLGetInteriorVehicleData(andSubscribeToModuleType: .radio, moduleId: "<#ModuleID#>")
sdlManager.send(request: getInteriorVehicleData) { (req, res, err) in
    guard let response = res as? SDLGetInteriorVehicleDataResponse else { return }
    // This can now be used to retrieve data
    <#Code#>
}
```
!@

@![android, javaSE, javaEE]
```java
// TODO: Update

sdlManager.addOnRPCNotificationListener(FunctionID.ON_INTERIOR_VEHICLE_DATA, new OnRPCNotificationListener() {
    @Override
    public void onNotified(RPCNotification notification) {
        OnInteriorVehicleData onInteriorVehicleData = (OnInteriorVehicleData) notification;
        // Perform action based on notification
    }
});
```
```java
GetInteriorVehicleData interiorVehicleData = new GetInteriorVehicleData(ModuleType.RADIO);
interiorVehicleData.setSubscribe(true);
interiorVehicleData.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse response) {
        GetInteriorVehicleDataResponse getResponse = (GetInteriorVehicleDataResponse) response;
        // This can now be used to retrieve data
    }
    @Override
    public void onError(int correlationId, Result resultCode, String info){
        Log.e(TAG, "onError: "+ resultCode+ " | Info: "+ info );
    }
});

sdlManager.sendRPC(interiorVehicleData);
```
!@

#### Getting One-Time Data
To get data from a module without subscribing send a @![iOS]`SDLGetInteriorVehicleData`!@@![android, javaSE, javaEE]`GetInteriorVehicleData`!@ request with the `subscribe` flag set to `false`.

@![iOS]
##### Objective-C
###### RPC < v6.0
```objc
SDLGetInteriorVehicleData *getInteriorVehicleData = [SDLGetInteriorVehicleData alloc] initWithModuleType:SDLModuleTypeRadio];
[self.sdlManager sendRequest:getInteriorVehicleData withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    SDLGetInteriorVehicleDataResponse *dataResponse = (SDLGetInteriorVehicleDataResponse *)response;
    // This can now be used to retrieve data
}];
```

###### RPC v6.0+
```objc
SDLGetInteriorVehicleData *getInteriorVehicleData = [SDLGetInteriorVehicleData alloc] initWithModuleType:SDLModuleTypeRadio moduleId:<#ModuleID#>];
[self.sdlManager sendRequest:getInteriorVehicleData withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    SDLGetInteriorVehicleDataResponse *dataResponse = (SDLGetInteriorVehicleDataResponse *)response;
    // This can now be used to retrieve data
}];
```

##### Swift
###### RPC < v6.0
```swift
let getInteriorVehicleData = SDLGetInteriorVehicleData(moduleType: .radio)
sdlManager.send(request: getInteriorVehicleData) { (req, res, err) in
    guard let response = res as? SDLGetInteriorVehicleDataResponse else { return }
    // This can now be used to retrieve data
    <#Code#>
}
```

###### RPC v6.0+
```objc
let getInteriorVehicleData =  SDLGetInteriorVehicleData(moduleType: .radio, moduleId: <#ModuleID#>)
sdlManager.send(request: getInteriorVehicleData) { (req, res, err) in
    guard let response = res as? SDLGetInteriorVehicleDataResponse else { return }
    // This can now be used to retrieve data
    <#Code#>
}
```
!@

@![android, javaEE, javaSE]
```java
// ToDo - Add example
```
!@

### Setting Module Data

Not only do you have the ability to get data from these modules, but, if you have the right permissions, you can also set module data.

#### Getting Consent to Control a Module (6.0+ Only)
Some OEMs may wish to ask the driver for consent before a user can control a module. The @![iOS]`SDLGetInteriorVehicleDataConsent`!@@![android, javaSE, javaEE]`GetInteriorVehicleDataConsent`!@ RPC will alert the driver for consent in some OEM head units if the module if not free (another user has control) and `allowMultipleAccess` (multiple users can access/set the data at the same time) is `true`. `allowMultipleAccess` is part of the `moduleInfo` in the module object.

Check the `allowed` property in the @![iOS]`SDLGetInteriorVehicleDataConsentResponse`!@@![android, javaSE, javaEE]`GetInteriorVehicleDataConsentResponse`!@ to see what modules can be controlled. Note that the order of the `allowed` array is 1-1 with the `moduleIds` array you passed into the @![iOS]`SDLGetInteriorVehicleDataConsent`!@@![android, javaSE, javaEE]`GetInteriorVehicleDataConsent`!@ RPC.

!!! Note
You should always try to get consent before setting any module data. If consent is not granted you should not attempt to set any module's data.
!!!

@![iOS]
##### Objective-C
```objc
SDLGetInteriorVehicleDataConsent *getInteriorVehicleDataConsent = [[SDLGetInteriorVehicleDataConsent alloc] initWithModuleType:@"<#ModuleType#>" moduleIds:@[@"<#ModuleID#>", @"<#ModuleID#>", @"<#ModuleID#>"]];
[self.sdlManager sendRequest:getInteriorVehicleDataConsent withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if (!response.success) { return; }
    <#Allowed is an array of true or false values#>
}];
```
##### Swift
```swift
let getInteriorVehicleDataConsent =  SDLGetInteriorVehicleDataConsent(moduleType: "<#ModuleType#>", moduleIds: ["<#ModuleID#>", "<#ModuleID#>", "<#ModuleID#>"])
sdlManager.send(request: getInteriorVehicleDataConsent , responseHandler: { (request, response, error) in
    guard let res = response as? SDLGetInteriorVehicleDataConsentResponse else { return }
    let allowed = res.allowed as! [NSNumber]
    let boolAllowed = allowed.map ({ (bool) -> Bool in
        return bool.boolValue
    })
    <#BoolAllowed is an array of true or false values#>
})
```
!@

@![android, javaEE, javaSE]
```java
// ToDo - Add example
```
!@

#### Controlling a Module
Below is an example of setting climate control data. It is likely that you will not need to set all the data as in the code example below. When connected to 6.0+ system, you must set the `moduleId` in @![iOS]`SDLSetInteriorVehicleData.moduleData`!@@![android, javaSE, javaEE]`SetInteriorVehicleData.setModuleData`!@. When connected to < 6.0 systems, there is only one module per module type, so you must only pass the type of the module you wish to control.

###### Controlling Modules on RPC 6.0+
When you received module information above in "Getting Remote Control Module Information" on 6.0+ systems, you received information on the `location` and `serviceArea` of the module. The permission area of a module depends on that `serviceArea`. The `location` of a module is like the `seats` array: it maps to the `grid` to tell you the physical location of a particular module. The `serviceArea` maps to the grid to show how far that module's scope reaches.

For example, a radio module usually serves all passengers in the vehicle, so its service area will likely cover the entirety of the vehicle grid, while a climate module may only cover a passenger area and not the driver or the back row. If a `serviceArea` is not included, it is assumed that the `serviceArea` is the same as the module's `location`. If neither is included, it is assumed that the `serviceArea` covers the whole area of the vehicle. If a user is not sitting within the `serviceArea`'s `grid`, they will not receive permission to control that module (attempting to set data will fail).

@![iOS]
##### Objective-C
###### RPC < v6.0
```objc
SDLTemperature *temperature = [[SDLTemperature alloc] initWithUnit:SDLTemperatureUnitFahrenheit value:74.1];
SDLClimateControlData *climateControlData = [[SDLClimateControlData alloc] initWithFanSpeed:@2 desiredTemperature:temperature acEnable:@YES circulateAirEnable:@NO autoModeEnable:@NO defrostZone:nil dualModeEnable:@NO acMaxEnable:@NO ventilationMode:SDLVentilationModeLower heatedSteeringWheelEnable:@YES heatedWindshieldEnable:@YES heatedRearWindowEnable:@YES heatedMirrorsEnable:@NO];
SDLModuleData *moduleData = [[SDLModuleData alloc] initWithClimateControlData:climateControlData];
SDLSetInteriorVehicleData *setInteriorVehicleData = [[SDLSetInteriorVehicleData alloc] initWithModuleData:moduleData];
[self.sdlManager sendRequest:setInteriorVehicleData withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if(!response.success) { return; }
}];
```

###### RPC v6.0+
```objc
SDLTemperature *temperature = [[SDLTemperature alloc] initWithUnit:SDLTemperatureUnitFahrenheit value:74.1];
SDLClimateControlData *climateControlData = [[SDLClimateControlData alloc] initWithFanSpeed:@2 desiredTemperature:temperature acEnable:@YES circulateAirEnable:@NO autoModeEnable:@NO defrostZone:nil dualModeEnable:@NO acMaxEnable:@NO ventilationMode:SDLVentilationModeLower heatedSteeringWheelEnable:@YES heatedWindshieldEnable:@YES heatedRearWindowEnable:@YES heatedMirrorsEnable:@NO];
SDLModuleData *moduleData = [[SDLModuleData alloc] initWithClimateControlData:climateControlData];
moduleData.moduleId = <#Module ID#>
SDLSetInteriorVehicleData *setInteriorVehicleData = [[SDLSetInteriorVehicleData alloc] initWithModuleData:moduleData];
[self.sdlManager sendRequest:setInteriorVehicleData withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if(!response.success) { return; }
}];
```

##### Swift
###### RPC < v6.0
```swift
let temperature = SDLTemperature(unit: .fahrenheit, value: 74.1)
let climateControlData = SDLClimateControlData(fanSpeed: 2 as NSNumber, desiredTemperature: temperature, acEnable: true as NSNumber, circulateAirEnable: false as NSNumber, autoModeEnable: false as NSNumber, defrostZone: nil, dualModeEnable: false as NSNumber, acMaxEnable: false as NSNumber, ventilationMode: .lower, heatedSteeringWheelEnable: true as NSNumber, heatedWindshieldEnable: true as NSNumber, heatedRearWindowEnable: true as NSNumber, heatedMirrorsEnable: false as NSNumber)
let moduleData = SDLModuleData(climateControlData: climateControlData)
let setInteriorVehicleData = SDLSetInteriorVehicleData(moduleData: moduleData)

sdlManager.send(request: setInteriorVehicleData) { (request, response, error) in
    guard response?.success.boolValue == true else { return }
}
```

###### RPC v6.0+
```swift
let temperature = SDLTemperature(unit: .fahrenheit, value: 74.1)
let climateControlData = SDLClimateControlData(fanSpeed: 2 as NSNumber, desiredTemperature: temperature, acEnable: true as NSNumber, circulateAirEnable: false as NSNumber, autoModeEnable: false as NSNumber, defrostZone: nil, dualModeEnable: false as NSNumber, acMaxEnable: false as NSNumber, ventilationMode: .lower, heatedSteeringWheelEnable: true as NSNumber, heatedWindshieldEnable: true as NSNumber, heatedRearWindowEnable: true as NSNumber, heatedMirrorsEnable: false as NSNumber)
let moduleData = SDLModuleData(climateControlData: climateControlData)
moduleData.moduleId = <#Module ID#>
let setInteriorVehicleData = SDLSetInteriorVehicleData(moduleData: moduleData)

sdlManager.send(request: setInteriorVehicleData) { (request, response, error) in
    guard response?.success.boolValue == true else { return }
}
```
!@

@![android, javaSE, javaEE]
```java
// ToDo - Updated if needed

Temperature temp = new Temperature(TemperatureUnit.FAHRENHEIT, 74.1f);

ClimateControlData climateControlData = new ClimateControlData();
climateControlData.setAcEnable(true);
climateControlData.setAcMaxEnable(true);
climateControlData.setAutoModeEnable(false);
climateControlData.setCirculateAirEnable(true);
climateControlData.setCurrentTemperature(temp);
climateControlData.setDefrostZone(DefrostZone.FRONT);
climateControlData.setDualModeEnable(true);
climateControlData.setFanSpeed(2);
climateControlData.setVentilationMode(VentilationMode.BOTH);
climateControlData.setDesiredTemperature(temp);

ModuleData moduleData = new ModuleData(ModuleType.CLIMATE);
moduleData.setClimateControlData(climateControlData);

SetInteriorVehicleData setInteriorVehicleData = new SetInteriorVehicleData(moduleData);
sdlManager.sendRPC(setInteriorVehicleData);
```
!@

@![android, javaEE, javaSE]
```java
// ToDo - Add example
```
!@

#### Button Presses
Another unique feature of remote control is the ability to send simulated button presses to the associated modules, imitating a button press on the hardware itself. Simply specify the module, the button, and the type of press you would like to simulate.

@![iOS]
##### Objective-C
###### RPC < v6.0
```objc
SDLButtonPress *buttonPress = [[SDLButtonPress alloc] initWithButtonName:SDLButtonNameEject moduleType:SDLModuleTypeRadio];
buttonPress.buttonPressMode = SDLButtonPressModeShort;

[self.sdlManager sendRequest:buttonPress withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if(!response.success) { return; }
}];
```

###### RPC v6.0+
```objc
SDLButtonPress *buttonPress = [[SDLButtonPress alloc] initWithButtonName:SDLButtonNameEject moduleType:SDLModuleTypeRadio moduleId:@"<#ModuleID#>"];
buttonPress.buttonPressMode = SDLButtonPressModeShort;

[self.sdlManager sendRequest:buttonPress withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if(!response.success) { return; }
}];
``` 

##### Swift
###### RPC < v6.0
```swift
let buttonPress = SDLButtonPress(buttonName: .eject, moduleType: .radio)
buttonPress.buttonPressMode = .short

sdlManager.send(request: buttonPress) { (request, response, error) in
    guard response?.success.boolValue == true else { return }
}
```

###### RPC v6.0+
```swift
let buttonPress = SDLButtonPress(buttonName: .eject, moduleType: .radio, moduleId: "<#ModuleID#>")
buttonPress.buttonPressMode = .short

sdlManager.send(request: buttonPress) { (request, response, error) in
    guard response?.success.boolValue == true else { return }
}
```
!@

@![android, javaSE, javaEE]
```java
ButtonPress buttonPress = new ButtonPress(ModuleType.RADIO, ButtonName.EJECT, ButtonPressMode.SHORT);
sdlManager.sendRPC(buttonPress);
```
!@

@![java,javaEE,javaSE]
```java
// ToDo - Add example
```
!@

#### Releasing the Module (RPC 6.0+ only)
When the user no longer needs control over a module, you should release the module so other users can control it. If you do not release the module, other users who would otherwise be able to control the module may be rejected from doing so.

@![iOS]
##### Objective-C
```objc
SDLReleaseInteriorVehicleDataModule * releaseInteriorVehicleDataModule = [[SDLReleaseInteriorVehicleDataModule alloc] initWithModuleType:<#Module Type#> moduleId:@"<#Module ID#>"]
[self.sdlManager sendRequest:releaseInteriorVehicleDataModule withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if(!response.success) { return; }
    <#Module Was Released#>
}];
```

##### Swift
```swift
let releaseInteriorVehicleDataModule = SDLReleaseInteriorVehicleDataModule(moduleType: <#Module Type#>, moduleId: "<#Module ID#>")
sdlManager.send(request: releaseInteriorVehicleDataModule) { (request, response, error) in
    guard response?.success.boolValue == true else { return }
    <#Module Was Released#>
}
```
!@

@![android, javaEE, javaSE]
```java
// ToDo - Add example
```
!@
