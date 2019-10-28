# Video Streaming
@![iOS]
To stream video from a SDL app use the `SDLStreamingMediaManager` class. A reference to this class is available from the `SDLManager`. You can choose to create your own video streaming manager or you can use the `CarWindow` API to easily stream video to the head unit.

!!! NOTE
Due to an iOS limitation, video can not be streamed when the app on the phone is backgrounded or when the phone is sleeping/locked. Text will automatically be displayed telling the user that they must bring the application to the foreground.
!!!

### Transports for Video Streaming
Transports are automatically handled for you. As of SDL v6.1, the iOS library will automatically manage primary transports and secondary transports for video streaming. If Wi-Fi is available, the app will automatically connect using it *after* connecting over USB / Bluetooth. This is the only way that Wi-Fi will be used in a production setting.

## CarWindow
`CarWindow` is a system to automatically video stream a view controller screen to the head unit. When you set the view controller, `CarWindow` will resize the view controller's frame to match the head unit's screen dimensions. Then, when the video service setup has completed, it will capture the screen and and send it to the head unit.

To start, you will have to set a `rootViewController`, which can easily be set using one of the convenience initializers: `autostreamingInsecureConfigurationWithInitialViewController:` or `autostreamingSecureConfigurationWithSecurityManagers:initialViewController:`

!!! MUST
The view controller you are streaming must be a subclass of `SDLCarWindowViewController` or have only one `supportedInterfaceOrientation`. The `SDLCarWindowViewController` class prevents the `rootViewController` from rotating. This is necessary because rotation between landscape and portrait modes can cause the app to crash while the `CarWindow` API is capturing an image.
!!!

There are several customizations you can make to `CarWindow` to optimize it for your video streaming needs:

1. Choose how `CarWindow` captures and renders the screen using the `carWindowRenderingType` enum.
2. By default, when using `CarWindow`, the `SDLTouchManager` will sync it's touch updates to the framerate. To disable this feature, set `SDLTouchManager.enableSyncedPanning` to `NO`.
3. `CarWindow` hard-dictates the framerate of the app. To change the framerate and other parameters, update `SDLStreamingMediaConfiguration.customVideoEncoderSettings`.

    Below are the video encoder defaults:

        @{
            (__bridge NSString *)kVTCompressionPropertyKey_ProfileLevel: (__bridge NSString *)kVTProfileLevel_H264_Baseline_AutoLevel,
            (__bridge NSString *)kVTCompressionPropertyKey_RealTime: @YES,
            (__bridge NSString *)kVTCompressionPropertyKey_ExpectedFrameRate: @15,
            (__bridge NSString *)kVTCompressionPropertyKey_AverageBitRate: @600000
        };


### Showing a New View Controller
Simply update `sdlManager.streamManager.rootViewController` to the new view controller. This will also update the [haptic parser](Video Streaming for Navigation Apps/Supporting Haptic Input).

### Mirroring the Device Screen vs. Off-Screen UI
It is recommended that you use an off-screen view controller for your UI. This view controller will appear on-screen in the car, while remaining off-screen on the device. It is possible to mirror your device screen, however we strongly recommend against this course of action.

#### Off-Screen
To set an off-screen view controller all you have to do is instantiate a new `UIViewController` class and use it to set the `rootViewController`. 

##### Objective-C
```objc
UIViewController *offScreenViewController = <#Acquire a UIViewController#>;
self.sdlManager.streamManager.rootViewController = offScreenViewController;
```

##### Swift
```swift
let offScreenViewController = <#Acquire a UIViewController#>
sdlManager.streamManager?.rootViewController = offScreenViewController
```

#### Mirroring the Device Screen
If you must use mirroring to stream video please be aware of the following limitations:

1. Getting the app's topmost view controller using `UIApplication.shared.keyWindow.rootViewController` will not work as this will give you SDL's lock screen view controller. The projected image you see in the car will be distorted because the view controller being projected will not be resized correctly. Instead, the `rootViewController` should be set in the `viewDidAppear:animated` method of the `UIViewController`.
1. If mirroring your device's screen, the `rootViewController` should only be set after `viewDidAppear:animated` is called. Setting the `rootViewController` in `viewDidLoad` or `viewWillAppear:animated` can cause weird behavior when setting the new frame.
1. If setting the `rootViewController` when the app returns to the foreground, the app should register for the `UIApplicationDidBecomeActive` notification and not the `UIApplicationWillEnterForeground` notification. Setting the frame after a notification from the latter can also cause weird behavior when setting the new frame.
1. Configure your SDL app so the lock screen is [always visible](Getting Started/Adding the Lock Screen). If you do not do this, video streaming can stop when the device is rotated.

### Showing a New View Controller
Simply update the streaming media manager's `rootViewController` to the new view controller. This will also automatically update the [haptic parser](Video Streaming for Navigation Apps/Supporting Haptic Input).

## Sending Raw Video Data
If you decide to send raw video data instead of relying on the `CarWindow` API to generate that video data from a view controller, you must maintain the lifecycle of the video stream as there are limitations to when video is allowed to stream. The app's HMI state on the head unit and the app's application state on the device determines whether video can stream. Due to an iOS limitation, video cannot be streamed when the app on the device is no longer in the foreground and/or the device is locked/sleeping.

The lifecycle of the video stream is maintained by the SDL library. The `SDLManager.streamingMediaManager` can be accessed once the `start` method of `SDLManager` is called. The `SDLStreamingMediaManager` automatically takes care of determining screen size and encoding to the correct video format.

!!! NOTE
It is not recommended to alter the default video format and resolution behavior as it can result in distorted video or the video not showing up at all on the head unit. However, that option is available to you by implementing `SDLStreamingMediaConfiguration.dataSource`.
!!!

### Sending Video Data
To check whether or not you can start sending data to the video stream, watch for the `SDLVideoStreamDidStartNotification`, `SDLVideoStreamDidStopNotification`, and `SDLVideoStreamSuspendedNotification` notifications. When you receive the start notification, start sending video data; stop when you receive the suspended or stop notifications. You will receive a video stream suspended notification when the app on the device is backgrounded. There are parallel start and stop notifications for audio streaming.

Video data must be provided to the `SDLStreamingMediaManager` as a `CVImageBufferRef` (Apple documentation [here](https://developer.apple.com/library/mac/documentation/QuartzCore/Reference/CVImageBufferRef/)). Once the video stream has started, you will not see video appear until Core has received a few frames. Refer to the code sample below for an example of how to send a video frame:

##### Objective-C
```objective-c
CVPixelBufferRef imageBuffer = <#Acquire Image Buffer#>;

if ([self.sdlManager.streamManager sendVideoData:imageBuffer] == NO) {
  NSLog(@"Could not send Video Data");
}
```

##### Swift
```swift
let imageBuffer = <#Acquire Image Buffer#>

guard let streamManager = self.sdlManager.streamManager, !streamManager.isVideoStreamingPaused else {
    return
}

if !streamManager.sendVideoData(imageBuffer) {
    print("Could not send Video Data")
}
```

### Best Practices
* A constant stream of map frames is not necessary to maintain an image on the screen. Because of this, we advise that a batch of frames are only sent on map movement or location movement. This will keep the application's memory consumption lower.
* For the best user experience, we recommend sending at least 15 frames per second.
!@

@![android]
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
                Log.i(TAG, "Button Clicked");
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
                Log.e(TAG, "Failed to start video streaming manager");
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
!@


### Handling HMI Scaling (RPC v6.0+)
If the HMI scales the video stream, you will have to handle scaling the projected view, touches and haptic rectangles yourself (this is all handled for you behind the scenes in the @![iOS]`CarWindow`!@ @![android]`SdlRemoteDisplay`!@  API). To find out if the HMI scales the video stream, you must for query and check the @![iOS]`SDLVideoStreamingCapability`!@ @![android]`VideoStreamingCapability`!@ for the `scale` property. Please check the [Adaptive Interface Capabilities](Displaying a User Interface/Adaptive Interface Capabilities) section for more information on how to query for this property using the system capability manager.  