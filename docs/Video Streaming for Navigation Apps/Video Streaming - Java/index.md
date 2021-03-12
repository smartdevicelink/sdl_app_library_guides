# Video Streaming (RPC v4.5+)
In order to stream video from an SDL app, we only need to manage a few things. For the most part, the library will handle the majority of logic needed to perform video streaming.

## SDL Remote Display
The `SdlRemoteDisplay` base class provides the easiest way to start streaming using SDL. The `SdlRemoteDisplay` is extended from Android's `Presentation` class with modifications to work with other aspects of the SDL Android library.

!!! Note
It is recommended that you extend this as a local class within the service that has the `SdlManager` instance.
!!!

Extending this class gives developers a familiar, native experience to handling layouts and events on screen.

```java
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

    //onViewResized is added in SDL v5.1+
    @Override
    public void onViewResized(int width, int height) {
        DebugTool.logInfo(TAG, "Remote view new width and height ("+ width + ", " + height + ")");
    }
}
```

!!! Note
If you are obfuscating the code in your app, make sure to exclude your class that extends `SdlRemoteDisplay`. For more information on how to do that, you can check [Proguard Guidelines](Getting Started/Proguard Guidelines).
!!!

## Managing the Stream
The `VideoStreamingManager` can be used to start streaming video after the `SdlManager` has successfully been started. This is performed by calling the method `startRemoteDisplayStream(Context context, final Class<? extends SdlRemoteDisplay> remoteDisplay, final VideoStreamingParameters parameters, final boolean encrypted, VideoStreamingRange supportedLandscapeStreamingRange, VideoStreamingRange supportedPortraitStreamingRange)`.

```java
public static class MyDisplay extends SdlRemoteDisplay {

    public MyDisplay(Context context, Display display) {
        super(context, display);
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.stream);

        String videoUri = "android.resource://" + context.getPackageName() + "/" + R.raw.sdl;
        VideoView videoView = findViewById(R.id.videoView);
        videoView.setVideoURI(Uri.parse(videoUri));
        videoView.start();
    }
    
    //onViewResized is added in SDL v5.1+
    @Override
    public void onViewResized(int width, int height) {
        DebugTool.logInfo(TAG, "Remote view new width and height ("+ width + ", " + height + ")");
    }
}
```

```java
if (sdlManager.getVideoStreamManager() != null) {
    sdlManager.getVideoStreamManager().start(new CompletionListener() {
        @Override
        public void onComplete(boolean success) {
            if (success) {
                sdlManager.getVideoStreamManager().startRemoteDisplayStream(getApplicationContext(), MyDisplay.class, null, false, null, null);
            } else {
                DebugTool.logError(TAG, "Failed to start video streaming manager");
            }
        }
    });
}
```

### Ending the Stream
When the `HMIStatus` is back to `HMI_NONE` it is time to stop the stream. This is accomplished through a method `stopStreaming()`.

```java
Map<FunctionID, OnRPCNotificationListener> onRPCNotificationListenerMap = new HashMap<>();
onRPCNotificationListenerMap.put(FunctionID.ON_HMI_STATUS, new OnRPCNotificationListener() {
    @Override
    public void onNotified(RPCNotification notification) {
        OnHMIStatus status = (OnHMIStatus) notification;
		if (status != null && status.getHmiLevel() == HMILevel.HMI_NONE) {

			//Stop the stream
			if (sdlManager.getVideoStreamManager() != null && sdlManager.getVideoStreamManager().isStreaming()) {
				sdlManager.getVideoStreamManager().stopStreaming();
			}

		}
    }
});
builder.setRPCNotificationListeners(onRPCNotificationListenerMap);
```

### Handling HMI Scaling (RPC v6.0+)
If the HMI scales the video stream, you will have to handle scaling the projected view, touches and haptic rectangles yourself (this is all handled for you behind the scenes in the `VideoStreamManager` API). To find out if the HMI scales the video stream, you must for query and check the `VideoStreamingCapability` for the `scale` property. Please check the [Adaptive Interface Capabilities](Displaying a User Interface/Adaptive Interface Capabilities) section for more information on how to query for this property using the system capability manager.

### Video Streaming Parameters (SDL v5.1+)
Starting with SDL version 5.1+ the `VideoStreamingParameters` you provide will automatically be aligned with the `VideoStreamingCapabilities` provided by the HMI.
If the HMI provides the scale or resolution in the `VideoStreamingCapabilities` the video stream will use that scale or resolution. Otherwise, the scale or resolution you defined in the `VideoStreamingParameters` will be used.
If the HMI provides the bitrate or preferred frame rate in the `VideoStreamingCapabilities` and they are also defined in the `VideoStreamingParamerters` you provided, the smaller bitrate or preferred frame rate will be used.

### Video FrameRate (RPC v7.1+)
Starting with RPC version 7.1+ you can define a frame rate within your `VideoStreamingParameters` that you would like your video to stream at. You can also have the option to turn this capability on or off by using the `stableFrameRate` flag in the `VideoStreamingParameters`.

```java
VideoStreamingParameters params = new VideoStreamingParameters();
//Turn on use of stable frame rate
params.setStableFrameRate(true);
//Set the frame rate that you would wish to stream at
params.setFrameRate(30);
```

### Supporting Different Video Streaming Window Sizes (RPC v7.1+)
You can specify two `VideoStreamingRange` parameters when you want to start your video stream using the `startRemoteDisplay` method, one range will be for landscape orientation and one range will be for portrait orientation.
In these `VideoStreamingRange` parameters you can define different view sizes that you wish to support in the event that the HMI resizes the view during the stream. (i.e. to a collapsed view, split screen, preview mode or picture-in-picture)
In the `VideoStreamingRange` you will define a minimum and maximum Resolution, minimum diagonal, and a minimum and maximum aspect Ratio. Any values you do not wish to use should be set to `null`.
If you want to support all possible landscape or portrait sizes you can simply pass `null` for `supportedLandscapeStreamingRange`, `supportedPortraitStreamingRange`, or both.
If you wish to only support landscape orientation or only support portrait orientation you "disable" the range by passing a `VideoStreamingRange` with all 0 values set.

```java
//This VideoStreamingRange represents a disabled Range and can be passed if you do not wish to support landscape orientation or portrait orientation
final VideoStreamingRange disabledRange = new VideoStreamingRange(new Resolution(0, 0), new Resolution(0, 0), 0.0, 0.0, 0.0);

//This VideoStreamingRange represents that we will support any resolution between 500x200 and 800x400 no matter the diagonal size or aspect ratio
final VideoStreamingRange landscapeRange = new VideoStreamingRange(new Resolution(500, 200), new Resolution(800, 400), null, null, null);

//This VideoStreamingRange represents that we will support any aspect ratio between 1.0 and 2.5 no matter the resolution or diagonal size
final VideoStreamingRange portraitRange = new VideoStreamingRange(null, null, null, 1.0, 2.5);

if (sdlManager.getVideoStreamManager() != null) {
    sdlManager.getVideoStreamManager().start(new CompletionListener() {
        @Override
        public void onComplete(boolean success) {
            if (success) {
                sdlManager.getVideoStreamManager().startRemoteDisplayStream(getApplicationContext(), MyDisplay.class, null, false, landscapeRange, portraitRange);
            } else {
                DebugTool.logError(TAG, "Failed to start video streaming manager");
            }
        }
    });
}
```

!!! NOTE
If you disable both the `supportedLandscapeStreamingRange` and `supportedPortraitStreamingRange`, the video will not stream
!!!

If the HMI resizes the view during the stream, the video stream will automatically restart with the new size and the `onViewResized` method you defined in your presentation class.
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
