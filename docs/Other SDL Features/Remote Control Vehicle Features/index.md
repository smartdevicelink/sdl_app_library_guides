# Remote Control Vehicle Features
The remote control framework allows apps to control certain modules, such as climate, radio, seat, lights, etc., within a vehicle. Some vehicles will have mutiple modules of the same type and each module can be controlled seperatley depending on seat location and permissions.

!!! Note
Not all head units support this feature. If using this feature in your app you will most likely need to request permission from the vehicle manufacturer.
!!!

## Why Use Remote Control?
Consider the following scenarios:

- A radio application wants to use the in-vehicle radio tuner. It needs the functionality to select the radio band (AM/FM/XM/HD/DAB), tune the radio frequency or change the radio station, as well as obtain general radio information for decision making.
- A climate control application needs to turn on the AC, control the air circulation mode, change the fan speed and set the desired cabin temperature.
- A user profile application wants to remember users' favorite settings and apply it later automatically when the users get into the same/another vehicle.

### Remote Control Modules
Currently, the remote control feature supports these modules:

| Remote Control Modules |
| ---------            |
| Climate              |
| Radio                |
| Seat                 |
| Audio                |
| Light                |
| HMI Settings         |

The following table lists what control items are in each control module.

#### Climate

| Control Item | RPC Item Name | Value Range | Type | Comments | Version Changes |
| --------------- | ------------ | ------------ | ------------ | ------------ | ------------ |
| Climate Enable | climateEnable | on, off | Get/Set/Notification | Enabled to turn on the climate system, Disabled to turn off the climate system. All other climate items need climate enabled to work. | Since SDL v6.0 |
| Current Cabin Temperature | currentTemperature | N/A | Get/Notification | read only, value range depends on OEM | Since SDL v4.5 |
| Desired Cabin Temperature | desiredTemperature | N/A | Get/Set/Notification | value range depends on OEM | Since SDL v4.5 |
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
| Radio Enabled | radioEnable | true,false  | Get/Set/Notification | read only, all other radio control items need radio enabled to work | Since SDL v4.5 |
| Radio Band | band | AM,FM,XM  | Get/Set/Notification | | Since SDL v4.5 |
| Radio Frequency | frequencyInteger / frequencyFraction | 0-1710, 0-9 | Get/Set/Notification | value range depends on band | Since SDL v4.5 |
| Radio RDS Data | rdsData | RdsData struct | Get/Notification | read only | Since SDL v4.5 |
| Available HD Channels | availableHdChannels | Array size 0-8, values 0-7 | Get/Notification | read only | Since SDL v6.0, replaces availableHDs |
| Available HD Channels (DEPRECATED) | availableHDs | 1-7 (Deprecated in SDL v.6.0) (1-3 before SDL v.5.0) | Get/Notification | read only | Since SDL v4.5, updated in v5.0, deprecated in v6.0 |
| Current HD Channel | hdChannel | 0-7 (1-3 before SDL v.5.0) (1-7 between SDL v.5.0-6.0) | Get/Set/Notification |  | Since SDL v4.5, updated in SDL v5.0, updated in SDL v6.0 |
| Radio Signal Strength | signalStrength | 0-100% | Get/Notification | read only | Since SDL v4.5 |
| Signal Change Threshold | signalStrengthThreshold | 0-100% | Get/Notification | read only | Since SDL v4.5 |
| Radio State | state | Acquiring, acquired, multicast, not_found | Get/Notification | read only | Since SDL v4.5 |
| SIS Data | sisData | See SisData struct | Get/Notification | read only | Since SDL v5.0 |

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
For remote control to work, the head unit must support SDL Core v.4.4 or newer. Also your app's @![iOS]`appType`!@ @![android, javaSE, javaEE]`appHMIType`!@ must be set to `REMOTE_CONTROL`.

### Checking Permissions
Prior to using any remote control RPCs, you must check that the head unit has the remote control capability. As you will encounter head units that do *not* support it, this check is important. To check for this capability, use the following call:

If you do have permission to use the remote control feature, the capability object will have a list of @![iOS]`SDLButtonCapabilities`!@ @![android, javaSE, javaEE]`ButtonCapabilities`!@ that can be obtained via the `buttonCapabilities` property.

