# Setting the Navigation Destination
Setting a Navigation Destination allows you to send a GPS location, prompting the user to navigate to that location using their embedded navigation. When using the `SendLocation` RPC, you will not receive a callback about how the user interacted with this location, only if it was successfully sent to Core and received. It will be handled by Core from that point on using the embedded navigation system.

!!! note
This currently is only supported for Embedded Navigation; it does not work with Mobile Navigation Apps at this time.
!!!

## Detecting if SendLocation is Available
To check if `SendLocation` is supported, you may look at `SDLManager`'s `systemCapabilityManager` property after the ready handler is called. Or, you may use `SDLManager`'s `permissionManager` property to ask for the permission status of `SendLocation`.

!!! note
`SendLocation` is an RPC that is usually restricted by OEMs. As a result, the OEM you are connecting to may limit app functionality if you are not approved to use it.
!!!

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

## Using Send Location
To use `SendLocation`, you must at least include the Longitude and Latitude of the location.

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

## Determining the Result of SendLocation
`SendLocation` has 3 possible results that you should expect:

1. SUCCESS - SendLocation was successfully sent.
2. INVALID_DATA - The request you sent contains invalid data and was rejected.
3. DISALLOWED - Your app does not have permission to use SendLocation.