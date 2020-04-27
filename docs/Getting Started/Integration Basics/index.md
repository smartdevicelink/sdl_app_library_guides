# Integration Basics
### How SDL Works
SmartDeviceLink works by sending remote procedure calls (RPCs) back and forth between a smartphone application and the SDL Core. These RPCs allow you to build the user interface, detect button presses, play audio, and get vehicle data, among other things. You will use the SDL library to build your app on the SDL Core.

@![iOS]
## Set Up a Proxy Manager Class
You will need a class that manages the RPCs sent back and forth between your app and SDL Core. Since there should be only one active connection to the SDL Core, you may wish to implement this proxy class using the singleton pattern.

##### Objective-C
###### ProxyManager.h

```objc
#import <Foundation/Foundation.h>

NS_ASSUME_NONNULL_BEGIN

@interface ProxyManager : NSObject

+ (instancetype)sharedManager;

@end

NS_ASSUME_NONNULL_END
```

###### ProxyManager.m
```objc
#import "ProxyManager.h"

NS_ASSUME_NONNULL_BEGIN

@interface ProxyManager ()

@end

@implementation ProxyManager

+ (instancetype)sharedManager {
    static ProxyManager* sharedManager = nil;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        sharedManager = [[ProxyManager alloc] init];
    });

    return sharedManager;
}

- (instancetype)init {
    self = [super init];
    if (!self) {
      return nil;
    }
}

@end

NS_ASSUME_NONNULL_END
```

##### Swift
```swift
class ProxyManager: NSObject {
  // Singleton
  static let sharedManager = ProxyManager()

  private override init() {
    super.init()
  }
}
```

Your app should always start passively watching for a connection with a SDL Core as soon as the app launches. The easy way to do this is by instantiating the `ProxyManager` class in the `didFinishLaunchingWithOptions()` method in your `AppDelegate` class.

The connect method will be implemented later. To see a full example, navigate to the bottom of this page.

##### Objective-C
```objc
@implementation AppDelegate

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
  // Initialize and start the proxy
  [[ProxyManager sharedManager] connect];
}

@end
```

##### Swift
```swift
class AppDelegate: UIResponder, UIApplicationDelegate {
  func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
    // Initialize and start the proxy
    ProxyManager.sharedManager.connect()

    return true
  }
}
```

## Importing the SDL Library
At the top of the `ProxyManager` class, import the SDL for iOS library.

##### Objective-C
```objc
#import <SmartDeviceLink/SmartDeviceLink.h>
```

##### Swift
```swift
import SmartDeviceLink
```

## Creating the SDL Manager
The `SDLManager` is the main class of SmartDeviceLink. It will handle setting up the initial connection with the head unit. It will also help you upload images and send RPCs.

##### Objective-C
```objc
#import "ProxyManager.h"
#import "SmartDeviceLink.h"

NS_ASSUME_NONNULL_BEGIN

@interface ProxyManager ()

@property (nonatomic, strong) SDLManager *sdlManager;

@end

@implementation ProxyManager

+ (instancetype)sharedManager {
    static ProxyManager *sharedManager = nil;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        sharedManager = [[ProxyManager alloc] init];
    });

    return sharedManager;
}

- (instancetype)init {
    self = [super init];
    if (!self) {
      return nil;
    }

    return self
}

@end

NS_ASSUME_NONNULL_END
```

##### Swift
```swift
class ProxyManager: NSObject {
  // Manager
  fileprivate var sdlManager: SDLManager!

  // Singleton
  static let sharedManager = ProxyManager()

  private override init() {
    super.init()
  }
}
```

### 1. Create a Lifecycle Configuration
In order to instantiate the `SDLManager` class, you must first configure an `SDLConfiguration`. To start, we will look at the `SDLLifecycleConfiguration`. You will at minimum need a `SDLLifecycleConfiguration` instance with the application name and application id. During the development stage, a dummy app id is usually sufficient. For more information about obtaining an application id, please consult the [SDK Configuration](Getting Started/SDK Configuration) section of this guide. You must also decide which network configuration to use to connect the app to the SDL Core. Optional, but recommended, configuration properties include short app name, app icon, and app type.

#### Network Connection Type
There are two different ways to connect your app to a SDL Core: with a TCP (Wi-Fi) network connection or with an iAP (USB / Bluetooth) network connection. Use TCP for debugging and use iAP for production level apps.

##### iAP
###### Objective-C
```objc
SDLLifecycleConfiguration* lifecycleConfiguration = [SDLLifecycleConfiguration defaultConfigurationWithAppName:@"<#App Name#>" fullAppId:@"<#App Id#>"];
```
###### Swift
```swift
let lifecycleConfiguration = SDLLifecycleConfiguration(appName:"<#App Name#>", fullAppId: "<#App Id#>")
```

##### TCP
###### Objective-C
```objc
SDLLifecycleConfiguration* lifecycleConfiguration = [SDLLifecycleConfiguration debugConfigurationWithAppName:@"<#App Name#>" fullAppId:@"<#App Id#>" ipAddress:@"<#IP Address#>" port:<#Port#>];
```
###### Swift
```swift
let lifecycleConfiguration = SDLLifecycleConfiguration(appName: "<#App Name#>", fullAppId: "<#App Id#>", ipAddress: "<#IP Address#>", port: <#Port#>)
```

!!! NOTE
If you are connecting your app to an emulator using a TCP connection, the IP address is your computer or virtual machine’s IP address, and the port number is usually 12345.
!!!

### 2. Short App Name (optional)
This is a shortened version of your app name that is substituted when the full app name will not be visible due to character count constraints. You will want to make this as short as possible.

