# Setting the Navigation Destination
The @![iOS]`SDLSendLocation`!@@![android,javaSE,javaEE,javascript]`SendLocation`!@ RPC gives you the ability to send a GPS location to the active navigation app on the module.

When using the @![iOS]`SDLSendLocation`!@@![android,javaSE,javaEE,javascript]`SendLocation`!@ RPC, you will not have access to any information about how the user interacted with this location, only if the request was successfully sent. The request will be handled by the module from that point on using the active navigation system.

## Checking Your App's Permissions
The @![iOS]`SDLSendLocation`!@@![android,javaSE,javaEE,javascript]`SendLocation`!@ RPC is restricted by most OEMs. As a result, a module may reject your request if your app does not have the correct permissions. Your SDL app may also be restricted to only being allowed to send a location when your app is open (i.e. the `hmiLevel` is non-`NONE`) or when it is the currently active app (i.e. the `hmiLevel` is `FULL`). 

@![iOS]
##### Objective-C
```objc
id observerId = [self.sdlManager.permissionManager addObserverForRPCs:@[SDLRPCFunctionNameSendLocation] groupType:SDLPermissionGroupTypeAny withHandler:^(NSDictionary<SDLPermissionRPCName,NSNumber *> * _Nonnull allChanges, SDLPermissionGroupStatus groupStatus) {
    if (groupStatus != SDLPermissionGroupStatusAllowed) {
        // Your app does not have permission to send the `SDLSendLocation` request for its current HMI level
        return;
    }

    // Your app has permission to send the `SDLSendLocation` request for its current HMI level
}];
```

##### Swift
```swift
let observerId = sdlManager.permissionManager.addObserver(forRPCs: [SDLRPCFunctionName.sendLocation.rawValue.rawValue], groupType: .any, withHandler: { (allChanges, groupStatus) in
    // This handler will be called whenever the permission status changes
    guard groupStatus == .allowed else {
        // Your app does not have permission to send the `SDLSendLocation` request for its current HMI level
        return
    }

    // Your app has permission to send the `SDLSendLocation` request for its current HMI level
})
```
!@

@![android,javaSE,javaEE]
```java
UUID listenerId = sdlManager.getPermissionManager().addListener(Arrays.asList(new PermissionElement(FunctionID.SEND_LOCATION, null)), PermissionManager.PERMISSION_GROUP_TYPE_ANY, new OnPermissionChangeListener() {
    @Override
    public void onPermissionsChange(@NonNull Map<FunctionID, PermissionStatus> allowedPermissions, @NonNull int permissionGroupStatus) {
        if (permissionGroupStatus != PermissionManager.PERMISSION_GROUP_TYPE_ANY) {
            // Your app does not have permission to send the `SendLocation` request for its current HMI level
            return;
        }

        // Your app has permission to send the `SendLocation` request for its current HMI level
    }
});
```
!@

@![javascript]
```js
const permissionElements = [];
permissionElements.push(new SDL.manager.permission.PermissionElement(SDL.rpc.enums.FunctionID.SendLocation, null));

const listenerId = sdlManager.getPermissionManager().addListener(permissionElements, SDL.manager.permission.enums.PermissionGroupType.ANY, function (allowedPermissions, permissionGroupStatus) {
    if (permissionGroupStatus != SDL.manager.permission.enums.PermissionGroupStatus.ALLOWED) {
        // Your app does not have permission to send the `SendLocation` request for its current HMI level
        return;
    }

    // Your app has permission to send the `SendLocation` request for its current HMI level
});
```
!@

## Checking if the Module Supports Sending a Location
Since some modules will not support sending a location, you should check if the module supports this feature before trying to use it. Once you have successfully connected to the module, you can check the module's capabilities via the @![iOS]`SDLManager.systemCapabilityManager`!@@![android,javaSE,javaEE,javascript]`sdlManager.getSystemCapabilityManager()`!@ as shown in the example below. Please note that you only need to check once if the module supports sending a location, however you must wait to perform this check until you know that the SDL app has been opened (i.e. the `hmiLevel` is non-`NONE`).  

!!! NOTE
If you discover that the module does not support sending a location or that your app does not have the right permissions, you should disable any buttons, voice commands, menu items, etc. in your app that would send the @![iOS]`SDLSendLocation`!@@![android,javaSE,javaEE,javascript]`SendLocation`!@ request.
!!!

