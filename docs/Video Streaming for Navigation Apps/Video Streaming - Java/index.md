# Video Streaming
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
}
```

!!! Note
If you are obfuscating the code in your app, make sure to exclude your class that extends `SdlRemoteDisplay`. For more information on how to do that, you can check [Proguard Guidelines](Getting Started/Proguard Guidelines).
!!!

## Managing the Stream
The `VideoStreamingManager` can be used to start streaming video after the `SdlManager` has successfully been started. This is performed by calling the method `startRemoteDisplayStream(Context context, final Class<? extends SdlRemoteDisplay> remoteDisplay, final VideoStreamingParameters parameters, final boolean encrypted)`.

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
}
```

```java
if (sdlManager.getVideoStreamManager() != null) {
    sdlManager.getVideoStreamManager().start(new CompletionListener() {
        @Override
        public void onComplete(boolean success) {
            if (success) {
                sdlManager.getVideoStreamManager().startRemoteDisplayStream(getApplicationContext(), MyDisplay.class, null, false);
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
If the HMI scales the video stream, you will have to handle scaling the projected view, touches and haptic rectangles yourself (this is all handled for you behind the scenes in the `VideoStreamManager`  API). To find out if the HMI scales the video stream, you must for query and check the `VideoStreamingCapability` for the `scale` property. Please check the [Adaptive Interface Capabilities](Displaying a User Interface/Adaptive Interface Capabilities) section for more information on how to query for this property using the system capability manager.  