@![iOS]
##### Objective-C
```objc
[self.sdlManager.systemCapabilityManager updateCapabilityType:SDLSystemCapabilityTypeRemoteControl completionHandler:^(NSError * _Nullable error, SDLSystemCapabilityManager * _Nonnull systemCapabilityManager) {
    if (systemCapabilityManager.remoteControlCapability == nil || error != nil) {
        // Remote control is not supported or capability could not be retrieved
        return;
    }

    SDLRemoteControlCapabilities *capabilities = systemCapabilityManager.remoteControlCapability;
    <#Code#>
}];
```

##### Swift
```swift
sdlManager.systemCapabilityManager.updateCapabilityType(.remoteControl) { (error, systemCapabilityManager) in
    if systemCapabilityManager.remoteControlCapability == nil || error != nil {
        // Remote control is not supported or capability could not be retrieved
        return
    }

    let capabilities = systemCapabilityManager.remoteControlCapability;
    <#Code#>
}
```
!@

@![android, javaSE, javaEE]
```java
// First you can check to see if the capability is supported on the module
if (sdlManager.getSystemCapabilityManager().isCapabilitySupported(SystemCapabilityType.REMOTE_CONTROL)){
    // Since the module does support this capability we can query it for more information
    sdlManager.getSystemCapabilityManager().getCapability(SystemCapabilityType.REMOTE_CONTROL, new OnSystemCapabilityListener(){

        @Override
        public void onCapabilityRetrieved(Object capability){
            RemoteControlCapabilities remoteControlCapabilities = (RemoteControlCapabilities) capability;
            // Now it is possible to get details on how this capability
            // is supported using the remoteControlCapabilities object
        }

        @Override
        public void onError(String info){
            Log.i(TAG, "Capability could not be retrieved: "+ info);
        }
    });
}
```
!@

### Controlling Module(s)
Starting in SDL 6.0 multiple modules can exist for each module type. Since many modules can exist for each module type you should provide the `moduleID` to tell the HMI the specific module you wish to control. A new struct `moduleInfo` was created to give developers the information they need to control a specific module. The `moduleInfo` is a struct on the `XYZControlCapabilities` objects. When sending remote control RPCs in SDL 6.0 the `moudleID` should be provided to control the desired module. If no `moduleID` is set, the HMI will use the default module of that module type.

Controlling a module is seat-based, depending on what seat you are sitting in you may or not be able to control certain modules. For example, only the person sitting in a specific seat can control that seat. Some modules allow multiple users to access and some only allow one at a time. Access to a module depends on the OEMs configuration. 

### Setting Your Seat
The first step and before sending any set moudle data RPCs you should have the user select their seat. Seat location may affect the modules a user can control depending on the OEMs rules. The default seat location is `Driver`. Seat location can be updated by setting the `userLocation` property in `SDLSetGlobalProperties` and then sending a `SDLSetGlobalProperties` RPC. In a real-life scenario, you may wish to show the user a map or list of all available seats. This example is only meant to show you how to access the data not build your UI/UX. An array of seats can be found in the `SDLGetSystemCapabilityResponse`s `seatLocationCapability`s `seat` array. Each `SeatLocation` object within the `seats` array will have a `grid` struct. This struct will tell you the seat placement of that particular seat. This information can be very useful for creating a map/list for users to select from. 

The `grid` system starts with the driver seat being (0,0,0). A `grid` of `col`=0, `row`=0 and `level`=0 would be referring to the drivers' location. A `col`=2, `row`=0 and `level`=0 would be referring to the front right passenger location, assuming the car has 3 columns. A negative `col` or `row` means it is outside the vehicle. The `colspan` and `rowspan` properties tell you how many rows and columns that module or seat takes up.

![Car](assets/Car.png)

| col=-1, row=-1  | col=0 | col=1 | col=2 |
| --- | --- | --- | --- |
| row=0 | driver's seat: {col=0, row=0, level=0, colspan=1, rowspan=1, levelspan=1} |   | front passenger's seat : {col=2, row=0, level=0, colspan=1, rowspan=1, levelspan=1}|
| row=1 | rear-left seat : {col=0, row=1, level=0, colspan=1, rowspan=1, levelspan=1}| rear-middle seat :  {col=1, row=1, level=0, colspan=1, rowspan=1, levelspan=1} | rear-right seat : {col=2, row=1, level=0, colspan=1, rowspan=1, levelspan=1} |