##### Objective-C
```objc
lifecycleConfiguration.shortAppName = @"<#Shortened App Name#>";
```

##### Swift
```swift
lifecycleConfiguration.shortAppName = "<#Shortened App Name#>"
```

### 3. App Icon
This is a custom icon for your application. Please refer to [Adaptive Interface Capabilities](Displaying a User Interface/Adaptive Interface Capabilities) for icon sizes.

##### Objective-C
```objc
UIImage* appImage = [UIImage imageNamed:@"<#AppIcon Name#>"];
if (appImage) {
  SDLArtwork* appIcon = [SDLArtwork persistentArtworkWithImage:appImage name:@"<#Name to Upload As#>" asImageFormat:SDLArtworkImageFormatPNG /* or SDLArtworkImageFormatJPG */];
  lifecycleConfiguration.appIcon = appIcon;
}
```

##### Swift
```swift
if let appImage = UIImage(named: "<#AppIcon Name#>") {
  let appIcon = SDLArtwork(image: appImage, name: "<#Name to Upload As#>", persistent: true, as: .JPG /* or .PNG */)
  lifecycleConfiguration.appIcon = appIcon
}
```

!!! NOTE
Persistent files are used when the image ought to remain on the remote system between ignition cycles. This is commonly used for menu artwork, soft button artwork and app icons. Non-persistent artwork is usually used for dynamic images like music album artwork.
!!!

### 4. App Type (optional)
The app type is used by car manufacturers to decide how to categorize your app. Each car manufacturer has a different categorization system. For example, if you set your app type as media, your app will also show up in the audio tab as well as the apps tab of Ford’s SYNC3 head unit. The app type options are: default, communication, media (i.e. music/podcasts/radio), messaging, navigation, projection, information, and social.

!!! NOTE
Navigation and projection applications both use video and audio byte streaming. However, navigation apps require special permissions from OEMs, and projection apps are only for internal use by OEMs.
!!!

##### Objective-C
```objc
lifecycleConfiguration.appType = SDLAppHMITypeMedia;
```

##### Swift
```swift
lifecycleConfiguration.appType = .media
```

#### Additional App Types
If one app type doesn't cover your full app use-case, you can add additional `AppHMIType`s as well.

##### Objective-C

```objc
lifecycleConfiguration.additionalAppTypes = @[SDLAppHMITypeInformation];
```

##### Swift

```swift
lifecycleConfiguration.additionalAppTypes = [.information];
```

### 5. Template Coloring
You can customize the color scheme of your templates. For more information, see the [Customizing the Template guide](Customizing Look and Functionality/Customizing the Template) section.

### 6. Determine SDL Support
You have the ability to determine a minimum SDL protocol and minimum SDL RPC version that your app supports. We recommend not setting these values until your app is ready for production. The OEMs you support will help you configure the correct `minimumProtocolVersion` and `minimumRPCVersion` during the application review process.

If a head unit is blocked by protocol version, your app icon will never appear on the head unit's screen. If you configure your app to block by RPC version, it will appear and then quickly disappear. So while blocking with `minimumProtocolVersion` is preferable, `minimumRPCVersion` allows you more granular control over which RPCs will be present.

##### Objective-C
```objc
lifecycleConfiguration.minimumProtocolVersion = [SDLVersion versionWithString:@"3.0.0"];
lifecycleConfiguration.minimumRPCVersion = [SDLVersion versionWithString:@"4.0.0"];
```

##### Swift
```swift
lifecycleConfiguration.minimumProtocolVersion = SDLVersion(string: "3.0.0")
lifecycleConfiguration.minimumRPCVersion = SDLVersion(string: "4.0.0")
```

### 7. Lock Screen
A lock screen is used to prevent the user from interacting with the app on the smartphone while they are driving. When the vehicle starts moving, the lock screen is activated. Similarly, when the vehicle stops moving, the lock screen is removed. You must implement a lock screen in your app for safety reasons. Any application without a lock screen will not get approval for release to the public.

The SDL SDK can take care of the lock screen implementation for you, automatically using your app logo and the connected vehicle logo. If you do not want to use the default lock screen, you can implement your own custom lock screen.

For more information, please refer to the [Adding the Lock Screen](Getting Started/Adding the Lock Screen) section, for this guide we will be using `SDLLockScreenConfiguration`'s basic `enabledConfiguration`.

##### Objective-C
```objc
[SDLLockScreenConfiguration enabledConfiguration]
```

##### Swift
```swift
SDLLockScreenConfiguration.enabled()
```

### 8. Logging
A logging configuration is used to define where and how often SDL will log. It will also allow you to set your own logging modules and filters. For more information about setting up logging, see [the logging guide](Developer Tools/Configuring SDL Logging).

##### Objective-C
```objc
[SDLLogConfiguration defaultConfiguration]
```

##### Swift
```swift
SDLLogConfiguration.default()
```

### 9. File Manager
The file manager configuration allows you to configure retry behavior for uploading files and images. The default configuration attempts one re-upload, but will fail after that.

##### Objective-C
```objc
[SDLFileManagerConfiguration defaultConfiguration];
```

##### Swift
```swift
SDLFileManagerConfiguration.default()
```

### 10. Set the Configuration
The `SDLConfiguration` class is used to set the lifecycle, lock screen, logging, and optionally (dependent on if you are a Navigation or Projection app) streaming media configurations for the app. Use the lifecycle configuration settings above to instantiate a `SDLConfiguration` instance.

