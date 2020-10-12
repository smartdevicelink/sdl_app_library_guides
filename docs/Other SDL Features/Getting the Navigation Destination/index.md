# Getting the Navigation Destination
The @![iOS]`SDLGetWayPoints`!@@![android,javaSE,javaEE,javascript]`GetWayPoints`!@ and @![iOS]`SDLSubscribeWayPoints`!@@![android,javaSE,javaEE,javascript]`SubscribeWayPoints`!@ RPCs are designed to allow you to get the navigation destination(s) from the active navigation app when the user has activated in-car navigation.

## Checking Your App's Permissions
Both the @![iOS]`SDLGetWayPoints`!@@![android,javaSE,javaEE,javascript]`GetWayPoints`!@ and @![iOS]`SDLSubscribeWayPoints`!@@![android,javaSE,javaEE,javascript]`SubscribeWayPoints`!@ RPCs are restricted by most OEMs. As a result, a module may reject your request if your app does not have the correct permissions. Your SDL app may also be restricted to only being allowed to get waypoints when your app is open (i.e. the `hmiLevel` is non-`NONE`) or when it is the currently active app (i.e. the `hmiLevel` is `FULL`). 

@![iOS]
##### Objective-C
```objc
id observerId = [self.sdlManager.permissionManager addObserverForRPCs:@[SDLRPCFunctionNameGetWayPoints, SDLRPCFunctionNameSubscribeWayPoints] groupType:SDLPermissionGroupTypeAny withHandler:^(NSDictionary<SDLPermissionRPCName,NSNumber *> * _Nonnull allChanges, SDLPermissionGroupStatus groupStatus) {
    // This handler will be called whenever the permission status changes
    BOOL getWayPointPermissionStatus = allChanges[SDLRPCFunctionNameGetWayPoints].boolValue;
    if (getWayPointPermissionStatus.boolValue) {
        // Your app has permission to send the `SDLGetWayPoints` request for its current HMI level
    } else {
        // Your app does not have permission to send the `SDLGetWayPoints` request for its current HMI level
    }

    BOOL subscribeWayPointsPermissionStatus = allChanges[SDLRPCFunctionNameSubscribeWayPoints].boolValue;
    if (subscribeWayPointsPermissionStatus.boolValue) {
        // Your app has permission to send the `SDLSubscribeWayPoints` request for its current HMI level
    } else {
        // Your app does not have permission to send the `SDLSubscribeWayPoints` request for its current HMI level
    }
}];
```

##### Swift
```swift
let getWayPointsPermissionElement = SDLPermissionElement(rpcName: SDLRPCFunctionName.getWayPoints, parameterPermissions: nil)
let subscribeWayPointsPermissionElement = SDLPermissionElement(rpcName: SDLRPCFunctionName.subscribeWayPoints, parameterPermissions: nil)

let observerId = sdlManager.permissionManager.subscribe(toRPCPermissions: [getWayPointsPermissionElement, subscribeWayPointsPermissionElement], groupType: .any, withHandler: { (allChanges, groupStatus) in
    // This handler will be called whenever the permission status changes
    if let getWayPointPermissionStatus = allChanges[SDLRPCFunctionName.getWayPoints], getWayPointPermissionStatus.isRPCAllowed == true {
        // Your app has permission to send the `SDLGetWayPoints` request for its current HMI level
    } else {
        // Your app does not have permission to send the `SDLGetWayPoints` request for its current HMI level
    }

    if let subscribeWayPointsPermissionStatus = allChanges[SDLRPCFunctionName.subscribeWayPoints], subscribeWayPointsPermissionStatus.isRPCAllowed == true {
        // Your app has permission to send the `SubscribeWayPoints` request for its current HMI level
    } else {
        // Your app does not have permission to send the `SubscribeWayPoints` request for its current HMI level
    }
})
```
!@

