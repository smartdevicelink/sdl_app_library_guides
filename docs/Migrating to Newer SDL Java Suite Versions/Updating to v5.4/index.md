# Upgrading to 5.4

## Overview

This guide is to help developers get set up with the SDL Java Suite library version 5.4.0. It is assumed that the developer is already updated to at least version 5.3.1 of the library.

The full release notes are published [here](https://github.com/smartdevicelink/sdl_java_suite/releases).

SDL Java Suite library version 5.4.0 adds support for Android 12.

## AndroidManifest Exported Flag
Starting in Android 12, any activities, services, or broadcast receivers that use intent filters will need to explicitly declare the `android:exported` attribute for the given app components. The SdlRouterService and SdlReceiver should already have the exported attribute defined and set to true, but the USBAccessoryAttachmentActivity will now also require this attribute to be set. Any activity that had an intent-filter would have a default exported value of true. Now we need to explicitly set it.

```xml
<activity
    android:name="com.smartdevicelink.transport.USBAccessoryAttachmentActivity"
    android:exported="true" <!--New Addition-->
    android:launchMode="singleTop">
    <intent-filter>
        <action android:name="android.hardware.usb.action.USB_ACCESSORY_ATTACHED" />
    </intent-filter>
    <!-- ... -->
</activity>
```

## Bluetooth Runtime Permissions
Starting in Android 12, for the library to be able to connect to the HMI over Bluetooth, app developers will need to request the new `BLUETOOTH_CONNECT` runtime permission.

This means the permission will need to be listed in the `AndroidManifest.xml` file.

```xml
<uses-permission android:name="android.permission.BLUETOOTH_CONNECT"
    tools:targetApi="31"/>
```

The developer will also need to request this permission from the user as it is a runtime permission.

```java
//MainActivity.java

//.....

private static final int REQUEST_CODE = 200;


@Override
protected void onCreate(Bundle savedInstanceState) {

    //......

    if (android.os.Build.VERSION.SDK_INT >= Build.VERSION_CODES.S && !checkPermission()) {
        requestPermission();
        return;
    }

    //We are either not targeting Android 12+ or permissions are granted so we can try to start out SdlService
    SdlReceiver.queryForConnectedService(this);

    //.....

}

private boolean checkPermission() {
    return PackageManager.PERMISSION_GRANTED == ContextCompat.checkSelfPermission(getApplicationContext(), BLUETOOTH_CONNECT);
}

private void requestPermission() {
    ActivityCompat.requestPermissions(this, new String[]{BLUETOOTH_CONNECT}, REQUEST_CODE);
}

@Override
public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
    switch (requestCode) {
        case REQUEST_CODE:
            if (grantResults.length > 0) {
                boolean btConnectGranted = grantResults[0] == PackageManager.PERMISSION_GRANTED;

                if (btConnectGranted) {
                    //Bluetooth permissions have been granted by the user so we can try to start out SdlService.
                    SdlReceiver.queryForConnectedService(this);
                }
            }
            break;
    }
}

//.....

```

## Starting Services from the Foreground

Starting with Android 12, apps cannot start services from the background. In order to allow the library to work as intended, the library will now need to start the "SdlService" from the context of the active router service.

To achieve this there are a few changes that will be required in your application.

First to allow your "SdlService" to be started from an external source (the active router service may belong to another app), you will need to export the service in your `AndroidManifest.xml`.

```xml
<service
    android:name="com.sdl.hellosdlandroid.SdlService"
    android:exported="true" <!--New Addition-->
    android:foregroundServiceType="connectedDevice">
</service>
```

Second in the `SdlReceiver.onSdlEnabled()` method the received intent will now have a PendingIntent extra when your service should be started.

You will need to get the PendingIntent extra and send the PendingIntent with the intent of the service that you intend to start.

The PendingIntent will start the service from the context of the active router service (which is running in the foreground).

```java
//Retrieve, Update, and Send the PendingIntent
@Override
public void onSdlEnabled(Context context, Intent intent) {
    DebugTool.logInfo(TAG, "SDL Enabled");
    intent.setClass(context, SdlService.class);

    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.S) {
        if (intent.getParcelableExtra(TransportConstants.PENDING_INTENT_EXTRA) != null) {
            PendingIntent pendingIntent = (PendingIntent) intent.getParcelableExtra(TransportConstants.PENDING_INTENT_EXTRA);
            try {
                //Here we are allowing the RouterService that is in the Foreground to start the SdlService on our behalf
                pendingIntent.send(context, 0, intent);
            } catch (PendingIntent.CanceledException e) {
                e.printStackTrace();
            }
        }
    } else {
        // SdlService needs to be foregrounded in Android O and above
        // This will prevent apps in the background from crashing when they try to start SdlService
        // Because Android O doesn't allow background apps to start background services
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            context.startForegroundService(intent);
        } else {
            context.startService(intent);
        }
    }
}
```
#### Alternative Method to Avoid Exporting the SdlService
If you do not wish to export your "SdlService" class then the library will not be able to start the service and there is no way to start the service from the background.

However you can start your own "SdlService" while your app is in a foreground context. To achieve this you will need a way to track if your app is in the foreground. While your app is in the foreground you can start your "SdlService" as you normally would.

```java
//MainActivity, Application, or where appropriate

//...
private androidx.lifecycle.LifecycleObserver lifecycleObserver;

@Override
protected void onCreate(Bundle savedInstanceState) {
    try {
        lifecycleObserver = new androidx.lifecycle.LifecycleObserver() {
            @androidx.lifecycle.OnLifecycleEvent(androidx.lifecycle.Lifecycle.Event.ON_START)
            public void onMoveToForeground() {
                SdlReceiver.setIsForeground(true);
            }

            @androidx.lifecycle.OnLifecycleEvent(androidx.lifecycle.Lifecycle.Event.ON_STOP)
            public void onMoveToBackground() {
                SdlReceiver.setIsForeground(false);
            }
        };

        if (androidx.lifecycle.ProcessLifecycleOwner.get() != null) {
            androidx.lifecycle.ProcessLifecycleOwner.get().getLifecycle().addObserver(lifecycleObserver);
        }
    } catch (Exception e) {
        e.printStackTrace();
    }
}

@Override
protected void onDestroy() {
    super.onDestroy();
    try {
        if (androidx.lifecycle.ProcessLifecycleOwner.get() != null && lifecycleObserver != null) {
            androidx.lifecycle.ProcessLifecycleOwner.get().getLifecycle().removeObserver(lifecycleObserver);
        }
    } catch (Exception e) {
        e.printStackTrace();
    }

    lifecycleObserver = null;
}
//...
```

```java
//SdlReceiver.java

//...
private static boolean isForeground;

public static void setIsForeground(boolean status) {
    isForeground = status;
}

@Override
    public void onSdlEnabled(Context context, Intent intent) {
        DebugTool.logInfo(TAG, "SDL Enabled");
        intent.setClass(context, SdlService.class);

        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.S) {
            if (isForeground) {
                context.startForegroundService(intent);
            } else {
                if (intent.getParcelableExtra(TransportConstants.PENDING_INTENT_EXTRA) != null) {
                    PendingIntent pendingIntent = (PendingIntent) intent.getParcelableExtra(TransportConstants.PENDING_INTENT_EXTRA);
                    try {
                        pendingIntent.send(context, 0, intent);
                    } catch (PendingIntent.CanceledException e) {
                        e.printStackTrace();
                    }
                }
            }
        } else {
            // SdlService needs to be foregrounded in Android O and above
            // This will prevent apps in the background from crashing when they try to start SdlService
            // Because Android O doesn't allow background apps to start background services
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
                context.startForegroundService(intent);
            } else {
                context.startService(intent);
            }
        }
    }
//...
```

## SdlService Class Name

There is now an overridable method, `getSdlServiceName` in the `SdlBroadcastReceiver` class. This method is used by the `SdlBroadcastReceiver` to catch a possible foreground exception.

When the app tries to start the `SdlService`, if the service does not enter the foreground within a set amount of time (this time is designated by the Android operating system), an exception will be thrown and the app may encounter an ANR.

The `SdlBroadcasterReceiver` can catch this exception and prevent the ANR but will need to know the name of the class that throws the exception.

By default the `getSdlServiceName` method will return "SdlService". If your app uses a name other than "SdlService" you will need to override `getSdlServiceName` in the `SdlReceiver` class to return the correct name.


```java
//...
@Override
public String getSdlServiceName() {
    return SDL_SERVICE_CLASS_NAME;
}
//...
```

## Known Corner Case

When the user connects their device over USB and the user has not been granted Bluetooth Permissions the user will be presented with a notification which will help navigate the user to grant Bluetooth Permissions for the app.

Once the permissions are granted the Router Service will open the Bluetooth connection.

If the user then revokes these permissions, the Android operating system will kill the application running the Router Service and the Router Service process, and none of the service's callbacks will be called. Even though the Router Service has been killed, the HMI will still show the previously connected apps.

Unplugging the USB cable will remove the apps from the HMI.
