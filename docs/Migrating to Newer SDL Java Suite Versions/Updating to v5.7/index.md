# Updating to 5.7

## Overview

This guide is to help developers get set up with the SDL Java Suite library version 5.7.0. It is assumed that the developer is already updated to at least version 5.6.0 of the library.

The full release notes are published [here](https://github.com/smartdevicelink/sdl_java_suite/releases).

SDL Java Suite library version 5.7.0 adds support for Android 14.

## FOREGROUND_SERVICE_CONNECTED_DEVICE Permissions

Starting in Android 14, we are required to specify a foreground service type of `connectedDevice` in your app's `AndroidManifest.xml` to be able to enter the foreground. 

```xml
 <uses-permission android:name="android.permission.FOREGROUND_SERVICE_CONNECTED_DEVICE" />
```
With a foreground service type of `connectedDevice`, it requires your app to have the `BLUETOOTH_CONNECT` permission or have been the app selected to receive the USB intent.

To prevent SdlService from crashing and allow you to test via a TCP connection, we recommend adding a try catch statement when entering the foreground.

```java
//SdlService.java

//.....

    // Helper method to let the service enter foreground mode
    public void enterForeground() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            try {
                NotificationChannel channel = new NotificationChannel(BuildConfig.APP_ID, "SdlService", NotificationManager.IMPORTANCE_DEFAULT);
                NotificationManager notificationManager = (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
                if (notificationManager != null) {
                    notificationManager.createNotificationChannel(channel);
                    Notification.Builder builder = new Notification.Builder(this, channel.getId())
                            .setContentTitle("Connected through SDL")
                            .setSmallIcon(R.drawable.ic_sdl);
                    if (android.os.Build.VERSION.SDK_INT >= Build.VERSION_CODES.S) {
                        builder.setForegroundServiceBehavior(Notification.FOREGROUND_SERVICE_IMMEDIATE);
                    }
                    Notification serviceNotification = builder.build();
                    startForeground(FOREGROUND_SERVICE_ID, serviceNotification);
                }
            } catch (Exception e) {
                // This should only catch for TCP connections on Android 14+ due to needing
                // permissions for ForegroundServiceType ConnectedDevice that don't make sense for
                // a TCP connection
                DebugTool.logError(TAG, "Unable to start service in foreground", e);
            }
        }
    }

//.....

```

In your SdlReceiver onSdlEnable method before you start your service when on Android 14 you must check to make sure you have permission to enter the foreground before you can start. We added a helper method `AndroidTools.ServicePermissionUtil.hasForegroundServiceTypePermission` where you just need to pass the context to check if your app has the `BLUETOOTH_CONNECT` permission or has USB accessory permission so that your app can enter the foreground. You should at minimum check fore theses permission before you start your service:

```java
                if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.UPSIDE_DOWN_CAKE) {
                    if (!AndroidTools.ServicePermissionUtil.hasForegroundServiceTypePermission(context)) {
                        DebugTool.logInfo(TAG, "Permission missing for ForegroundServiceType connected device." + context);
                        return;
                    }
                }
````

If you want to request the USB accessory permission at this point and still be able to start your service after:
```java
//SdlReceiver.java

//.....
    private PendingIntent pendingIntent;
    private Context storedContext;
    private Intent storedIntent;

    @Override
    public void onSdlEnabled(Context context, Intent intent) {
        DebugTool.logInfo(TAG, "SDL Enabled");
        intent.setClass(context, SdlService.class);

        // Starting with Android S SdlService needs to be started from a foreground context.
        // We will check the intent for a pendingIntent parcelable extra
        // This pendingIntent allows us to start the SdlService from the context of the active router service which is in the foreground
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.S) {
            PendingIntent pendingIntent = (PendingIntent) intent.getParcelableExtra(TransportConstants.PENDING_INTENT_EXTRA);
            if (pendingIntent != null) {
                if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.UPSIDE_DOWN_CAKE) {
                    if (!AndroidTools.ServicePermissionUtil.hasForegroundServiceTypePermission(context)) {
                        requestUsbAccessory(context);
                        storedIntent = intent;
                        storedContext = context;
                        this.pendingIntent = pendingIntent;
                        DebugTool.logInfo(TAG, "Permission missing for ForegroundServiceType connected device." + context);
                        return;
                    }
                }
                try {
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


    private void requestUsbAccessory(Context context) {
        UsbManager manager = (UsbManager) context.getSystemService(Context.USB_SERVICE);
        if (manager.getAccessoryList() == null) {
            return;
        }
        PendingIntent mPermissionIntent = PendingIntent.getBroadcast(context, 0, new Intent(ACTION_USB_PERMISSION), PendingIntent.FLAG_IMMUTABLE);
        IntentFilter filter = new IntentFilter(ACTION_USB_PERMISSION);
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.UPSIDE_DOWN_CAKE) {
            context.registerReceiver(mUsbReceiver, filter, Context.RECEIVER_EXPORTED);
        }
        for (final UsbAccessory usbAccessory : manager.getAccessoryList()) {
            manager.requestPermission(usbAccessory, mPermissionIntent);
        }
    }

    private final BroadcastReceiver mUsbReceiver = new BroadcastReceiver() {

        public void onReceive(Context context, Intent intent) {
            String action = intent.getAction();
            if (ACTION_USB_PERMISSION.equals(action) && storedContext != null && storedIntent != null && pendingIntent != null) {
                if (AndroidTools.ServicePermissionUtil.hasForegroundServiceTypePermission(storedContext)) {
                    try {
                        pendingIntent.send(storedContext, 0, storedIntent);
                    } catch (PendingIntent.CanceledException e) {
                        e.printStackTrace();
                    }
                }
            }
        }
    };

//.....

```
 
