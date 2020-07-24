# Updating from 4.6 to 4.7


## Overview

This guide is to help developers get setup with the SDL Android library version 4.7. It is assumed that the developer is already updated to 4.6 of the library. This version includes the addition of the SdlManagers and a re-working of the transports which greatly enhances the use of the `SdlRouterService`, along with adding the functionality for secondary transports on supporting versions of SDL Core.

In this guide we will be focusing on the transitioning from the proxy, which implemented `SdlProxyALM` into using the `SdlManager` system, which includes specialized sub-managers that you can interact with through the `SdlManager`. We will follow the naming convention of the guides, highlighting the previous way of implementing SDL and showing the new ways of implementing it.

!!! NOTE
Moving from the `SdlProxyALM` implementation to the `SdlManager` API will require you to manually subscribe to the notifications and responses that you wish to receive instead of all of the notifications and responses being passed through the `IProxyListenerALM` interface.
!!!

## Integration Basics

The `SdlService` class will contain a great deal of changes as it acts as the main bridge to SDL functionality. There are going to be two main differences with how this class was set up in 4.6 versus 4.7.

### Removal of IProxyListenerALM

Previously, your `SdlService` had to implement the `IProxyListenerALM` interface. This often added many unnecessary lines of code to the class due to the need to override all of its functions. The need to do this has been removed in 4.7 with the inclusion of the `SdlManager` APIs. Developers now only have to add the listeners they need.

##### 4.6:

```java
public class SdlService extends Service implements IProxyListenerALM {

    // The proxy handles communication between the application and SDL
    private SdlProxyALM proxy = null;

    //...

    @Override
    public void someListener(){}
    //...
}
```

##### 4.7: The requirement to implement `IProxyListenerALM` is removed:

```java
public class SdlService extends Service {

	// The SdlManager exposes the APIs needed to communicate between the application and SDL
	private SdlManager sdlManager = null;

	//...
}
```

After removing `IProxyListenerALM ` from the `SdlService`, all of its previously overridden functions will need to be removed. If your app used any of these callback methods, it will help to document which ones they were, as you will need to add in the listeners that you need using the `SdlManager`'s `addOnRPCNotificationListener`.

!!! note
When you start using the managers, you have to make sure that your app subscribes to notifications before sending the corresponding RPC requests and subscriptions or else some notifications may be missed.
!!!


### Creation of SdlManager

As we no longer want to directly instantiate `SdlProxyALM`, we need to instantiate the `SdlManager` instead. This is best done using the `SdlManager.Builder` class using your application's details and configurations. In order to receive life cycle events from the `SdlManager`, an `SdlManagerListener` must be provided. The new code should resemble the following:

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
                	// RPC listeners and other functionality can be called once this callback is triggered.
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

    //...

}
```

Once you receive the `onStart` callback from `SdlManager`, you can add in your listeners and start adding UI elements. There will be more about adding the UI elements later. The last example in this section will be about adding specific listeners. Because we removed the `IProxyListenerALM` implementation, you will have to set listeners for the needs of your app.


### Listening for RPC notifications and events

We can listen for specific events using `SdlManager`'s `addOnRPCNotificationListener`. These listeners can be added either in the `onStart()` callback of the `SdlManagerListener` or after it has been triggered. The following example shows how to listen for HMI Status notifications. Additional listeners can be added for specific RPCs by using their corresponding `FunctionID` in place of the `ON_HMI_STATUS` in the following example and casting the `RPCNotification` object to the correct type.

##### Example of a listener for HMI Status:

```java
sdlManager.addOnRPCNotificationListener(FunctionID.ON_HMI_STATUS, new OnRPCNotificationListener() {
		@Override
		public void onNotified(RPCNotification notification) {
			OnHMIStatus status = (OnHMIStatus) notification;
			if (status.getHmiLevel() == HMILevel.HMI_FULL && ((OnHMIStatus) notification).getFirstRun()) {
				// first time in HMI Full
			}
		}
	});
```



### Sending RPCs

There are new method names and locations that mimic previous functionality for sending RPCs. These methods are located in the `SdlManager` and have the new names of `sendRPC`, `sendRPCs`, and `sendSequentialRPCs`.

##### 4.6:

```java
// single RPC
proxy.sendRPCRequest(request);

// muliple RPCs, non-sequential
proxy.sendRequests(rpcs, new OnMultipleRequestListener() {
	//...
});

// multiple RPCs, sequential
proxy.sendSequentialRequests(rpcs, new OnMultipleRequestListener() {
	//...
});
```

In 4.7, we use the `SdlManager` to send the requests.

##### 4.7:

```java
// single RPC
sdlManager.sendRPC(request);

// muliple RPCs, non-sequential
sdlManager.sendRPCs(rpcs, new OnMultipleRequestListener() {
	//...
});

// multiple RPCs, sequential
sdlManager.sendSequentialRPCs(rpcs, new OnMultipleRequestListener() {
	//...
});
```

### Using AOA Protocol

If your app uses USB to connect to SDL, this update provides a very useful enhancement. AOA connections now work with the `SdlRouterService`. This means that multiple USB apps can be connected to the head unit at once.

#### SdlBroadcastReceiver

Since the AOA transport will now use the multiplexing feature, it is important that your app correctly adds funcitonality for the `SdlRouterService`. This starts in the `SdlBroadcastReciever`.

##### 4.6:
```java
public class SdlReceiver extends com.smartdevicelink.SdlBroadcastReceiver {

