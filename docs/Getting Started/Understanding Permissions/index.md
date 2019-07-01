# Understanding Permissions
While creating your SDL app, remember that just because your app is connected to a head unit it does not mean that the app has permission to send any RPCs you want. If your app does not have the required permissions, requests will be rejected. There are three important things to remember in regards to permissions:

1. You may not be able to send a RPC when the SDL app is closed, in the background, or obscured by an alert. Each RPC has a set of `hmiLevel`s during which it can be sent.
1. For some RPCs, like those that access vehicle data or make a phone call, you may need special permissions from the OEM to use. This permission is granted when you submit your app to the OEM for approval. Each OEM decides which RPCs it will restrict access to, so it is up you to check if you are allowed to use the RPC with the head unit.
1. Some head units may not support all RPCs.

## HMI Levels
When your app is connected to the head unit you will receive notifications when the SDL app's HMI status changes. Your app can be in one of four different `hmiLevel`s:

HMI Level   | What does this mean?
------------|------------------------------------------------------------
NONE        | The user has not yet opened your app, or the app has been killed.
BACKGROUND  | The user has opened your app, but is currently in another part of the head unit.
LIMITED     | This level only applies to media and navigation apps (i.e. apps with an `appType` of `MEDIA` or `NAVIGATION`). The user has opened your app, but is currently in another part of the head unit. The app can receive button presses from the play, seek, tune, and preset buttons.
FULL        | Your app is currently in focus on the screen.

Be careful with sending user interface related RPCs in the `NONE` and `BACKGROUND` levels; some head units may reject RPCs sent in those states. We recommended that you wait until your app's `hmiLevel` enters `FULL` to set up your app's UI.

To get more detailed information about the state of your SDL app check the current system context. The system context will let you know if a menu is open, a VR session is in progress, an alert is showing, or if the main screen is unobstructed. You can find more information about the system context below.

### Monitoring the HMI Level
@![iOS]
The easiest way to monitor the `hmiLevel` of your SDL app is through a required delegate callback of `SDLManagerDelegate`. The function `hmiLevel:didChangeToLevel:` is called every time your app's `hmiLevel` changes.
!@

@![android,javaSE,javaEE]
Monitoring HMI Status is possible through an `OnHMIStatus` notification that you can subscribe to via the `SdlManager`'s `addOnRPCNotificationListener`.
!@

@![iOS]
##### Objective-C
```objc
- (void)hmiLevel:(SDLHMILevel)oldLevel didChangeToLevel:(SDLHMILevel)newLevel {
    if (![newLevel isEqualToEnum:SDLHMILevelNone] && (self.firstHMILevel == SDLHMIFirstStateNone)) {
        // This is our first time in a non-`NONE` state
        self.firstHMILevel = newLevel;
        <#Send static menu RPCs#>
    }

    if ([newLevel isEqualToEnum:SDLHMILevelFull]) {
        <#Send user interface RPCs#>
    } else if ([newLevel isEqualToEnum:SDLHMILevelLimited]) {
        <#Code#>
    } else if ([newLevel isEqualToEnum:SDLHMILevelBackground]) {
        <#Code#>
    } else if ([newLevel isEqualToEnum:SDLHMILevelNone]) {
        <#Code#>
    }
}
```

##### Swift
```swift
fileprivate var firstHMILevel: SDLHMILevel = .none
func hmiLevel(_ oldLevel: SDLHMILevel, didChangeToLevel newLevel: SDLHMILevel) {
    if newLevel != .none && firstHMILevel == .none {
        // This is our first time in a non-`NONE` state
        firstHMILevel = newLevel
        <#Send static menu RPCs#>
    }

    switch newLevel {
    case .full:
        <#Send user interface RPCs#>
    case .limited: break 
    case .background: break
    case .none: break
    default: break
    }
}
```
!@

@![android,javaSE,javaEE]
```java
Map<FunctionID, OnRPCNotificationListener> onRPCNotificationListenerMap = new HashMap<>();
onRPCNotificationListenerMap.put(FunctionID.ON_HMI_STATUS, new OnRPCNotificationListener() {
    @Override
    public void onNotified(RPCNotification notification) {
        OnHMIStatus onHMIStatus = (OnHMIStatus) notification;
        if (onHMIStatus.getHmiLevel() == HMILevel.HMI_FULL && onHMIStatus.getFirstRun()){
            // first time in HMI Full
        }
    }
});
builder.setRPCNotificationListeners(onRPCNotificationListenerMap);
```
!@

## Permission Manager
The PermissionManager allows developers to easily query whether specific RPCs are allowed or not. It also allows a listener to be added for a list of RPCs so that if there are changes in their permissions, the app will be notified.