@![iOS]
##### Objective-C
```objc
- (void)isSendLocationSupportedWithHandler:(void (^) (BOOL success, NSError * _Nullable error))handler {
    // Check if the module has navigation capabilities
    if (![self.sdlManager.systemCapabilityManager isCapabilitySupported:SDLSystemCapabilityTypeNavigation]) {
        return handler(false, nil);
    }

    // Legacy modules (pre-RPC Spec v4.5) do not support system capabilities, so for versions less than 4.5 we will assume `SDLSendLocation` is supported if `isCapabilitySupported` returns true
    SDLMsgVersion *sdlMsgVersion = self.sdlManager.registerResponse.sdlMsgVersion;
    if (sdlMsgVersion == nil) {
        return handler(true, nil);
    }
    SDLVersion *rpcSpecVersion = [[SDLVersion alloc] initWithSDLMsgVersion:sdlMsgVersion];
    if (![rpcSpecVersion isGreaterThanOrEqualToVersion:[[SDLVersion alloc] initWithMajor:4 minor:5 patch:0]]) {
        return handler(true, nil);
    }

    // Check if the navigation capability has already been retrieved from the module
    SDLNavigationCapability *navigationCapability = self.sdlManager.systemCapabilityManager.navigationCapability;
    if (navigationCapability != nil) {
        return handler(navigationCapability.sendLocationEnabled.boolValue, nil);
    }

    // Retrieve the navigation capability from the module
    [self.sdlManager.systemCapabilityManager updateCapabilityType:SDLSystemCapabilityTypeNavigation completionHandler:^(NSError * _Nullable error, SDLSystemCapabilityManager * _Nonnull systemCapabilityManager) {
        if (error != nil) {
            return handler(NO, error);
        }

        return handler(systemCapabilityManager.navigationCapability.sendLocationEnabled.boolValue, nil);
    }];
}
```

##### Swift
```swift
func isSendLocationSupported(handler: @escaping (_ success: Bool, _ error: Error?) -> Void) {
    // Check if the module has navigation capabilities
    guard sdlManager.systemCapabilityManager.isCapabilitySupported(type: .navigation) else {
        return handler(false, nil)
    }

    // Legacy modules (pre-RPC Spec v4.5) do not support system capabilities, so for versions less than 4.5 we will assume `SDLSendLocation` is supported if `isCapabilitySupported` returns true
    guard let sdlMsgVersion = sdlManager.registerResponse?.sdlMsgVersion, SDLVersion(sdlMsgVersion: sdlMsgVersion).isGreaterThanOrEqual(to: SDLVersion(major: 4, minor: 5, patch: 0)) else {
        return handler(true, nil)
    }

    // Check if the navigation capability has already been retrieved from the module
    if let navigationCapability = sdlManager.systemCapabilityManager.navigationCapability {
        return handler(navigationCapability.sendLocationEnabled?.boolValue ?? false, nil)
    }

    // Retrieve the navigation capability from the module
    sdlManager.systemCapabilityManager.updateCapabilityType(.navigation) { (error, systemCapabilityManager) in
        if (error != nil) {
            return handler(false, error)
        }

        return handler(systemCapabilityManager.navigationCapability?.sendLocationEnabled?.boolValue ?? false, nil)
    }
}
```
!@

@![android, javaSE, javaEE]
```java
private void isSendLocationSupported(final OnCapabilitySupportedListener capabilitySupportedListener) {
    // Check if the module has navigation capabilities
    if (!sdlManager.getSystemCapabilityManager().isCapabilitySupported(SystemCapabilityType.NAVIGATION)) {
        capabilitySupportedListener.onCapabilitySupported(false);
        return;
    }

    // Legacy modules (pre-RPC Spec v4.5) do not support system capabilities, so for versions less than 4.5 we will assume `SendLocation` is supported if `isCapabilitySupported()` returns true
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

    // Retrieve the navigation capability
    sdlManager.getSystemCapabilityManager().getCapability(SystemCapabilityType.NAVIGATION, new OnSystemCapabilityListener() {
        @Override
        public void onCapabilityRetrieved(Object capability) {
            NavigationCapability navigationCapability = (NavigationCapability) capability;
            capabilitySupportedListener.onCapabilitySupported(navigationCapability != null ? navigationCapability.getSendLocationEnabled() : false);
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

@![javascript]
```js
async isSendLocationSupported() {
    // Check if the module has navigation capabilities
    if (!sdlManager.getSystemCapabilityManager()._getCapabilityMethodForType(SDL.rpc.enums.SystemCapabilityType.NAVIGATION)) {
        return false;
    }

    // Legacy modules (pre-RPC Spec v4.5) do not support system capabilities, so for versions less than 4.5 we will assume `SendLocation` is supported if `getCapabilityMethodForType` returns true
    let sdlMsgVersion = sdlManager.getRegisterAppInterfaceResponse().getSdlMsgVersion();
    if (sdlMsgVersion == null) {
        return true;
    }
    let rpcSpecVersion = new SDL.util.Version(sdlMsgVersion.getMajorVersion(), sdlMsgVersion.getMinorVersion(), sdlMsgVersion.getPatchVersion());
    if (rpcSpecVersion.isNewerThan(new SDL.util.Version(4, 5, 0)) < 0) {
        return true;
    }

    // Retrieve the navigation capability
    let isNavigationSupported = false;
    const navCapability = await sdlManager.getSystemCapabilityManager().updateCapability(SDL.rpc.enums.SystemCapabilityType.NAVIGATION)
        .catch(error => {
            throw error;
        });
    if (navCapability !== null) {
        isNavigationSupported = navCapability.getSendLocationEnabled();
    }

    return isNavigationSupported;
}
```
!@

## Using Send Location
To use the @![iOS]`SDLSendLocation`!@@![android,javaSE,javaEE,javascript]`SendLocation`!@ request, you must at minimum include the longitude and latitude of the location.

@![iOS]
##### Objective-C
```objc
SDLSendLocation *sendLocation = [[SDLSendLocation alloc] initWithLongitude:-97.380967 latitude:42.877737 locationName:@"The Center" locationDescription:@"Center of the United States" address:@[@"900 Whiting Dr", @"Yankton, SD 57078"] phoneNumber:nil image:nil];

