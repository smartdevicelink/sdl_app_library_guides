# Calling a Phone Number
The @![iOS]`SDLDialNumber`!@@![android,javaSE,javaEE]`DialNumber`!@ RPC allows you make a phone call via the user's phone. In order to make dial a phone number you must be sure that the device is connected via Bluetooth (even if using a USB cord) for this request to work. If the phone is not connected via Bluetooth, you will receive a result of `REJECTED` from the module.

## Checking Your App's Permissions
@![iOS]`SDLDialNumber`!@@![android,javaSE,javaEE]`DialNumber`!@ is an RPC that is usually restricted by OEMs. As a result, a module may reject your request if your app does not have the correct permissions. Your SDL app may also be restricted to only being allowed to making a phone call when your app is open (i.e. the `hmiLevel` is non-`NONE`) or when it is the currently active app (i.e. the `hmiLevel` is `FULL`). 

@![iOS]
##### Objective-C
```objc
// TODO
```

##### Swift
```swift
let observerId  = sdlManager.permissionManager.addObserver(forRPCs: [SDLRPCFunctionName.dialNumber.rawValue.rawValue], groupType: .any, withHandler: { (allChanges, groupStatus) in
    // This handler will be called whenever the permission status changes
    guard groupStatus == SDLPermissionGroupStatus.allowed else { 
        // Your app does not have permission to send the `DialNumber` request for its current HMI level
        return 
    }

    // Your app has permission to send the `DialNumber` request for its current HMI level
})
```
!@

@![android,javaSE,javaEE]
```java
// TODO
```
!@

## Checking if the Module Supports Calling a Phone Number
Since making a phone call is a newer feature, there is a possibility that some legacy modules will reject your request because the module does not support the @![iOS]`SDLDialNumber`!@@![android,javaSE,javaEE]`DialNumber`!@ request. Once you have successfully connected to the module and you know that your SDL app has been opened, you can check the module's capabilities via the @![iOS]`SDLManager.systemCapabilityManager`!@@![android, javaSE, javaEE]`sdlManager.getSystemCapabilityManager`!@ as shown in the example below. Please note that you only need to check once if the module supports calling a phone number, however you must wait to perform this check until you know that the SDL app has been opened (i.e. the `hmiLevel` is non-`NONE`). 

If you discover that the module does not support calling a phone number or that your app does not have the right permissions, you should disable any buttons, voice commands, menu items, etc. in your app that would send the @![iOS]`SDLDialNumber`!@@![android,javaSE,javaEE]`DialNumber`!@ request.

@![iOS]
##### Objective-C
```objc
// TODO
```

##### Swift
```swift
func isDialNumberSupported(handler: @escaping (_ success: Bool, _ error: Error?) -> ()) {
    // Check if the module has phone capabilities
    guard (sdlManager.systemCapabilityManager.isCapabilitySupported(type: .phoneCall)) else {
        return handler(false, nil)
    }

    // Legacy modules (pre RPC Spec v4.5) do not support the phone call capability so for versions less than 4.5 we will assume `DialNumber` is supported
    guard let sdlMsgVersion = sdlManager.registerResponse?.sdlMsgVersion, SDLVersion(sdlMsgVersion: sdlMsgVersion).isGreaterThanOrEqual(to: SDLVersion(major: 4, minor: 5, patch: 0)) else {
        return handler(true, nil)
    }

    // Check if the module supports the `DialNumber` request
    sdlManager.systemCapabilityManager.updateCapabilityType(.phoneCall) { (error, systemCapabilityManager) in
        if (error != nil) {
            return handler(false, error)
        }

        let isDialNumberSupported: Bool
        if let phoneCapability = systemCapabilityManager.phoneCapability {
            isDialNumberSupported = phoneCapability.dialNumberEnabled?.boolValue ?? false
        } else {
            isDialNumberSupported = false
        }
        return handler(isDialNumberSupported, nil)
    }
}
```
!@

@![android,javaSE,javaEE]
```java
// TODO
```
!@

## Sending a Dial Number Request
Once you know that the module supports dialing a phone number and that your SDL app has permission to send a @![iOS]`SDLDialNumber`!@@![android,javaSE,javaEE]`DialNumber`!@ request, you can create and send the request. 

!!! note
@![iOS]`SDLDialNumber`!@@![android,javaSE,javaEE]`DialNumber`!@ strips all characters except for `0`-`9`, `*`, `#`, `,`, `;`, and `+`.
!!!

@![iOS]
##### Objective-C
```objc
SDLDialNumber *dialNumber = [[SDLDialNumber alloc] initWithNumber: @"1238675309"];

[self.sdlManager sendRequest:dialNumber withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if (error != nil || ![response isKindOfClass:SDLDialNumberResponse.class]) {
        <#Encountered error sending DialNumber#>
        return;
    }

    SDLDialNumberResponse* dialNumber = (SDLDialNumberResponse *)response;
    SDLResult *resultCode = dialNumber.resultCode;
    if (!resultCode.success.boolValue) {
        if ([resultCode isEqualToEnum:SDLResultRejected]) {
            <#DialNumber was rejected. Either the call was sent and cancelled or there is no device connected#>
        } else if ([resultCode isEqualToEnum:SDLResultDisallowed]) {
            <#Your app is not allowed to use DialNumber#>
        } else {
            <#Some unknown error has occurred#>
        }
        return;
    }

    <#Dial number successfully sent#>
}];
```

##### Swift
```swift
let dialNumber = SDLDialNumber(number: "1238675309")

sdlManager.send(request: dialNumber) { (request, response, error) in
    guard let response = response as? SDLDialNumberResponse, error == nil else {
        <#Encountered error sending DialNumber#>
        return
    }

    guard response?.success.boolValue == true else {
        switch response.resultCode {
        case .rejected:
            <#DialNumber was rejected. Either the call was sent and cancelled or there is no device connected#>
        case .disallowed:
            <#Your app is not allowed to use DialNumber#>
        default:
            <#Some unknown error has occurred#>
        }
        return
    }

    <#Dial number successfully sent#>
}
```
!@

@![android,javaSE,javaEE]
```java
DialNumber dialNumber = new DialNumber();
dialNumber.setNumber("1238675309");
dialNumber.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse response) {
        Result result = response.getResultCode();
        if(result.equals(Result.SUCCESS)){
            // `DialNumber` was successfully sent, and a phone call was initiated by the user.
        }else if(result.equals(Result.REJECTED)){
            // `DialNumber` was sent, and a phone call was cancelled by the user. Also, this could mean that there is no phone connected via Bluetooth.
        }else if(result.equals(Result.DISALLOWED)){
            // Your app does not have permission to use DialNumber.
        }
    }

    @Override
    public void onError(int correlationId, Result resultCode, String info){
        Log.e(TAG, "onError: "+ resultCode+ " | Info: "+ info );
    }
});

sdlManager.sendRPC(dialNumber);
```
!@

### Dial Number Responses
The @![iOS]`SDLDialNumber`!@@![android,javaSE,javaEE]`DialNumber`!@ request has three possible responses that you should expect:

1. `SUCCESS` - The request was successfully sent, and a phone call was initiated by the user.
1. `REJECTED` - This can mean either: 

    * The user rejected the request to make the phone call. 
    * The phone is not connected to the module via Bluetooth.

1. `DISALLOWED` - Your app does not have permission to use the @![iOS]`SDLDialNumber`!@@![android,javaSE,javaEE]`DialNumber`!@ request.