### Checking Current Permissions of a Single RPC
@![iOS]
##### Objective-C
```objc
BOOL isAllowed = [self.sdlManager.permissionManager isRPCAllowed:<#RPC name#>];
```

##### Swift
```swift
let isAllowed = sdlManager.permissionManager.isRPCAllowed(<#RPC name#>)
```
!@

@![android,javaSE,javaEE]
```java
boolean allowed = sdlManager.getPermissionManager().isRPCAllowed(FunctionID.SHOW);

// You can also check if a permission parameter is allowed  
boolean parameterAllowed = sdlManager.getPermissionManager().isPermissionParameterAllowed(FunctionID.GET_VEHICLE_DATA, GetVehicleData.KEY_RPM);
```
!@

### Checking Current Permissions of a Group of RPCs
@![iOS]
##### Objective-C
```objc
SDLPermissionGroupStatus groupPermissionStatus = [self.sdlManager.permissionManager groupStatusOfRPCs:@[<#RPC name#>, <#RPC name#>]];
NSDictionary *individualPermissionStatuses = [self.sdlManager.permissionManager statusOfRPCs:@[<#RPC name#>, <#RPC name#>]];
```

##### Swift
```swift
let groupPermissionStatus = sdlManager.permissionManager.groupStatus(ofRPCs:[<#RPC name#>, <#RPC name#>])
let individualPermissionStatuses = sdlManager.permissionManager.status(ofRPCs:[<#RPC name#>, <#RPC name#>])
```
!@

@![android,javaSE,javaEE]
```java
List<PermissionElement> permissionElements = new ArrayList<>();
permissionElements.add(new PermissionElement(FunctionID.SHOW, null));
permissionElements.add(new PermissionElement(FunctionID.GET_VEHICLE_DATA, Arrays.asList(GetVehicleData.KEY_RPM, GetVehicleData.KEY_SPEED)));

int groupStatus = sdlManager.getPermissionManager().getGroupStatusOfPermissions(permissionElements);

switch (groupStatus) {
    case PermissionManager.PERMISSION_GROUP_STATUS_ALLOWED:
        // Every permission in the group is currently allowed
        break;
    case PermissionManager.PERMISSION_GROUP_STATUS_DISALLOWED:
        // Every permission in the group is currently disallowed
        break;
    case PermissionManager.PERMISSION_GROUP_STATUS_MIXED:
        // Some permissions in the group are allowed and some disallowed
        break;
    case PermissionManager.PERMISSION_GROUP_STATUS_UNKNOWN:
        // The current status of the group is unknown
        break;
}
```

The previous snippet will give a quick generic status for all permissions together. However, if developers want to get a more detailed result about the status of every permission or parameter in the group, they can use `getStatusOfPermissions` method:

```java
List<PermissionElement> permissionElements = new ArrayList<>();
permissionElements.add(new PermissionElement(FunctionID.SHOW, null));
permissionElements.add(new PermissionElement(FunctionID.GET_VEHICLE_DATA, Arrays.asList(GetVehicleData.KEY_RPM, GetVehicleData.KEY_AIRBAG_STATUS)));

Map<FunctionID, PermissionStatus> status = sdlManager.getPermissionManager().getStatusOfPermissions(permissionElements);

if (status.get(FunctionID.GET_VEHICLE_DATA).getIsRPCAllowed()){
    // GetVehicleData RPC is allowed
}

if (status.get(FunctionID.GET_VEHICLE_DATA).getAllowedParameters().get(GetVehicleData.KEY_RPM)){
    // rpm parameter in GetVehicleData RPC is allowed
}
```
!@

### Observing Permissions
If desired, you can set an observer for a group of permissions. The observer's handler will be called when the permissions for the group changes. If you want to be notified when the permission status of any of RPCs in the group change, set the `groupType` to @![iOS]`SDLPermissionGroupTypeAny`!@ @![android,javaSE,javaEE]`PERMISSION_GROUP_TYPE_ANY`!@. If you only want to be notified when all of the RPCs in the group are allowed, set the `groupType` to @![iOS]`SDLPermissionGroupTypeAllAllowed`!@ @![android,javaSE,javaEE]`PERMISSION_GROUP_TYPE_ALL_ALLOWED`!@.

@![iOS]
##### Objective-C
```objc
SDLPermissionObserverIdentifier observerId = [self.sdlManager.permissionManager addObserverForRPCs:@[<#RPC name#>, <#RPC name#>] groupType:<#SDLPermissionGroupType#> withHandler:^(NSDictionary<SDLPermissionRPCName, NSNumber<SDLBool> *> * _Nonnull change, SDLPermissionGroupStatus status) {
    <#RPC group status changed#>
}];
```

