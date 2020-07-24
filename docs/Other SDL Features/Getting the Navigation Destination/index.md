# Getting the Navigation Destination
The `GetWayPoints` and `SubscribeWayPoints` RPCs are designed to allow you to get the navigation destination(s) from the active navigation app if the user is navigating.

## Checking If Your App Has Permission to Use GetWayPoints
The `GetWayPoints` and `SubscribeWayPoints` RPCs are restricted by most vehicle manufacturers. As a result, the head unit you are connecting to will reject the request if you do not have the correct permissions. Please check the [Understanding Permissions](Getting Started/Understanding Permissions) section for more information on how to check permissions for an RPC.

### Checking if Head Unit Supports GetWaypoints
Since there is a possibility that some head units will not support getting the navigation destination, you should check head unit support before attempting to send the request. You should also update your app's UI based on whether or not you can use `GetWayPoints`.

You can use the @![iOS]`SDLSystemCapabilityManager`!@@![android, javaSE, javaEE, javascript]`SystemCapabilityManager`!@ to check the navigation capability returned by Core as shown in the code sample below.

@![iOS]
##### Objective-C
```objc
[self.sdlManager.systemCapabilityManager updateCapabilityType:SDLSystemCapabilityTypeNavigation completionHandler:^(NSError * _Nullable error, SDLSystemCapabilityManager * _Nonnull systemCapabilityManager) {
    BOOL isNavigationSupported = NO;
    if (error == nil) {
        isNavigationSupported = systemCapabilityManager.navigationCapability.getWayPointsEnabled.boolValue;
    } else {
        isNavigationSupported = systemCapabilityManager.hmiCapabilities.navigation.boolValue;
    }

    <#If navigation is supported, send the `GetWayPoints` RPC#>
}];
```

##### Swift
```swift
sdlManager.systemCapabilityManager.updateCapabilityType(.navigation) { (error, systemCapabilityManager) in
    var isNavigationSupported = false
    if error == nil {
        isNavigationSupported = systemCapabilityManager.navigationCapability?.getWayPointsEnabled?.boolValue ?? false;
    } else {
        isNavigationSupported = systemCapabilityManager.hmiCapabilities?.navigation?.boolValue ?? false
    }

    <#If navigation is supported, send the `GetWayPoints` RPC#>
}
```
!@

@![android, javaSE, javaEE]
```java
sdlManager.getSystemCapabilityManager().getCapability(SystemCapabilityType.NAVIGATION, new OnSystemCapabilityListener() {
    @Override
    public void onCapabilityRetrieved(Object capability) {
        NavigationCapability navCapability = (NavigationCapability) capability;
        boolean isNavigationSupported = navCapability != null && navCapability.getWayPointsEnabled();
    }

    @Override
    public void onError(String info) {
        HMICapabilities hmiCapabilities = (HMICapabilities) sdlManager.getSystemCapabilityManager().getCapability(SystemCapabilityType.HMI);
        boolean isNavigationSupported = hmiCapabilities.isNavigationAvailable();
    }
});
```
!@

@![javascript]
```js
const navCapability = await sdlManager.getSystemCapabilityManager().updateCapability(SDL.rpc.enums.SystemCapabilityType.NAVIGATION);
const isNavigationSupported = navCapability !== null && navCapability.getWayPointsEnabled();
```
!@

## Subscribing to WayPoints
To subscribe to the waypoints, you will have to set up your callback for whenever the waypoints are updated, then send the `SubscribeWayPoints` RPC.

@![iOS]
##### Objective-C
```objc
// Any time before SDL would send the notification (such as when you call `sdlManager.start` or at initialization of your manager)
[self.sdlManager subscribeToRPC:SDLDidReceiveWaypointNotification withObserver:self selector:@selector(waypointsDidUpdate:)];

// Create this method to receive the subscription callback
- (void)waypointsDidUpdate:(SDLRPCNotificationNotification *)notification {
    SDLOnWayPointChange *waypointUpdate = (SDLOnWayPointChange *)notification.notification;
    NSArray<SDLLocationData> *waypoints = waypointUpdate.wayPoints;

    <#Use the waypoint data#>
}

// After SDL has started your connection, at whatever point you want to subscribe, send the subscribe RPC
SDLSubscribeWayPoints *subscribeWaypoints = [[SDLSubscribeWayPoints alloc] init];
[self.sdlManager sendRequest:subscribeWaypoints withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if (error != nil || !response.success) {
        // Handle the error
        return;
    }

    // You are now subscribed!
}];
```

