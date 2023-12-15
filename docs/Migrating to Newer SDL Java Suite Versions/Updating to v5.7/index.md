# Updating to 5.7

## Overview

This guide is to help developers get set up with the SDL Java Suite library version 5.7.0. It is assumed that the developer is already updated to at least version 5.6.0 of the library.

The full release notes are published [here](https://github.com/smartdevicelink/sdl_java_suite/releases).

SDL Java Suite library version 5.7.0 adds support for Android 14.

## FOREGROUND_SERVICE_CONNECTED_DEVICE Permissions

Starting in Android 14, it is required to specify a foreground service type of `connectedDevice` in your app's `AndroidManifest.xml` to be able to enter the foreground. 

```xml
<uses-permission android:name="android.permission.FOREGROUND_SERVICE_CONNECTED_DEVICE"
        tools:targetApi="34"/>
```
With a foreground service type of `connectedDevice`, your app must either have the `BLUETOOTH_CONNECT` permission or have been the app selected to receive the USB Intent. You must now check to see if your app has been granted either of these permissions by the user in the `SdlReceiver.onSdlEnabled` callback before you start your service to ensure your SdlService can enter the foreground. We added a helper method, `AndroidTools.hasForegroundServiceTypePermission`, to check for both permissions that can be used at that time.

```java
                if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.UPSIDE_DOWN_CAKE) {
                    if (!AndroidTools.hasForegroundServiceTypePermission(context)) {
                        DebugTool.logInfo(TAG, "Permission missing for ForegroundServiceType connected device." + context);
                        return;
                    }
                }
````

### Alternative way to satisfy `connectedDevice` requirement for foreground service

If your app does not have the `BLUETOOTH_CONNECT` permission, and was not selected to receive the USB accessory intent, it is still possible to request access to a connected device (USB) so that your app's SDL related services can start in the foreground. This can be accomplished during the the `onSdlEnabled` callback in your `SdlBroadcastReceiver` as follows:

```java
//SdlReceiver.java

//.....
    private PendingIntent pendingIntentToStartService;
    private Intent startSdlServiceIntent;

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
                    if (!AndroidTools.hasForegroundServiceTypePermission(context)) {
                        requestUsbAccessory(context);
                        startSdlServiceIntent = intent;
                        this.pendingIntentToStartService = pendingIntent;
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


    private final BroadcastReceiver usbPermissionReceiver = new BroadcastReceiver() {
        public void onReceive(Context context, Intent intent) {
            String action = intent.getAction();
            if (ACTION_USB_PERMISSION.equals(action) && context != null && startSdlServiceIntent != null && pendingIntentToStartService != null) {
                if (AndroidTools.hasForegroundServiceTypePermission(context)) {
                    try {
                        pendingIntentToStartService.send(context, 0, startSdlServiceIntent);
                        context.unregisterReceiver(this);
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
            }
        }
    };


    private void requestUsbAccessory(Context context) {
        UsbManager manager = (UsbManager) context.getSystemService(Context.USB_SERVICE);
        UsbAccessory[] accessoryList = manager.getAccessoryList();
        if (accessoryList == null || accessoryList.length == 0) {
            startSdlServiceIntent = null;
            pendingIntentToStartService = null;
            return;
        }
        PendingIntent mPermissionIntent = PendingIntent.getBroadcast(context, 0, new Intent(ACTION_USB_PERMISSION), PendingIntent.FLAG_IMMUTABLE);
        IntentFilter filter = new IntentFilter(ACTION_USB_PERMISSION);

        AndroidTools.registerReceiver(context, usbPermissionReceiver, filter,
                Context.RECEIVER_EXPORTED);

        for (final UsbAccessory usbAccessory : accessoryList) {
            manager.requestPermission(usbAccessory, mPermissionIntent);
        }
    }


//.....

```
 
