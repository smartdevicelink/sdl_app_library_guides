# Video Streaming (RPC v4.5+)
One of the most powerful features provided by SDL is the ability to stream an interactive Android layout directly to a vehicle's heads up display. The following guide explains the basics of this functionality.

## SDL Remote Display
The `SdlRemoteDisplay` base class provides the easiest way to start streaming using SDL. The `SdlRemoteDisplay` is extended from Android's `Presentation` class with modifications to work with other aspects of the SDL Android library.

!!! NOTE
It is recommended that you extend this as a local class within the service that has the `SdlManager` instance.
!!!

Extending this class gives developers a familiar, native experience to handle layouts and events on screen.

!!! NOTE
You must have a valid and approved application ID from an OEM in order receive user input from a remote display. 
!!!

The following example uses a simple layout containing a single button:

__Inside SdlService.java:__
```java
//...

//This class should be nested in SdlService or another class containing the application's SdlManager instance
public static class MyDisplay extends SdlRemoteDisplay{
    public MyDisplay(Context context, Display display) {
        super(context, display);
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        setContentView(R.layout.stream);

        Button button = findViewById(R.id.button);

        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                DebugTool.logInfo(TAG, "Button Clicked");
            }
        });
    }
}

//...
```
__res/layout/stream.xml:__
```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
  android:layout_width="match_parent"
  android:layout_height="match_parent">

  <Button
    android:id="@+id/button"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_centerInParent="true" />

</RelativeLayout>
```

!!! Note
If you are obfuscating the code in your app, make sure to exclude your class that extends `SdlRemoteDisplay`. For more information on how to do that, you can check [Proguard Guidelines](Getting Started/Proguard Guidelines).
!!!

## Managing the Stream
The `VideoStreamManager` can be used to start streaming a remote display after the `SdlManager` has successfully been started. This is performed by calling the method `startRemoteDisplayStream()`.

__Inside SdlService.java:__
```java
//...

if (sdlManager.getVideoStreamManager() != null) {
    sdlManager.getVideoStreamManager().start(new CompletionListener () {
        @Override
        public void onComplete(boolean success) {
            if (success) {
                sdlManager.getVideoStreamManager().startRemoteDisplayStream(
                    getApplicationContext(),
                    MyDisplay.class,
                    null,
                    false,
                    null,
                    null
                );
            } else {
                DebugTool.logError(TAG, "Failed to start video streaming manager");
            }
        }
    });
}

//...
```

### Ending the Stream
When the `HMIStatus` is back to `HMI_NONE` it is time to stop the stream. This is accomplished through a method `stopStreaming()`.

__Inside SdlService.java:__
```java
//...

/*
 OnRPCNotificationListeners are commonly used to perform tasks at specific HMI levels
 In this case, it is used to end streaming at HMI_NONE
*/ 
Map<FunctionID, OnRPCNotificationListener> onRPCNotificationListenerMap = new HashMap<>();
    onRPCNotificationListenerMap.put(FunctionID.ON_HMI_STATUS, new OnRPCNotificationListener() {
    @Override
    public void onNotified(RPCNotification notification) {
        OnHMIStatus status = (OnHMIStatus) notification;
        
        //...

        if (status != null && status.getHmiLevel() == HMILevel.HMI_NONE) {
            
            //...
 
            //Stop the stream
            if (sdlManager.getVideoStreamManager() != null
                && sdlManager.getVideoStreamManager().isStreaming()) {
                sdlManager.getVideoStreamManager().stopStreaming();
            }

            //...

        }

        //...

    }
});

//...

/*
 builder refers to an instance of SdlManager.Builder
 The line below needs to be placed after builder is created and before builder.build() is called
*/
builder.setRPCNotificationListeners(onRPCNotificationListenerMap);

//...
```

### Handling HMI Scaling (RPC v6.0+)
If the HMI scales the video stream, you will have to handle scaling the projected view, touches, and haptic rectangles yourself (this is all handled for you behind the scenes in the `VideoStreamManager` API). To find out if the HMI scales the video stream, you must for query and check the `VideoStreamingCapability` for the `scale` property. Please check the [Adaptive Interface Capabilities](Displaying a User Interface/Adaptive Interface Capabilities) section for more information on how to query for this property using the system capability manager. For information on manually handling haptic rectangles, refer to the [Haptic Input](Video Streaming for Navigation Apps/Supporting Haptic Input) guide.