##### Objective-C
```objc
SDLConfiguration* configuration = [SDLConfiguration configurationWithLifecycle:lifecycleConfiguration lockScreen:[SDLLockScreenConfiguration enabledConfiguration] logging:[SDLLogConfiguration defaultConfiguration] fileManager:[SDLFileManagerConfiguration defaultConfiguration]];
```

##### Swift
```swift
let configuration = SDLConfiguration(lifecycle: lifecycleConfiguration, lockScreen: .enabled(), logging: .default(), fileManager: .default())
```

### 11. Create a SDLManager
Now you can use the `SDLConfiguration` instance to instantiate the `SDLManager`.

##### Objective-C
```objc
self.sdlManager = [[SDLManager alloc] initWithConfiguration:configuration delegate:self];
```

##### Swift
```swift
sdlManager = SDLManager(configuration: configuration, delegate: self)
```

### 12. Start the SDLManager
The manager should be started as soon as possible in your application's lifecycle. We suggest doing this in the `didFinishLaunchingWithOptions()` method in your `AppDelegate` class. Once the manager has been initialized, it will immediately start watching for a connection with the remote system. The manager will passively search for a connection with a SDL Core during the entire lifespan of the app. If the manager detects a connection with a SDL Core, the `startWithReadyHandler` will be called.

Create a new function in the `ProxyManager` class called `connect`.

##### Objective-C
```objc
- (void)connect {
    [self.sdlManager startWithReadyHandler:^(BOOL success, NSError * _Nullable error) {
        if (success) {
            // Your app has successfully connected with the SDL Core
        }
    }];
}
```

##### Swift
```swift
func connect() {
    // Start watching for a connection with a SDL Core
    sdlManager.start { (success, error) in
        if success {
            // Your app has successfully connected with the SDL Core
        }
    }
}
```

!!! NOTE
In production, your app will be watching for connections using iAP, which will not use any more battery power than normal.
!!!

If the connection is successful, you can start sending RPCs to the SDL Core. However, some RPCs can only be sent when the HMI is in the `FULL` or `LIMITED` state. If the SDL Core's HMI is not ready to accept these RPCs, your requests will be ignored. If you want to make sure that the SDL Core will not ignore your RPCs, use the `SDLManagerDelegate` methods in the next section.

#### Implement the SDL Manager Delegate
The `ProxyManager` class should conform to the `SDLManagerDelegate` protocol. This means that the `ProxyManager` class must implement the following required methods:

1. `managerDidDisconnect` This function is called when the proxy disconnects from the SDL Core. Do any cleanup you need to do in this function.
2. `hmiLevel:didChangeToLevel:` This function is called when the HMI level changes for the app. The HMI level can be `FULL`, `LIMITED`, `BACKGROUND`, or `NONE`. It is important to note that most RPCs sent while the HMI is in `BACKGROUND` or `NONE` mode will be ignored by the SDL Core. For more information, please refer to [Understanding Permissions](Getting Started/Understanding Permissions).

In addition, there are three optional methods:

1. `audioStreamingState:didChangeToState:` Called when the audio streaming state of this application changes on the remote system. For more information, please refer to [Understanding Permissions](Getting Started/Understanding Permissions).
1. `systemContext:didChangeToContext:` Called when the system context (i.e. a menu is open, an alert is visible,  a voice recognition session is in progress) of this application changes on the remote system. For more information, please refer to [Understanding Permissions](Getting Started/Understanding Permissions).
1. `managerShouldUpdateLifecycleToLanguage:` Called when the head unit language does not match the `language` set in the `SDLLifecycleConfiguration` but does match a language included in `languagesSupported`. If desired, you can customize the `appName`, the `shortAppName`,  and `ttsName` for the head unit's current language. For more information about supporting more than one language in your app please refer to [Getting Started/Adapting to the Head Unit Language](Getting Started/Adapting to the Head Unit Language).

### Example Implementation of a Proxy Class
The following code snippet has an example of setting up both a TCP and iAP connection.

##### Objective-C
###### ProxyManager.h
```objc
#import <Foundation/Foundation.h>

NS_ASSUME_NONNULL_BEGIN

@interface ProxyManager : NSObject

+ (instancetype)sharedManager;
- (void)start;

@end

NS_ASSUME_NONNULL_END
```

###### ProxyManager.m
```objc
#import <SmartDeviceLink/SmartDeviceLink.h>

NS_ASSUME_NONNULL_BEGIN

static NSString* const AppName = @"<#App Name#>";
static NSString* const AppId = @"<#App Id#>";
@interface ProxyManager () <SDLManagerDelegate>

@property (nonatomic, strong) SDLManager* sdlManager;

@end

@implementation ProxyManager

+ (instancetype)sharedManager {
    static ProxyManager *sharedManager = nil;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        sharedManager = [[ProxyManager alloc] init];
    });

    return sharedManager;
}

- (instancetype)init {
    self = [super init];
    if (!self) {
        return nil;
    }

    // Used for USB Connection
    SDLLifecycleConfiguration* lifecycleConfiguration = [SDLLifecycleConfiguration defaultConfigurationWithAppName:AppName fullAppId:AppId];

    // Used for TCP/IP Connection
//    SDLLifecycleConfiguration* lifecycleConfiguration = [SDLLifecycleConfiguration debugConfigurationWithAppName:AppName fullAppId:AppId  ipAddress:@"<#IP Address#>" port:<#Port#>];

    UIImage* appImage = [UIImage imageNamed:@"<#AppIcon Name#>"];
    if (appImage) {
        SDLArtwork* appIcon = [SDLArtwork persistentArtworkWithImage:appImage name:@"<#Name to Upload As#>" asImageFormat:SDLArtworkImageFormatJPG /* or SDLArtworkImageFormatPNG */];
        lifecycleConfiguration.appIcon = appIcon;
    }

    lifecycleConfiguration.shortAppName = @"<#Shortened App Name#>";
    lifecycleConfiguration.appType = [SDLAppHMIType MEDIA];

    SDLConfiguration* configuration = [SDLConfiguration configurationWithLifecycle:lifecycleConfiguration lockScreen:[SDLLockScreenConfiguration enabledConfiguration] logging:[SDLLogConfiguration defaultConfiguration] fileManager:[SDLFileManager defaultConfiguration]];

    self.sdlManager = [[SDLManager alloc] initWithConfiguration:configuration delegate:self];

    return self;
}

- (void)connect {
    [self.sdlManager startWithReadyHandler:^(BOOL success, NSError * _Nullable error) {
        if (success) {
            // Your app has successfully connected with the SDL Core
        }
    }];
}

#pragma mark SDLManagerDelegate
- (void)managerDidDisconnect {
    NSLog(@"Manager disconnected!");
}

- (void)hmiLevel:(SDLHMILevel *)oldLevel didChangeToLevel:(SDLHMILevel *)newLevel {
    NSLog(@"Went from HMI level %@ to HMI Level %@", oldLevel, newLevel);
}

@end

NS_ASSUME_NONNULL_END
```