@![iOS]
##### Objective-C
```objc
SDLGetSystemCapability *getSeatLocationCapability = [[SDLGetSystemCapability alloc] initWithType: SDLSystemCapabilityTypeSeatLocation];
[self.sdlManager sendRequest:getSeatLocationCapability withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if (!response.success) { return; }
    SDLGetSystemCapabilityResponse *res = (SDLGetSystemCapabilityResponse *)response;
    
    NSArray<SDLSeatLocation *> *seats = res.systemCapability.seatLocationCapability.seats;
    <#Save seats array#>
}];
```
##### Swift
```swift
let getSeatLocationCapability = SDLGetSystemCapability(type: .seatLocation)
self.sdlManager.send(request: getSeatLocationCapability) { (request, response, error) in
    guard let res = response as? SDLGetSystemCapabilityResponse, let seats = res.systemCapability.seatLocationCapability?.seats else { return }
    <#Save seats array#>
}
```
!@

When the user selects their seat you should send an `SDLSetGlobalProperties` RPC and set the `userLocation` property. 

@![iOS]
##### Objective-C
```objc
SDLSetGlobalProperties *seatLocation = [[SDLSetGlobalProperties alloc] init];
seatLocation.userLocation = <#Selected Seat#>;
[self.sdlManager sendRPC:seatLocation];
```
##### Swift
```swift
let seatLocation = SDLSetGlobalProperties()
seatLocation.userLocation = <#Selected Seat#>;
self.sdlManager.send(seatLocation)
```
!@

### Getting Remote Control Modules
If the vehicle supports multiple modules of a module type you will have to pass in the `moduleID` along with the module type to access that module. To get a list of all modules you can check the `SDLGetSystemCapabilityResponse`s `remoteControlCapability` object. The `moduleID` is contained within `moduleInfo` in the remote control capability module. 

@![iOS]
##### Objective-C
```objc
SDLGetSystemCapability *remoteControl = [[SDLGetSystemCapability alloc] initWithType: SDLSystemCapabilityTypeRemoteControl];
[self.sdlManager sendRequest:remoteControl withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if (!response.success) { return; }
    SDLRemoteControlCapabilities *remoteCapabilities = ((SDLGetSystemCapabilityResponse *)response).systemCapability.remoteControlCapability;
    <#Save Remote Capabilities#>
}];
```
##### Swift
```swift
let getRemoteCapability = SDLGetSystemCapability(type: .remoteControl)
self.sdlManager.send(request: getRemoteCapability) { (request, response, error) in
    guard let res =  response as? SDLGetSystemCapabilityResponse, let radioCapabilities = res.systemCapability.remoteControlCapability else { return }
    <#Save Remote Capabilities#>
}
```
!@

With the saved Remote Capabilities object saved you can build a UI/UX to display modules to a user. When the user selects the module you can send a Remote Control RPCs using the `moduleInfo`s `moduleId` property. 

@![iOS]
##### Objective-C
```objc
NSString *selectedModuleID = <#SelectedModule#>.moduleInfo.moduleID;
```
##### Swift
```swift
let selectedModuleID = <#SelectedModule#>.moduleInfo?.moduleId
```
!@

### Get Consent
Some OEMs may wish to ask the driver for consent before a user can control a module. This is typically done automatically by the HMI depending on the configuration. However, the `SDLGetInteriorVehicleDataConsent` RPC will alert the driver for consent in some OEMs if the module if not free and `allowMultipleAccess` is true. `allowMultipleAccess` is part of the `moduleInfo` in the module object.

Check the `allowed` property in the `SDLGetInteriorVehicleDataConsentResponse` to see what modules can be controlled. Note the order of the `allowed` array is 1-1 of the `moduleIDs` array you passed into the `SDLGetInteriorVehicleDataConsent` RPC.

!!! Note
Getting consent should always be sent before setting any module data. If consent is not granted you should not send any set module data RPC.
!!!

@![iOS]
##### Objective-C
```objc
[self.sdlManager sendRequest:[[SDLGetInteriorVehicleDataConsent alloc] initWithModuleType:@"<#ModuleType#>" moduleIds:@[@"<#ModuleID#>",@"<#ModuleID#>",@"<#ModuleID#>"]] withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if (!response.success) { return; }
    <#Allowed is an arry of true or false values#>
}];
```
##### Swift
```swift
self.sdlManager?.send(request: SDLGetInteriorVehicleDataConsent(moduleType: "<#ModuleType#>", moduleIds: ["<#ModuleID#>","<#ModuleID#>","<#ModuleID#>"]), responseHandler: { (request, response, error) in
    guard let res = response as? SDLGetInteriorVehicleDataConsentResponse else { return }
    let allowed = res.allowed as! [NSNumber]
    <#Allowed is an arry of true or false values#>
})
```
!@