@![android,javaSE,javaEE]
```java
UUID listenerId = sdlManager.getPermissionManager().addListener(Arrays.asList(new PermissionElement(FunctionID.GET_WAY_POINTS, null), new PermissionElement(FunctionID.SUBSCRIBE_WAY_POINTS, null)), PermissionManager.PERMISSION_GROUP_TYPE_ANY, new OnPermissionChangeListener() {
    @Override
    public void onPermissionsChange(@NonNull Map<FunctionID, PermissionStatus> allowedPermissions, @NonNull int permissionGroupStatus) {
        PermissionStatus getWayPointPermissionStatus = allowedPermissions.get(FunctionID.GET_WAY_POINTS);
        if (getWayPointPermissionStatus != null && getWayPointPermissionStatus.getIsRPCAllowed()) {
            // Your app has permission to send the `GetWayPoints` request for its current HMI level
        } else {
            // Your app does not have permission to send the `GetWayPoints` request for its current HMI level
        }

        PermissionStatus subscribeWayPointsPermissionStatus = allowedPermissions.get(FunctionID.SUBSCRIBE_WAY_POINTS);
        if (subscribeWayPointsPermissionStatus != null && subscribeWayPointsPermissionStatus.getIsRPCAllowed()) {
            // Your app has permission to send the `SubscribeWayPoints` request for its current HMI level
        } else {
            // Your app does not have permission to send the `SubscribeWayPoints` request for its current HMI level
        }
    }
});
```
!@

@![javascript]
```js
const permissionElements = [];
permissionElements.push(new SDL.manager.permission.PermissionElement(SDL.rpc.enums.FunctionID.GetWayPoints, null));
permissionElements.push(new SDL.manager.permission.PermissionElement(SDL.rpc.enums.FunctionID.SubscribeWayPoints, null));

const listenerId = sdlManager.getPermissionManager().addListener(permissionElements, SDL.manager.permission.enums.PermissionGroupType.ANY, function (allowedPermissions, permissionGroupStatus) {
    if (allowedPermissions[SDL.rpc.enums.FunctionID.GetWayPoints].getIsRpcAllowed()) {
        // Your app has permission to send the `GetWayPoints` request for its current HMI level
    } else {
        // Your app does not have permission to send the `GetWayPoints` request for its current HMI level
    }

    if (allowedPermissions[SDL.rpc.enums.FunctionID.SubscribeWayPoints].getIsRpcAllowed()) {
        // Your app has permission to send the `SubscribeWayPoints` request for its current HMI level
    } else {
        // Your app does not have permission to send the `SubscribeWayPoints` request for its current HMI level
    }
});
```
!@

## Checking if the Module Supports Waypoints 
Since some modules will not support getting waypoints, you should check if the module supports this feature before trying to use it. Once you have successfully connected to the module, you can check the module's capabilities via the @![iOS]`SDLManager.systemCapabilityManager`!@@![android, javaSE, javaEE,javascript]`sdlManager.getSystemCapabilityManager()`!@ as shown in the example below. Please note that you only need to check once if the module supports getting waypoints, however you must wait to perform this check until you know that the SDL app has been opened (i.e. the `hmiLevel` is non-`NONE`).  

!!! NOTE
If you discover that the module does not support getting navigation waypoints or that your app does not have the right permissions, you should disable any buttons, voice commands, menu items, etc. in your app that would send the @![iOS]`SDLGetWayPoints`!@@![android,javaSE,javaEE,javascript]`GetWayPoints`!@ or @![iOS]`SDLSubscribeWayPoints`!@@![android,javaSE,javaEE,javascript]`SubscribeWayPoints`!@ requests.
!!!

@![iOS]
##### Objective-C
```objc
- (void)isGetWaypointsSupportedWithHandler:(void (^) (BOOL success, NSError * _Nullable error))handler {
    // Check if the module has navigation capabilities
    if (![self.sdlManager.systemCapabilityManager isCapabilitySupported:SDLSystemCapabilityTypeNavigation]) {
        return handler(false, nil);
    }

    // Legacy modules (pre-RPC Spec v4.5) do not support system capabilities, so for versions less than 4.5 we will assume `SDLGetWayPoints` and `SDLSubscribeWayPoints` are supported if `isCapabilitySupported` returns true
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
        return handler(navigationCapability.getWayPointsEnabled.boolValue, nil);
    }

    // Retrieve the navigation capability from the module
    [self.sdlManager.systemCapabilityManager updateCapabilityType:SDLSystemCapabilityTypeNavigation completionHandler:^(NSError * _Nullable error, SDLSystemCapabilityManager * _Nonnull systemCapabilityManager) {
        if (error != nil) {
            return handler(NO, error);
        }

        return handler(systemCapabilityManager.navigationCapability.getWayPointsEnabled.boolValue, nil);
    }];
}
```