##### Swift
```swift
let observerId = sdlManager.permissionManager.addObserver(forRPCs: <#RPC name#>, <#RPC name#>, groupType:<#SDLPermissionGroupType#>, withHandler: { (individualStatuses, groupStatus) in
    <#RPC group status changed#>
})
```
!@

@![android,javaSE,javaEE]
```java
List<PermissionElement> permissionElements = new ArrayList<>();
permissionElements.add(new PermissionElement(FunctionID.SHOW, null));
permissionElements.add(new PermissionElement(FunctionID.GET_VEHICLE_DATA, Arrays.asList(GetVehicleData.KEY_RPM, GetVehicleData.KEY_AIRBAG_STATUS)));


UUID listenerId = sdlManager.getPermissionManager().addListener(permissionElements, PermissionManager.PERMISSION_GROUP_TYPE_ANY, new OnPermissionChangeListener() {
    @Override
    public void onPermissionsChange(@NonNull Map<FunctionID, PermissionStatus> allowedPermissions, @NonNull int permissionGroupStatus) {
        if (allowedPermissions.get(FunctionID.GET_VEHICLE_DATA).getIsRPCAllowed()) {
            // GetVehicleData RPC is allowed
        }

        if (allowedPermissions.get(FunctionID.GET_VEHICLE_DATA).getAllowedParameters().get(GetVehicleData.KEY_RPM)){
            // rpm parameter in GetVehicleData RPC is allowed
        }
    }
});
```
!@

### Stopping Observation of Permissions
When you set up the observer, you will get an unique id back. Use this id to unsubscribe to the permissions at a later date.

@![iOS]
##### Objective-C
```objc
[self.sdlManager.permissionManager removeObserverForIdentifier:observerId];
```

##### Swift
```swift
sdlManager.permissionManager.removeObserver(forIdentifier: observerId)
```
!@

@![android,javaSE,javaEE]
```java
sdlManager.getPermissionManager().removeListener(listenerId);
```
!@

## Additional HMI State Information
If you want more detail about the current state of your SDL app you can monitor the audio playback state as well as get notifications when something blocks the main screen of your app.

### Audio Streaming State
The Audio Streaming State informs your app whether or not the driver will be able to hear your app's audio. It will be either `AUDIBLE`, `NOT_AUDIBLE`, or `ATTENUATED`.

You will get these notifications when an alert pops up, when you start recording the in-car audio, when voice recognition is active, when another app takes audio control, when a navigation app is giving directions, etc.

Audio Streaming State   | What does this mean?
------------------------|------------------------------------------------------------
AUDIBLE     			| Any audio you are playing will be audible to the user
ATTENUATED  			| Some kind of audio mixing is occurring between what you are playing, if anything, and some system level audio or navigation application audio.
NOT_AUDIBLE 			| Your streaming audio is not audible. This could occur during a `VRSESSION` System Context.

@![iOS]
##### Objective-C
```objc
- (void)audioStreamingState:(nullable SDLAudioStreamingState)oldState didChangeToState:(SDLAudioStreamingState)newState {
    <#code#>
}
```
##### Swift
```swift
func audioStreamingState(_ oldState: SDLAudioStreamingState?, didChangeToState newState: SDLAudioStreamingState) {
    <#code#>
}
```
!@

@![android,javaSE,javaEE]
```java
@Override
public void onNotified(RPCNotification notification) {
    OnHMIStatus status = (OnHMIStatus) notification;
    AudioStreamingState streamingState = notification.getAudioStreamingState();
}
```
!@

### System Context
The System Context informs your app if there is potentially a blocking HMI component while your app is still visible. An example of this would be if your application is open and you display an alert. Your app will receive a system context of `ALERT` while it is presented on the screen, followed by `MAIN` when it is dismissed.

System Context State   | What does this mean?
-----------------------|------------------------------------------------------------
MAIN        		   | No user interaction is in progress that could be blocking your app's visibility.
VRSESSION  			   | Voice recognition is currently in progress.
MENU     			   | A menu interaction is currently in-progress.
HMI_OBSCURED    	   | The app's display HMI is being blocked by either a system or other app's overlay (another app's alert, for instance).
ALERT 				   | An alert that you have sent is currently visible.

@![iOS]
##### Objective-C
```objc
- (void)systemContext:(nullable SDLSystemContext)oldContext didChangeToContext:(SDLSystemContext)newContext {
    <#code#>
}
```

##### Swift
```swift
func systemContext(_ oldContext: SDLSystemContext?, didChangeToContext newContext: SDLSystemContext) {
    <#code#>
}
```
!@

@![android,javaSE,javaEE]
```java
@Override
public void onNotified(RPCNotification notification) {
    OnHMIStatus status = (OnHMIStatus) notification;
    SystemContext systemContext = notification.getSystemContext();
}
```
!@