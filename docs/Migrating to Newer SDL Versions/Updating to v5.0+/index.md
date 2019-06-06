# Updating from v4.3+ to v5.0+
A number of breaking changes have been made to the SDL library in v5.0+. This means that it is unlikely your project will compile without changes.

## Changes to SDL Enums
SDLEnums have changed from being objects in SDL v4.X to strings in Obj-C and Enums in Swift. This means that every usage of SDL enums in your app integration will need changes.

##### Obj-C
Old:
```objc
- (void)hmiLevel:(SDLHMILevel *)oldLevel didChangeToLevel:(SDLHMILevel *)newLevel {
    if (![newLevel isEqualToEnum:[SDLHMILevel NONE]] && (self.firstTimeState == SDLHMIFirstStateNone)) {
        // This is our first time in a non-NONE state
    }
    
    if ([newLevel isEqualToEnum:[SDLHMILevel FULL]] && (self.firstTimeState != SDLHMIFirstStateFull)) {
        // This is our first time in a FULL state
    }
    
    if ([newLevel isEqualToEnum:[SDLHMILevel FULL]]) {
        // We entered full
    }
}
```

New:
```objc
- (void)hmiLevel:(SDLHMILevel)oldLevel didChangeToLevel:(SDLHMILevel)newLevel {
    if (![newLevel isEqualToEnum:SDLHMILevelNone] && (self.firstTimeState == SDLHMIFirstStateNone)) {
        // This is our first time in a non-NONE state
    }
    
    if ([newLevel isEqualToEnum:SDLHMILevelFull] && (self.firstTimeState != SDLHMIFirstStateFull)) {
        // This is our first time in a FULL state
    }
    
    if ([newLevel isEqualToEnum:SDLHMILevelFull]) {
        // We entered full
    }
}
```

Note the differences between, e.g. `[SDLHMILevel FULL]` and `SDLHMILevelFull`.

##### Swift
Old: (Swift 3)
```swift
func hmiLevel(_ oldLevel: SDLHMILevel, didChangeTo newLevel: SDLHMILevel) {
    // On our first HMI level that isn't none, do some setup
    if newLevel != .none() && firstTimeState == .none {
        // This is our first time in a non-NONE state
    }

    // HMI state is changing from NONE or BACKGROUND to FULL or LIMITED
    if (newLevel == .full() && firstTimeState != .full) {
        // This is our first time in a FULL state
    }

    if (newLevel == .full()) {
        // We entered full
    }
}
```

New: (Swift 4)
```swift
func hmiLevel(_ oldLevel: SDLHMILevel, didChangeToLevel: SDLHMILevel) {
    // On our first HMI level that isn't none, do some setup
    if didChangeToLevel != .none && firstTimeState == .none {
        // This is our first time in a non-NONE state
    }

    // HMI state is changing from NONE or BACKGROUND to FULL or LIMITED
    if (didChangeToLevel == .full && firstTimeState != .full) {
        // This is our first time in a FULL state
    }

    if (didChangeToLevel == .full {
        // We entered full
    }
}
```

Note the differences between, e.g. `.full()` and `.full`.

## Changes to RPC Handlers
Old:
```objc
SDLSubscribeButton *button = [[SDLSubscribeButton alloc] initWithHandler:^(__kindof SDLRPCNotification * _Nonnull notification) {
    if (![notification isKindOfClass:[SDLOnButtonPress class]]) {
        return;
    }
    SDLOnButtonPress *buttonPress = (SDLOnButtonPress *)notification;
    if (buttonPress.buttonPressMode != SDLButtonPressMode.SHORT) {
        return;
    }
}];
[manager sendRequest:button];
```

New:
```objc
SDLSubscribeButton *button = [[SDLSubscribeButton alloc] initWithHandler:^(SDLOnButtonPress * _Nullable buttonPress, SDLOnButtonEvent * _Nullable buttonEvent) {
    if (buttonPress != nil && buttonPress.buttonPressMode != SDLButtonPressModeShort) {
        return;
    }
}];

[manager sendRequest:button];
```

##### Swift
Old:
```swift
let button = SDLSubscribeButton { [unowned store] (notification) in
    guard let buttonPress = notification as? SDLOnButtonPress else { return }
    guard buttonPress.buttonPressMode == .short() else { return }

    // Button was pressed
}!
button.buttonName = .seekleft()
manager.send(request: button)
```