    @Override
    public void onSdlEnabled(Context context, Intent intent) {
        //Use the provided intent but set the class to your SdlService
        intent.setClass(context, SdlService.class);
        context.startService(intent);
    }

    @Override
    public Class<? extends SdlRouterService> defineLocalSdlRouterClass() {
         return null;
    }

}
```
##### 4.7:
```java
public class SdlReceiver extends com.smartdevicelink.SdlBroadcastReceiver {

    @Override
    public void onSdlEnabled(Context context, Intent intent) {
        //Use the provided intent but set the class to your SdlService
        intent.setClass(context, SdlService.class);
        context.startService(intent);
    }

    @Override
    public Class<? extends SdlRouterService> defineLocalSdlRouterClass() {
        // define your local router service. For example:
        return com.sdl.hellosdlandroid.SdlRouterService.class;
    }

}
```

#### SdlRouterService


The `SdlRouterService` will listen for a connection with an SDL enabled module. When a connection happens, it will alert all SDL enabled apps that a connection has been established and they should start their SDL services.

##### 4.6:

(No implementation required).

##### 4.7:
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
Make sure this local class (SdlRouterService.java) is in the same package of SdlReceiver.java
!!!

#### SdlService

##### 4.6:

```java
transport = new USBTransportConfig(getBaseContext(), (UsbAccessory) intent.getParcelableExtra(UsbManager.EXTRA_ACCESSORY), false, false);
```
##### 4.7:

```java
MultiplexTransportConfig transport = new MultiplexTransportConfig(this, APP_ID, MultiplexTransportConfig.FLAG_MULTI_SECURITY_MED);
```

##### Additional configurations:

If your app requires high bandwidth transport, you can now specify that:

```java
transport.setRequiresHighBandwidth(true);
```

!!! Note
If your app only works when a high bandwidth transport is available, you should set `setRequiresHighBandwidth` to `true`. You cannot be certain that all core implementations support multiple transports. You could also set `TransportType.USB` as your only supported primary transport
!!!

Since the `SdlRouterService` now works with multiple transports, you can set your own configuration, for example:

```java
static final List<TransportType> multiplexPrimaryTransports = Arrays.asList(TransportType.USB, TransportType.BLUETOOTH);
static final List<TransportType> multiplexSecondaryTransports = Arrays.asList(TransportType.TCP, TransportType.USB, TransportType.BLUETOOTH);

//...

transport.setPrimaryTransports(multiplexPrimaryTransports);
transport.setSecondaryTransports(multiplexSecondaryTransports);
```
!!! NOTE
Multiple transports only work on supported versions of SDL Core.
!!!

#### AndroidManifest

##### 4.6

```xml
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />

    <uses-feature android:name="android.hardware.usb.accessory"/>

	<service
   		android:name=".SdlService"
       android:enabled="true"/>


	<receiver
		android:name=".SdlReceiver"
       android:enabled="true"
       android:exported="true"
       tools:ignore="ExportedReceiver">
       <intent-filter>
       	<action android:name="com.smartdevicelink.USB_ACCESSORY_ATTACHED"/> <!--For AOA -->
          <action android:name="sdl.router.startservice" />
   		</intent-filter>
	</receiver>

	<activity android:name="com.smartdevicelink.transport.USBAccessoryAttachmentActivity"
   		android:launchMode="singleTop">
       <intent-filter>
         	<action android:name="android.hardware.usb.action.USB_ACCESSORY_ATTACHED" />
       </intent-filter>

       <meta-data
         	android:name="android.hardware.usb.action.USB_ACCESSORY_ATTACHED"
          android:resource="@xml/accessory_filter" />
	</activity>

```

##### 4.7

```xml
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.BLUETOOTH"/>
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.FOREGROUND_SERVICE" />

    <uses-feature android:name="android.hardware.usb.accessory"/>


        <service
        android:name=".SdlService"
        android:enabled="true"/>

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
	<receiver
		android:name=".SdlReceiver"
       android:enabled="true"
       android:exported="true"
       tools:ignore="ExportedReceiver">
       <intent-filter>
       	<action android:name="com.smartdevicelink.USB_ACCESSORY_ATTACHED"/> <!--For AOA -->
          <action android:name="android.bluetooth.device.action.ACL_CONNECTED" />
          <action android:name="sdl.router.startservice" />
   		</intent-filter>
	</receiver>

	<activity android:name="com.smartdevicelink.transport.USBAccessoryAttachmentActivity"
   		android:launchMode="singleTop">
       <intent-filter>
         	<action android:name="android.hardware.usb.action.USB_ACCESSORY_ATTACHED" />
       </intent-filter>

       <meta-data
         	android:name="android.hardware.usb.action.USB_ACCESSORY_ATTACHED"
          android:resource="@xml/accessory_filter" />
	</activity>

