# Setting the Built-In Navigation Destination
The `SendLocation` RPC gives you the ability to send a GPS location to the embedded navigation app on the head unit. When the request is sent the user will be prompted to navigate to that location using the embedded navigation app. 

When using the `SendLocation` RPC, you will not have access to any information about how the user interacted with this location, only if the request was successfully sent to Core. The request will be handled by Core from that point on using the embedded navigation system.

!!! NOTE
As of library v.@![iOS]6.2!@@![android, javaSE, javaEE]4.8!@ and SDL Core v.5.1, the `SendLocation` RPC can be sent from one SDL app to another. Both SDL apps will have to implement the **App Services** API. Please refer to [Other SDL Features/Creating an App Service](Other SDL Features/Creating an App Service) and [Other SDL Features/Using an App Service](Other SDL Features/Using an App Service) for more information on how to implement this feature.
!!!

## Checking if App has Permission to Use Send Location
The `SendLocation` RPC is restricted by most vehicle manufacturers. As a result, the head unit you are connecting to will reject the request if you do not have the correct permissions. Please check the [Getting Started/Understanding Permissions](Getting Started/Understanding Permissions) section for more information on how to check permissions for an RPC.

## Checking if Head Unit Supports Send Location 
Since is a possibility that some head units will **not** support the send location feature, you should check head unit support before attempting to send the request. 

If connecting to SDL Core v.4.5 or newer and using library v.@![iOS]6.0!@@![android, javaSE, javaEE]4.4!@, you can use the @`SDLSystemCapabilityManager` to check the navigation capability returned by Core as shown in the code sample. 

If connecting to older versions of Core (or using older versions of the library), you will have to check the @![iOS]`SDLManager.registerResponse.hmiCapabilities.navigation`!@ @![android, javaSE, javaEE]`SdlManager.registerAppInterfaceResponse.hmiCapabilities.isNavigationAvailable`!@ after the SDL app has started successfully.

@![iOS]
##### Objective-C
```objc
[self.sdlManager.systemCapabilityManager updateCapabilityType:SDLSystemCapabilityTypeNavigation completionHandler:^(NSError * _Nullable error, SDLSystemCapabilityManager * _Nonnull systemCapabilityManager) {
    BOOL isNavigationSupported = systemCapabilityManager.navigationCapability.sendLocationEnabled.boolValue;
}];
```

##### Swift
```swift
sdlManager.systemCapabilityManager.updateCapabilityType(.navigation) { (error, systemCapabilityManager) in
    let isNavigationSupported = systemCapabilityManager.navigationCapability?.sendLocationEnabled?.boolValue;
}
```
!@

@![android, javaSE, javaEE]
```java
sdlManager.getSystemCapabilityManager().getCapability(SystemCapabilityType.NAVIGATION, new OnSystemCapabilityListener() {
	@Override
	public void onCapabilityRetrieved(Object capability) {
		NavigationCapability navCapability = (NavigationCapability)capability;
		Boolean isNavigationSupported = navCapability.getSendLocationEnabled();
	}

	@Override
	public void onError(String info) {

	}
});
```
!@

## Using Send Location
To use the `SendLocation` request, you must at minimum include the longitude and latitude of the location.

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