##### Swift
```swift
import SmartDeviceLink

class ProxyManager: NSObject {
    private let appName = "<#App Name#>"
    private let appId = "<#App Id#>"

    // Manager
    fileprivate var sdlManager: SDLManager!

    // Singleton
    static let sharedManager = ProxyManager()

    private override init() {
        super.init()

        // Used for USB Connection
        let lifecycleConfiguration = SDLLifecycleConfiguration(appName: appName, fullAppId: appId)

        // Used for TCP/IP Connection
        // let lifecycleConfiguration = SDLLifecycleConfiguration(appName: appName, fullAppId: appId, ipAddress: "<#IP Address#>", port: <#Port#>)

        // App icon image
        if let appImage = UIImage(named: "<#AppIcon Name#>") {
            let appIcon = SDLArtwork(image: appImage, name: "<#Name to Upload As#>", persistent: true, as: .JPG /* or .PNG */)
            lifecycleConfiguration.appIcon = appIcon
        }

        lifecycleConfiguration.shortAppName = "<#Shortened App Name#>"
        lifecycleConfiguration.appType = .media

        let configuration = SDLConfiguration(lifecycle: lifecycleConfiguration, lockScreen: .enabled(), logging: .default(), fileManager: .default())

        sdlManager = SDLManager(configuration: configuration, delegate: self)
    }

    func connect() {
        // Start watching for a connection with a SDL Core
        sdlManager.start { (success, error) in
            if success {
                // Your app has successfully connected with the SDL Core
            }
        }
    }
}

//MARK: SDLManagerDelegate
extension ProxyManager: SDLManagerDelegate {
  func managerDidDisconnect() {
    print("Manager disconnected!")
  }

  func hmiLevel(_ oldLevel: SDLHMILevel, didChangeToLevel newLevel: SDLHMILevel) {
    print("Went from HMI level \(oldLevel) to HMI level \(newLevel)")
  }
}
```

## Where to Go From Here
You should now be able to connect to a head unit or emulator. From here, [learn about designing your main interface](Displaying a User Interface/Main Screen Templates). For further details on connecting, see [Connecting to a SDL Core](Getting Started/Connecting to an Infotainment System).
!@

@![android]
In this guide, we exclusively use Android Studio. We are going to set-up a bare-bones application so you get started using SDL.

