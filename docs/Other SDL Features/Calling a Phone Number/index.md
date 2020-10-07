# Calling a Phone Number
The @![iOS]`SDLDialNumber`!@@![android,javaSE,javaEE]`DialNumber`!@ RPC allows you make a phone call via the user's phone. In order to dial a phone number you must be sure that the device is connected via Bluetooth (even if your device is also connected using a USB cord) for this request to work. If the phone is not connected via Bluetooth, you will receive a result of `REJECTED` from the module.

## Checking Your App's Permissions
@![iOS]`SDLDialNumber`!@@![android,javaSE,javaEE]`DialNumber`!@ is an RPC that is usually restricted by OEMs. As a result, a module may reject your request if your app does not have the correct permissions. Your SDL app may also be restricted to only being allowed to making a phone call when your app is open (i.e. the `hmiLevel` is non-`NONE`) or when it is the currently active app (i.e. the `hmiLevel` is `FULL`). 

@![iOS]
##### Objective-C
```objc
id observerId = [self.sdlManager.permissionManager addObserverForRPCs:@[SDLRPCFunctionNameDialNumber] groupType:SDLPermissionGroupTypeAny withHandler:^(NSDictionary<SDLPermissionRPCName,NSNumber *> * _Nonnull allChanges, SDLPermissionGroupStatus groupStatus) {
    if (groupStatus != SDLPermissionGroupStatusAllowed) {
        // Your app does not have permission to send the `SDLDialNumber` request for its current HMI level
        return;
    }

    // Your app has permission to send the `SDLDialNumber` request for its current HMI level
}];
```

##### Swift
```swift
let observerId = sdlManager.permissionManager.addObserver(forRPCs: [SDLRPCFunctionName.dialNumber.rawValue.rawValue], groupType: .any, withHandler: { (allChanges, groupStatus) in
    guard groupStatus == .allowed else { 
        // Your app does not have permission to send the `SDLDialNumber` request for its current HMI level
        return 
    }

    // Your app has permission to send the `SDLDialNumber` request for its current HMI level
})
```
!@

@![android,javaSE,javaEE]
```java
UUID listenerId = sdlManager.getPermissionManager().addListener(Arrays.asList(new PermissionElement(FunctionID.DIAL_NUMBER, null)), PermissionManager.PERMISSION_GROUP_TYPE_ANY, new OnPermissionChangeListener() {
    @Override
    public void onPermissionsChange(@NonNull Map<FunctionID, PermissionStatus> allowedPermissions, @NonNull int permissionGroupStatus) {
        if (permissionGroupStatus != PermissionManager.PERMISSION_GROUP_TYPE_ALL_ALLOWED) {
            // Your app does not have permission to send the `DialNumber` request for its current HMI level
            return;
        }

        // Your app has permission to send the `DialNumber` request for its current HMI level
    }
});
```
!@

## Checking if the Module Supports Calling a Phone Number
Since making a phone call is a newer feature, there is a possibility that some legacy modules will reject your request because the module does not support the @![iOS]`SDLDialNumber`!@@![android,javaSE,javaEE]`DialNumber`!@ request. Once you have successfully connected to the module, you can check the module's capabilities via the @![iOS]`SDLManager.systemCapabilityManager`!@@![android, javaSE, javaEE]`sdlManager.getSystemCapabilityManager`!@ as shown in the example below. Please note that you only need to check once if the module supports calling a phone number, however you must wait to perform this check until you know that the SDL app has been opened (i.e. the `hmiLevel` is non-`NONE`). 

!!! NOTE
If you discover that the module does not support calling a phone number or that your app does not have the right permissions, you should disable any buttons, voice commands, menu items, etc. in your app that would send the @![iOS]`SDLDialNumber`!@@![android,javaSE,javaEE]`DialNumber`!@ request.
!!!

@![iOS]
##### Objective-C
```objc
- (void)isDialNumberSupportedWithHandler:(void (^) (BOOL success, NSError * _Nullable error))handler {
    // Check if the module has phone capabilities
    if (![self.sdlManager.systemCapabilityManager isCapabilitySupported:SDLSystemCapabilityTypePhoneCall]) {
        return handler(false, nil);
    }

    // Legacy modules (pre-RPC Spec v4.5) do not support system capabilities, so for versions less than 4.5 we will assume `DialNumber` is supported if `isCapabilitySupported` returns true
    SDLMsgVersion *sdlMsgVersion = self.sdlManager.registerResponse.sdlMsgVersion;
    if (sdlMsgVersion == nil) {
        return handler(true, nil);
    }
    SDLVersion *rpcSpecVersion = [[SDLVersion alloc] initWithSDLMsgVersion:sdlMsgVersion];
    if (![rpcSpecVersion isGreaterThanOrEqualToVersion:[[SDLVersion alloc] initWithMajor:4 minor:5 patch:0]]) {
        return handler(true, nil);
    }

    // Check if the phone capability has already been retrieved from the module
    SDLPhoneCapability *phoneCapability = self.sdlManager.systemCapabilityManager.phoneCapability;
    if (phoneCapability != nil) {
        return handler(phoneCapability.dialNumberEnabled.boolValue, nil);
    }

    // Retrieve the phone capability from the module
    [self.sdlManager.systemCapabilityManager updateCapabilityType:SDLSystemCapabilityTypePhoneCall completionHandler:^(NSError * _Nullable error, SDLSystemCapabilityManager * _Nonnull systemCapabilityManager) {
        if (error != nil) {
            return handler(NO, error);
        }

        return handler(systemCapabilityManager.phoneCapability.dialNumberEnabled.boolValue, nil);
    }];
}
```

