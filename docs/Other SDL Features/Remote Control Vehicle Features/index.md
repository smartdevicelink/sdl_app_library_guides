# Remote Control Vehicle Features
The remote control framework allows apps to control certain modules, such as climate, radio, seat, lights, etc., within a vehicle.

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

| RC Module | Control Item | Value Range |Type | Comments |
| --------------- | ------------ |------------ | ------------ | ------------ |
| **Climate**     | Current Cabin Temperature |  | Get/Notification | read only, value range depends on OEM |
|                 | Desired Cabin Temperature |  | Get/Set/Notification | value range depends on OEM |
|                 | AC Setting | on, off | Get/Set/Notification |  |
|                 | AC MAX Setting | on, off  | Get/Set/Notification |  |
|                 | Air Recirculation Setting | on, off  | Get/Set/Notification |  |
|                 | Auto AC Mode Setting | on, off  | Get/Set/Notification |  |
|                 | Defrost Zone Setting | front, rear, all, none  | Get/Set/Notification |  |
|                 | Dual Mode Setting | on, off  | Get/Set/Notification |  |
|                 | Fan Speed Setting | 0%-100% | Get/Set/Notification |  |
|                 | Ventilation Mode Setting | upper, lower, both, none  | Get/Set/Notification |  |
| **Radio**       | Radio Enabled | true,false  | Get/Set/Notification| read only, all other radio control items need radio enabled to work|
|                 | Radio Band | AM,FM,XM  | Get/Set/Notification| |
|                 | Radio Frequency | | Get/Set/Notification | value range depends on band |
|                 | Radio RDS Data | | Get/Notification | read only |
|                 | Available HD Channel | 1-3 | Get/Notification | read only |
|                 | Current HD Channel | 1-3 | Get/Set/Notification |
|                 | Radio Signal Strength |  | Get/Notification | read only |
|                 | Signal Change Threshold |  | Get/Notification | read only |
|                 | Radio State | Acquiring, acquired, multicast, not_found | Get/Notification | read only |
| **Seat**        | Seat Heating Enabled | true, false | Get/Set/Notification | Indicates whether heating is enabled for a seat |
|                 | Seat Cooling Enabled | true, false | Get/Set/Notification | Indicates whether cooling is enabled for a seat |
|                 | Seat Heating  level | 0-100% | Get/Set/Notification | Level of the seat heating |
|                 | Seat Cooling  level | 0-100% | Get/Set/Notification | Level of the seat cooling |
|                 | Seat Horizontal Positon | 0-100% | Get/Set/Notification | Adjust a seat forward/backward, 0 means the nearest position to the steering wheel, 100% means the furthest position from the steering wheel |
|                 | Seat Vertical Position | 0-100% | Get/Set/Notification | Adjust seat height (up or down) in case there is only one actuator for seat height, 0 means the lowest position, 100% means the highest position|
|                 | Seat-Front Vertical Position | 0-100% | Get/Set/Notification | Adjust seat front height (in case there are two actuators for seat height), 0 means the lowest position, 100% means the highest position|
|                 | Seat-Back Vertical Position | 0-100% | Get/Set/Notification | Adjust seat back height (in case there are two actuators for seat height), 0 means the lowest position, 100% means the highest position|
|                 | Seat Back Tilt Angle | 0-100% | Get/Set/Notification | Backrest recline, 0 means the angle that back top is nearest to the steering wheel, 100% means the angle that back top is furthest from the steering wheel|
|                 | Head Support Horizontal Positon | 0-100% | Get/Set/Notification | Adjust head support forward/backward, 0 means the nearest position to the front, 100% means the furthest position from the front |
|                 | Head Support Vertical Position | 0-100% | Get/Set/Notification | Adjust head support height (up or down), 0 means the lowest position, 100% means the highest position|
|                 | Seat Massaging Enabled | true, false | Get/Set/Notification | Indicates whether massage is enabled for a seat |
|                 | Massage Mode | List of Struct {MassageZone, MassageMode} | Get/Set/Notification | list of massage mode of each zone |
|                 | Massage Cushion Firmness | List of Struct {Cushion, 0-100%} | Get/Set/Notification | list of firmness of each massage cushion|
|                 | Seat memory    | Struct{ id, label, action (SAVE/RESTORE/NONE)} | Get/Set/Notification | seat memory |
| **Audio**       | Audio volume | 0%-100%| Get/Set/Notification | The audio source volume level |
|                 | Audio Source | MOBILE_APP, RADIO_TUNER, CD, BLUETOOTH, USB, etc. see PrimaryAudioSource| Get/Set/Notification | defines one of the available audio sources |
|                 | keep Context | true, false| Set only | control whether HMI shall keep current application context or switch to default media UI/APP associated with the audio source|
|                 | Equilizer Settings | Struct {Channel ID as integer, Channel setting as 0%-100%} | Get/Set/Notification | Defines the list of supported channels (band) and their current/desired settings on HMI
| **Light**       | Light Status | ON, OFF| Get/Set/Notification | turn on/off a single light or all lights in a group |
|                 | Light Density | float 0.0-1.0| Get/Set/Notification | change the density/dim a single light or all lights in a group|
|                 | Light Color | RGB color| Get/Set/Notification | change the color scheme of a single light or all lights in a group|
| **HMI Settings** | Display Mode | DAY, NIGHT, AUTO | Get/Set/Notification | Current display mode of the HMI display |
|                 | Distance Unit | MILES, KILOMETERS | Get/Set/Notification | Distance Unit used in the HMI (for maps/tracking distances) |
|                 | Temperature Unit | FAHRENHEIT, CELSIUS | Get/Set/Notification | Temperature Unit used in the HMI (for temperature measuring systems) |

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