!!! NOTE
The SDL Mobile library supports [Android 2.2.x (API Level 8)](https://developer.android.com/about/versions/android-2.2.html) or higher.
!!!
!@

@![javaSE,javaEE]
In this guide, we exclusively use IntelliJ. We are going to set-up a bare-bones application so you get started using SDL.

!!! NOTE
The SDL Java library supports Java 7 and above.
!!!
!@


@![android,javaSE,javaEE]
## SmartDeviceLink Service
A SmartDeviceLink Service should be created to manage the lifecycle of the SDL session. The `SdlService` should build and start an instance of the `SdlManager` which will automatically connect with a head unit when available. This `SdlManager` will handle sending and receiving messages to and from SDL after it is connected.

@![android]
!!! NOTE
Please be aware that using an Activity to host the SDL implementation will not work. Android 10 has [restrictions](https://developer.android.com/guide/components/activities/background-starts) on starting activities from the background and that is how the SDL library will start the supplied component. SDL apps should only use a foreground service to host the SDL implementation.
!!!
!@

Create a new service and name it appropriately, for this guide we are going to call it `SdlService`.
!@

@![android]
```java
public class SdlService extends Service {
    //...
}
```
!@

@![javaSE,javaEE]
```java
public class SdlService {
    //...
}
```
!@

@![android]
If you created the service using the Android Studio template then the service should have been added to your `AndroidManifest.xml`. If not, then service needs to be defined in the manifest:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.company.mySdlApplication">

    <application>

        <service
        android:name=".SdlService"
        android:enabled="true"/>

    </application>

</manifest>
```

### Entering the Foreground
Because of Android Oreo's requirements, it is mandatory that services enter the foreground for long running tasks. The first bit of integration is ensuring that happens in the `onCreate` method of the `SdlService` or similar. Within the service that implements the SDL lifecycle you will need to add a call to start the service in the foreground. This will include creating a notification to sit in the status bar tray. This information and icons should be relevant for what the service is doing/going to do. If you already start your service in the foreground, you can ignore this section.

```java

public void onCreate() {
    super.onCreate();
    //...
    if(Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
    	NotificationManager notificationManager = (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
     	notificationManager.createNotificationChannel(...);
     	Notification serviceNotification = new Notification.Builder(this, *Notification Channel*)
         	.setContentTitle(...)
         	.setSmallIcon(....)
         	.setLargeIcon(...)
         	.setContentText(...)
         	.setChannelId(channel.getId())
         	.build();
     	startForeground(id, serviceNotification);
     }
}

```
!!! NOTE
The sample code checks if the OS is of Android Oreo or newer to start a foreground service. It is up to the app developer if they wish to start the notification in previous versions.
!!!


### Exiting the Foreground
It's important that you don't leave your notification in the notification tray as it is very confusing to users. So in the `onDestroy` method in your service, simply call the `stopForeground` method.

```java
@Override
public void onDestroy(){
    //...
	if(Build.VERSION.SDK_INT>=Build.VERSION_CODES.O){
		NotificationManager notificationManager = (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
		if(notificationManager!=null){ //If this is the only notification on your channel
			notificationManager.deleteNotificationChannel(* Notification Channel*);
		}
		stopForeground(true);
	}
}
```
!@

@![android,javaSE,javaEE]
### Implementing SDL Manager
In order to correctly connect to an SDL enabled head unit developers need to implement methods for the proper creation and disposing of an `SdlManager` in our `SdlService`.

!!! NOTE
An instance of SdlManager cannot be reused after it is closed and properly disposed of. Instead, a new instance must be created. Only one instance of SdlManager should be in use at any given time.
!!!
!@

@![android]
```java
public class SdlService extends Service {

    //The manager handles communication between the application and SDL
    private SdlManager sdlManager = null;

    //...

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {

        if (sdlManager == null) {
            MultiplexTransportConfig transport = new MultiplexTransportConfig(this, APP_ID, MultiplexTransportConfig.FLAG_MULTI_SECURITY_OFF);

            // The app type to be used
            Vector<AppHMIType> appType = new Vector<>();
            appType.add(AppHMIType.MEDIA);

            // The manager listener helps you know when certain events that pertain to the SDL Manager happen
            SdlManagerListener listener = new SdlManagerListener() {

                @Override
                public void onStart() {
                	// After this callback is triggered the SdlManager can be used to interact with the connected SDL session (updating the display, sending RPCs, etc)
                }

                @Override
                public void onDestroy() {
                    SdlService.this.stopSelf();
                }

                @Override
                public void onError(String info, Exception e) {
                }
            };

            // Create App Icon, this is set in the SdlManager builder
            SdlArtwork appIcon = new SdlArtwork(ICON_FILENAME, FileType.GRAPHIC_PNG, R.mipmap.ic_launcher, true);

            // The manager builder sets options for your session
            SdlManager.Builder builder = new SdlManager.Builder(this, APP_ID, APP_NAME, transport, listener);
            builder.setShortAppName(shortAppName);
            builder.setAppTypes(appType);
            builder.setAppIcon(appIcon);
            builder.setFileManagerConfig(fileManagerConfig);
            builder.setLockScreenConfig(lockScreenConfig);
            sdlManager = builder.build();
            sdlManager.start();
        }

}
```

The `onDestroy()` method from the `SdlManagerListener` is called whenever the manager detects some disconnect in the connection, whether initiated by the app, by SDL, or by the device’s connection.

!!! IMPORTANT
The `sdlManager` must be shutdown properly in the `SdlService.onDestroy()` callback using the method `sdlManager.dispose()`.
!!!
!@

@![javaSE,javaEE]
```java
public class SdlService {

    //The manager handles communication between the application and SDL
    private SdlManager sdlManager = null;

    public SdlService(BaseTransportConfig config){
        buildSdlManager(config);
    }

    public void start() {
        if(sdlManager != null){
            sdlManager.start();
        }
    }

    public void stop() {
        if (sdlManager != null) {
            sdlManager.dispose();
            sdlManager = null;
        }
    }

    //...

    private void buildSdlManager(BaseTransportConfig transport) {

        if (sdlManager == null) {

            // The app type to be used
            Vector<AppHMIType> appType = new Vector<>();
            appType.add(AppHMIType.MEDIA);

            // The manager listener helps you know when certain events that pertain to the SDL Manager happen
            SdlManagerListener listener = new SdlManagerListener() {

                @Override
                public void onStart(SdlManager sdlManager) {
                    // After this callback is triggered the SdlManager can be used to interact with the connected SDL session (updating the display, sending RPCs, etc)
                }

                @Override
                public void onDestroy(SdlManager sdlManager) {
                }

                @Override
                public void onError(SdlManager sdlManager, String info, Exception e) {
                }
            };

            // Create App Icon, this is set in the SdlManager builder
            SdlArtwork appIcon = new SdlArtwork(ICON_FILENAME, FileType.GRAPHIC_PNG, ICON_PATH, true);

            // The manager builder sets options for your session
            SdlManager.Builder builder = new SdlManager.Builder(APP_ID, APP_NAME, listener);
            builder.setShortAppName(shortAppName);
            builder.setAppTypes(appType);
            builder.setTransportType(transport);
            builder.setAppIcon(appIcon);
            builder.setFileManagerConfig(fileManagerConfig);
            sdlManager = builder.build();
            sdlManager.start();
        }

    }
}
```

!!! IMPORTANT
The `sdlManager` must be shutdown properly if this class is shutting down in the respective method using the method `sdlManager.dispose()`.
!!!
!@

@![android,javaSE,javeEE]
## SdlManager Builder options


### Required
@![android]
1. Context - Current context
2. AppID - ID of applicaiton
3. AppName - Name of application
4. BaseTransportConfig - the type of transport that should be used for this SdlManager instance
5. SdlManagerListener - Listener that helps you know when certain events that pertain to the SDL Manager happen
```java
 SdlManager.Builder builder = new SdlManager.Builder(this, APP_ID, APP_NAME, transport, listener);
```

!!! IMPORTANT
Alternatively you can use a constructor without transport, transport will then have to be set by:
```java
SdlManager.Builder builder = new SdlManager.Builder(this, APP_ID, APP_NAME, listener);
builder.setTransportType(transport);
```
!!!
!@
@![javaSE, javaEE]
1. AppID - ID of applicaiton
2. AppName - Name of applicaiton
3. SdlManagerListener - Listener that helps you know when certain events that pertain to the SDL Manager happen

```java
SdlManager.Builder builder = new SdlManager.Builder(APP_ID, APP_NAME, listener);
```
!@

!@

### App Icon
This is a custom icon for your application. Please refer to [Adaptive Interface Capabilities](Displaying a User Interface/Adaptive Interface Capabilities) for icon sizes.

```java
builder.setAppIcon(appIcon);
```

### App Type 
The app type is used by car manufacturers to decide how to categorize your app. Each car manufacturer has a different categorization system. For example, if you set your app type as media, your app will also show up in the audio tab as well as the apps tab of Ford’s SYNC3 head unit. The app type options are: default, communication, media (i.e. music/podcasts/radio), messaging, navigation, projection, information, and social.

```java
Vector<AppHMIType> appHMITypes = new Vector<>();
appHMITypes.add(AppHMIType.MEDIA);
appHMITypes.add(AppHMIType.PROJECTION);

builder.setAppTypes(appHMITypes);
```

!!! NOTE
Navigation and projection applications both use video and audio byte streaming. However, navigation apps require special permissions from OEMs, and projection apps are only for internal use by OEMs.
!!!

#### Additional App Types
If one app type doesn't cover your full app use-case, you can add additional `AppHMIType`s as well.

### Short App Name 
This is a shortened version of your app name that is substituted when the full app name will not be visible due to character count constraints. You will want to make this as short as possible.

```java
builder.setShortAppName(shortAppName);
```

### Template Coloring
You can customize the color scheme of your templates. For more information, see the [Customizing the Template guide](Customizing Look and Functionality/Customizing the Template) section.

```java
TemplateColorScheme dayColorScheme = new TemplateColorScheme();
TemplateColorScheme nightColorScheme = new TemplateColorScheme();

builder.setDayColorScheme(dayColorScheme);
builder.setNightColorScheme(nightColorScheme);
```

### Determining SDL Support
You have the ability to determine a minimum SDL protocol and a minimum SDL RPC version that your app supports. We recommend not setting these values until your app is ready for production. The OEMs you support will help you configure the correct `minimumProtocolVersion` and `minimumRPCVersion` during the application review process.

If a head unit is blocked by protocol version, your app icon will never appear on the head unit's screen. If you configure your app to block by RPC version, it will appear and then quickly disappear. So while blocking with `minimumProtocolVersion` is preferable, `minimumRPCVersion` allows you more granular control over which RPCs will be present.


```java
builder.setMinimumProtocolVersion(new Version("3.0.0"));
builder.setMinimumRPCVersion(new Version("4.0.0"));
```

@![android]
### Lock Screen 
A lock screen is used to prevent the user from interacting with the app on the smartphone while they are driving. When the vehicle starts moving, the lock screen is activated. Similarly, when the vehicle stops moving, the lock screen is removed. You must implement a lock screen in your app for safety reasons. Any application without a lock screen will not get approval for release to the public.

The SDL SDK can take care of the lock screen implementation for you, automatically using your app logo and the connected vehicle logo. If you do not want to use the default lock screen, you can implement your own custom lock screen.

```java
LockScreenConfig lockScreenConfig = new LockScreenConfig();

builder.setLockScreenConfig(lockScreenConfig);
```

For more information, please refer to the [Adding the Lock Screen](Getting Started/Adding the Lock Screen) section, for this guide we will be using `SDLLockScreenConfiguration`'s basic `enabledConfiguration`.
!@

### SdlSecurity
Security Libary

```java
builder.setSdlSecurity()
```

### Text-To-Speech 
Set the Text-to-Speech name of application

```java
Vector<TTSChunk> ttschunk = new Vector<>();

builder.setTtsName(ttschunk);
```

### Voice Recognition synonyms
Voice Recognition synonyms can be usd to identify the application
```java
Vector<String> vrSynonyms = new Vector<>();
vrSynonyms.add("App Name");

builder.setVrSynonyms(vrSynonyms);
```


### File Manager Configuration
The file manager configuration allows you to configure retry behavior for uploading files and images. The default configuration attempts one re-upload, but will fail after that.

```java
FileManagerConfig fileManagerConfig = new FileManagerConfig();
fileManagerConfig.setArtworkRetryCount(2);
fileManagerConfig.setFileRetryCount(2);

builder.setFileManagerConfig(fileManagerConfig);
```

### Language
The desired language to be used on display/HMI of connected module can be set.
```java
builder.setLanguage(Language.EN_US)
```


@![android,javaSE,javaEE]
### Listening for RPC notifications and events

We can listen for specific events using `SdlManager`'s builder `setRPCNotificationListeners`. The following example shows how to listen for HMI Status notifications. Additional listeners can be added for specific RPCs by using their corresponding `FunctionID` in place of the `ON_HMI_STATUS` in the following example and casting the `RPCNotification` object to the correct type.

##### Example of a listener for HMI Status:

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

@![android]
## SmartDeviceLink Router Service
The `SdlRouterService` will listen for a connection with an SDL enabled module. When a connection happens, it will alert all SDL enabled apps that a connection has been established and they should start their SDL services.

We must implement a local copy of the `SdlRouterService` into our project. The class doesn't need any modification, it's just important that we include it. We will extend the `com.smartdevicelink.transport.SdlRouterService` in our class named `SdlRouterService`:

!!! NOTE
Do not include an import for `com.smartdevicelink.transport.SdlRouterService`. Otherwise, we will get an error for `'SdlRouterService' is already defined in this compilation unit`.
!!!

```Java
public class SdlRouterService extends  com.smartdevicelink.transport.SdlRouterService {
    //Nothing to do here
}
```
!!! MUST
The local extension of the `com.smartdevicelink.transport.SdlRouterService` must be named `SdlRouterService`.
!!!

!!! MUST
Make sure this local class `SdlRouterService.java` is in the same package of `SdlReceiver.java` (described below)
!!!

If you created the service using the Android Studio template then the service should have been added to your `AndroidManifest.xml` otherwise the service needs to be added in the manifest. Because we want our service to be seen by other SDL enabled apps, we need to set `android:exported="true"`. The system may issue a lint warning because of this, so we can suppress that using `tools:ignore="ExportedService"`.

!!! MUST
The `SdlRouterService` must be placed in a separate process with the name `com.smartdevicelink.router`. If it is not in that process during its start up it will stop itself.
!!!

### Intent Filter
```xml
<intent-filter>
    <action android:name="com.smartdevicelink.router.service"/>
</intent-filter>
```

The new versions of the SDL Android library rely on the `com.smartdevicelink.router.service` action to query SDL enabled apps that host router services. This allows the library to determine which router service to start.

!!! MUST
This `intent-filter` MUST be included.
!!!

### Metadata

#### Router Service Version
```xml
<meta-data android:name="sdl_router_version"  android:value="@integer/sdl_router_service_version_value" />
```

Adding the `sdl_router_version` metadata allows the library to know the version of the router service that the app is using. This makes it simpler for the library to choose the newest router service when multiple router services are available.

#### Custom Router Service
```xml
<meta-data android:name="sdl_custom_router" android:value="false" />
```

!!! NOTE
This is only for specific OEM applications, therefore normal developers do not need to worry about this.
!!!

Some OEMs choose to implement custom router services. Setting the `sdl_custom_router` metadata value to `true` means that the app is using something custom over the default router service that is included in the SDL Android library. Do not include this `meta-data` entry unless you know what you are doing.


## SmartDeviceLink Broadcast Receiver
The Android implementation of the `SdlManager` relies heavily on the OS's bluetooth and USB intents. When the phone is connected to SDL and the router service has sent a connection intent, the app needs to create an `SdlManager`, which will bind to the already connected router service. As mentioned previously, the `SdlManager` cannot be re-used. When a disconnect between the app and SDL occurs, the current `SdlManager` must be disposed of and a new one created.

The SDL Android library has a custom broadcast receiver named `SdlBroadcastReceiver` that should be used as the base for your `BroadcastReceiver`. It is a child class of Android's `BroadcastReceiver` so all normal flow and attributes will be available. Two abstract methods will be automatically populate the class, we will fill them out soon.

Create a new `SdlBroadcastReceiver` and name it appropriately, for this guide we are going to call it `SdlReceiver`:

```java
public class SdlReceiver extends SdlBroadcastReceiver {

    @Override
	public void onSdlEnabled(Context context, Intent intent) {
		//...

	}

	@Override
	public Class<? extends SdlRouterService> defineLocalSdlRouterClass() {
        //...
	}
}
```

!!! MUST
SdlBroadcastReceiver must call super if ```onReceive``` is overridden
!!!

``` java
	@Override
	public void onReceive(Context context, Intent intent) {
		super.onReceive(context, intent);
		//your code here
	}
```

If you created the `BroadcastReceiver` using the Android Studio template then the service should have been added to your `AndroidManifest.xml` otherwise the receiver needs to be defined in the manifest. Regardless, the manifest needs to be edited so that the `SdlBroadcastReceiver` needs to respond to the following intents:

* [android.bluetooth.device.action.ACL_CONNECTED](https://developer.android.com/reference/android/bluetooth/BluetoothDevice.html#ACTION_ACL_CONNECTED)
* sdl.router.startservice

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.company.mySdlApplication">

    <application>

        <receiver
            android:name=".SdlReceiver"
            android:exported="true"
            android:enabled="true">

            <intent-filter>
                <action android:name="android.bluetooth.device.action.ACL_CONNECTED" />
                <action android:name="sdl.router.startservice" />
            </intent-filter>

        </receiver>

    </application>

</manifest>
```
!!! NOTE
The intent `sdl.router.startservice` is a custom intent that will come from the `SdlRouterService` to tell us that we have just connected to an SDL enabled piece of hardware.
!!!

!!! MUST
SdlBroadcastReceiver has to be exported, or it will not work correctly
!!!

Next, we want to make sure we supply our instance of the `SdlBroadcastService` with our local copy of the `SdlRouterService`. We do this by simply returning the class object in the method `defineLocalSdlRouterClass`:

```java
public class SdlReceiver extends SdlBroadcastReceiver {
   @Override
	public void onSdlEnabled(Context context, Intent intent) {

	}

	@Override
	public Class<? extends SdlRouterService> defineLocalSdlRouterClass() {
        //Return a local copy of the SdlRouterService located in your project
		return com.company.mySdlApplication.SdlRouterService.class;
	}
}
```

We want to start the `SdlManager` when an SDL connection is made via the `SdlRouterService`. We do this by taking action in the `onSdlEnabled` method:

!!! MUST
Apps must start their service in the foreground as of Android Oreo (API 26).
!!!

```java
public class SdlReceiver extends SdlBroadcastReceiver {

   @Override
	public void onSdlEnabled(Context context, Intent intent) {
		//Use the provided intent but set the class to the SdlService
		intent.setClass(context, SdlService.class);
		if(Build.VERSION.SDK_INT < Build.VERSION_CODES.O) {
			context.startService(intent);
		}else{
			context.startForegroundService(intent);
		}
	}

	@Override
	public Class<? extends SdlRouterService> defineLocalSdlRouterClass() {
        //Return a local copy of the SdlRouterService located in your project
		return com.company.mySdlApplication.SdlRouterService.class;
	}
}
```
!!! IMPORTANT
The `onSdlEnabled` method will be the main start point for our SDL connection session. We define exactly what we want to happen when we find out we are connected to SDL enabled hardware.
!!!

### Main Activity
Now that the basic connection infrastructure is in place, we should add methods to start the `SdlService` when our application starts. In `onCreate()` in your main activity, you need to call a method that will check to see if there is currently an SDL connection made. If there is one, the `onSdlEnabled` method will be called and we will follow the flow we already set up:

```java
public class MainActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        //If we are connected to a module we want to start our SdlService
		SdlReceiver.queryForConnectedService(this);
    }
}
```
!@

@![javaSE]
### Main Class
Now that the basic connection infrastructure is in place, we should add methods to start the `SdlService` when our application starts. In `main(String[] args)` in your main class, you will create and start an instance of the `SdlService` class.

You will also need to fill in what port the app should listen on for an incoming web socket connection.

```java
public class Main {

	Thread thread;
	SdlService sdlService;

    public static void main(String[] args) {
        Main main = new Main();
        main.startSdlService();

    }

private void startSdlService() {

        thread = new Thread(new Runnable() {

            @Override
            public void run() {
                sdlService  = new SdlService(new WebSocketServerConfig(PORT, -1));
                sdlService.start();


            }
        });
        thread.start();
    }
}
```
!@

@![javaEE]
### Adding EJB and Websockets
Create a new package where all the JavaEE-specific code will go.

The SDL Java library comes with a `CustomTransport` class which takes the role of sending messages between incoming sdl_core connections and your SDL application. You need to pass that class to the `SdlManager` builder to make the SDL Java library aware that you want to use your JavaEE websocket server as the transport.

Create a Java class in the new package which will be the `SDLSessionBean` class. This class utilizes the `CustomTransport` class and EJB JavaEE API which will make it the entry point of your app when a connection is made. It will open up a websocket server at `/` and create stateful beans, where the bean represents the logic of your cloud app. Every new connection to this endpoint creates a new bean containing your app logic, allowing for load balancing across all the instances of your app that were automatically created.

```java
import com.smartdevicelink.transport.CustomTransport;
import javax.ejb.Stateful;
import javax.websocket.*;
import javax.websocket.server.ServerEndpoint;
import java.io.IOException;
import java.nio.ByteBuffer;

@ServerEndpoint("/")
@Stateful(name = "SDLSessionEJB")
public class SDLSessionBean {

    CustomTransport websocket;

    public class WebSocketEE extends CustomTransport {
        Session session;
        public WebSocketEE(String address, Session session) {
            super(address);
            this.session = session;
        }
        public void onWrite(byte[] bytes, int i, int i1) {
            try {
                session.getBasicRemote().sendBinary(ByteBuffer.wrap(bytes));
            }
            catch (IOException e) {

            }
        }
    }

    @OnOpen
    public void onOpen (Session session) {
        websocket = new WebSocketEE("http://localhost", session) {};
        //TODO: pass your CustomTransport instance to your SDL app here
    }

    @OnMessage
    public void onMessage (ByteBuffer message, Session session) {
        websocket.onByteBufferReceived(message); //received message from core
    }
}
```

Unfortunately, [there's no way to get a client's IP address using the standard API](https://stackoverflow.com/a/23025059), so localhost is passed to the `CustomTransport` for now as the transport address (this is only used locally in the library so it is not necessary).

The `SDLSessionBean` class’s `@OnOpen` method is where you will start your app, and should call your entry of your application and invoke whatever is needed to start it. You need to pass the instantiated `CustomTransport` object to your application so that the connection can be passed into the `SdlManager`.

The `SdlManager` will need you to create a `CustomTransportConfig`, pass in the `CustomTransport` instance from the `SDLSessionBean` instance, then set the `SdlManager` Builder’s transport type to that config. This will set your transport type into `CUSTOM` mode and will use your `CustomTransport` instance to handle the read and write operations.

```java
// Set transport config. builder is a SdlManager.Builder
CustomTransportConfig transport = new CustomTransportConfig(websocket);
builder.setTransportType(transport);
```

!!! IMPORTANT
The `SDLSessionBean` should be inside a Java package other than the default package in order for it to work properly.
!!!

##### Add a New Artifact:

* Right-click project -> Open Module Settings -> Artifacts -> + ->
  Web Application: Archive -> for your war: exploded artifact which should already exist
* Create Manifest. Apply + OK.
* Run Build -> Build Artifacts to get a .war file in the /out folder.
!@