##### Swift
```swift
// Any time before SDL would send the notification (such as when you call `sdlManager.start` or at initialization of your manager)
sdlManager.subscribe(to: .SDLDidReceiveWaypoint, observer: self, selector: #selector(waypointsDidUpdate(_:)))

// Create this method to receive the subscription callback
func waypointsDidUpdate(_ notification: SDLRPCNotificationNotification) {
    guard let waypointUpdate = notification.notification as? SDLOnWayPointChange, let waypoints = waypointUpdate.wayPoints else { return }

    <#Use the waypoint data#>
}

// After SDL has started your connection, at whatever point you want to subscribe, send the subscribe RPC
let subscribeWaypoints = SDLSubscribeWayPoints()
sdlManager.send(request: subscribeWaypoints) { (request, response, error) in
    guard error == nil, let response = response, response.success == true else {
        // Handle the errors
        return
    }

    // You are now subscribed!
}
```
!@

@![android, javaSE, javaEE]
##### Java
```java
// Create this method to receive the subscription callback
sdlManager.addOnRPCNotificationListener(FunctionID.ON_WAY_POINT_CHANGE, new OnRPCNotificationListener() {
    @Override
    public void onNotified(RPCNotification notification) {
        OnWayPointChange onWayPointChangeNotification = (OnWayPointChange) notification;
        //<#Use the waypoint data#>
    }
});

// After SDL has started your connection, at whatever point you want to subscribe, send the subscribe RPC
SubscribeWayPoints subscribeWayPoints = new SubscribeWayPoints();
subscribeWayPoints.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse rpcResponse) {
        if (rpcResponse.getSuccess()){
            // You are now subscribed!
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
    //<#Use the waypoint data#>
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
    // You are now subscribed!
} else {
    // Handle the errors
}
```
!@

### Unsubscribing from Waypoints
To unsubscribe from waypoint data, you must send the @![iOS]`SDLUnsubscribeWayPoints`!@@![android, javaSE, javaEE, javascript]`UnsubscribeWayPoints`!@ RPC.

@![iOS]
!!! NOTE
You do not have to unsubscribe from the `sdlManager.subscribe` method, you must simply send the unsubscribe RPC and no more callbacks will be received.
!!!
!@

@![iOS]
##### Objective-C
```objc
// Whenever you want to unsubscribe
SDLUnsubscribeWayPoints *unsubscribeWaypoints = [[SDLUnsubscribeWayPoints alloc] init];
[self.sdlManager sendRequest:unsubscribeWaypoints withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if (error != nil || !response.success) {
        // Handle the error
        return;
    }

    // You are now unsubscribed!
}];
```

##### Swift
```swift
// Whenever you want to unsubscribe
let unsubscribeWaypoints = SDLUnsubscribeWayPoints()
sdlManager.send(request: unsubscribeWaypoints) { (request, response, error) in
    guard error == nil, let response = response, response.success == true else {
        // Handle the errors
        return
    }

    // You are now subscribed!
}
```

!@

@![android, javaSE, javaEE]
##### Java
```java
UnsubscribeWayPoints unsubscribeWayPoints = new UnsubscribeWayPoints();
unsubscribeWayPoints.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse rpcResponse) {
        if (rpcResponse.getSuccess()){
            // You are now unsubscribed!
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
    // You are now unsubscribed!
} else {
    // Handle the errors
}
```
!@

## One-Time Waypoints Request
If you only need waypoint data once without an ongoing subscription, you can use `GetWayPoints` instead of `SubscribeWayPoints`.

@![iOS]
##### Objective-C
```objc
// Whenever you want to get waypoint data
SDLGetWayPoints *getWaypoints = [[SDLGetWayPoints alloc] initWithType:SDLWayPointTypeAll];
[self.sdlManager sendRequest:getWaypoints withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if (error != nil || !response.success) {
        // Handle the error
        return;
    }

    SDLGetWayPointsResponse *waypointResponse = response;
    NSArray<SDLLocationDetails *> *waypointLocations = waypointResponse.waypoints;

    <#Use the waypoint information#>
}];
```

##### Swift
```swift
// Whenever you want to unsubscribe
let getWaypoints = SDLGetWayPoints(type: .all)
sdlManager.send(request: getWaypoints) { (request, response, error) in
    guard error == nil, let response = response as? SDLGetWayPointsResponse, response.success == true else {
        // Handle the errors
        return
    }

    let waypointLocations = response.waypoints;
    <#Use the waypoint information#>
}
```

!@

@![android, javaSE, javaEE]
##### Java
```java
GetWayPoints getWayPoints = new GetWayPoints();
getWayPoints.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse rpcResponse) {
        if (rpcResponse.getSuccess()){
            GetWayPointsResponse getWayPointsResponse = (GetWayPointsResponse) rpcResponse;
            <#Use the waypoint information#>
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
    <#Use the waypoint information#>
} else {
    // Handle the errors
}
```
!@