### Getting Data
Once you know you have permission to use the remote control feature, you can retrieve the data. The following code is an example of how to get data from the radio module. The example also subscribes to updates to radio data, which will be discussed later on in this guide.

@![iOS]
##### Objective-C
```objc
SDLGetInteriorVehicleData *getInteriorVehicleData = [[SDLGetInteriorVehicleData alloc] initAndSubscribeToModuleType:SDLModuleTypeRadio];
[self.sdlManager sendRequest:getInteriorVehicleData withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    SDLGetInteriorVehicleDataResponse *dataResponse = (SDLGetInteriorVehicleDataResponse *)response;
    // This can now be used to retrieve data
    <#Code#>
}];
```

##### Swift
```swift
let getInteriorVehicleData = SDLGetInteriorVehicleData(andSubscribeToModuleType: .radio)
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
});

sdlManager.sendRPC(interiorVehicleData);
```
!@

### Setting Data
Of course, the ability to set these modules is the point of the remote control framework. Setting data is similar to getting it. Below is an example of setting climate control data. It is likely that you will not need to set all the data as in the code example. If there are settings you don't wish to modify you can skip setting them.

@![iOS]
##### Objective-C
```objc
SDLTemperature *temperature = [[SDLTemperature alloc] initWithUnit:SDLTemperatureUnitFahrenheit value:74.1];
SDLClimateControlData *climateControlData = [[SDLClimateControlData alloc] initWithFanSpeed:@2 desiredTemperature:temperature acEnable:@YES circulateAirEnable:@NO autoModeEnable:@NO defrostZone:nil dualModeEnable:@NO acMaxEnable:@NO ventilationMode:SDLVentilationModeLower heatedSteeringWheelEnable:@YES heatedWindshieldEnable:@YES heatedRearWindowEnable:@YES heatedMirrorsEnable:@NO];
SDLModuleData *moduleData = [[SDLModuleData alloc] initWithClimateControlData:climateControlData];
SDLSetInteriorVehicleData *setInteriorVehicleData = [[SDLSetInteriorVehicleData alloc] initWithModuleData:moduleData];
[self.sdlManager sendRequest:setInteriorVehicleData];
```

##### Swift
```swift
let temperature = SDLTemperature(unit: .fahrenheit, value: 74.1)
let climateControlData = SDLClimateControlData(fanSpeed: 2 as NSNumber, desiredTemperature: temperature, acEnable: true as NSNumber, circulateAirEnable: false as NSNumber, autoModeEnable: false as NSNumber, defrostZone: nil, dualModeEnable: false as NSNumber, acMaxEnable: false as NSNumber, ventilationMode: .lower, heatedSteeringWheelEnable: true as NSNumber, heatedWindshieldEnable: true as NSNumber, heatedRearWindowEnable: true as NSNumber, heatedMirrorsEnable: false as NSNumber)
let moduleData = SDLModuleData(climateControlData: climateControlData)
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
```objc
SDLButtonPress *buttonPress = [[SDLButtonPress alloc] initWithButtonName:SDLButtonNameEject moduleType:SDLModuleTypeRadio];
buttonPress.buttonPressMode = SDLButtonPressModeShort;

[self.sdlManager sendRequest:buttonPress];
```

##### Swift
```swift
let buttonPress = SDLButtonPress(buttonName: .eject, moduleType: .radio)
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

let getInteriorVehicleData = SDLGetInteriorVehicleData(andSubscribeToModuleType: .radio)
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