##### Swift
```swift
func isGetWaypointsSupported(handler: @escaping (_ success: Bool, _ error: Error?) -> Void) {
    // Check if the module has navigation capabilities
    guard sdlManager.systemCapabilityManager.isCapabilitySupported(type: .navigation) else {
        return handler(false, nil)
    }

    // Legacy modules (pre-RPC Spec v4.5) do not support system capabilities, so for versions less than 4.5 we will assume `SDLGetWayPoints` and `SDLSubscribeWayPoints` are supported if `isCapabilitySupported` returns true
    guard let sdlMsgVersion = sdlManager.registerResponse?.sdlMsgVersion, SDLVersion(sdlMsgVersion: sdlMsgVersion).isGreaterThanOrEqual(to: SDLVersion(major: 4, minor: 5, patch: 0)) else {
        return handler(true, nil)
    }

    // Check if the navigation capability has already been retrieved from the module
    if let navigationCapability = sdlManager.systemCapabilityManager.navigationCapability {
        return handler(navigationCapability.getWayPointsEnabled?.boolValue ?? false, nil)
    }

    // Retrieve the navigation capability from the module
    sdlManager.systemCapabilityManager.updateCapabilityType(.navigation) { (error, systemCapabilityManager) in
        if (error != nil) {
            return handler(false, error)
        }

        return handler(systemCapabilityManager.navigationCapability?.getWayPointsEnabled?.boolValue ?? false, nil)
    }
}
```
!@

@![android, javaSE, javaEE]
```java
private void isGetWaypointsSupported(final OnCapabilitySupportedListener capabilitySupportedListener) {
    // Check if the module has navigation capabilities
    if (!sdlManager.getSystemCapabilityManager().isCapabilitySupported(SystemCapabilityType.NAVIGATION)) {
        capabilitySupportedListener.onCapabilitySupported(false);
        return;
    }

    // Legacy modules (pre-RPC Spec v4.5) do not support system capabilities, so for versions less than 4.5 we will assume `GetWayPoints` and `SubscribeWayPoints` are supported if `isCapabilitySupported()` returns true
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
            capabilitySupportedListener.onCapabilitySupported(navigationCapability != null ? navigationCapability.getWayPointsEnabled() : false);
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
async function isGetWaypointsSupported() {
    // Check if the module has navigation capabilities
    if (!sdlManager.getSystemCapabilityManager().isCapabilitySupported(SDL.rpc.enums.SystemCapabilityType.NAVIGATION)) {
        return false;
    }

    // Legacy modules (pre-RPC Spec v4.5) do not support system capabilities, so for versions less than 4.5 we will assume `GetWayPoints` and `SubscribeWayPoints` are supported if `isCapabilitySupported` returns true
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
        isNavigationSupported = navCapability.getGetWayPointsEnabled();
    }

    return isNavigationSupported;
}
```
!@

## Subscribing to Waypoints
To subscribe to the navigation waypoints, you will have to set up your callback for whenever the waypoints are updated, then send the @![iOS]`SDLSubscribeWayPoints`!@@![android,javaSE,javaEE,javascript]`SubscribeWayPoints`!@ RPC.