[self.sdlManager sendRequest:sendLocation withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if (error != nil) {
        // Encountered an error sending `SendLocation`
        return;
    }

    SDLSendLocationResponse *sendLocation = (SDLSendLocationResponse *)response;
    SDLResult resultCode = sendLocation.resultCode;
    if (!sendLocation.success) {
        if ([resultCode isEqualToEnum:SDLResultInvalidData]) {
            // `SDLSendLocation` was rejected. The request contained invalid data
        } else if ([resultCode isEqualToEnum:SDLResultDisallowed]) {
            // Your app is not allowed to use `SDLSendLocation`
        } else {
            // Some unknown error has occurred
        }
        return;
    }

    // `SDLSendLocation` successfully sent
}];
```

##### Swift
```swift
let sendLocation = SDLSendLocation(longitude: -97.380967, latitude: 42.877737, locationName: "The Center", locationDescription: "Center of the United States", address: ["900 Whiting Dr", "Yankton, SD 57078"], phoneNumber: nil, image: nil)

sdlManager.send(request: sendLocation) { (request, response, error) in
    guard let response = response as? SDLSendLocationResponse else {
        // Encountered an error sending `SendLocation`
        return
    }

    guard response.success.boolValue == true else {
        case .invalidData:
            // `SDLSendLocation` was rejected. The request contained invalid data
        case .disallowed:
            // Your app is not allowed to use `SDLSendLocation`
        default: break
            // Some unknown error has occurred
        }
        return
    }

    // `SDLSendLocation` successfully sent
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

OasisAddress address = new OasisAddress();
address.setSubThoroughfare("900");
address.setThoroughfare("Whiting Dr");
address.setLocality("Yankton");
address.setAdministrativeArea("SD");
address.setPostalCode("57078");
address.setCountryCode("US-SD");
address.setCountryName("United States");

sendLocation.setAddress(address);

sendLocation.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse response) {
        Result result = response.getResultCode();
        if(result.equals(Result.SUCCESS)){
            // `SendLocation` successfully sent
        }else if(result.equals(Result.INVALID_DATA)){
            // `SendLocation` was rejected. The request contained invalid data
        }else if(result.equals(Result.DISALLOWED)){
            // Your app is not allowed to use `SendLocation`
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

@![javascript]
```js
const sendLocation = new SDL.rpc.messages.SendLocation()
    .setLatitudeDegrees(42.877737)
    .setLongitudeDegrees(-97.380967)
    .setLocationName('The Center')
    .setLocationDescription('Center of the United States');

const address = new SDL.rpc.structs.OasisAddress()
    .setSubThoroughfare('900')
    .setThoroughfare('Whiting Dr')
    .setLocality('Yankton')
    .setAdministrativeArea('SD')
    .setPostalCode('57078')
    .setCountryCode('US-SD')
    .setCountryName('United States');

sendLocation.setAddress(address);

const response = await sdlManager.sendRpc(sendLocation).catch(error => error);

const result = response.getResultCode();
if (result === SDL.rpc.enums.Result.SUCCESS) {
    // `SendLocation` was successfully sent
} else if (result === SDL.rpc.enums.Result.INVALID_DATA) {
    // `SendLocation` was rejected. The request contained invalid data
} else if (result === SDL.rpc.enums.Result.DISALLOWED) {
    // Your app is not allowed to use `SendLocation`
}
```
!@

## Checking the Result of Send Location
The @![iOS]`SDLSendLocation`!@@![android,javaSE,javaEE,javascript]`SendLocation`!@ request has three possible responses that you should expect:

1. `SUCCESS` - Successfully sent.
1. `INVALID_DATA` - The request contains invalid data and was rejected.
1. `DISALLOWED` - Your app does not have permission to use the @![iOS]`SDLSendLocation`!@@![android,javaSE,javaEE,javascript]`SendLocation`!@ request.
