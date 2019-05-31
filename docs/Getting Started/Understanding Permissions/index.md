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
The easiest way to monitor the `hmiLevel` of your SDL app is through a required delegate callback of `SDLManagerDelegate`. The function `hmiLevel:didChangeToLevel:` is called every time your app's `hmiLevel` changes.

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

## Permission Manager
When your app first connects to the head unit, it will receive an `OnPermissionsChange` notification. This notification contains all RPCs the head unit supports and the `hmiLevel` permissions for each RPC. Use the `SDLManager`'s permission manager to check the current permission status of a specific RPC or group of RPCs. If desired, you may also subscribe to get notifications when the RPC(s) permission status changes. 

### Checking Current Permissions of a Single RPC
##### Objective-C
```objc
BOOL isAllowed = [self.sdlManager.permissionManager isRPCAllowed:<#RPC name#>];
```

##### Swift
```swift
let isAllowed = sdlManager.permissionManager.isRPCAllowed(<#RPC name#>)
```

### Checking Current Permissions of a Group of RPCs
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

### Observing Permissions
If desired, you can set an observer for a group of permissions. The observer's handler will be called when the permissions for the group changes. If you want to be notified when the permission status of any of RPCs in the group change, set the `groupType` to `SDLPermissionGroupTypeAny`. If you only want to be notified when all of the RPCs in the group are allowed, set the `groupType` to `SDLPermissionGroupTypeAllAllowed`.

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

### Stopping Observation of Permissions
When you set up the observer, you will get an unique id back. Use this id to unsubscribe to the permissions at a later date.

##### Objective-C
```objc
[self.sdlManager.permissionManager removeObserverForIdentifier:observerId];
```

##### Swift
```swift
sdlManager.permissionManager.removeObserver(forIdentifier: observerId)
```

## Additional HMI State Information
If you want more detail about the current state of your SDL app you can monitor the audio playback state as well as get notifications when something blocks the main screen of your app.

### Audio Streaming State
The Audio Streaming State informs your app whether or not the driver will be able to hear your app's audio. It will be either `AUDIBLE`, `NOT_AUDIBLE`, or `ATTENUATED`.

You will get these notifications when an alert pops up, when you start recording the in-car audio, when voice recognition is active, when another app takes audio control, when a navigation app is giving directions, etc.

Audio Streaming State   | What does this mean?
------------------------|------------------------------------------------------------
AUDIBLE     			| Any audio you are playing will be audible to the user. 
ATTENUATED  			| Some kind of audio mixing is occuring between what you are playing, if anything, and some system level audio or navigation application audio.
NOT_AUDIBLE 			| Your streaming audio is not audible. This could occur during a `VRSESSION` System Context.

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

### System Context
The System Context informs your app if there is potentially a blocking HMI component while your app is still visible. An example of this would be if your application is open and you display an alert. Your app will receive a system context of `ALERT` while it is presented on the screen, followed by `MAIN` when it is dismissed.

System Context State   | What does this mean?
-----------------------|------------------------------------------------------------
MAIN        		   | No user interaction is in progress that could be blocking your app's visibility.
VRSESSION  			   | Voice recognition is currently in progress.
MENU     			   | A menu interaction is currently in-progress.
HMI_OBSCURED    	   | The app's display HMI is being blocked by either a system or other app's overlay (another app's alert, for instance).
ALERT 				   | An alert that you have sent is currently visible.

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