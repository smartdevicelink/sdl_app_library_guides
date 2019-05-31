# Updating from 4.2 and below to 4.3+
This guide is used to show the update process for a developer using a version before 4.3, using the `SDLProxy` to using the new `SDLManager` class available in 4.3 and newer. Although this is not a breaking change, v4.3+ makes significant deprecations and additions that will simplify your code. For our examples through this guide, we are going to be using the version 1.0.0 of the Hello SDL project.

You can download this version [here](https://d83tozu1c8tt6.cloudfront.net/media/resources/hello_sdl_ios-1.0.0.zip). 

## Updating the Podfile
For this guide, we will be using the most recent version of SDL at this time: 4.5.5. To change the currently used version, you can open up the `Podfile` located in the root of the `hello_sdl_ios-1.0.0` directory. 

Change the following line

```
pod 'SmartDeviceLink-iOS', '~> 4.2.3'
```

to

```
pod 'SmartDeviceLink-iOS', '~> 4.5.5'
```

You may then be able to run `pod install` to install this version of SDL into the app. For more information on how to use Cocoapods, check out the [Getting Started > Installation](Getting Started/Installation) section.

After SDL has been updated, open up the `HelloSDL.xcworkspace`.

You will notice that the project will still compile, but with deprecation warnings.

!!! note
Currently, `SDLProxy` is still supported in versions but is deprecated, however in the future this accessibility will be removed.
!!!


## Response and Event Handlers
A big change with migration to versions of SDL 4.3 and later is the change from a delegate-based to a notification-based and/or completion-handler based infrastructure. All delegate callbacks relating to Responses and Notifications within `SDLProxyListener.h` will now be available as iOS notifications, with their names listed in `SDLNotificationConstants.h`.

We have also added the ability to have completion handlers for when a request's response comes back. This allows you to simply set a response handler when sending a request and be notified in the block when the response returns or fails. Additional handlers are available on certain RPCs that are associated with SDL notifications, such as `SubscribeButton`, when that button is pressed, you will receive a call on the handler.

Because of this, any delegate function will become non-functional when migrating to `SDLManager` from `SDLProxy`, but changing these to use the new handlers is simple and will be described in the section **SDLProxy to SDLManager**.

## Deprecating SDLRPCRequestFactory
If you are using the `SDLRPCRequestFactory` class, you will need to update the initializers of all RPCs to use this. This will be the following migrations:

```objc 
- (void)hsdl_performWelcomeMessage {
    NSLog(@"Send welcome message");
    SDLShow *show = [[SDLShow alloc] init];
    show.mainField1 = WelcomeShow;
    show.alignment = [SDLTextAlignment CENTERED];
    show.correlationID = [self hsdl_getNextCorrelationId];
    [self.proxy sendRPC:show];

    SDLSpeak *speak = [SDLRPCRequestFactory buildSpeakWithTTS:WelcomeSpeak correlationID:[self hsdl_getNextCorrelationId]];
    [self.proxy sendRPC:speak];
}
```

to

```objc
- (void)hsdl_performWelcomeMessage {
    NSLog(@"Send welcome message");
    SDLShow *show = [[SDLShow alloc] initWithMainField1:WelcomeShow mainField2:nil alignment:SDLTextAlignment.CENTERED];
    [self.proxy sendRPC:show];

    SDLSpeak *speak = [[SDLSpeak alloc] initWithTTS:WelcomeSpeak];
    [self.proxy sendRPC:speak];
}
```

!!! note
We are not updating all of the functions that utilize `SDLRPCRequestFactory`, because we are going to be deleting those functions later on in the guide.
!!!

## SDLProxy to SDLManager                               
In versions following 4.3, the `SDLProxy` is no longer your main point of interaction with SDL. Instead, `SDLManager` was introduced to allow for apps to more easily integrate with SDL, and to not worry about things such as registering their app, uploading images, and showing a lock screen.

Our first step of removing the usage of `SDLProxy` is to add in an `SDLManager` instance variable. `SDLManager` is started with a `SDLConfiguration`, which contains settings relating to the application.

```objc
@interface HSDLProxyManager () <SDLManagerDelegate> // Replace SDLProxyListener with SDLManagerDelegate

@property (nonatomic, strong) SDLManager *manager; // New instance variable
@property (nonatomic, strong) SDLLifecycleConfiguration *lifecycleConfiguration; // New instance variable
@property (nonatomic, strong) SDLProxy *proxy;
@property (nonatomic, assign) NSUInteger correlationID;								
@property (nonatomic, strong) NSNumber *appIconId;									
@property (nonatomic, strong) NSMutableSet *remoteImages;							
@property (nonatomic, assign, getter=isGraphicsSupported) BOOL graphicsSupported;
@property (nonatomic, assign, getter=isFirstHmiFull) BOOL firstHmiFull;
@property (nonatomic, assign, getter=isFirstHmiNotNone) BOOL firstHmiNotNone;
@property (nonatomic, assign, getter=isVehicleDataSubscribed) BOOL vehicleDataSubscribed;

@end
```

### SDLProxyListener to SDLManagerDelegate
`SDLManagerDelegate` is a small protocol that gives back only 2 callbacks, as compared to `SDLProxyListener`'s 67 callbacks. As mentioned before, all of these callbacks from `SDLProxyListener` are now sent out as `NSNotification`s, and the names for these are located in `SDLNotificationConstants.h`. From these delegate changes, we can modify the following functions to use the new `SDLManagerDelegate` callbacks

`onProxyClosed` to `managerDidDisconnect`:

```objc
- (void)onProxyClosed {
    NSLog(@"SDL Disconnect");

    // Reset state variables
    self.firstHmiFull = YES;
    self.firstHmiNotNone = YES;
    self.graphicsSupported = NO;
    [self.remoteImages removeAllObjects];
    self.vehicleDataSubscribed = NO;
    self.appIconId = nil;

    // Notify the app delegate to clear the lockscreen
    [self hsdl_postNotification:HSDLDisconnectNotification info:nil];

    // Cycle the proxy
    [self disposeProxy];
    [self startProxy];
}
```

to

```objc
- (void)managerDidDisconnect {
    NSLog(@"SDL Disconnect");
    
    // Reset state variables
    self.firstHmiFull = YES;
    self.firstHmiNotNone = YES;
    self.graphicsSupported = NO;
    self.vehicleDataSubscribed = NO;
    
    // Notify the app delegate to clear the lockscreen
    [self hsdl_postNotification:HSDLDisconnectNotification info:nil];
}
```

`onOnHMIStatus:` to `hmiLevel:didChangeToLevel:`
```objc
- (void)onOnHMIStatus:(SDLOnHMIStatus *)notification {
    NSLog(@"HMIStatus notification from SDL");

    // Send welcome message on first HMI FULL
    if ([[SDLHMILevel FULL] isEqualToEnum:notification.hmiLevel]) {
        if (self.isFirstHmiFull) {
            self.firstHmiFull = NO;
            [self hsdl_performWelcomeMessage];
        }

        // Other HMI (Show, PerformInteraction, etc.) would go here
    }

    // Send AddCommands in first non-HMI NONE state (i.e., FULL, LIMITED, BACKGROUND)
    if (![[SDLHMILevel NONE] isEqualToEnum:notification.hmiLevel]) {
        if (self.isFirstHmiNotNone) {
            self.firstHmiNotNone = NO;
            [self hsdl_addCommands];

            // Other app setup (SubMenu, CreateChoiceSet, etc.) would go here
        }
    }
}
```

to

```objc
- (void)hmiLevel:(SDLHMILevel *)oldLevel didChangeToLevel:(SDLHMILevel *)newLevel {
    NSLog(@"HMIStatus notification from SDL");
    
    // Send welcome message on first HMI FULL
    if ([[SDLHMILevel FULL] isEqualToEnum:newLevel]) {
        if (self.isFirstHmiFull) {
            self.firstHmiFull = NO;
            [self hsdl_performWelcomeMessage];
        }
        
        // Other HMI (Show, PerformInteraction, etc.) would go here
    }
    
    // Send AddCommands in first non-HMI NONE state (i.e., FULL, LIMITED, BACKGROUND)
    if (![[SDLHMILevel NONE] isEqualToEnum:newLevel]) {
        if (self.isFirstHmiNotNone) {
            self.firstHmiNotNone = NO;
            [self hsdl_addCommands];
            
            // Other app setup (SubMenu, CreateChoiceSet, etc.) would go here
        }
    }
}
```

We can also remove the functions relating to lifecycle management and app icons, as the **Creating the Manager** section of the migration guide will handle this:
- `hsdl_uploadImages`
- `onListFilesResponse:`
- `onPutFileResponse:`
- `hsdl_setAppIcon`

And can also remove the `remoteImages` property from the list of instance variables.

### Correlation Ids
We no longer require developers to keep track of correlation ids, as `SDLManager` does this for you. Because of this, you can remove `correlationID` and `appIconId` from the list of instance variables. 

Because of this, we can remove the `hsdl_getNextCorrelationId` method as well.

!!! note
If you set the correlation id, it will be overwritten by `SDLManager`.
!!!

### Creating the Manager
In `HSDLProxyManager`'s `init` function, we will build these components, and begin removing components that are no longer needed, as `SDLManager` handles it.

```objc
- (instancetype)init {
    if (self = [super init]) {
        _correlationID = 1;	// No longer needed, remove instance variable.
        _graphicsSupported = NO;
        _firstHmiFull = YES;
        _firstHmiNotNone = YES;
        _remoteImages = [[NSMutableSet alloc] init]; // No longer needed, remove instance variable.
        _vehicleDataSubscribed = NO;

        // SDLManager initialization
        
         // If connecting via USB (to a vehicle).
//        _lifecycleConfiguration = [SDLLifecycleConfiguration defaultConfigurationWithAppName:AppName appId:AppId];
        
        // If connecting via TCP/IP (to an emulator).
        _lifecycleConfiguration = [SDLLifecycleConfiguration debugConfigurationWithAppName:AppName appId:AppId ipAddress:RemoteIpAddress port:RemotePort];
        
        _lifecycleConfiguration.appType = AppIsMediaApp ? [SDLAppHMIType MEDIA] : [SDLAppHMIType DEFAULT];
        _lifecycleConfiguration.shortAppName = ShortAppName;
        _lifecycleConfiguration.voiceRecognitionCommandNames = @[AppVrSynonym];
        _lifecycleConfiguration.ttsName = [SDLTTSChunk textChunksFromString:AppName];

        UIImage* appIcon = [UIImage imageNamed:IconFile];
        if (appIcon) {
            _lifecycleConfiguration.appIcon = [SDLArtwork artworkWithImage:appIcon name:IconFile asImageFormat:SDLArtworkImageFormatPNG];
        }
        
        // SDLConfiguration contains the lifecycle and lockscreen configurations
        SDLConfiguration *configuration = [SDLConfiguration configurationWithLifecycle:_lifecycleConfiguration lockScreen:[SDLLockScreenConfiguration enabledConfiguration]];
        
        _manager = [[SDLManager alloc] initWithConfiguration:configuration delegate:self];
    }
    return self;
}
```

We must also update the `RemotePort` constant from a type of `NSString *` to `UInt16`.

Because we the way we configure the app's properties via `SDLLifecycleConfiguration` now, we do not need to actually send an `SDLRegisterAppInterface` request. Because of this, we can remove the `onProxyOpened` method and it's contents.

### Built-In Lock Screen
Versions of SDL moving forward contain a lock screen manager to allow for easily customizing and using a lock screen. For more information, please check out the [Adding the Lock Screen](Adding the Lock Screen) section.

With the lockscreen handles for us now, we can remove the following from `HSDLProxyManager`:

Constants (from .h and .m)
- `HSDLDisconnectNotification`
- `HSDLLockScreenStatusNotification`
- `HSDLNotificationUserInfoObject`

Functions
- `hsdl_postNotification:info:`
- last line of `managerDidDisconnect`
- `onOnLockScreenNotification:`

We also can eliminate the `LockScreenViewController` files, and remove the following line numbers/ranges from `AppDelegate.m`:
- lines 61-121
- lines 25-31
- lines 15-16

We also can open Main.storyboard, and remove the `LockScreenViewController`.

### Starting the Manager and Register App Interface
In previous implementations, a developer would need to react to the `onRegisterAppInterfaceResponse:` to get information regarding their application and the currently connected Core. Now, however, we can simply access these properties after the `SDLManager` has been started.

First, we must start the manager. Change the `startProxy` function from:

```objc
- (void)startProxy {
    NSLog(@"startProxy");
    
    // If connecting via USB (to a vehicle).
//    self.proxy = [SDLProxyFactory buildSDLProxyWithListener:self];

    // If connecting via TCP/IP (to an emulator).
    self.proxy = [SDLProxyFactory buildSDLProxyWithListener:self tcpIPAddress:RemoteIpAddress tcpPort:RemotePort];
}
```

to:

```objc
- (void)startProxy {
    NSLog(@"startProxy");
    
    __weak typeof(self) weakself = self;
    [self.manager startWithReadyHandler:^(BOOL success, NSError * _Nullable error) {
        if (!success) {
            NSLog(@"Error trying to start SDLManager: %@", error);
            return;
        }
        
        NSNumber<SDLBool> graphicSupported = weakself.systemCapabilityManager.displayCapabilities.graphicSupported
        if (graphicSupported != nil) {
            weakself.graphicsSupported = graphicSupported;
        }
    }];
}
```

We can now remove `onRegisterAppInterfaceResponse:`.

### Stopping the Manager
Stopping the manager is simply changing from

```objc
- (void)disposeProxy {
    NSLog(@"disposeProxy");
    [self.proxy dispose];
    self.proxy = nil;
}
```

to

```objc
- (void)disposeProxy {
    NSLog(@"disposeProxy");
    [self.manager stop];
}
```

### Adding Notification Handlers
Registering for a notification is similar to registering for `NSNotification`s. The list of these subscribable notifications is in `SDLNotificationConstants.h`. For this project, we are observing the `onDriverDistraction:` notification and logging a string. We will modify this to instead listen for a notification and the log the same string.

Remove this function
```objc
- (void)onOnDriverDistraction:(SDLOnDriverDistraction *)notification {
    NSLog(@"OnDriverDistraction notification from SDL");
    // Some RPCs (depending on region) cannot be sent when driver distraction is active.
}
```

And add in the notification observer
```objc
- (instancetype)init {
    if (self = [super init]) {
    	// Previous code setting up SDLManager

        // Add in the notification observer
        [[NSNotificationCenter defaultCenter] addObserverForName:SDLDidChangeDriverDistractionStateNotification object:nil queue:nil usingBlock:^(NSNotification * _Nonnull note) {
            SDLRPCNotificationNotification* notification = (SDLRPCNotificationNotification*)note;
            
            if (![notification.notification isKindOfClass:SDLOnDriverDistraction.class]) {
                return;
            }
            
            NSLog(@"OnDriverDistraction notification from SDL");
            // Some RPCs (depending on region) cannot be sent when driver distraction is active.
        }];
    }
    return self;
}
```

We will also remove all of the remaining delegate functions from `SDLProxyListener`, except for `onAddCommandResponse:`.

### Handling command notifications

`SDLAddCommand` utilizes the new handler mechanism for responding to when a user interacts with the command you have added. When using the initializer, you can see we set the new `handler` property to use the same code we originally wrote in `onOnCommand:`.

```objc
- (void)hsdl_addCommands {
    NSLog(@"hsdl_addCommands");
    SDLMenuParams *menuParams = [[SDLMenuParams alloc] init];
    menuParams.menuName = TestCommandName;
    SDLAddCommand *command = [[SDLAddCommand alloc] init];
    command.vrCommands = [NSMutableArray arrayWithObject:TestCommandName];
    command.menuParams = menuParams;
    command.cmdID = @(TestCommandID);

    [self.proxy sendRPC:command];
}

- (void)onOnCommand:(SDLOnCommand *)notification {
    NSLog(@"OnCommand notification from SDL");

    // Handle sample command when triggered
    if ([notification.cmdID isEqual:@(TestCommandID)]) {
        SDLShow *show = [[SDLShow alloc] init];
        show.mainField1 = @"Test Command";
        show.alignment = [SDLTextAlignment CENTERED];
        show.correlationID = [self hsdl_getNextCorrelationId];
        [self.proxy sendRPC:show];

        SDLSpeak *speak = [SDLRPCRequestFactory buildSpeakWithTTS:@"Test Command" correlationID:[self hsdl_getNextCorrelationId]];
        [self.proxy sendRPC:speak];
    }
}
```

to

```objc
- (void)hsdl_addCommands {
    NSLog(@"hsdl_addCommands");
    SDLAddCommand *command = [[SDLAddCommand alloc] initWithId:TestCommandID vrCommands:@[TestCommandName] menuName:TestCommandName handler:^(__kindof SDLRPCNotification * _Nonnull notification) {
        if (![notification isKindOfClass:SDLOnCommand.class]) {
            return;
        }
        
        NSLog(@"OnCommand notification from SDL");
        SDLOnCommand* onCommand = (SDLOnCommand*)notification;
        
        // Handle sample command when triggered
        if ([onCommand.cmdID isEqual:@(TestCommandID)]) {
            SDLShow *show = [[SDLShow alloc] initWithMainField1:@"Test Command" mainField2:nil alignment:SDLTextAlignment.CENTERED];
            [self.proxy sendRPC:show];
            
            SDLSpeak *speak = [[SDLSpeak alloc] initWithTTS:@"Test Command"];
            [self.proxy sendRPC:speak];
        }
    }];
    
    [self.proxy sendRPC:command];
}
```


### Sending Requests via `SDLManager`
As mentioned in **Response and Event Handlers**, `SDLManager` provides the ability to easily react to responses for RPCs we send out. `SDLManager` has two functions for sending RPCs:
- `sendRequest:withResponseHandler:`
- `sendRequest:`

We will update our `SDLAddCommand` request to both send the request via `SDLManager` instead of `SDLProxy`, as well as react to the response that we get back.

From
```objc
- (void)hsdl_addCommands {
    NSLog(@"hsdl_addCommands");
    SDLAddCommand *command = [[SDLAddCommand alloc] initWithId:TestCommandID vrCommands:@[TestCommandName] menuName:TestCommandName handler:^(__kindof SDLRPCNotification * _Nonnull notification) {
        if (![notification isKindOfClass:SDLOnCommand.class]) {
            return;
        }
        
        NSLog(@"OnCommand notification from SDL");
        SDLOnCommand* onCommand = (SDLOnCommand*)notification;
        
        // Handle sample command when triggered
        if ([onCommand.cmdID isEqual:@(TestCommandID)]) {
            SDLShow *show = [[SDLShow alloc] initWithMainField1:@"Test Command" mainField2:nil alignment:SDLTextAlignment.CENTERED];
            [self.proxy sendRPC:show];
            
            SDLSpeak *speak = [[SDLSpeak alloc] initWithTTS:@"Test Command"];
            [self.proxy sendRPC:speak];
        }
    }];
    
    [self.proxy sendRPC:command];
}
```

to

```objc
- (void)hsdl_addCommands {
    NSLog(@"hsdl_addCommands");
    SDLAddCommand *command = [[SDLAddCommand alloc] initWithId:TestCommandID vrCommands:@[TestCommandName] menuName:TestCommandName handler:^(__kindof SDLRPCNotification * _Nonnull notification) {
        if (![notification isKindOfClass:SDLOnCommand.class]) {
            return;
        }
        
        NSLog(@"OnCommand notification from SDL");
        SDLOnCommand* onCommand = (SDLOnCommand*)notification;
        
        // Handle sample command when triggered
        if ([onCommand.cmdID isEqual:@(TestCommandID)]) {
            SDLShow *show = [[SDLShow alloc] initWithMainField1:@"Test Command" mainField2:nil alignment:SDLTextAlignment.CENTERED];
            [self.manager sendRequest:show];
            
            SDLSpeak *speak = [[SDLSpeak alloc] initWithTTS:@"Test Command"];
            [self.manager sendRequest:speak];
        }
    }];
    
    [self.manager sendRequest:command withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
        NSLog(@"AddCommand response from SDL: %@ with info: %@", response.resultCode, response.info);
    }];
}
```

And now, we can remove `onAddCommandResponse:`

We can also update `hsdl_performWelcomeMessage`
```objc
- (void)hsdl_performWelcomeMessage {
    NSLog(@"Send welcome message");
    SDLShow *show = [[SDLShow alloc] initWithMainField1:WelcomeShow mainField2:nil alignment:SDLTextAlignment.CENTERED];
    [self.proxy sendRPC:show];

    SDLSpeak *speak = [[SDLSpeak alloc] initWithTTS:WelcomeSpeak];
    [self.proxy sendRPC:speak];
}
```

to

```objc
- (void)hsdl_performWelcomeMessage {
    NSLog(@"Send welcome message");
    SDLShow *show = [[SDLShow alloc] initWithMainField1:WelcomeShow mainField2:nil alignment:SDLTextAlignment.CENTERED];
    [self.manager sendRequest:show];

    SDLSpeak *speak = [[SDLSpeak alloc] initWithTTS:WelcomeSpeak];
    [self.manager sendRequest:speak];
}
```


## Uploading Files via `SDLManager`'s `SDLFileManager`
`SDLPutFile` is the original means of uploading a file. In 4.3+, we have abstracted this out, and instead provide the functionality via two new classes: `SDLFile` and `SDLArtwork`. For more information on these, check out the [Uploading Files and Graphics](Uploading Files and Graphics) section.

We can change `hsdl_uploadImage:withCorrelationID:` from
```objc
- (void)hsdl_uploadImage:(NSString *)imageName withCorrelationID:(NSNumber *)corrId {
    NSLog(@"hsdl_uploadImage: %@", imageName);
    if (imageName) {
        UIImage *pngImage = [UIImage imageNamed:IconFile];
        if (pngImage) {
            NSData *pngData = UIImagePNGRepresentation(pngImage);
            if (pngData) {
                SDLPutFile *putFile = [[SDLPutFile alloc] init];
                putFile.syncFileName = imageName;
                putFile.fileType = [SDLFileType GRAPHIC_PNG];
                putFile.persistentFile = @YES;
                putFile.systemFile = @NO;
                putFile.offset = @0;
                putFile.length = [NSNumber numberWithUnsignedLong:pngData.length];
                putFile.bulkData = pngData;
                putFile.correlationID = corrId;
                [self.proxy sendRPC:putFile];
            }
        }
    }
}
```

to

```objc
- (void)hsdl_uploadImage:(NSString *)imageName {
    NSLog(@"hsdl_uploadImage: %@", imageName);
    if (!imageName) {
        return;
    }
    
    UIImage *pngImage = [UIImage imageNamed:imageName];
    if (!pngImage) {
        return;
    }
    
    SDLFile *file = [SDLArtwork persistentArtworkWithImage:pngImage name:imageName asImageFormat:SDLArtworkImageFormatPNG];
    [self.manager.fileManager uploadFile:file completionHandler:^(BOOL success, NSUInteger bytesAvailable, NSError * _Nullable error) {
        if (!success) {
            NSLog(@"Error uploading file: %@", error);
        }
        
        NSLog(@"File uploaded");
    }];
}
```

We can now finally remove `SDLProxy` from the project's instance variables.