@![iOS]
##### Objective-C
```objc
// You can subscribe any time before SDL would send the notification (such as when you call `sdlManager.start` or at initialization of your manager)
[self.sdlManager subscribeToRPC:SDLDidReceiveWaypointNotification withObserver:self selector:@selector(waypointsDidUpdate:)];

// Create this method to receive the subscription callback
- (void)waypointsDidUpdate:(SDLRPCNotificationNotification *)notification {
    SDLOnWayPointChange *waypointUpdate = (SDLOnWayPointChange *)notification.notification;
    NSArray<SDLLocationDetails *> *waypoints = waypointUpdate.waypoints;

    <#Use the waypoint data#>
}

// After SDL has started your connection, at whatever point you want to subscribe, send the subscribe RPC
SDLSubscribeWayPoints *subscribeWaypoints = [[SDLSubscribeWayPoints alloc] init];
[self.sdlManager sendRequest:subscribeWaypoints withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if (!response.success) {
        // Handle the error
        return;
    }

    // You are now subscribed
}];
```

##### Swift
```swift
// You can subscribe any time before SDL would send the notification (such as when you call `sdlManager.start` or at initialization of your manager)
sdlManager.subscribe(to: .SDLDidReceiveWaypoint, observer: self, selector: #selector(waypointsDidUpdate(_:)))

// Create this method to receive the subscription callback
@objc func waypointsDidUpdate(_ notification: SDLRPCNotificationNotification) {
    guard let waypointUpdate = notification.notification as? SDLOnWayPointChange else { return }
    let waypoints = waypointUpdate.waypoints

    <#Use the waypoint data#>
}

// After SDL has started your connection, at whatever point you want to subscribe, send the subscribe RPC
let subscribeWaypoints = SDLSubscribeWayPoints()
sdlManager.send(request: subscribeWaypoints) { (request, response, error) in
    guard let response = response, response.success.boolValue else {
        // Handle the errors
        return
    }

    // You are now subscribed
}
```
!@

@![android, javaSE, javaEE]
```java
// You can subscribe any time before SDL would send the notification (such as when you call `sdlManager.start` or at initialization of your manager)
sdlManager.addOnRPCNotificationListener(FunctionID.ON_WAY_POINT_CHANGE, new OnRPCNotificationListener() {
    @Override
    public void onNotified(RPCNotification notification) {
        OnWayPointChange onWayPointChangeNotification = (OnWayPointChange) notification;
        // Use the waypoint data
    }
});

// After SDL has started your connection, at whatever point you want to subscribe, send the subscribe RPC
SubscribeWayPoints subscribeWayPoints = new SubscribeWayPoints();
subscribeWayPoints.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse rpcResponse) {
        if (rpcResponse.getSuccess()){
            // You are now subscribed
        } else {
            // Handle the errors
        }
    }
});

sdlManager.sendRPC(subscribeWayPoints);
```
!@


@![javascript]
```js
// Create this method to receive the subscription callback
sdlManager.addRpcListener(SDL.rpc.enums.FunctionID.OnWayPointChange, (onWayPointChangeNotification) => {
    // Use the waypoint data
});

// After SDL has started your connection, at whatever point you want to subscribe, send the subscribe RPC
const subscribeWayPoints = new SDL.rpc.messages.SubscribeWayPoints();

// sdl_javascript_suite v1.1+
const response = await sdlManager.sendRpcResolve(subscribeWayPoints);
if (response.getSuccess()) {
    // You are now subscribed!
} else {
    // Handle the errors
}
// thrown exceptions should be caught by a parent function via .catch()

// Pre sdl_javascript_suite v1.1
const response = await sdlManager.sendRpc(subscribeWayPoints).catch(error => error);
if (response.getSuccess()) {
    // You are now subscribed
} else {
    // Handle the errors
}
```
!@

### Unsubscribing from Waypoints
To unsubscribe from waypoint data, you must send the @![iOS]`SDLUnsubscribeWayPoints`!@@![android,javaSE,javaEE,javascript]`UnsubscribeWayPoints`!@ RPC.

@![iOS]
!!! NOTE
You do not have to unsubscribe from the `sdlManager.subscribe` method, you must simply send the unsubscribe RPC and no more callbacks will be received.
!!!
!@

@![iOS]
##### Objective-C
```objc
SDLUnsubscribeWayPoints *unsubscribeWaypoints = [[SDLUnsubscribeWayPoints alloc] init];
[self.sdlManager sendRequest:unsubscribeWaypoints withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if (!response.success) {
        // Handle the error
        return;
    }

    // You are now unsubscribed
}];
```