```


## Lock Screen

There has been a major overhaul for lock screens in 4.7. Complicated lock screen setups are no longer required due to the addition of the `LockScreenManager`. Instead of going over the previous lock screen tutorial and then writing another one I will give brief instructions on how to either continue using your lock screen implementation, or upgrading to the new managed system. This review is brief, it is recommended that you look at the full [lock screen guide](https://smartdevicelink.com/en/guides/android/adding-the-lock-screen/)

#### Using your current implementation

If you would like to keep your current lock screen implementation, but would like to use the `SdlManager` for its other functionalities, you must disable the `LockScreenManager`. (This is not recommended as the new `LockScreenManager` takes care of a lot of boiler plate code and reduces possible errors)

##### Disabling the Lock Screen Manager:

To disable, create a `LockScreenConfig` object and set it in the `SdlManager.Builder` in your `SdlService.java` class.

```java
lockScreenConfig.setEnabled(false);
//...
builder.setLockScreenConfig(lockScreenConfig);
```

#### Using the new LockScreenManager

If you want SDL to handle the lock screen logic for you, it is simple. You will remove the classes that currently handle your lock screen, and set the variables you want for your new lock screen as defined in the [lock screen guide](https://smartdevicelink.com/en/guides/android/adding-the-lock-screen/). This simple addition is handled during the instantiation of the the `SdlManager` within `SdlService.java`.


##### Lock Screen Activity

You must declare the `SDLLockScreenActivity` in your manifest. To do so, simply add the following to your app's `AndroidManifest.xml` if you have not already done so:

```xml
<activity android:name="com.smartdevicelink.managers.lockscreen.SDLLockScreenActivity"
                  android:launchMode="singleTop"/>
```

!!! MUST
This manifest entry must be added for the lock screen feature to work.
!!!

##### Configurations

The default configurations should work for most app developers and is simple to get up and and running. However, it is easy to perform deeper configurations to the lock screen for your app. Below are the options that are available to customize your lock screen which builds on top of the logic already implemented in the `LockScreenManager`.

There is a setter in the `SdlManager.Builder` that allows you to set a `LockScreenConfig` by calling `builder.setLockScreenConfig(lockScreenConfig)`. The following options are available to be configured with the`LockScreenConfig`.

In order to to use these features, create a `LockScreenConfig` object and set it using `SdlManager.Builder` before you build `SdlManager`.


###### Custom Background Color

In your `LockScreenConfig` object, you can set the background color to a color resource that you have defined in your `Colors.xml` file:

```java
lockScreenConfig.setBackgroundColor(resourceColor); // For example, R.color.black
```

###### Custom App Icon

In your `LockScreenConfig` object, you can set the resource location of the drawable icon you would like displayed:

```java
lockScreenConfig.setAppIcon(appIconInt); // For example, R.drawable.lockscreen_icon
```

###### Showing The Device Logo

This sets whether or not to show the connected device's logo on the default lock screen. The logo will come from the connected hardware if set by the manufacturer. When using a Custom View, the custom layout will have to handle the logic to display the device logo or not. The default setting is false, but some OEM partners may require it.

In your `LockScreenConfig` object, you can set the boolean of whether or not you want the device logo shown, if available:

```java
lockScreenConfig.showDeviceLogo(true);
```

###### Setting A Custom Lock Screen View

If you'd rather provide your own layout, it is easy to set. In your `LockScreenConfig` object, you can set the reference to the custom layout to be used for the lock screen. If this is set, the other customizations described above will be ignored:

```java
lockScreenConfig.setCustomView(customViewInt);
```

## Displaying Information

### Setting text:

Previously, to set text fields, the developer had to create a `Show` RPC, set the text fields, and then send the PRC. It was also the developer's responsibility to make sure that they set only the lines of text that are supported by the template. In 4.7, the `ScreenManager` can be used and handles such logic internally. If a specific text field is not supported, it will be automatically hyphenated with other texts to make sure that everything is displayed correctly.

##### 4.6:

```java
Show show = new Show();
show.setMainField1("Hello, this is MainField1.");
show.setMainField2("Hello, this is MainField2.");
show.setMainField3("Hello, this is MainField3.");
show.setMainField4("Hello, this is MainField4.");
show.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse response) {
        if (((ShowResponse) response).getSuccess()) {
            Log.i("SdlService", "Successfully showed.");
        } else {
            Log.i("SdlService", "Show request was rejected.");
        }
    }
});
proxy.sendRPCRequest(show);
```

##### 4.7:

```java
sdlManager.getScreenManager().beginTransaction();
sdlManager.getScreenManager().setTextField1("Hello, this is MainField1.");
sdlManager.getScreenManager().setTextField2("Hello, this is MainField2.");
sdlManager.getScreenManager().setTextField3("Hello, this is MainField3.");
sdlManager.getScreenManager().setTextField4("Hello, this is MainField4.");
sdlManager.getScreenManager().commit(new CompletionListener() {
	@Override
	public void onComplete(boolean success) {
		Log.i(TAG, "ScreenManager update complete: " + success);

	}
});
```

### Setting images:

Previously, to set an image, the developer had to upload the image using the `PutFile` RPC. When it is uploaded, a `Show` RPC was then created and sent to display the image. In 4.7, the `ScreenManager` handles uploading the image and sending the RPCs internally.

##### 4.6:

```java
Image image = new Image();
image.setImageType(ImageType.DYNAMIC);
image.setValue("appImage.jpeg"); // a previously uploaded filename using PutFile RPC

