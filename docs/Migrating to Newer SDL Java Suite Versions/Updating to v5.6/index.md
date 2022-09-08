# Upgrading to 5.6

## Overview

This guide is to help developers get set up with the SDL Java Suite library version 5.6.0. It is assumed that the developer is already updated to at least version 5.5.0 of the library.

The full release notes are published [here](https://github.com/smartdevicelink/sdl_java_suite/releases).

SDL Java Suite library version 5.6.0 adds support for Android 13.

## POST_NOTIFICATIONS Runtime Permissions
Starting in Android 13, for the library and app to display notifications, app developers will need to request the new `POST_NOTIFICATIONS` runtime permission.

This means the permission will need to be listed in the `AndroidManifest.xml` file.

```xml
<uses-permission android:name="android.permission.POST_NOTIFICATIONS"
        tools:targetApi="33"/>
```

The developer will also need to request this permission from the user as it is a runtime permission, given that apps need to request the BLUETOOTH_CONNECT permission with Andoid S API 31 and above, below is an example on how to request both at the same time.

```java
//MainActivity.java

//.....

private static final int REQUEST_CODE = 200;


@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    String[] permissionsNeeded = permissionsNeeded();
    if (permissionsNeeded.length > 0) {
        requestPermission(permissionsNeeded, REQUEST_CODE);
        for (String permission : permissionsNeeded) {
            if (Manifest.permission.BLUETOOTH_CONNECT.equals(permission)) {
                // We need to request BLUETOOTH_CONNECT permission to connect to SDL via Bluetooth
                return;
            }
        }
    }

    //If we are connected to a module we want to start our SdlService
    SdlReceiver.queryForConnectedService(this);

}

/**
* Boolean method that checks API level and check to see if we need to request BLUETOOTH_CONNECT permission
* @return false if we need to request BLUETOOTH_CONNECT permission
*/
private boolean hasBTPermission() {
    return Build.VERSION.SDK_INT >= Build.VERSION_CODES.S ? checkPermission(Manifest.permission.BLUETOOTH_CONNECT) : true;
}

/**
* Boolean method that checks API level and check to see if we need to request POST_NOTIFICATIONS permission
* @return false if we need to request POST_NOTIFICATIONS permission
*/
private boolean hasPNPermission() {
    return Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU ? checkPermission(Manifest.permission.POST_NOTIFICATIONS) : true;
}

private boolean checkPermission(String permission) {
    return PackageManager.PERMISSION_GRANTED == ContextCompat.checkSelfPermission(getApplicationContext(), permission);
}

private void requestPermission(String[] permissions, int REQUEST_CODE) {
    ActivityCompat.requestPermissions(this, permissions, REQUEST_CODE);
}

private @NonNull String[] permissionsNeeded() {
    ArrayList<String> result = new ArrayList<>();
    if (!hasBTPermission()) {
        result.add(Manifest.permission.BLUETOOTH_CONNECT);
    }
    if (!hasPNPermission()) {
        result.add(Manifest.permission.POST_NOTIFICATIONS);
    }
    return (result.toArray(new String[result.size()]));
}

@Override
public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
    switch (requestCode) {
        case REQUEST_CODE:
            if (grantResults.length > 0) {
                for (int i = 0; i < grantResults.length; i++) {
                    if (permissions[i].equals(Manifest.permission.BLUETOOTH_CONNECT)) {
                        boolean btConnectGranted =
                                grantResults[i] == PackageManager.PERMISSION_GRANTED;
                        if (btConnectGranted) {
                            SdlReceiver.queryForConnectedService(this);
                        }
                    } else if (permissions[i].equals(Manifest.permission.POST_NOTIFICATIONS)) {
                    boolean postNotificationGranted =
                                grantResults[i] == PackageManager.PERMISSION_GRANTED;
                        if (!postNotificationGranted) {
                            // User denied permission, Notifications for SDL will not appear
                            // on Android 13 devices.
                        }
                    }
                }
            }
            break;
    }
}

//.....

```
 
