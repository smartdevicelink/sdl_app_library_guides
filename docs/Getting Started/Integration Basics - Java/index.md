# Integration Basics
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


## SmartDeviceLink Service
A SmartDeviceLink Service should be created to manage the lifecycle of the SDL session. The `SdlService` should build and start an instance of the `SdlManager` which will automatically connect with a head unit when available. This `SdlManager` will handle sending and receiving messages to and from SDL after it is connected.

@![android]
!!! NOTE
Please be aware that using an Activity to host the SDL implementation will not work. Android 10 has [restrictions](https://developer.android.com/guide/components/activities/background-starts) on starting activities from the background and that is how the SDL library will start the supplied component. SDL apps should only use a foreground service to host the SDL implementation.
!!!
!@

Create a new service and name it appropriately, for this guide we are going to call it `SdlService`.

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

!!! NOTE
Android API 29 adds a new attribute [foregroundServiceType](https://developer.android.com/reference/android/R.attr#foregroundServiceType) to specify the type of foreground service.
Starting with Android API 29 please include `android:foregroundServiceType='connectedDevice'` to the service tag for SdlService in your AndroidManifest.xml
!!!

### Entering the Foreground
Because of Android Oreo's requirements, it is mandatory that services enter the foreground for long running tasks. The first bit of integration is ensuring that happens in the `onCreate` method of the `SdlService` or similar. Within the service that implements the SDL lifecycle you will need to add a call to start the service in the foreground. This will include creating a notification to sit in the status bar tray. This information and icons should be relevant for what the service is doing/going to do. If you already start your service in the foreground, you can ignore this section.

```java
@Override
public void onCreate() {
    super.onCreate();
    //...
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
        NotificationChannel channel = new NotificationChannel("channelId", "channelName", NotificationManager.IMPORTANCE_DEFAULT);
        NotificationManager notificationManager = (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
        if (notificationManager != null) {
            notificationManager.createNotificationChannel(channel);
            Notification serviceNotification = new Notification.Builder(this, channel.getId())
                    .setContentTitle(...)
                    .setSmallIcon(...)
                    .setContentText(...)
                    .setChannelId(channel.getId())
                    .build();
            startForeground(FOREGROUND_SERVICE_ID, serviceNotification);
        }
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

### Implementing SDL Manager
In order to correctly connect to an SDL enabled head unit developers need to implement methods for the proper creation and disposing of an `SdlManager` in our `SdlService`.

!!! NOTE
An instance of SdlManager cannot be reused after it is closed and properly disposed of. Instead, a new instance must be created. Only one instance of SdlManager should be in use at any given time.
!!!

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
    
                @Override
                public LifecycleConfigurationUpdate managerShouldUpdateLifecycle(Language language, Language hmiLanguage) {
                    return null;
                }
            };
    
            // Create App Icon, this is set in the SdlManager builder
            SdlArtwork appIcon = new SdlArtwork(ICON_FILENAME, FileType.GRAPHIC_PNG, R.mipmap.ic_launcher, true);
    
            // The manager builder sets options for your session
            SdlManager.Builder builder = new SdlManager.Builder(this, APP_ID, APP_NAME, listener);
            builder.setAppTypes(appType);
            builder.setTransportType(transport);
            builder.setAppIcon(appIcon);
            sdlManager = builder.build();
            sdlManager.start();
        }
    
        return START_STICKY;
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

                @Override
                public LifecycleConfigurationUpdate managerShouldUpdateLifecycle(Language language, Language hmiLanguage) {
                  return null;
                }
            };

            // Create App Icon, this is set in the SdlManager builder
            SdlArtwork appIcon = new SdlArtwork(ICON_FILENAME, FileType.GRAPHIC_PNG, ICON_PATH, true);

            // The manager builder sets options for your session
            SdlManager.Builder builder = new SdlManager.Builder(APP_ID, APP_NAME, listener);
            builder.setAppTypes(appType);
            builder.setTransportType(transport);
            builder.setAppIcon(appIcon);
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

#### Optional SdlManager Builder Parameters

##### App Icon
This is a custom icon for your application. Please refer to [Adaptive Interface Capabilities](Displaying a User Interface/Adaptive Interface Capabilities) for icon sizes.

```java
builder.setAppIcon(appIcon);
```

##### App Type 
The app type is used by car manufacturers to decide how to categorize your app. Each car manufacturer has a different categorization system. For example, if you set your app type as media, your app will also show up in the audio tab as well as the apps tab of Ford’s SYNC® 3 head unit. The app type options are: default, communication, media (i.e. music/podcasts/radio), messaging, navigation, projection, information, and social.

```java
Vector<AppHMIType> appHMITypes = new Vector<>();
appHMITypes.add(AppHMIType.MEDIA);

builder.setAppTypes(appHMITypes);
```

@![android]
!!! NOTE
Navigation and projection applications both use video and audio byte streaming. However, navigation apps require special permissions from OEMs, and projection apps are only for internal use by OEMs.
!!!
!@

##### Short App Name 
This is a shortened version of your app name that is substituted when the full app name will not be visible due to character count constraints. You will want to make this as short as possible.

```java
builder.setShortAppName(shortAppName);
```

##### Template Coloring
You can customize the color scheme of your initial template on head units that support this feature using the `builder`. For more information, see the [Customizing the Template guide](Customizing Look and Functionality/Customizing the Template) section.

##### Determining SDL Support
You have the ability to determine a minimum SDL protocol and a minimum SDL RPC version that your app supports. We recommend not setting these values until your app is ready for production. The OEMs you support will help you configure the correct `minimumProtocolVersion` and `minimumRPCVersion` during the application review process.

If a head unit is blocked by protocol version, your app icon will never appear on the head unit's screen. If you configure your app to block by RPC version, it will appear and then quickly disappear. So while blocking with `minimumProtocolVersion` is preferable, `minimumRPCVersion` allows you more granular control over which RPCs will be present.


```java
builder.setMinimumProtocolVersion(new Version("3.0.0"));
builder.setMinimumRPCVersion(new Version("4.0.0"));
```

@![android]
##### Lock Screen Configuration
A lock screen is used to prevent the user from interacting with the app on the smartphone while they are driving. When the vehicle starts moving, the lock screen is activated. Similarly, when the vehicle stops moving, the lock screen is removed. You must implement a lock screen in your app for safety reasons. Any application without a lock screen will not get approval for release to the public.

The SDL SDK can take care of the lock screen implementation for you, automatically using your app logo and the connected vehicle logo. If you do not want to use the default lock screen, you can implement your own custom lock screen.

```java
LockScreenConfig lockScreenConfig = new LockScreenConfig();
builder.setLockScreenConfig(lockScreenConfig);
```

You should also declare the `SDLLockScreenActivity` in your manifest. For more information, please refer to the [Adding the Lock Screen](Getting Started/Adding the Lock Screen) section.
!@

##### SdlSecurity
Some OEMs may want to encrypt messages passed between your SDL app and the head unit. If this is the case, when you submit your app to the OEM for review, they will ask you to add a security library to your SDL app. See the [Encryption](Other SDL Features/Encryption) section.

##### File Manager Configuration
The file manager configuration allows you to configure retry behavior for uploading files and images. The default configuration attempts one re-upload, but will fail after that.

```java
FileManagerConfig fileManagerConfig = new FileManagerConfig();
fileManagerConfig.setArtworkRetryCount(2);
fileManagerConfig.setFileRetryCount(2);

builder.setFileManagerConfig(fileManagerConfig);
```

##### Language
The desired language to be used on display/HMI of connected module can be set.

```java
builder.setLanguage(Language.EN_US);
```

##### Listening for RPC notifications and events
You can listen for specific events using `SdlManager`'s builder `setRPCNotificationListeners`. The following example shows how to listen for HMI Status notifications. Additional listeners can be added for specific RPCs by using their corresponding `FunctionID` in place of the `ON_HMI_STATUS` in the following example and casting the `RPCNotification` object to the correct type.

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

##### Hash Resumption
Set a `hashID` for your application that can be used over connection cycles (i.e. loss of connection, ignition cycles, etc.).

```java
builder.setResumeHash(hashID);
```

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

!!! NOTE
Android API 29 adds a new attribute [foregroundServiceType](https://developer.android.com/reference/android/R.attr#foregroundServiceType) to specify the type of foreground service.
Starting with Android API 29 please include `android:foregroundServiceType='connectedDevice'` to the service tag for SdlRouterService in your AndroidManifest.xml
!!!

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

The final router service entry in the `AndroidManifest.xml` file should look like the following:
```xml
<service
    android:name=".SdlRouterService"
    android:enabled="true"
    android:exported="true"
    android:foregroundServiceType="connectedDevice"
    android:process="com.smartdevicelink.router">

    <intent-filter>
        <action android:name="com.smartdevicelink.router.service" />
    </intent-filter>

    <meta-data
        android:name="sdl_router_version"
        android:value="@integer/sdl_router_service_version_value" />
</service>
```


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