# Getting the Navigation Destination
The `GetWayPoints` and `SubscribeWayPoints` RPCs are designed to allow you to get the navigation destination(s) from the active navigation app if the user is navigating.

## Checking If Your App Has Permission to Use GetWayPoints
The `GetWayPoints` and `SubscribeWayPoints` RPCs are restricted by most vehicle manufacturers. As a result, the head unit you are connecting to will reject the request if you do not have the correct permissions. Please check the [Understanding Permissions](Getting Started/Understanding Permissions) section for more information on how to check permissions for an RPC.

### Checking if Head Unit Supports GetWaypoints
Since there is a possibility that some head units will not support getting the navigation destination, you should check head unit support before attempting to send the request. You should also update your app's UI based on whether or not you can use `GetWayPoints`.

If using library v.@![iOS]6.0!@@![android, javaSE, javaEE]4.4!@+, you can use the @![iOS]`SDLSystemCapabilityManager`!@@![android, javaSE, javaEE]`SystemCapabilityManager`!@ to check the navigation capability returned by Core as shown in the code sample below.

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
// TODO: Change for GetWayPoints

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
// TODO
```
!@

### Unsubscribing from Waypoints
To unsubscribe from waypoint data, you must send the @![iOS]SDLUnsubscribeWayPoints!@@![android, javaSE, javaEE]UnsubscribeWayPoints!@ RPC.

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
// TODO
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
// TODO
```
!@