##### Swift
```swift
let unsubscribeWaypoints = SDLUnsubscribeWayPoints()
sdlManager.send(request: unsubscribeWaypoints) { (request, response, error) in
    guard let response = response as? SDLGetWayPointsResponse, response.success.boolValue else {
        // Handle the error
        return
    }

    // You are now unsubscribed
}
```

!@

@![android, javaSE, javaEE]
```java
UnsubscribeWayPoints unsubscribeWayPoints = new UnsubscribeWayPoints();
unsubscribeWayPoints.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse rpcResponse) {
        if (rpcResponse.getSuccess()){
            // You are now unsubscribed
        } else {
            // Handle the errors
        }
    }
});

sdlManager.sendRPC(unsubscribeWayPoints);
```
!@

@![javascript]
```js
// sdl_javascript_suite v1.1+
const unsubscribeWayPoints = new SDL.rpc.messages.UnsubscribeWayPoints();
const response = await sdlManager.sendRpcResolve(unsubscribeWayPoints);
if (response.getSuccess()) {
    // You are now unsubscribed!
} else {
    // Handle the errors
}
// thrown exceptions should be caught by a parent function via .catch()

// Pre sdl_javascript_suite v1.1
const unsubscribeWayPoints = new SDL.rpc.messages.UnsubscribeWayPoints();
const response = await sdlManager.sendRpc(unsubscribeWayPoints).catch(error => error);
if (response.getSuccess()) {
    // You are now unsubscribed
} else {
    // Handle the errors
}
```
!@

## One-Time Waypoints Request
If you only need waypoint data once without an ongoing subscription, you can use @![iOS]`SDLGetWayPoints`!@@![android,javaSE,javaEE,javascript]`GetWayPoints`!@ instead of @![iOS]`SDLSubscribeWayPoints`!@@![android,javaSE,javaEE,javascript]`SubscribeWayPoints`!@.

@![iOS]
##### Objective-C
```objc
SDLGetWayPoints *getWaypoints = [[SDLGetWayPoints alloc] initWithType:SDLWayPointTypeAll];
[self.sdlManager sendRequest:getWaypoints withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if (!response.success) {
        // Handle the error
        return;
    }

    SDLGetWayPointsResponse *waypointResponse = response;
    NSArray<SDLLocationDetails *> *waypointLocations = waypointResponse.waypoints;

    <#Use the waypoint data#>
}];
```

##### Swift
```swift
let getWaypoints = SDLGetWayPoints(type: .all)
sdlManager.send(request: getWaypoints) { (request, response, error) in
    guard let response = response as? SDLGetWayPointsResponse, response.success.boolValue else {
        // Handle the errors
        return
    }

    let waypointLocations = response.waypoints;
    <#Use the waypoint data#>
}
```

!@

@![android, javaSE, javaEE]
```java
GetWayPoints getWayPoints = new GetWayPoints();
getWayPoints.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse rpcResponse) {
        if (rpcResponse.getSuccess()){
            GetWayPointsResponse getWayPointsResponse = (GetWayPointsResponse) rpcResponse;
            // Use the waypoint data
        } else {
            // Handle the errors
        }
    }
});

sdlManager.sendRPC(getWayPoints);
```
!@

@![javascript]
```js
// sdl_javascript_suite v1.1+
const getWayPoints = new SDL.rpc.messages.GetWayPoints();
const response = await sdlManager.sendRpcResolve(getWayPoints);
if (response.getSuccess()) {
    <#Use the waypoint information#>
} else {
    // Handle the errors
}
// thrown exceptions should be caught by a parent function via .catch()

// Pre sdl_javascript_suite v1.1
const getWayPoints = new SDL.rpc.messages.GetWayPoints();
const response = await sdlManager.sendRpc(getWayPoints).catch(error => error);
if (response.getSuccess()) {
    // Use the waypoint data
} else {
    // Handle the errors
}
```
!@