Show show = new Show();
show.setGraphic(image);
show.setCorrelationID(CorrelationIdGenerator.generateId());
show.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse response) {
        if (((ShowResponse) response).getSuccess()) {
            Log.i("SdlService", "Successfully showed.");
        } else {
            Log.i("SdlService", "Show request was rejected.");
        }
    }
});
proxy.sendRPCRequest(show);
```

##### 4.7:

```java
SdlArtwork sdlArtwork = new SdlArtwork("appImage.jpeg", FileType.GRAPHIC_JPEG, R.drawable.appImage, true);
sdlManager.getScreenManager().setPrimaryGraphic(sdlArtwork);
```



### Using soft buttons:

Previously, to add a soft button with an image the developer had to upload the image by sending a `PutFile` RPC, and after the image is uploaded, creating a `SoftButton` object, then creating a `Show` RPC. They would then need to set the button in the RPC, and then send the request. In 4.7, the `ScreenManager` takes care of sending the RPCs. The developer just has to create `softButtonObject`, add a state to it, then use the `ScreenManager` to set soft button objects.

##### 4.6:

```java
Image cancelImage = new Image();
cancelImage.setImageType(ImageType.DYNAMIC);
cancelImage.setValue("cancel.jpeg"); // a previously uploaded filename using PutFile RPC

List<SoftButton> softButtons = new ArrayList<>();

SoftButton cancelButton = new SoftButton();
cancelButton.setType(SoftButtonType.SBT_IMAGE);
cancelButton.setImage(cancelImage);
cancelButton.setSoftButtonID(1);

softButtons.add(cancelButton);

Show show = new Show();
show.setSoftButtons(softButtons);
proxy.sendRPCRequest(show);
```

##### 4.7:

```java
SoftButtonState softButtonState = new SoftButtonState("state1", "cancel", new SdlArtwork("cancel.jpeg", FileType.GRAPHIC_JPEG, R.drawable.cancel, true));
SoftButtonObject softButtonObject = new SoftButtonObject("object", Collections.singletonList(softButtonState), softButtonState.getName(), null);
sdlManager.getScreenManager().setSoftButtonObjects(Collections.singletonList(softButtonObject));
```

Receiving button events on previous versions of SDL had to be done using `onOnButtonEvent` and `onOnButtonPress` callbacks from the `IProxyListenerALM` interface. The id had to be checked to know the exact button that received the event. In 4.7, it is much cleaner: a listener can be added to the `SoftButtonObject`, so the developer can easily tell when and which soft button received the event.

##### 4.6:

```java
@Override
public void onOnButtonEvent(OnButtonEvent notification) {
   Log.i(TAG, "onOnButtonEvent: ");

   if (notification.getButtonName() == CUSTOM_BUTTON){
        int ID = notification.getCustomButtonName();
        Log.i(TAG, "Button event received for button " + ID);
    }
}

@Override
public void onOnButtonPress(OnButtonPress notification) {
    Log.i(TAG, "onOnButtonPress: ");

    if (notification.getButtonName() == CUSTOM_BUTTON){
        int ID = notification.getCustomButtonName();
        Log.i(TAG, "Button press received for button " + ID);
    }
}
```


##### 4.7:

```java
softButtonObject.setOnEventListener(new SoftButtonObject.OnEventListener() {
    @Override
    public void onPress(SoftButtonObject softButtonObject, OnButtonPress onButtonPress) {
        Log.i(TAG, "OnButtonPress: ");
    }

    @Override
    public void onEvent(SoftButtonObject softButtonObject, OnButtonEvent onButtonEvent) {
        Log.i(TAG, "OnButtonEvent: ");
    }
});
```


### Receiving Subscribe Buttons Events
 Previously, your `SdlService` had to implement `IProxyListenerALM` interface which means your `SdlService` class had to override all of the `IProxyListenerALM` callback methods including `OnButtonEvent` and `OnButtonPress`.


##### 4.6

```java
@Override
public void onOnHMIStatus(OnHMIStatus notification) {
    if(notification.getHmiLevel() == HMILevel.HMI_FULL && notification.getFirstRun()) {
        SubscribeButton subscribeButtonRequest = new SubscribeButton();
        subscribeButtonRequest.setButtonName(ButtonName.SEEKRIGHT);
        proxy.sendRPCRequest(subscribeButtonRequest);
    }
}

@@Override
public void onOnButtonEvent(OnButtonEvent notification) {
    switch(notification.getButtonName()){
            case OK:
                break;
            case SEEKLEFT:
                break;
            case SEEKRIGHT:
                break;
            case TUNEUP:
                break;
            case TUNEDOWN:
                break;
        }
}

