# Retrieving Vehicle Data
You can use the @![iOS]`SDLGetVehicleData` and `SDLSubscribeVehicleData`!@@![android, javase, javaee]`GetVehicleData` and `SubscribeVehicleData`!@ RPC requests to get vehicle data. Each vehicle manufacturer decides which data it will expose and to whom they will expose it. Please check the response from Core to find out which data you will have permission to access. Additionally, be aware that the user may have the ability to disable vehicle data access through the settings menu of their head unit. It may be possible to access vehicle data when the `hmiLevel` is `NONE` (i.e. the user has not opened your SDL app) but you will have to request this permission from the vehicle manufacturer.

!!! NOTE
You will only have access to vehicle data that is allowed to your `appName` & `appId` combination. Permissions will be granted by each OEM separately. See [Understanding Permissions](Getting Started/Understanding Permissions) for more details.
!!!

| Vehicle Data | Parameter Name  |  Description |
| ------------- | ------------- | ------------- |
| Acceleration Pedal Position | accPedalPosition | Accelerator pedal position (percentage depressed) |
| Airbag Status | airbagStatus | Status of each of the airbags in the vehicle: yes, no, no event, not supported, fault |
| Belt Status | beltStatus | The status of each of the seat belts: no, yes, not supported, fault, or no event |
| Body Information | bodyInformation | Door ajar status for each door. The Ignition status. The ignition stable status. The park brake active status. |
| Cloud App Vehicle Id | cloudAppVehicleID | The id for the vehicle when connecting to cloud applications |
| Cluster Mode Status | clusterModeStatus | Whether or not the power mode is active. The power mode qualification status: power mode undefined, power mode evaluation in progress, not defined, power mode ok. The car mode status: normal, factory, transport, or crash. The power mode status: key out, key recently out, key approved, post accessory, accessory, post ignition, ignition on, running, crank |
| Device Status | deviceStatus | Contains information about the smartphone device. Is voice recognition on or off, has a bluetooth connection been established, is a call active, is the phone in roaming mode, is a text message available, the battery level, the status of the mono and stereo output channels, the signal level, the primary audio source, whether or not an emergency call is currently taking place |
| Driver Braking | driverBraking | The status of the brake pedal: yes, no, no event, fault, not supported |
| E-Call Infomation | eCallInfo | Information about the status of an emergency call |
| Electronic Parking Brake Status | electronicParkingBrakeStatus | The status of the electronic parking brake. Available states: closed, transition, open, drive active, fault |
| Emergency event | emergencyEvent | The type of emergency: frontal, side, rear, rollover, no event, not supported, fault. Fuel cutoff status: normal operation, fuel is cut off, fault. The roll over status: yes, no, no event, not supported, fault. The maximum change in velocity. Whether or not multiple emergency events have occurred |
| Engine Oil Life | engineOilLife | The estimated percentage (0% - 100%) of remaining oil life of the engine |
| Engine Torque | engineTorque | Torque value for engine (in Nm) on non-diesel variants |
| External Temperature | externalTemperature | The external temperature in degrees celsius |
| Fuel Level | fuelLevel | The fuel level in the tank (percentage) |
| Fuel Level State | fuelLevel_State | The fuel level state: Unknown, Normal, Low, Fault, Alert, or Not Supported |
| Fuel Range | fuelRange | The estimate range in KM the vehicle can travel based on fuel level and consumption |
| GPS | gps | Longitude and latitude, current time in UTC, degree of precision, altitude, heading, speed, satellite data vs dead reckoning, and supported dimensions of the GPS |
| Head Lamp Status | headLampStatus | Status of the head lamps: whether or not the low and high beams are on or off. The ambient light sensor status: night, twilight 1, twilight 2, twilight 3, twilight 4, day, unknown, invalid |
| Instant Fuel Consumption | instantFuelConsumption | The instantaneous fuel consumption in microlitres |
| My Key | myKey | Information about whether or not the emergency 911 override has been activated |
| Odometer | odometer | Odometer reading in km |
| PRNDL | prndl | The selected gear the car is in: park, reverse, neutral, drive, sport, low gear, first, second, third, fourth, fifth, sixth, seventh or eighth gear, unknown, or fault |
| Speed | speed | Speed in KPH |
| Steering Wheel Angle | steeringWheelAngle | Current angle of the steering wheel (in degrees) |
| Tire Pressure | tirePressure | Tire status of each wheel in the vehicle: normal, low, fault, alert, or not supported. Warning light status for the tire pressure: off, on, flash, or not used |
| Turn Signal | turnSignal | The status of the turn signal. Available states: off, left, right, both |
| RPM | rpm | The number of revolutions per minute of the engine |
| VIN | vin | The Vehicle Identification Number |
| Wiper Status | wiperStatus | The status of the wipers: off, automatic off, off moving, manual interaction off, manual interaction on, manual low, manual high, manual flick, wash, automatic low, automatic high, courtesy wipe, automatic adjust, stalled, no data exists |

