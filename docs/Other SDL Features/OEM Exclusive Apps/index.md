# OEM Exclusive Apps
The Android library can prevent apps from starting their `SdlRouterService` based on if they support the vehicle type if the connection is over Bluetooth.

!!! NOTE
If older apps are installed that use a version less then SDL Android 4.12.0, on the first connection to the vehicle an OEM app may host the `SdlRouterService` but on subsequent connections it may pass it off to another app.
!!!

!!! NOTE
This does not work over AOA connections.
!!!


## Create file for supported vehicle types and add to Android Manifest
To implement this feature, you will need to define an XML file for supported vehicles in resources of the project called `supported_vehicle_type.xml`, and add it as metaData for `SdlRouterService` in its AndroidManifest.  If an app defines a vehicle-type element, then it should always have a make attribute, all other attributes; are optional. However, if the you want to use model year or trim, you should define make and model attributes as well. The Java Suite app library will check only the defined attributes. The below example shows a valid vehicle type resource file.


```XML
<?xml version="1.0" encoding="utf-8"?>

<resource>
    <!-- Vehicle filter for vehicle make-->
    <vehicle-type
        make="SDL"/>
    <!-- Vehicle filter for vehicle make, model and model year-->
    <vehicle-type
        make="SDL"
        model="Generic"
        modelYear="2021"/>
    <!-- Vehicle filter for vehicle make, model and trim-->
    <vehicle-type
        make="SDL"
        model="Generic"
        trim="SE"/>
</resource>
```
Add supported vehicle type file as metaData for `SdlRouterService` in AndroidManifest

```XML
<meta-data
    android:name="@string/sdl_oem_vehicle_type_filter_name"
    android:resource="@xml/supported_vehicle_type" />

```

## Prevent app from connecting to unsupported vehicles 
Apps can still receive an intent to start when `SDL` is enabled from other apps. To prevent an OEM app from starting their `SdlService`, vehicle type can be retrieved in `SdlReceiver.onSdlEnabled` and the app can choose to not to start `SdlService` for that app.

```java

@Override
public void onSdlEnabled(Context context, Intent intent) {
    DebugTool.logInfo(TAG, "SDL Enabled");
    intent.setClass(context, SdlService.class);

    VehicleType vehicleType = null;
    if (intent.hasExtra(TransportConstants.VEHICLE_INFO_EXTRA)) {
        vehicleType = intent.getParcelableExtra(TransportConstants.VEHICLE_INFO_EXTRA);
    }

    VehicleType vehicleType1 = new VehicleType().setMake("SDL");
    VehicleType vehicleType2 = new VehicleType().setMake("SDL").setModel("Generic").setModelYear("2021");
    VehicleType vehicleType3 = new VehicleType().setMake("SDL").setModel("Generic").setTrim("SE");
    List<VehicleType> supportedVehicleList = Arrays.asList(vehicleType1, vehicleType2, vehicleType3);

    if (vehicleType != null && SdlAppInfo.checkIfVehicleSupported(supportedVehicleList, vehicleType)) {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            context.startForegroundService(intent);
        } else {
            context.startService(intent);
        }
    }
}

```
