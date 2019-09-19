# Setting the Navigation Destination
The `SendLocation` RPC gives you the ability to send a GPS location to the active navigation app on the head unit.

When using the `SendLocation` RPC, you will not have access to any information about how the user interacted with this location, only if the request was successfully sent. The request will be handled by Core from that point on using the active navigation system.

## Checking If Your App Has Permission to Use SendLocation
The `SendLocation` RPC is restricted by most vehicle manufacturers. As a result, the head unit you are connecting to will reject the request if you do not have the correct permissions. Please check the [Understanding Permissions](Getting Started/Understanding Permissions) section for more information on how to check permissions for an RPC.

## Checking if Head Unit Supports SendLocation
Since there is a possibility that some head units will not support the send location feature, you should check head unit support before attempting to send the request. You should also update your app's UI based on whether or not you can use `SendLocation`.

If using library v.@![iOS]6.0!@@![android, javaSE, javaEE]4.4!@+, you can use the @![iOS]`SDLSystemCapabilityManager`!@@![android, javaSE, javaEE]`SystemCapabilityManager`!@ to check the navigation capability returned by Core as shown in the code sample below.

@![iOS]
##### Objective-C
```objc
[self.sdlManager.systemCapabilityManager updateCapabilityType:SDLSystemCapabilityTypeNavigation completionHandler:^(NSError * _Nullable error, SDLSystemCapabilityManager * _Nonnull systemCapabilityManager) {
    BOOL isNavigationSupported = NO;
    if (error == nil) {
        isNavigationSupported = systemCapabilityManager.navigationCapability.sendLocationEnabled.boolValue;
    }
    else {
        isNavigationSupported = systemCapabilityManager.hmiCapabilities.navigation.boolValue;
    }

    <#If navigation is supported, send the `SendLocation` RPC#>
}];
```

##### Swift
```swift
sdlManager.systemCapabilityManager.updateCapabilityType(.navigation) { (error, systemCapabilityManager) in
    var isNavigationSupported = false
    if error == nil {
        isNavigationSupported = systemCapabilityManager.navigationCapability?.sendLocationEnabled?.boolValue ?? false;
    } else {
        isNavigationSupported = systemCapabilityManager.hmiCapabilities?.navigation?.boolValue ?? false
    }

    <#If navigation is supported, send the `SendLocation` RPC#>
}
```
!@

@![android, javaSE, javaEE]
```java
sdlManager.getSystemCapabilityManager().getCapability(SystemCapabilityType.NAVIGATION, new OnSystemCapabilityListener() {
	@Override
	public void onCapabilityRetrieved(Object capability) {
		NavigationCapability navCapability = (NavigationCapability) capability;
		Boolean isNavigationSupported = navCapability.getSendLocationEnabled();
	}

	@Override
	public void onError(String info) {
		HMICapabilities hmiCapabilities = (HMICapabilities) sdlManager.getSystemCapabilityManager().getCapability(SystemCapabilityType.HMI);
		Boolean isNavigationSupported = hmiCapabilities.isNavigationAvailable();
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
        <#Encountered error sending SendLocation#>
        return;
    }

    SDLSendLocationResponse *sendLocation = (SDLSendLocationResponse *)response;
    SDLResult resultCode = sendLocation.resultCode;
    if (![resultCode isEqualToEnum:SDLResultSuccess]) {
        if ([resultCode isEqualToEnum:SDLResultInvalidData]) {
            <#SendLocation was rejected. The request contained invalid data.#>
        } else if ([resultCode isEqualToEnum:SDLResultDisallowed]) {
            <#Your app is not allowed to use SendLocation#>
        } else {
            <#Some unknown error has occured#>
        }
        return;
    }

    <#SendLocation successfully sent#>
}];
```

##### Swift
```swift
let sendLocation = SDLSendLocation(longitude: -97.380967, latitude: 42.877737, locationName: "The Center", locationDescription: "Center of the United States", address: ["900 Whiting Dr", "Yankton, SD 57078"], phoneNumber: nil, image: nil)

sdlManager.send(request: sendLocation) { (request, response, error) in
    guard let response = response as? SDLSendLocationResponse, error == nil else {
        <#Encountered error sending SendLocation#>
        return
    }

    guard response.resultCode == .success else {
        switch response.resultCode {
        case .invalidData:
            <#SendLocation was rejected. The request contained invalid data.#>
        case .disallowed:
            <#Your app is not allowed to use SendLocation#>
        default:
            <#Some unknown error has occured#>
        }
        return
    }

    <#SendLocation successfully sent#>
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

    @Override
    public void onError(int correlationId, Result resultCode, String info){
        Log.e(TAG, "onError: "+ resultCode+ " | Info: "+ info );
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