## One-Time Vehicle Data Retrieval
To get vehicle data a single time, use the @![iOS]`SDLGetVehicleData`!@@![android, javaSE, javaEE]`GetVehicleData`!@ RPC. 

@![iOS]
##### Objective-C
```objc
SDLGetVehicleData *getVehicleData = [[SDLGetVehicleData alloc] init];
getVehicleData.prndl = @YES;
[self.sdlManager sendRequest:getVehicleData withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if (error || ![response isKindOfClass:SDLGetVehicleDataResponse.class]) {
        NSLog(@"Encountered Error sending GetVehicleData: %@", error);
        return;
    }
    
    SDLGetVehicleDataResponse* getVehicleDataResponse = (SDLGetVehicleDataResponse *)response;
    SDLResult *resultCode = getVehicleDataResponse.resultCode;
    if (![resultCode isEqualToEnum:SDLResultSuccess]) {
        if ([resultCode isEqualToEnum:SDLResultRejected]) {
            NSLog(@"GetVehicleData was rejected. Are you in an appropriate HMI?");
        } else if ([resultCode isEqualToEnum:SDLResultDisallowed]) {
            NSLog(@"Your app is not allowed to use GetVehicleData");
        } else {
            NSLog(@"Some unknown error has occured!");
        }
        return;
    }
    
    SDLPRNDL *prndl = getVehicleDataResponse.prndl;
}];
```

##### Swift
```swift
let getVehicleData = SDLGetVehicleData()
getVehicleData.prndl = true
sdlManager.send(getVehicleData) { (request, response, error) in
    guard let response = response as? SDLGetVehicleDataResponse else { return }
    
    if let error = error {
        print("Encountered Error sending GetVehicleData: \(error)")
        return
    }
    
    if !response.resultCode == .success {
        if response.resultCode == .rejected {
            print("GetVehicleData was rejected. Are you in an appropriate HMI?")
        } else if response.resultCode == .disallowed {
            print("Your app is not allowed to use GetVehicleData")
        } else {
            print("Some unknown error has occured!")
        }
        return
    }
    
    guard let prndl = response.prndl else { return }
}
```
!@

@![android, javaSE, javaEE]
```java
GetVehicleData vdRequest = new GetVehicleData();
vdRequest.setPrndl(true);
vdRequest.setOnRPCResponseListener(new OnRPCResponseListener() {
	@Override
	public void onResponse(int correlationId, RPCResponse response) {
		if(response.getSuccess()){
			PRNDL prndl = ((GetVehicleDataResponse) response).getPrndl();
			Log.i("SdlService", "PRNDL status: " + prndl.toString());
		}else{
			Log.i("SdlService", "GetVehicleData was rejected.");
		}
	}
});
sdlManager.sendRPC(vdRequest);
```
!@

## Subscribing to Vehicle Data
Subscribing to vehicle data allows you to get notifications whenever new data is available. You should not rely upon getting this data in a consistent manner. New vehicle data is available roughly every second, but this is totally dependent on which head unit you are connected to.

@![iOS]
**First**, register to observe the `SDLDidReceiveVehicleDataNotification` notification: 

##### Objective-C
```objc
[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(vehicleDataAvailable:) name:SDLDidReceiveVehicleDataNotification object:nil];
```

##### Swift
```swift
NotificationCenter.default.addObserver(self, selector: #selector(vehicleDataAvailable(_:)), name: .SDLDidReceiveVehicleData, object: nil)
```

**Second**, send the `SubscribeVehicleData` request:

##### Objective-C
```objc
SDLSubscribeVehicleData *subscribeVehicleData = [[SDLSubscribeVehicleData alloc] init];
subscribeVehicleData.prndl = @YES;

[self.sdlManager sendRequest:subscribeVehicleData withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if (![response isKindOfClass:[SDLSubscribeVehicleDataResponse class]]) {
        return;
    }
    
    SDLSubscribeVehicleDataResponse *subscribeVehicleDataResponse = (SDLSubscribeVehicleDataResponse*)response;
    SDLVehicleDataResult *prndlData = subscribeVehicleDataResponse.prndl;
    if (!response.success.boolValue) {
        if ([response.resultCode isEqualToEnum:SDLResultDisallowed]) {
            // Not allowed to register for this vehicle data.
        } else if ([response.resultCode isEqualToEnum:SDLResultUserDisallowed]) {
            // User disabled the ability to give you this vehicle data
        } else if ([response.resultCode isEqualToEnum:SDLResultIgnored]) {
            if ([prndlData.resultCode isEqualToEnum:SDLVehicleDataResultCodeDataAlreadySubscribed]) {
                // You have access to this data item, and you are already subscribed to this item so we are ignoring.
            } else if ([prndlData.resultCode isEqualToEnum:SDLVehicleDataResultCodeVehicleDataNotAvailable]) {
                // You have access to this data item, but the vehicle you are connected to does not provide it.
            } else {
            	NSLog(@"Unknown reason for being ignored: %@", prndlData.resultCode.value);
            }
        } else if (error) {
            NSLog(@"Encountered Error sending SubscribeVehicleData: %@", error);
        }
        
        return;
    }
    
    // Successfully subscribed
}];
```

##### Swift
```swift
let subscribeVehicleData = SDLSubscribeVehicleData()
subscribeVehicleData.prndl = true

sdlManager.send(request: subscribeVehicleData) { (request, response, error) in
    guard let response = response as? SDLSubscribeVehicleDataResponse else { return }
    
    guard response.success.boolValue == true else {
        if response.resultCode == .disallowed {
            // Not allowed to register for this vehicle data.
        } else if response.resultCode == .userDisallowed {
            // User disabled the ability to give you this vehicle data
        } else if response.resultCode == .ignored {
            if let prndlData = response.prndl {
                if prndlData.resultCode == .dataAlreadySubscribed {
                    // You have access to this data item, and you are already subscribed to this item so we are ignoring.
                } else if prndlData.resultCode == .vehicleDataNotAvailable {
                    // You have access to this data item, but the vehicle you are connected to does not provide it.
                } else {
                    print("Unknown reason for being ignored: \(prndlData.resultCode)")
                }
            } else {
                print("Unknown reason for being ignored: \(String(describing: response.info))")
            }
        } else if let error = error {
            print("Encountered Error sending SubscribeVehicleData: \(error)")
        }
        return
    }
    
    // Successfully subscribed
}
```

**Third**, react to the notification when vehicle data is received:

##### Objective-C
``` objc
- (void)vehicleDataAvailable:(SDLRPCNotificationNotification *)notification {
    if (![notification.notification isKindOfClass:SDLOnVehicleData.class]) {
        return;
    }
    
    SDLOnVehicleData *onVehicleData = (SDLOnVehicleData *)notification.notification;
    
    SDLPRNDL *prndl = onVehicleData.prndl;
}
```

##### Swift
```swift
func vehicleDataAvailable(_ notification: SDLRPCNotificationNotification) {
    guard let onVehicleData = notification.notification as? SDLOnVehicleData else {
        return
    }
    
    let prndl = onVehicleData.prndl
}
```
!@

@![android, javaSE, javaEE]
**First**, you should add a notification listener for the `OnVehicleData` notification: 

```java
sdlManager.addOnRPCNotificationListener(FunctionID.ON_VEHICLE_DATA, new OnRPCNotificationListener() {
    @Override
    public void onNotified(RPCNotification notification) {
        OnVehicleData onVehicleDataNotification = (OnVehicleData) notification;
        if (onVehicleDataNotification.getPrndl() != null) {
            Log.i("SdlService", "PRNDL status was updated to: " + onVehicleDataNotification.getPrndl());
        }
    }
});
```