### Getting Data
Once you know you have permission to use the remote control feature and you have the moudleID, you can retrieve the data. The following code is an example of how to get data from a radio module. The example also subscribes to updates to radio data, which will be discussed later on in this guide. Since not all vehicles will suport mutiple modules it is recommended to have both of the following implemented. Seat location does not effect the ability to obtain information about a module. However, to set module data, seat location in taken into account.

@![iOS]
##### Objective-C
###### No ModuleID
```objc
SDLGetInteriorVehicleData *getInteriorVehicleData = [[SDLGetInteriorVehicleData alloc] initAndSubscribeToModuleType:SDLModuleTypeRadio];
[self.sdlManager sendRequest:getInteriorVehicleData withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    SDLGetInteriorVehicleDataResponse *dataResponse = (SDLGetInteriorVehicleDataResponse *)response;
    // This can now be used to retrieve data
    <#Code#>
}];
```
###### With ModuleID
```objc
SDLGetInteriorVehicleData *getInteriorVehicleData = [[SDLGetInteriorVehicleData alloc] initAndSubscribeToModuleType:SDLModuleTypeRadio moduleId:@"<#ModuleID#>"]];
[self.sdlManager sendRequest:getInteriorVehicleData withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    SDLGetInteriorVehicleDataResponse *dataResponse = (SDLGetInteriorVehicleDataResponse *)response;
    // This can now be used to retrieve data
    <#Code#>
}];
```

##### Swift
###### No ModuleID
```swift
let getInteriorVehicleData = SDLGetInteriorVehicleData(andSubscribeToModuleType: .radio)
sdlManager.send(request: getInteriorVehicleData) { (req, res, err) in
    guard let response = res as? SDLGetInteriorVehicleDataResponse else { return }
    // This can now be used to retrieve data
    <#Code#>
}
```
###### With ModuleID
```swift
let getInteriorVehicleData = SDLGetInteriorVehicleData(andSubscribeToModuleType: .radio, moudleID: "<#ModuleID#>")
sdlManager.send(request: getInteriorVehicleData) { (req, res, err) in
    guard let response = res as? SDLGetInteriorVehicleDataResponse else { return }
    // This can now be used to retrieve data
    <#Code#>
}
```
!@

@![android, javaSE, javaEE]
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

### Setting Data
Of course, the ability to set these modules is the point of the remote control framework. Setting data is similar to getting it. Below is an example of setting climate control data. It is likely that you will not need to set all the data as in the code example. If there are settings you don't wish to modify you can skip setting them. If multiple modules are allowed on the HMI this request may fail if that module is already "In Use" by another user or is out of your seat range. To check if the module is accessed send an `SDLGetInteriorVehicleDataConsent` with the `moduleID` of the module you wish to control as shown above. To set the `moduleID` in a `SDLSetInteriorVehicleData` set the `moduleID` property on the `moduleData` object.

The permission area of a module depends on the `serviceArea`s which can be found in `moduleInfo`. Each module will have its own `location` and `serviceArea`. The `location` of a module is just like the `seats` grid, it tells you the physical location of a particular module. The `serviceArea` tells you how far that module reaches. A radio control serves all passengers in the vehicle. Its service area will cover its physical location as well as other locations in the vehicle. If a `serviceArea` is not included, it is assumed the `serviceArea` is the same as `location`. If both are not included, it is assumed that the `serviceArea` covers the whole area of the vehicle. If a user is not sitting within the `serviceArea`s `grid` they will not be able to control that module.