@Override
public void onOnButtonPress(OnButtonPress notification) {
        switch(notification.getButtonName()){
            case OK:
                break;
            case SEEKLEFT:
                break;
            case SEEKRIGHT:
                break;
            case TUNEUP:
                break;
            case TUNEDOWN:
                break;
        }
}
```


 In 4.7 and the new manager APIs, in order to receive the `OnButtonEvent` and `OnButtonPress` notifications, your app must add a `OnRPCNotificationListener` using the `SdlManager`'s method `addOnRPCNotificationListener`. This will subscribe the app to any notifications of the provided type, in this case `ON_BUTTON_EVENT` and `ON_BUTTON_PRESS`. The listener should be added before sending the corresponding RPC request/subscription or else some notifications may be missed.


```java
sdlManager.addOnRPCNotificationListener(FunctionID.ON_BUTTON_EVENT, new OnRPCNotificationListener() {
    @Override
    public void onNotified(RPCNotification notification) {
        OnButtonPress onButtonPressNotification = (OnButtonPress) notification;
        switch (onButtonPressNotification.getButtonName()) {
            case OK:
                break;
            case SEEKLEFT:
                break;
            case SEEKRIGHT:
                break;
            case TUNEUP:
                break;
            case TUNEDOWN:
                break;
        }
    }
});

sdlManager.addOnRPCNotificationListener(FunctionID.ON_BUTTON_PRESS, new OnRPCNotificationListener() {
    @Override
    public void onNotified(RPCNotification notification) {
        OnButtonPress onButtonPressNotification = (OnButtonPress) notification;
        switch (onButtonPressNotification.getButtonName()) {
            case OK:
                break;
            case SEEKLEFT:
                break;
            case SEEKRIGHT:
                break;
            case TUNEUP:
                break;
            case TUNEDOWN:
                break;
        }
    }
});

SubscribeButton subscribeButtonRequest = new SubscribeButton();
subscribeButtonRequest.setButtonName(ButtonName.SEEKRIGHT);
sdlManager.sendRPC(subscribeButtonRequest);
```

### Changing The Template:

Previously, developers had to pass a string that represents the name of the template to `SetDisplayLayout`. In 4.7, a new `PredefinedLayout` enum is introduced to hold all possible values for the templates.

##### 4.6:

```java
SetDisplayLayout setDisplayLayoutRequest = new SetDisplayLayout();
setDisplayLayoutRequest.setDisplayLayout("GRAPHIC_WITH_TEXT");
try{
    proxy.sendRPCRequest(setDisplayLayoutRequest);
}catch (SdlException e){
    e.printStackTrace();
}
```

##### 4.7:

```java
SetDisplayLayout setDisplayLayoutRequest = new SetDisplayLayout();
setDisplayLayoutRequest.setDisplayLayout(PredefinedLayout.GRAPHIC_WITH_TEXT.toString());

sdlManager.sendRPC(setDisplayLayoutRequest);

```



## Uploading Files and Graphics

SDL Android 4.7 introduces the `FileManager`, which is accessible through the `SdlManager`. Previous methods of uploading files and performing their functions still work, but now there are a set of convenience methods that do a lot of the boilerplate work for you.

Check out the [Uploading Files and Graphics](https://smartdevicelink.com/en/guides/android/uploading-files-and-graphics/) guide for code examples and detailed explanations.

### SDL File and SDL Artwork

New to version 4.7 of the SDL Android library are `SdlFile` and `SdlArtwork` objects. These have been created in parallel with the `FileManager` to help streamline SDL workflow. `SdlArtwork` is an extension of `SdlFile` that pertains only to graphic specific file types, and its use case is similar. For the rest of this document, `SdlFile` will be described, but everything also applies to `SdlArtwork`.

#### Creation

One of the hardest parts about getting a file into SDL was the boilerplate code needed to convert the file into a byte array that was used by the head unit. Now, you can instantiate a `SdlFile` with:

##### A resource ID

```java
new SdlFile(@NonNull String fileName, @NonNull FileType fileType, int id, boolean persistentFile)
```
##### A URI

```java
new SdlFile(@NonNull String fileName, @NonNull FileType fileType, Uri uri, boolean persistentFile)
```
And last but not least

##### A byte array

```java
new SdlFile(@NonNull String fileName, @NonNull FileType fileType, byte[] data, boolean persistentFile)
```

without the need to implement the methods needed to do the conversion of data yourself.

### Uploading a File

Uploading a file with the `FileManager` is a simple process. With an instantiated `SdlManager`,
you can simply call:

```java
sdlManager.getFileManager().uploadFile(sdlFile, new CompletionListener() {
    @Override
    public void onComplete(boolean success) {

    }
});
```


## Getting Vehicle Data and Subscribing to Notifications

Previously, your `SdlService` had to implement `IProxyListenerALM` interface which means your `SdlService` class had to override all of the `IProxyListenerALM` callback methods including `onOnVehicleData`.

##### 4.6:

```java
@Override
public void onOnHMIStatus(OnHMIStatus notification) {
    if(notification.getHmiLevel() == HMILevel.HMI_FULL && notification.getFirstRun()) {
        SubscribeVehicleData subscribeRequest = new SubscribeVehicleData();
        subscribeRequest.setPrndl(true);
        subscribeRequest.setOnRPCResponseListener(new OnRPCResponseListener() {
            @Override
            public void onResponse(int correlationId, RPCResponse response) {
                if(response.getSuccess()){
                    Log.i("SdlService", "Successfully subscribed to vehicle data.");
                }else{
                    Log.i("SdlService", "Request to subscribe to vehicle data was rejected.");
                }
            }
        });
        try {
            proxy.sendRPCRequest(subscribeRequest);
        } catch (SdlException e) {
            e.printStackTrace();
        }
    }
}

