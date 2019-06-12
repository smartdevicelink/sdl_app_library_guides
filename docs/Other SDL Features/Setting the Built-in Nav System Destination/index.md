# Setting the Built-in Navigation Destination
The `SendLocation` RPC gives you the ability to send a GPS location to the embedded navigation app on the head unit. When the request is sent the user will be prompted to navigate to that location using the embedded navigation app. 

When using the `SendLocation` RPC, you will not have access to any information about how the user interacted with this location, only if the request was successfully sent to Core. The request will be handled by Core from that point on using the embedded navigation system.

!!! NOTE
The send location feature is only supported for the embedded navigation app; it does not work with mobile navigation apps at this time.
!!!

## Checking if Send Location is Available
Since `SendLocation` is a newer RPC, there is a possibility that not all head units will support this request. To check if the request is supported, you may look at the @![iOS]`SDLManager`!@ @![android, javaSE, javaEE]`SdlManager`!@'s `systemCapabilityManager` property after the manager has started successfully.

!!! NOTE
`SendLocation` is an RPC that is usually restricted by OEMs. As a result, the head unit you are connecting to may limit app functionality if you do not have permission to use the request.
!!!

@![iOS]
##### Objective-C
```objc
BOOL isNavigationSupported = NO;

__weak typeof (self) weakSelf = self;
[self.sdlManager startWithReadyHandler:^(BOOL success, NSError * _Nullable error) {
    if (!success) {
        NSLog(@"SDL errored starting up: %@", error);
        return;
    }

    SDLHMICapabilities *hmiCapabilities = self.sdlManager.systemCapabilityManager.hmiCapabilities;
    if (hmiCapabilities != nil) {
        isNavigationSupported = hmiCapabilities.navigation.boolValue;
    }
}];
```

##### Swift
```swift
var isNavigationSupported = false

sdlManager.start { (success, error) in
    if !success {
        print("SDL errored starting up: \(error.debugDescription)")
        return
    }

    if let hmiCapabilities = self.sdlManager.systemCapabilityManager.hmiCapabilities, let navigationSupported = hmiCapabilities.navigation?.boolValue {
        isNavigationSupported = navigationSupported
    }
}
```
!@

@![android, javaSE, javaEE]
```java
HMICapabilities hmiCapabilities = (HMICapabilities) sdlManager.getSystemCapabilityManager().getCapability(SystemCapabilityType.HMI);
if (hmiCapabilities.isNavigationAvailable()) {
    // SendLocation supported
} else {
    // SendLocation is not supported
}
```
!@

## Using Send Location
To use the `SendLocation` request, you must, at minimum, include the longitude and latitude of the location.

@![iOS]
##### Objective-C
```objc
SDLSendLocation *sendLocation = [[SDLSendLocation alloc] initWithLongitude:-97.380967 latitude:42.877737 locationName:@"The Center" locationDescription:@"Center of the United States" address:@[@"900 Whiting Dr", @"Yankton, SD 57078"] phoneNumber:nil image:nil];
[self.sdlManager sendRequest:sendLocation withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if (error || ![response isKindOfClass:SDLSendLocationResponse.class]) {
        NSLog(@"Encountered Error sending SendLocation: %@", error);
        return;
    }
    
    SDLSendLocationResponse *sendLocation = (SDLSendLocationResponse *)response;
    SDLResult *resultCode = sendLocation.resultCode;
    if (![resultCode isEqualToEnum:SDLResultSuccess]) {
        if ([resultCode isEqualToEnum:SDLResultInvalidData]) {
            NSLog(@"SendLocation was rejected. The request contained invalid data.");
        } else if ([resultCode isEqualToEnum:SDLResultDisallowed]) {
            NSLog(@"Your app is not allowed to use SendLocation");
        } else {
            NSLog(@"Some unknown error has occured!");
        }
        return;
    }
    
    // Successfully sent!
}];
```

##### Swift
```swift
let sendLocation = SDLSendLocation(longitude: -97.380967, latitude: 42.877737, locationName: "The Center", locationDescription: "Center of the United States", address: ["900 Whiting Dr", "Yankton, SD 57078"], phoneNumber: nil, image: nil)

sdlManager.send(request: sendLocation) { (request, response, error) in
    guard let response = response as? SDLSendLocationResponse else { return }
    
    if let error = error {
        print("Encountered Error sending SendLocation: \(error)")
        return
    }
    
    if response.resultCode != .success {
        if response.resultCode == .invalidData {
            print("SendLocation was rejected. The request contained invalid data.")
        } else if response.resultCode == .disallowed {
            print("Your app is not allowed to use SendLocation")
        } else {
            print("Some unknown error has occured!")
        }
        return
    }
    
    // Successfully sent!
}
```
!@

@![android, javaSE, javaEE]
```java
SendLocation sendLocation = new SendLocation();
sendLocation.setLatitudeDegrees(42.877737);
sendLocation.setLongitudeDegrees(-97.380967);
sendLocation.setLocationName("The Center");
sendLocation.setLocationDescription("Center of the United States");

// Create Address
OasisAddress address = new OasisAddress();
address.setSubThoroughfare("900");
address.setThoroughfare("Whiting Dr");
address.setLocality("Yankton");
address.setAdministrativeArea("SD");
address.setPostalCode("57078");
address.setCountryCode("US-SD");
address.setCountryName("United States");

sendLocation.setAddress(address);

// Monitor response
sendLocation.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse response) {
        Result result = response.getResultCode();
        if(result.equals(Result.SUCCESS)){
            // SendLocation was successfully sent.
        }else if(result.equals(Result.INVALID_DATA)){
            // The request you sent contains invalid data and was rejected.
        }else if(result.equals(Result.DISALLOWED)){
            // Your app does not have permission to use SendLocation.
        }
    }
});

sdlManager.sendRPC(sendLocation);
```
!@

## Checking the Result of Send Location
The `SendLocation` response has 3 possible results that you should expect:

1. `SUCCESS` - Successfully sent.
2. `INVALID_DATA` - The request contains invalid data and was rejected.
3. `DISALLOWED` - Your app does not have permission to use `SendLocation`.