@![iOS]
##### Objective-C
###### No ModuleID
```objc
SDLTemperature *temperature = [[SDLTemperature alloc] initWithUnit:SDLTemperatureUnitFahrenheit value:74.1];
SDLClimateControlData *climateControlData = [[SDLClimateControlData alloc] initWithFanSpeed:@2 desiredTemperature:temperature acEnable:@YES circulateAirEnable:@NO autoModeEnable:@NO defrostZone:nil dualModeEnable:@NO acMaxEnable:@NO ventilationMode:SDLVentilationModeLower heatedSteeringWheelEnable:@YES heatedWindshieldEnable:@YES heatedRearWindowEnable:@YES heatedMirrorsEnable:@NO];
SDLModuleData *moduleData = [[SDLModuleData alloc] initWithClimateControlData:climateControlData];
SDLSetInteriorVehicleData *setInteriorVehicleData = [[SDLSetInteriorVehicleData alloc] initWithModuleData:moduleData];
[self.sdlManager sendRequest:setInteriorVehicleData];
```
###### With ModuleID
```objc
SDLTemperature *temperature = [[SDLTemperature alloc] initWithUnit:SDLTemperatureUnitFahrenheit value:74.1];
SDLClimateControlData *climateControlData = [[SDLClimateControlData alloc] initWithFanSpeed:@2 desiredTemperature:temperature acEnable:@YES circulateAirEnable:@NO autoModeEnable:@NO defrostZone:nil dualModeEnable:@NO acMaxEnable:@NO ventilationMode:SDLVentilationModeLower heatedSteeringWheelEnable:@YES heatedWindshieldEnable:@YES heatedRearWindowEnable:@YES heatedMirrorsEnable:@NO];
SDLModuleData *moduleData = [[SDLModuleData alloc] initWithClimateControlData:climateControlData];
moduleData.moduleId = <#Module ID#>
SDLSetInteriorVehicleData *setInteriorVehicleData = [[SDLSetInteriorVehicleData alloc] initWithModuleData:moduleData];
[self.sdlManager sendRequest:setInteriorVehicleData];
```

##### Swift
###### No ModuleID
```swift
let temperature = SDLTemperature(unit: .fahrenheit, value: 74.1)
let climateControlData = SDLClimateControlData(fanSpeed: 2 as NSNumber, desiredTemperature: temperature, acEnable: true as NSNumber, circulateAirEnable: false as NSNumber, autoModeEnable: false as NSNumber, defrostZone: nil, dualModeEnable: false as NSNumber, acMaxEnable: false as NSNumber, ventilationMode: .lower, heatedSteeringWheelEnable: true as NSNumber, heatedWindshieldEnable: true as NSNumber, heatedRearWindowEnable: true as NSNumber, heatedMirrorsEnable: false as NSNumber)
let moduleData = SDLModuleData(climateControlData: climateControlData)
let setInteriorVehicleData = SDLSetInteriorVehicleData(moduleData: moduleData)

sdlManager.send(setInteriorVehicleData)
```
###### With ModuleID
```swift
let temperature = SDLTemperature(unit: .fahrenheit, value: 74.1)
let climateControlData = SDLClimateControlData(fanSpeed: 2 as NSNumber, desiredTemperature: temperature, acEnable: true as NSNumber, circulateAirEnable: false as NSNumber, autoModeEnable: false as NSNumber, defrostZone: nil, dualModeEnable: false as NSNumber, acMaxEnable: false as NSNumber, ventilationMode: .lower, heatedSteeringWheelEnable: true as NSNumber, heatedWindshieldEnable: true as NSNumber, heatedRearWindowEnable: true as NSNumber, heatedMirrorsEnable: false as NSNumber)
let moduleData = SDLModuleData(climateControlData: climateControlData)
moduleData.moduleId = <#Module ID#>
let setInteriorVehicleData = SDLSetInteriorVehicleData(moduleData: moduleData)

sdlManager.send(setInteriorVehicleData)
```
!@

@![android, javaSE, javaEE]
```java
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

### Button Presses
Another unique feature of remote control is the ability to send simulated button presses to the associated modules, imitating a button press on the hardware itself. Simply specify the module, the button, and the type of press you would like.

@![iOS]
##### Objective-C
###### No ModuleID
```objc
SDLButtonPress *buttonPress = [[SDLButtonPress alloc] initWithButtonName:SDLButtonNameEject moduleType:SDLModuleTypeRadio];
buttonPress.buttonPressMode = SDLButtonPressModeShort;

[self.sdlManager sendRequest:buttonPress];
```
###### With ModuleID
```objc
SDLButtonPress *buttonPress = [[SDLButtonPress alloc] initWithButtonName:SDLButtonNameEject moduleType:SDLModuleTypeRadio moduleId: @"<#ModuleID#>"];
buttonPress.buttonPressMode = SDLButtonPressModeShort;

[self.sdlManager sendRequest:buttonPress];
``` 
##### Swift
###### No ModuleID
```swift
let buttonPress = SDLButtonPress(buttonName: .eject, moduleType: .radio)
buttonPress.buttonPressMode = .short