@Override
public void onOnVehicleData(OnVehicleData notification) {
    PRNDL prndl = notification.getPrndl();
    Log.i("SdlService", "PRNDL status was updated to: " prndl.toString());
}
```

In 4.7 and the new manager APIs, in order to receive the  `OnVehicleData ` notifications, your app must add a `OnRPCNotificationListener` using the `SdlManager`'s method `addOnRPCNotificationListener`. This will subscribe the app to any notifications of the provided type, in this case `ON_VEHICLE_DATA`. The listener should be added before sending the corresponding RPC request/subscription or else some notifications may be missed.

##### 4.7:

```java
sdlManager.addOnRPCNotificationListener(FunctionID.ON_VEHICLE_DATA, new OnRPCNotificationListener() {
    @Override
    public void onNotified(RPCNotification notification) {
        OnVehicleData onVehicleDataNotification = (OnVehicleData) notification;
        if (onVehicleDataNotification.getPrndl() != null) {
            Log.i("SdlService", "PRNDL status was updated to: " + onVehicleDataNotification.getPrndl());
        }
    }
});


SubscribeVehicleData subscribeRequest = new SubscribeVehicleData();
subscribeRequest.setPrndl(true);
subscribeRequest.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse response) {
        if(response.getSuccess()){
            Log.i("SdlService", "Successfully subscribed to vehicle data.");
        }else{
            Log.i("SdlService", "Request to subscribe to vehicle data was rejected.");
        }
    }
});
sdlManager.sendRPC(subscribeRequest);
```



## Getting In-Car Audio

### Subscribing to AudioPassThru Notifications

Previously, your `SdlService` had to implement `IProxyListenerALM` interface which means your `SdlService` class had to override all of the `IProxyListenerALM` callback methods including `onOnAudioPassThru`.

##### 4.6:

```java
@Override
public void onOnHMIStatus(OnHMIStatus notification) {
    if(notification.getHmiLevel() == HMILevel.HMI_FULL && notification.getFirstRun()) {
        PerformAudioPassThru performAPT = new PerformAudioPassThru();
        performAPT.setAudioPassThruDisplayText1("Ask me \"What's the weather?\"");
        performAPT.setAudioPassThruDisplayText2("or \"What's 1 + 2?\"");
        performAPT.setInitialPrompt(TTSChunkFactory.createSimpleTTSChunks("Ask me What's the weather? or What's 1 plus 2?"));
        performAPT.setSamplingRate(SamplingRate._22KHZ);
        performAPT.setMaxDuration(7000);
        performAPT.setBitsPerSample(BitsPerSample._16_BIT);
        performAPT.setAudioType(AudioType.PCM);
        performAPT.setMuteAudio(false);
        proxy.sendRPCRequest(performAPT);
    }
}

@Override
public void onOnAudioPassThru(OnAudioPassThru notification) {
    byte[] dataRcvd = notification.getAPTData();
    processAPTData(dataRcvd); // Do something with audio data
}
```

In 4.7 and the new manager APIs, in order to receive the  `OnAudioPassThru ` notifications, your app must add a `OnRPCNotificationListener` using the `SdlManager`'s method `addOnRPCNotificationListener`. This will subscribe the app to any notifications of the provided type, in this case `ON_AUDIO_PASS_THRU`. The listener should be added before sending the corresponding RPC request/subscription or else some notifications may be missed.

##### 4.7:
```java
sdlManager.addOnRPCNotificationListener(FunctionID.ON_AUDIO_PASS_THRU, new OnRPCNotificationListener() {
    @Override
    public void onNotified(RPCNotification notification) {
        OnAudioPassThru onAudioPassThru = (OnAudioPassThru) notification;
        byte[] dataRcvd = onAudioPassThru.getAPTData();
        processAPTData(dataRcvd); // Do something with audio data
    }
});

