# Using Android Open Accessory Protocol

Incorporating AOA into an SDL enabled app allows it to create and register an SDL session over USB. This guide will assume the app has already integrated the SDL library as laid out in the previous guides. AOA connections are sent through the `SDLRouterService` to bypass an Android limitation of only one app being able to be used through the AOA intent.

Prerequisites:

* [Installation guide](Getting Started/Installation)
* [SDK Configuration guide](Getting Started/SDK Configuration)
* [Integration Basics guide](Getting Started/Integration Basics - Java)

We will add or make changes to:

* Android Manifest __(of your app)__
* SdlService _(optional)_

## Prerequisites

The Installation, SDK Configuration, and Integration Basics guides must be completed before enabling the use of the AOA USB transport. The remainder of the guide will assume all steps will be followed.  


## Android Manifest

To use the AOA protocol, you must specify so in your app's Manifest with:

```xml
<uses-feature android:name="android.hardware.usb.accessory"/>
```

!!! MUST
This feature will not work without including this line!
!!!

The SDL Android library houses a `USBAccessoryAttachmentActivity` that you need to add between your Manifest's `<application>…</application>` tags:

```xml
<activity android:name="com.smartdevicelink.transport.USBAccessoryAttachmentActivity"
	android:launchMode="singleTop"
	android:exported=true>
	<intent-filter>
		<action android:name="android.hardware.usb.action.USB_ACCESSORY_ATTACHED" />
	</intent-filter>

	<meta-data
		android:name="android.hardware.usb.action.USB_ACCESSORY_ATTACHED"
		android:resource="@xml/accessory_filter" />
</activity>
```
!!! MUST
Android API 31 now requires any activity with an intent filter to explicitly set the exported flag to true (`exported=true`).
!!!

!!! NOTE
The accessory_filter.xml file is included with the SDL Android Library 
!!!

## Media Apps

Media applications do not register over AOA since by default there are no audio streaming methods available.
To get media applications to register, when creating the connection you need to set flag requiresAudioSupport to false:

```java
MultiplexTransportConfig multiplexTransportConfig = new MultiplexTransportConfig(getBaseContext(), APP_ID, MultiplexTransportConfig.FLAG_MULTI_SECURITY_OFF);
multiplexTransportConfig.setRequiresAudioSupport(false);
```

## SmartDeviceLink Service

As long as the app doesn't require high bandwidth, it shouldn't matter which transport is being connected. A multiplex transport should be used like the one that follows:

```java
@Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        
        if (sdlManager == null) {
            MultiplexTransportConfig transport = new MultiplexTransportConfig(this, APP_ID, MultiplexTransportConfig.FLAG_MULTI_SECURITY_OFF);
           
            SdlManagerListener listener = new SdlManagerListener() {
                //...
            };

            // ...
            
            builder.setTransportType(transport);
            sdlManager = builder.build();
            sdlManager.start();
        }
        ...
    }
```

### Using only USB / AOA

The new `MultiplexingConfig` allows for apps to be able to connect via Bluetooth and USB as primary transports. If you want your app to only use USB / AOA, then you should specifically only set that as the only allowed primary transport.

When defining your transport, also pass in a custom list that only contains the USB:

```java
List<TransportType> multiplexPrimaryTransports = Arrays.asList(TransportType.USB);

MultiplexTransportConfig transport = new MultiplexTransportConfig(this, appId, MultiplexTransportConfig.FLAG_MULTI_SECURITY_MED);

transport.setPrimaryTransports(multiplexPrimaryTransports);
```

### Multiple Transports

Since the `SdlRouterService` now handles both bluetooth and AOA/USB connections, an app will be connected to the transport that connects first if the app includes it in their transport config. If a module supports secondary transports, the second transport to be connected of bluetooth or USB will be available as well as potentially TCP. This means even though the app might register over bluetooth, if USB or TCP are available those transports will be available for high bandwidth services. For more information please see the [Multiple Transport Guide](/guides/android/getting-started/multiple-transports/). 