sdlManager.send(buttonPress)
```
###### With ModuleID
```swift
let buttonPress = SDLButtonPress(buttonName: .eject, moduleType: .radio, moduleId: "<#ModuleID#>")
buttonPress.buttonPressMode = .short

sdlManager.send(buttonPress)
```
!@

@![android, javaSE, javaEE]
```java
ButtonPress buttonPress = new ButtonPress(ModuleType.RADIO, ButtonName.EJECT, ButtonPressMode.SHORT);
sdlManager.sendRPC(buttonPress);
```
!@

You should wrap any setting data RPC in the handler of the `SDLGetInteriorVehicleDataConsent` request.

@![iOS]

##### Objective-C
```objc
[self.sdlManager sendRequest:[[SDLGetInteriorVehicleDataConsent alloc] initWithModuleType:@"<#ModuleType#>" moduleIds:@[@"<#ModuleID#>"]] withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if (!response.success) { return; }
    SDLGetInteriorVehicleDataConsentResponse *res = (SDLGetInteriorVehicleDataConsentResponse *)response;
    if (!res.allowed[0].boolValue) { return; }
    <#Set Module Data#>
}];
```
##### Swift
```swift
self.sdlManager?.send(request: SDLGetInteriorVehicleDataConsent(moduleType: "<#ModuleID#>", moduleIds: ["<#ModuleID#>"]), responseHandler: { (request, response, error) in
    guard let res = response as? SDLGetInteriorVehicleDataConsentResponse else { return }
    let allowed = res.allowed as! [NSNumber]
    if(allowed[0].boolValue) { return }
    <#Set Module Data#>
})
```
!@

### Subscribing to Changes
It is also possible to subscribe to changes in data associated with supported modules. To do so, during your request for data, simply set `subscribe` to `true`. To unsubscribe, send the request again with `subscribe` set to `false`. The response to a subscription will come in a form of a notification. You can receive this notification by adding a notification listener for `OnInteriorVehicleData`.

!!! NOTE
The notification listener should be added before sending the @![iOS]`SDLGetInteriorVehicleData`!@ @![android, javaSE, javaEE]`GetInteriorVehicleData`!@ request.
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

// No MoudleID
SDLGetInteriorVehicleData *getInteriorVehicleData = [[SDLGetInteriorVehicleData alloc] initAndSubscribeToModuleType:SDLModuleTypeRadio moduleId:@"<#ModuleID#>"];
[self.sdlManager sendRequest:getInteriorVehicleData withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    SDLGetInteriorVehicleDataResponse *dataResponse = (SDLGetInteriorVehicleDataResponse *)response;
    // This can now be used to retrieve data
    <#Code#>
}];

// With ModuleID
SDLGetInteriorVehicleData *getInteriorVehicleData = [[SDLGetInteriorVehicleData alloc] initAndSubscribeToModuleType:SDLModuleTypeRadio];
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

let getInteriorVehicleData = SDLGetInteriorVehicleData(andSubscribeToModuleType: .radio, moduleId: "<#ModuleID#>")
sdlManager.send(request: getInteriorVehicleData) { (req, res, err) in
    guard let response = res as? SDLGetInteriorVehicleDataResponse else { return }
    // This can now be used to retrieve initialData data
    <#Code#>
}
```
!@

@![android, javaSE, javaEE]
```java
sdlManager.addOnRPCNotificationListener(FunctionID.ON_INTERIOR_VEHICLE_DATA, new OnRPCNotificationListener() {
    @Override
    public void onNotified(RPCNotification notification) {
        OnInteriorVehicleData onInteriorVehicleData = (OnInteriorVehicleData) notification;
        // Perform action based on notification
    }
});

// Then send the GetInteriorVehicleData with subscription set to true
GetInteriorVehicleData interiorVehicleData = new GetInteriorVehicleData(ModuleType.RADIO);
interiorVehicleData.setSubscribe(true);
sdlManager.sendRPC(interiorVehicleData);

```
!@

### Releasing the Moudle
When the user no longer needs control over a module, you should release the module so other users can control it. 

@![iOS]
##### Objective-C
```objc
[self.sdlManager subscribeToRPC:[[SDLReleaseInteriorVehicleDataModule alloc] initWithModuleType:<#Module Type#> moduleId:@"<#Module ID#>"]];
```

##### Swift
```swift
self.sdlManager.send(request: SDLReleaseInteriorVehicleDataModule(moduleType: <#Module Type#>, moduleId: "<#Module ID#>"))
```
!@