##### Swift
```swift
func isDialNumberSupported(handler: @escaping (_ success: Bool, _ error: Error?) -> Void) {
    // Check if the module has phone capabilities
    guard sdlManager.systemCapabilityManager.isCapabilitySupported(type: .phoneCall) else {
        return handler(false, nil)
    }

    // Legacy modules (pre-RPC Spec v4.5) do not support system capabilities, so for versions less than 4.5 we will assume `SDLDialNumber` is supported if `isCapabilitySupported` returns true
    guard let sdlMsgVersion = sdlManager.registerResponse?.sdlMsgVersion, SDLVersion(sdlMsgVersion: sdlMsgVersion).isGreaterThanOrEqual(to: SDLVersion(major: 4, minor: 5, patch: 0)) else {
        return handler(true, nil)
    }

    // Check if the phone capability has already been retrieved from the module
    if let phoneCapability = sdlManager.systemCapabilityManager.phoneCapability {
        return handler(phoneCapability.dialNumberEnabled?.boolValue ?? false, nil)
    }

    // Retrieve the phone capability from the module
    sdlManager.systemCapabilityManager.updateCapabilityType(.phoneCall) { (error, systemCapabilityManager) in
        if (error != nil) {
            return handler(false, error)
        }

        return handler(systemCapabilityManager.phoneCapability?.dialNumberEnabled?.boolValue ?? false, nil)
    }
}
```
!@

@![android,javaSE,javaEE]
```java
private void isDialNumberSupported(final OnCapabilitySupportedListener capabilitySupportedListener) {
    // Check if the module has phone capabilities
    if (!sdlManager.getSystemCapabilityManager().isCapabilitySupported(SystemCapabilityType.PHONE_CALL)) {
        capabilitySupportedListener.onCapabilitySupported(false);
        return;
    }

    // Legacy modules (pre-RPC Spec v4.5) do not support system capabilities, so for versions less than 4.5 we will assume `DialNumber` is supported if `isCapabilitySupported()` returns true
    SdlMsgVersion sdlMsgVersion = sdlManager.getRegisterAppInterfaceResponse().getSdlMsgVersion();
    if (sdlMsgVersion == null) {
        capabilitySupportedListener.onCapabilitySupported(true);
        return;
    }
    Version rpcSpecVersion = new Version(sdlMsgVersion);
    if (rpcSpecVersion.isNewerThan(new Version(4, 5, 0)) < 0) {
        capabilitySupportedListener.onCapabilitySupported(true);
        return;
    }

    // Retrieve the phone capability
    sdlManager.getSystemCapabilityManager().getCapability(SystemCapabilityType.PHONE_CALL, new OnSystemCapabilityListener() {
        @Override
        public void onCapabilityRetrieved(Object capability) {
            PhoneCapability phoneCapability = (PhoneCapability) capability;
            capabilitySupportedListener.onCapabilitySupported(phoneCapability != null ? phoneCapability.getDialNumberEnabled() : false);
        }

        @Override
        public void onError(String info) {
            capabilitySupportedListener.onError(info);
        }
    }, false);
}

public interface OnCapabilitySupportedListener {
    void onCapabilitySupported(Boolean supported);
    void onError(String info);
}
```
!@

## Sending a DialNumber Request
Once you know that the module supports dialing a phone number and that your SDL app has permission to send the @![iOS]`SDLDialNumber`!@@![android,javaSE,javaEE]`DialNumber`!@ request, you can create and send the request. 

!!! note
@![iOS]`SDLDialNumber`!@@![android,javaSE,javaEE]`DialNumber`!@ strips all characters except for `0`-`9`, `*`, `#`, `,`, `;`, and `+`.
!!!

@![iOS]
##### Objective-C
```objc
SDLDialNumber *dialNumber = [[SDLDialNumber alloc] initWithNumber: @"1238675309"];

[self.sdlManager sendRequest:dialNumber withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if (error != nil || ![response isKindOfClass:SDLDialNumberResponse.class]) {
        // Encountered error sending `SDLDialNumber`
        return;
    }

    SDLDialNumberResponse *dialNumber = (SDLDialNumberResponse *)response;
    SDLResult *resultCode = dialNumber.resultCode;
    if (!resultCode.success.boolValue) {
        if ([resultCode isEqualToEnum:SDLResultRejected]) {
            // `SDLDialNumber` was rejected. Either the call was sent and cancelled or there is no device connected
        } else if ([resultCode isEqualToEnum:SDLResultDisallowed]) {
            // Your app is not allowed to use `SDLDialNumber`
        } else {
            // Some unknown error has occurred
        }
        return;
    }

    // `SDLDialNumber` successfully sent
}];
```

##### Swift
```swift
let dialNumber = SDLDialNumber(number: "1238675309")

sdlManager.send(request: dialNumber) { (request, response, error) in
    guard let response = response as? SDLDialNumberResponse, error == nil else {
        // Encountered error sending `SDLDialNumber`
        return
    }

    guard response?.success.boolValue == true else {
        switch response.resultCode {
        case .rejected:
            // `SDLDialNumber` was rejected. Either the call was sent and cancelled or there is no device connected
        case .disallowed:
            // Your app is not allowed to use `SDLDialNumber`
        default:
            // Some unknown error has occurred
        }
        return
    }

    // `SDLDialNumber` successfully sent
}
```
!@

@![android,javaSE,javaEE]
```java
DialNumber dialNumber = new DialNumber()
    .setNumber("1238675309");
dialNumber.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse response) {
        Result result = response.getResultCode();
        if(result.equals(Result.SUCCESS)){
            // `DialNumber` successfully sent
        }else if(result.equals(Result.REJECTED)){
            // `DialNumber` was rejected. Either the call was sent and cancelled or there is no device connected
        }else if(result.equals(Result.DISALLOWED)){
            // Your app is not allowed to use `DialNumber`
        }
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