PerformAudioPassThru performAPT = new PerformAudioPassThru();
performAPT.setAudioPassThruDisplayText1("Ask me \"What's the weather?\"");
performAPT.setAudioPassThruDisplayText2("or \"What's 1 + 2?\"");
performAPT.setInitialPrompt(TTSChunkFactory.createSimpleTTSChunks("Ask me What's the weather? or What's 1 plus 2?"));
performAPT.setSamplingRate(SamplingRate._22KHZ);
performAPT.setMaxDuration(7000);
performAPT.setBitsPerSample(BitsPerSample._16_BIT);
performAPT.setAudioType(AudioType.PCM);
performAPT.setMuteAudio(false);
sdlManager.sendRPC(performAPT);
```

## Mobile Navigation

### Video Streaming:

Previously, developers had to make sure that the app was in HMI_FULL before starting the video stream, In 4.7, after the `SdlManager` has called its `onStart` method, the developer can start video streaming in `VideoStreamingManager.start()`'s `CompletionListener`. The `VideoStreamingManager` will take care of starting the video when the app becomes ready.

##### 4.6:

```java
    if(notification.getHmiLevel().equals(HMILevel.HMI_FULL)){
        if (notification.getFirstRun()) {
            proxy.startRemoteDisplayStream(getApplicationContext(), MyDisplay.class, null, false);
        }
    }

}
```

##### 4.7:

```java
sdlManager.getVideoStreamManager().start(new CompletionListener() {
    @Override
    public void onComplete(boolean success) {
        if (success) {
            sdlManager.getVideoStreamManager().startRemoteDisplayStream(getApplicationContext(), MyDisplay.class, null, false);
        }
    }
});
```

### Audio Streaming


With the addition of the `AudioStreamingManager`, which is accessed through `SdlManager`, you can now use `mp3` files in addition to `raw`. The `AudioStreamingManager` also handles `AudioStreamingCapabilities` for you, so your stream will use the correct capabilities for the connected head unit. We suggest that for any audio streaming that this is now used. Below is the difference in streaming from 4.6 to 4.7

##### 4.6

```java
 private void startAudioStream(){

        final InputStream is = getResources().openRawResource(R.raw.audio_file);

        AudioStreamingParams audioParams = new AudioStreamingParams(44100, 1);
        listener = proxy.startAudioStream(false, AudioStreamingCodec.LPCM, audioParams);
        if (listener != null){
            try {
                listener.sendAudio(readToByteBuffer(is), -1);

            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    private void stopAudioStream(){
        proxy.endAudioStream();
    }

    static ByteBuffer readToByteBuffer(InputStream inStream) throws IOException {
        byte[] buffer = new byte[8000];
        ByteArrayOutputStream outStream = new ByteArrayOutputStream(8000);
        int read;
        while (true) {
            read = inStream.read(buffer);
            if (read == -1)
                break;
            outStream.write(buffer, 0, read);
        }
        ByteBuffer byteData = ByteBuffer.wrap(outStream.toByteArray());
        return byteData;
    }
```

##### 4.7

```java
if (sdlManager.getAudioStreamManager() != null) {
    Log.i(TAG, "Trying to start audio streaming");
    sdlManager.getAudioStreamManager().start(new CompletionListener() {
        @Override
        public void onComplete(boolean success) {
            if (success) {
                sdlManager.getAudioStreamManager().startAudioStream(false, new CompletionListener() {
                    @Override
                    public void onComplete(boolean success) {
                        if (success) {
                            Resources resources = getApplicationContext().getResources();
                            int resourceId = R.raw.audio_file;
                            Uri uri = new Uri.Builder()
                            .scheme(ContentResolver.SCHEME_ANDROID_RESOURCE)
                            .authority(resources.getResourcePackageName(resourceId))
                            .appendPath(resources.getResourceTypeName(resourceId))
                            .appendPath(resources.getResourceEntryName(resourceId))
                            .build();
                            sdlManager.getAudioStreamManager().pushAudioSource(uri, new CompletionListener() {
                                @Override
                                public void onComplete(boolean success) {
                                    if (success) {
                                        Log.i(TAG, "Audio file played successfully!");
                                    } else {
                                        Log.i(TAG, "Audio file failed to play!");
                                    }
                                }
                            });
                        } else {
                            Log.d(TAG, "Audio stream failed to start!");
                        }
                    }
                });
            } else {
                Log.i(TAG, "Failed to start audio streaming manager");
            }
        }
    });
}
```



## Checking Permissions:

Previously, it was not easy to check if specific permission had changed. Developers had to keep checking `onOnHMIStatus` and `onOnPermissionsChange` callbacks and manually check the responses to see if the permission is allowed. In 4.7, the `PermissionManager` implements all of this logic internally. It keeps a cached copy of the callback responses whenever an update is received. So developer can call `isRPCAllowed()` any time to know if a permission is allowed. It also makes it very simple to add a listener.

##### 4.6:

```java
@Override
public void onOnHMIStatus(OnHMIStatus notification) {
    hmiLevel = notification.getHmiLevel();
    if (checkShowPermission(FunctionID.SHOW.toString(), hmiLevel, permissionItems)){
        // Show RPC is allowed
    }
}

@Override
public void onOnPermissionsChange(OnPermissionsChange notification) {
    permissionItems = notification.getPermissionItem();
    if (checkShowPermission(FunctionID.SHOW.toString(), hmiLevel, permissionItems)){
        // Show RPC is allowed
    }
}

private boolean checkShowPermission(String rpcName, HMILevel hmiLevel, List<PermissionItem> permissionItems){
    PermissionItem permissionItem = null;
    for (PermissionItem item : permissionItems) {
        if (rpcName.equals(item.getRpcName())){
            permissionItem = item;
            break;
        }
    }
    if (hmiLevel == null || permissionItem == null || permissionItem.getHMIPermissions() == null || permissionItem.getHMIPermissions().getAllowed() == null){
            return false;
    } else if (permissionItem.getHMIPermissions().getUserDisallowed() != null){
        return permissionItem.getHMIPermissions().getAllowed().contains(hmiLevel) && !permissionItem.getHMIPermissions().getUserDisallowed().contains(hmiLevel);
    } else {
        return permissionItem.getHMIPermissions().getAllowed().contains(hmiLevel);
    }
}
```

##### 4.7:

To check if a permission is allowed:

```java
boolean allowed = sdlManager.getPermissionManager().isRPCAllowed(FunctionID.SHOW);
```

To setup a permission listener:

```java
List<PermissionElement> permissionElements = Collections.singletonList(new PermissionElement(FunctionID.SHOW, null));
UUID listenerId = sdlManager.getPermissionManager().addListener(permissionElements, PermissionManager.PERMISSION_GROUP_TYPE_ANY, new OnPermissionChangeListener() {
    @Override
    public void onPermissionsChange(@NonNull Map<FunctionID, PermissionStatus> allowedPermissions, @NonNull int permissionGroupStatus) {
        if (allowedPermissions.get(FunctionID.SHOW).getIsRPCAllowed()) {
            // Show RPC is allowed
        }
    }
});
```

For more information about `PermissionManager`, you can check [this page](/guides/android/permission-manager/).


### Handling a Language Change

Previously, to let your app reconnect after the user changes the head unit language, your app had to send an intent in the `onProxyClosed` callback. That intent should be received by `SdlReceiver` to start the `SdlService`. The `SdlReceiver` part did not change so we will only cover the changes in sending the intent which was done in previous versions as the following:

```java
@Override
public void onProxyClosed(String info, Exception e, SdlDisconnectedReason reason) {
    stopSelf();
    if(reason.equals(SdlDisconnectedReason.LANGUAGE_CHANGE)){
        Intent intent = new Intent(TransportConstants.START_ROUTER_SERVICE_ACTION);
        intent.putExtra(SdlReceiver.RECONNECT_LANG_CHANGE, true);
        sendBroadcast(intent);
    }
}
```

In 4.7, the app has to send the intent in a `ON_LANGUAGE_CHANGE` notification listener as the following:

```java
sdlManager.addOnRPCNotificationListener(FunctionID.ON_LANGUAGE_CHANGE, new OnRPCNotificationListener() {
    @Override
    public void onNotified(RPCNotification notification) {
        SdlService.this.stopSelf();
        Intent intent = new Intent(TransportConstants.START_ROUTER_SERVICE_ACTION);
        intent.putExtra(SdlReceiver.RECONNECT_LANG_CHANGE, true);
        AndroidTools.sendExplicitBroadcast(context, intent, null);
    }
});
```

Fore more information about handling language changes please visit [this page](/guides/android/handling-language-change/)


## Remote Control

### Subscribing to OnInteriorVehicleData Notifications
 Previously, your `SdlService` had to implement `IProxyListenerALM` interface which means your `SdlService` class had to override all of the `IProxyListenerALM` callback methods including `onOnInteriorVehicleData`.

##### 4.6:

```java
@Override
public void onOnHMIStatus(OnHMIStatus notification) {
    if(notification.getHmiLevel() == HMILevel.HMI_FULL && notification.getFirstRun()) {
        GetInteriorVehicleData interiorVehicleData = new GetInteriorVehicleData();
        interiorVehicleData.setModuleType(ModuleType.RADIO);
        interiorVehicleData.setSubscribe(true);
        interiorVehicleData.setOnRPCResponseListener(new OnRPCResponseListener() {
                @Override
                public void onResponse(int correlationId, RPCResponse response) {
                    GetInteriorVehicleData getResponse = (GetInteriorVehicleData) response;
                    //This can now be used to retrieve data
                }
        });
        proxy.sendRPCRequest(interiorVehicleData);
    }
}

@Override
public void onOnInteriorVehicleData(OnInteriorVehicleData response) {
    //Perform action based on notification
}
```


 In 4.7 and the new manager APIs, in order to receive the  `OnInteriorVehicleData ` notifications, your app must add a `OnRPCNotificationListener` using the `SdlManager`'s method `addOnRPCNotificationListener`. This will subscribe the app to any notifications of the provided type, in this case `ON_INTERIOR_VEHICLE_DATA`. The listener should be added before sending the corresponding RPC request/subscription or else some notifications may be missed.

#### 4.7:

```java
sdlManager.addOnRPCNotificationListener(FunctionID.ON_INTERIOR_VEHICLE_DATA, new OnRPCNotificationListener() {
    @Override
    public void onNotified(RPCNotification notification) {
        OnInteriorVehicleData onInteriorVehicleData = (OnInteriorVehicleData) notification;
        //Perform action based on notification
    }
});

GetInteriorVehicleData interiorVehicleData = new GetInteriorVehicleData();
interiorVehicleData.setModuleType(ModuleType.RADIO);
interiorVehicleData.setSubscribe(true);
interiorVehicleData.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse response) {
        GetInteriorVehicleData getResponse = (GetInteriorVehicleData) response;
        //This can now be used to retrieve data
    }
});
sdlManager.sendRPC(interiorVehicleData);
```