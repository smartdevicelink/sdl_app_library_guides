
# Updating to 4.4 (Upgrading To Multiplexing)


This guide is to help developers get setup with the SDL Android library 4.4. Upgrading apps to utilize the multiplexing transport flow will require us to do a few steps. This guide will assume the SDL library is already integrated into the app.

We will make changes to:

* SdlService
* SdlRouterService <b>(new)</b>
* SdlBroadcastReceiver
* MainActivity




## SmartDeviceLink Service

The SmartDeviceLink proxy object instantiation needs to change to the new constructor. We also need to check for a boolean extra supplied through the intent that started the service.

The old instantiation should look similar to this:

```java
 proxy = new SdlProxyALM(this, APP_NAME, true, APP_ID);

```
 The new constructor should look like this

```java
public class SdlService extends Service implements IProxyListenerALM {
   
   //...

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        boolean forceConnect = intent !=null && intent.getBooleanExtra(TransportConstants.FORCE_TRANSPORT_CONNECTED, false);
        if (proxy == null) {
            try {
                //Create a new proxy using Bluetooth transport
                //The listener, app name, 
                //whether or not it is a media app and the applicationId are supplied.
                proxy = new SdlProxyALM(this.getBaseContext(),this, APP_NAME, true, APP_ID);
            } catch (SdlException e) {
                //There was an error creating the proxy
                if (proxy == null) {
                //Stop the SdlService
                    stopSelf();
                }
            }
        }else if(forceConnect){
			proxy.forceOnConnected();
		}

        //use START_STICKY because we want the SDLService to be explicitly started and stopped as needed.
        return START_STICKY;
    }
```

Notice we now gather the extra boolean from the intent and add to our if-else statement. If the proxy is not null, we need to check if the supplied boolean extra is true and if so, take action.

```java
    if (proxy == null) {
       //...         
    }else if(forceConnect){
		proxy.forceOnConnected();
	}
```

## SmartDeviceLink Router Service (New)

The SdlRouterService will listen for a bluetooth connection with an SDL enabled module. When a connection happens, it will alert all SDL enabled apps that a connection has been established and they should start their SDL services.

We must implement a local copy of the SdlRouterService into our project. The class doesn't need any modification, it's just important that we include it. We will extend the `com.smartdevicelink.transport.SdlRouterService` in our class named `SdlRouterService`:

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
Make sure this local class (SdlRouterService.java) is in the same package of SdlReceiver.java (described below)
!!!

If you created the service using the Android Studio template then the service should have been added to your `AndroidManifest.xml` otherwise the service needs to be added in the manifest. Because we want our service to be seen by other SDL enabled apps, we need to set `android:exported="true"`. The system may issue a lint warning because of this, so we can suppress that using `tools:ignore="ExportedService"`.  Once added, it should be defined like below:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.company.mySdlApplication">

    <application>
    
    ...

        <service
        	android:name="com.company.mySdlApplication.SdlRouterService"
        	android:exported="true" 
        	android:process="com.smartdevicelink.router"
        	tools:ignore="ExportedService">
        </service>
    
    </application>

    ...

</manifest>
```

!!! MUST
The `SdlRouterService` must be placed in a separate process with the name `com.smartdevicelink.router`. If it is not in that process during it's start up it will stop itself.
!!!

## SmartDeviceLink Broadcast Receiver

The SmartDeviceLink Android Library now includes a base BroadcastReceiver that needs to be used. It's called `SdlBroadcastReceiver`. Our old BroadcastReceiver will just need to extend this class instead of the Android BroadcastReceiver.  Two abstract methods will be automatically populate the class, we will fill them out soon.


```java
public class SdlReceiver extends SdlBroadcastReceiver {

      	@Override
	public void onSdlEnabled(Context context, Intent intent) {...}

	@Override
	public Class<? extends SdlRouterService> defineLocalSdlRouterClass() {...}
	
}

``` 

Next, we want to make sure we supply our instance of the SdlBroadcastService with our local copy of the SdlRouterService. We do this by simply returning the class object in the method defineLocalSdlRouterClass:

```java
	public Class<? extends SdlRouterService> defineLocalSdlRouterClass() {
        //Return a local copy of the SdlRouterService located in your project
		return com.company.mySdlApplication.SdlRouterService.class;
	}
```


We want to start the SDL Proxy when an SDL connection is made via the `SdlRouterService`. This is likely code included on the `onReceive` method call previously. We do this by taking action in the onSdlEnabled method:
!!! NOTE
The actual package definition for the SdlRouterService might be different. Just make sure to return your local copy and not the class object from the library itself.
!!!

```java
public class SdlReceiver extends SdlBroadcastReceiver {
   
   @Override
	public void onSdlEnabled(Context context, Intent intent) {
		//Use the provided intent but set the class to the SdlService
		intent.setClass(context, SdlService.class);
		context.startService(intent);
		
	}


	@Override
	public Class<? extends SdlRouterService> defineLocalSdlRouterClass() {
        //Return a local copy of the SdlRouterService located in your project. 
		return com.company.mySdlApplication.SdlRouterService.class;
	}
}

```
!!! IMPORTANT
The `onSdlEnabled` method will be the main start point for our SDL connection session. We define exactly what we want to happen when we find out we are connected to SDL enabled hardware.
!!!

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

Now we need to add two extra intent actions to or our intent filter for the SdlBroadcastReceiver:

* [android.bluetooth.adapter.action.STATE_CHANGED](https://developer.android.com/reference/android/bluetooth/BluetoothAdapter.html#ACTION_CONNECTION_STATE_CHANGED)
* <b>sdl.router.startservice</b>

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.company.mySdlApplication">

    <application>

        ...

        <receiver
            android:name=".SdlReceiver"
            android:exported="true"
            android:enabled="true">
    
            <intent-filter>
                <action android:name="android.bluetooth.device.action.ACL_CONNECTED" />
                <action android:name="android.bluetooth.device.action.ACL_DISCONNECTED"/>
                <action android:name="android.bluetooth.adapter.action.STATE_CHANGED"/>
                <action android:name="android.media.AUDIO_BECOMING_NOISY" />
                <action android:name="sdl.router.startservice" />
            </intent-filter>
    
        </receiver>
    
    </application>

...

</manifest>
```

!!! MUST
SdlBroadcastReceiver has to be exported, or it will not work correctly
!!!


### Main Activity

Our previous MainActivity class probably looked similar to this:

```java
public class MainActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // Start the SDLService
        Intent sdlServiceIntent = new Intent(this, SdlService.class);
        startService(sdlServiceIntent);
    }
}
```

However now instead of starting the service every time we launch the application we can do a query that will let us know if we are connected to SDL enabled hardware or not. If we are, the `onSdlEnabled` method in our SdlBroadcastReceiver will be called and the proper flow should start. We do this by removing the intent creation and startService call and instead replace them with a single call to `SdlReceiver.queryForConnectedService(Context)`.

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

