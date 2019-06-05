## Getting Started on Android

`// TODO: Align with iOS Integration Basics more + Move configuration content to SDK Configuration guide`
In this guide, we exclusively use Android Studio. We are going to set-up a bare-bones application so you get started using SDL.

!!! IMPORTANT
The SDL Mobile library supports [Android 2.2.x (API Level 8)](https://developer.android.com/about/versions/android-2.2.html) or higher.
!!!
 
## Required System Permissions

In the AndroidManifest for our sample project we need to ensure we have the following system permissions: 

* [Internet](https://developer.android.com/reference/android/Manifest.permission.html#INTERNET) - Used by the mobile library to communicate with a SDL Server
* [Bluetooth](https://developer.android.com/reference/android/Manifest.permission.html#BLUETOOTH) - Primary transport for SDL communication between the device and the vehicle's head-unit
* [Access Network State](https://developer.android.com/reference/android/Manifest.permission.html#ACCESS_NETWORK_STATE) - Required to check if WiFi is enabled on the device

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.company.mySdlApplication">
    
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.BLUETOOTH"/>
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />

</manifest>
```

!!! IMPORTANT
If the app is targeting Android P (API Level 28) or higher, the Android Manifest file should also have the following permission to allow the app to start a foreground service:

```xml
<uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
```
!!!

## SmartDeviceLink Service

A SmartDeviceLink Android Service should be created to manage the lifecycle of the SDL session. The `SdlService` should build and start an instance of the `SdlManager` which will automatically connect with a headunit when available. This `SdlManager` will handle sending and receiving messages to and from SDL after connected.

Create a new service and name it appropriately, for this guide we are going to call it `SdlService`. 
 
```java
public class SdlService extends Service {
    //...
}
```
 
If you created the service using the Android Studio template then the service should have been added to your `AndroidManifest.xml` otherwise the service needs to be defined in the manifest:

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

It's important that you don't leave you notification in the notification tray as it is very confusing to users. So in the `onDestroy` method in your service, simply call the `stopForeground` method.

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

### Implementing SDL Manager

In order to correctly connect to an SDL enabled head unit developers need to implement methods for the proper creation and disposing of an `SdlManager` in our `SdlService`.

!!! NOTE
An instance of SdlManager cannot be reused after it is closed and properly disposed of. Instead, a new instance must be created. Only one instance of SdlManager should be in use at any given time.
!!!

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
            SdlManager.Builder builder = new SdlManager.Builder(this, APP_ID, APP_NAME, listener);
            builder.setAppTypes(appType);
            builder.setTransportType(transport);
            builder.setAppIcon(appIcon);
            sdlManager = builder.build();
            sdlManager.start();
        }

}
```

The `onDestroy()` method from the `SdlManagerListener` is called whenever the manager detects some disconnect in the connection, whether initiated by the app, by SDL, or by the device’s connection.

!!! IMPORTANT
The `sdlManager` must be shutdown properly in the `SdlService.onDestroy()` callback using the method `sdlManager.dispose()`.
!!!

### Determining SDL Support
You have the ability to determine a minimum SDL protocol and a minimum SDL RPC version that your app supports. We recommend not setting these values until your app is ready for production. The OEMs you support will help you configure the correct `minimumProtocolVersion` and `minimumRPCVersion` during the application review process.

If a head unit is blocked by protocol version, your app icon will never appear on the head unit's screen. If you configure your app to block by RPC version, it will appear and then quickly disappear. So while blocking with `minimumProtocolVersion` is preferable, `minimumRPCVersion` allows you more granular control over which RPCs will be present.


```java
builder.setMinimumProtocolVersion(new Version("3.0.0"));
builder.setMinimumRPCVersion(new Version("4.0.0"));

```

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
The `SdlRouterService` must be placed in a separate process with the name `com.smartdevicelink.router`. If it is not in that process during it's start up it will stop itself.
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

The Android implementation of the SdlManager relies heavily on the OS's bluetooth and USB intents. When the phone is connected to SDL and the router service has sent a connection intent, the app needs to create an SdlManager, which will bind to the already connected router service. As mentioned previously, the SdlManager cannot be re-used. When a disconnect between the app and SDL occurs, the current SdlManager must be disposed of and a new one created.

The SDL Android library has a custom broadcast receiver named `SdlBroadcastReceiver` that should be used as the base for your BroadcastReceiver. It is a child class of Android's BroadcastReceiver so all normal flow and attributes will be available. Two abstract methods will be automatically populate the class, we will fill them out soon.

Create a new SdlBroadcastReceiver and name it appropriately, for this guide we are going to call it `SdlReceiver`:

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

If you created the BroadcastReceiver using the Android Studio template then the service should have been added to your `AndroidManifest.xml` otherwise the receiver needs to be defined in the manifest. Regardless, the manifest needs to be edited so that the `SdlBroadcastReceiver` needs to respond to the following intents:

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
    
    <!-- Required to use the lock screen -->
    <activity android:name="com.smartdevicelink.managers.lockscreen.SDLLockScreenActivity"
                  android:launchMode="singleTop"/>

</manifest>
```
!!! NOTE
The intent `sdl.router.startservice` is a custom intent that will come from the SdlRouterService to tell us that we have just connected to an SDL enabled piece of hardware.
!!!

!!! MUST
SdlBroadcastReceiver has to be exported, or it will not work correctly
!!!

Next, we want to make sure we supply our instance of the SdlBroadcastService with our local copy of the SdlRouterService. We do this by simply returning the class object in the method defineLocalSdlRouterClass:

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


We want to start the SdlManager when an SDL connection is made via the `SdlRouterService`. We do this by taking action in the onSdlEnabled method:

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

## Lock Screen Activity

An Activity entry must also be added to the manifest for the SDL lock
screen. For more information about lock screens, please see the [Adding the Lock Screen](Getting Started With Android/Adding the Lock Screen) section.


!!! NOTE
When using `SdlManager`, the lock screen is enabled by default via the `LockScreenManager`. Please see the link above for more information
!!!

Once added, your `AndroidManifest.xml` should be defined like below:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.company.mySdlApplication">

    <application>

        <service
        	android:name="com.company.mySdlApplication.SdlRouterService"
        	android:exported="true" 
        	android:process="com.smartdevicelink.router"
            tools:ignore="ExportedService">
            <intent-filter>
                <action android:name="com.smartdevicelink.router.service"/>
            </intent-filter>
            <meta-data android:name="sdl_router_version"  android:value="@integer/sdl_router_service_version_value" />
        </service>
        
        <!-- Required to use the lock screen -->
        <activity android:name="com.smartdevicelink.managers.lockscreen.SDLLockScreenActivity"
                  android:launchMode="singleTop"/>
    
    </application>

</manifest>
```


### Main Activity

Now that the basic connection infrastructure is in place, we should add methods to start the SdlService when our application starts. In `onCreate()` in your main activity, you need to call a method that will check to see if there is currently an SDL connection made. If there is one, the onSdlEnabled method will be called and we will follow the flow we already set up. In our `MainActivity.java` we need to check for an SDL connection:

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