### Video Streaming Parameters (SDL v5.1+)
Starting with SDL version 5.1+ the `VideoStreamingParameters` you provide will automatically be aligned with the `VideoStreamingCapabilities` provided by the HMI.
If the HMI provides the scale or resolution in the `VideoStreamingCapabilities` the video stream will use that scale or resolution. Otherwise, the scale or resolution you defined in the `VideoStreamingParameters` will be used.
If the HMI provides the bitrate or preferred frame rate in the `VideoStreamingCapabilities` and they are also defined in the `VideoStreamingParamerters` you provided, the smaller bitrate or preferred frame rate will be used.

### Video Framerate
Starting with SDL v5.1, the video stream manager works behind the scene to create a consistent video stream that matches the framerate set in `VideoStreamingParameters`. This is the now the default behavior but you have the option to revert to the old behavior by setting the `stableFrameRate` flag to `false` in the `VideoStreamingParameters`.

```java
VideoStreamingParameters params = new VideoStreamingParameters();
//Turn on use of stable frame rate
params.setStableFrameRate(true);
//Set the frame rate that you would wish to stream at
params.setFrameRate(30);
```

### Supporting Different Video Streaming Window Sizes (RPC v7.1+)
Some HMIs support multiple view sizes and may resize your SDL app's view during video streaming (i.e. to a collapsed view, split screen, preview mode or picture-in-picture). By default, your app will support all the view sizes and the `VideoStreamManager` will resize the video stream when the HMI notifies the app of the updated screen size. 
If you wish to support only some screen sizes, you can configure the two `VideoStreamingRange` parameters when starting your video stream using the `startRemoteDisplay` method. One range is for landscape orientations and one range is for portrait orientations.
In these `VideoStreamingRange` parameters you can define different view sizes that you wish to support in the event that the HMI resizes the view during the stream.
In the `VideoStreamingRange` you will define a minimum and maximum resolution, minimum diagonal, and a minimum and maximum aspect ratio. Any values you do not wish to use should be set to `null`.
If you want to support all possible landscape or portrait sizes you can simply pass `null` for `supportedLandscapeStreamingRange`, `supportedPortraitStreamingRange`, or both.
If you wish to only support landscape orientation or only support portrait orientation you "disable" the range by passing a `VideoStreamingRange` with all 0 values set.

```java
/*
 This VideoStreamingRange represents a disabled range
 It can be passed if you do not wish to support landscape orientation or portrait orientation
*/
final VideoStreamingRange disabledRange = new VideoStreamingRange(
    new Resolution(0, 0), 
    new Resolution(0, 0), 
    0.0, 
    0.0, 
    0.0
);

/*
 This VideoStreamingRange represents that we will support any resolution between 500x200 and 800x400
 In this case, it does not directly restrict the diagonal size or aspect ratio
*/
final VideoStreamingRange landscapeRange = new VideoStreamingRange(new Resolution(500, 200),
    new Resolution(800, 400), null, null, null);
/*
 This VideoStreamingRange represents that we will support any aspect ratio between 1.0 and 2.5 
 In this case, the resolution and diagonal size are not directly restricted
*/
final VideoStreamingRange portraitRange = new VideoStreamingRange(null, null, null, 1.0, 2.5);

if (sdlManager.getVideoStreamManager() != null) {
    sdlManager.getVideoStreamManager().start(new CompletionListener() {
        @Override
        public void onComplete(boolean success) {
            if (success) {
                sdlManager.getVideoStreamManager().startRemoteDisplayStream(
                    getApplicationContext(),
                    MyDisplay.class,
                    null,
                    false,
                    landscapeRange,
                    portraitRange
                );
            } else {
                DebugTool.logError(TAG, "Failed to start video streaming manager");
            }
        }
    });
}
```

!!! NOTE
If you disable both the `supportedLandscapeStreamingRange` and `supportedPortraitStreamingRange`, video will not stream.
!!!

If the HMI resizes the view during the stream, the video stream will automatically restart with the new size and the `onViewResized` method you defined in your presentation class will be notified of the new screen size.
```java
public static class MyDisplay extends SdlRemoteDisplay{
    public MyDisplay(Context context, Display display) {
        super(context, display);
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        //...
    }

    //onViewResized is added in SDL v5.1+
    @Override
    public void onViewResized(int width, int height) {
        DebugTool.logInfo(TAG, "Remote view new width and height ("+ width + ", " + height + ")");
        //Update presentation based on new resolution
    }
}   
```