New:
```swift
let button = SDLSubscribeButton { [unowned store] (press, event) in
    guard press.buttonPressMode == .short() else { return }

    // Button was pressed
}
button.buttonName = .seekLeft
manager.send(request: button)
```

RPC handlers for `SDLAddCommand`, `SDLSoftButton`, and `SDLSubscribeButton` have been altered to provide more accurate notifications within the handler.

## SDLConfiguration Changes
`SDLConfiguration`, used to initialize `SDLManager` has changed slightly. When creating a configuration, a logging configuration is now required. Furthermore, if you are creating a video streaming `NAVIGATION` or `PROJECTION` app, you must now create an `SDLStreamingMediaConfiguration` and add it to your `SDLConfiguration` before initializing the `SDLManager`. Additionally, if your app is in Swift, your initialization may have changed.

##### Obj-C
```objc
SDLConfiguration *config = [SDLConfiguration configurationWithLifecycle:lifecycleConfig lockScreen:[SDLLockScreenConfiguration enabledConfiguration] logging:[SDLLogConfiguration debugConfiguration]];
```

##### Swift
```swift
let configuration: SDLConfiguration = SDLConfiguration(lifecycle: lifecycleConfiguration, lockScreen: SDLLockScreenConfiguration.enabledConfiguration(), logging: SDLLogConfiguration())
```

## Multiple File Uploads
You can now upload multiple files (such as images) with one method call and be notified when all finish uploading.

##### Obj-C
```objc
// Upload a batch of files with a completion handler when done
[self.sdlManager.fileManager uploadFiles:files completionHandler:^(NSError * _Nullable error) {
    <#code#>
}];

// Upload a batch of files, being notified in the progress handler when each completes (returning whether or not to continue uploading), and a completion handler when done
[self.sdlManager.fileManager uploadFiles:files progressHandler:^BOOL(SDLFileName * _Nonnull fileName, float uploadPercentage, NSError * _Nullable error) {
    <#code#>
} completionHandler:^(NSError * _Nullable error) {
    <#code#>
}];
```

##### Swift
```swift
// Upload a batch of files with a completion handler when done
sdlManager.fileManager.upload(files: softButtonImages) { (error) in
    <#code#>
}

// Upload a batch of files, being notified in the progress handler when each completes (returning whether or not to continue uploading), and a completion handler when done
sdlManager.fileManager.upload(files: softButtonImages, progressHandler: { (fileName, uploadPercentage, error) -> Bool in
    <#code#>
}) { (error) in
    <#code#>
}
```

## Logging Changes
For a comprehensive look at logging with SDL iOS 5.0, [see the section dedicated to the subject](Developer Tools/Configuring SDL Logging).

## Immutable RPC Collections & Generics
In any RPC that has an array, that array will now be immutable. Any array and dictionary will also now expose what it contains via generics.

For example, within `SDLAlert.h`:

```objc
@property (nullable, strong, nonatomic) NSArray<SDLSoftButton *> *softButtons;
```

### Nullability
SDL now exposes nullability tags for all APIs. This primarily means that you no longer need to use the force-unwrap operator `!` in Swift when creating RPCs.

## Video Streaming Enhancements
Video streaming has been overhauled in SDL 5.0; SDL now takes care of just about everything for you automatically including HMI changes and app state changes.

When setting your `SDLConfiguration`, you will have to set up an `SDLStreamingMediaConfiguration`.

```objc
SDLStreamingMediaConfiguration *streamingConfig = [SDLStreamingMediaConfiguration insecureConfiguration];
SDLConfiguration *config = [[SDLConfiguration alloc] initWithLifecycle:lifecycleConfig lockScreen:[SDLLockScreenConfiguration enabledConfiguration] logging:[SDLLogConfiguration debugConfiguration] streamingMedia:streamingConfig];
```

When you have a `NAVIGATION` or `PROJECTION` app and set this streaming configuration, SDL will automatically start the video streaming session on behalf of your app. When you receive the `SDLVideoStreamDidStartNotification`, you're good to go!

For more information about Video Streaming, see the [dedicated section](Video Streaming for Navigation Apps/Video Streaming).

### Touch Manager Delegate Changes
The touch manager delegate calls have all changed and previous delegate methods won't work. If you are streaming video and set the window, the new callbacks may return the view that was touched, otherwise it will return nil. For example:

```objc
- (void)touchManager:(SDLTouchManager *)manager didReceiveSingleTapForView:(UIView *_Nullable)view atPoint:(CGPoint)point;
```