**Second**, send the `SubscribeVehicleData` request:

```java
SubscribeVehicleData subscribeRequest = new SubscribeVehicleData();
subscribeRequest.setPrndl(true);
subscribeRequest.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse response) {
        if(response.getSuccess()){
            Log.i("SdlService", "Successfully subscribed to vehicle data.");
        }else{
            Log.i("SdlService", "Request to subscribe to vehicle data was rejected.");
        }
    }
}); 
sdlManager.sendRPC(subscribeRequest);
```

**Third**, the `onNotified` method will be called when there is an update to the subscribed vehicle data.
!@

## Unsubscribing from Vehicle Data
We suggest that you only subscribe to vehicle data as needed. To stop listening to specific vehicle data use the @![iOS]`SDLUnsubscribeVehicleData`!@@![android, javase, javaee]`UnsubscribeVehicleData`!@ RPC.

@![iOS]
##### Objective-C
```objc
SDLUnsubscribeVehicleData *unsubscribeVehicleData = [[SDLUnsubscribeVehicleData alloc] init];
unsubscribeVehicleData.prndl = @YES;

[self.sdlManager sendRequest:unsubscribeVehicleData withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if (![response isKindOfClass:[SDLUnsubscribeVehicleDataResponse class]]) {
        return;
    }

    SDLUnsubscribeVehicleDataResponse *unsubscribeVehicleDataResponse = (SDLUnsubscribeVehicleDataResponse*)response;
    SDLVehicleDataResult *prndlData = unsubscribeVehicleDataResponse.prndl;

    if (!response.success.boolValue) {
        if ([response.resultCode isEqualToEnum:SDLResultDisallowed]) {
            // Not allowed to register for this vehicle data, so unsubscribe also will not work.
        } else if ([response.resultCode isEqualToEnum:SDLResultUserDisallowed]) {
            // User disabled the ability to give you this vehicle data, so unsubscribe also will not work.
        } else if ([response.resultCode isEqualToEnum:SDLResultIgnored]) {
            if ([prndlData.resultCode isEqualToEnum:SDLVehicleDataResultCodeDataNotSubscribed]) {
                // You have access to this data item, but it was never subscribed to so we ignored it.
            } else {
                NSLog(@"Unknown reason for being ignored: %@", prndlData.resultCode.value);
            }
        } else if (error) {
            NSLog(@"Encountered Error sending UnsubscribeVehicleData: %@", error);
        }
        return;
    }
    
    // Successfully unsubscribed
}];
```

##### Swift
```swift
let unsubscribeVehicleData = SDLUnsubscribeVehicleData()
unsubscribeVehicleData.prndl = true

sdlManager.send(request: unsubscribeVehicleData) { (request, response, error) in
    guard let response = response as? SDLUnsubscribeVehicleDataResponse else { return }
    
    guard response.success.boolValue == true else {
        if response.resultCode == .disallowed {
            
        } else if response.resultCode == .userDisallowed {
            
        } else if response.resultCode == .ignored {
            if let prndlData = response.prndl {
                if prndlData.resultCode == .dataNotSubscribed {
                    // You have access to this data item, and you are already unsubscribed to this item so we are ignoring.
                } else if prndlData.resultCode == .vehicleDataNotAvailable {
                    // You have access to this data item, but the vehicle you are connected to does not provide it.
                } else {
                    print("Unknown reason for being ignored: \(prndlData.resultCode)")
                }
            } else {
                print("Unknown reason for being ignored: \(String(describing: response.info))")
            }
        } else if let error = error {
            print("Encountered Error sending UnsubscribeVehicleData: \(error)")
        }
        return
    }
    
    // Successfully unsubscribed
}
```
!@

@![android, javaSE, javaEE]
```java
UnsubscribeVehicleData unsubscribeRequest = new UnsubscribeVehicleData();
unsubscribeRequest.setPrndl(true); // unsubscribe to PRNDL data
unsubscribeRequest.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse response) {
        if(response.getSuccess()){
            Log.i("SdlService", "Successfully unsubscribed to vehicle data.");
        }else{
            Log.i("SdlService", "Request to unsubscribe to vehicle data was rejected.");
        }
    }
});
sdlManager.sendRPC(unsubscribeRequest);
```
!@
