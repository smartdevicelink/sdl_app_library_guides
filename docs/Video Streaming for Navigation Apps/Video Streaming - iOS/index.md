# Video Streaming (RPC v4.5+)
To stream video from a SDL app use the `SDLStreamingMediaManager` class. A reference to this class is available from the `SDLManager`. You can choose to create your own video streaming manager or you can use the `CarWindow` API to easily stream video to the head unit.

!!! NOTE
Due to an iOS limitation, video can not be streamed when the app on the phone is in the background or the screen is off. Text will automatically be displayed telling the user that they must bring the application to the foreground. This text can be disabled by setting the `SDLStreamingMediaManager`'s  `showVideoBackgroundDisplay` property to `false`.
!!!

### Transports for Video Streaming
Transports are automatically handled for you. As of SDL v6.1+, the iOS library will automatically manage primary transports and secondary transports for video streaming. If Wi-Fi is available, the app will automatically connect using it *after* connecting over USB / Bluetooth. This is the only way that Wi-Fi will be used in a production setting.

## CarWindow
`CarWindow` is a system to automatically video stream a view controller screen to the head unit. When you set the view controller, `CarWindow` will resize the view controller's frame to match the head unit's screen dimensions. Then, when the video service setup has completed, it will capture the screen and send it to the head unit.

To start, you will have to set a `rootViewController`, which can easily be set using one of the convenience initializers: `autostreamingInsecureConfigurationWithInitialViewController:` or `autostreamingSecureConfigurationWithSecurityManagers:initialViewController:`

!!! MUST
The view controller you are streaming must be a subclass of `SDLCarWindowViewController` or have only one `supportedInterfaceOrientation`. The `SDLCarWindowViewController` class prevents the `rootViewController` from rotating. This is necessary because rotation between landscape and portrait modes can cause the app to crash while the `CarWindow` API is capturing an image.
!!!

There are several customizations you can make to `CarWindow` to optimize it for your video streaming needs:

1. Choose how `CarWindow` captures and renders the screen using the `carWindowRenderingType` enum.
2. By default, when using `CarWindow`, the `SDLTouchManager` will sync its touch updates to the framerate. To disable this feature, set `SDLTouchManager.enableSyncedPanning` to `NO`.
3. As of SDL v7.1, if the HMI returns a desired framerate or max bitrate, the HMI's preferred settings will be use to configure the video encoder. You do have the option to change the default framerate and average bitrate via the `SDLStreamingMediaConfiguration.customVideoEncoderSettings`. Please note that your custom settings will override any settings received from the HMI except in the case where your custom framerate or average bitrate is larger than what the HMI says it can support.

Below are the video encoder defaults:

```objc
 @{
    __bridge NSString *)kVTCompressionPropertyKey_ProfileLevel: (__bridge NSString *)kVTProfileLevel_H264_Baseline_AutoLevel,
    (__bridge NSString *)kVTCompressionPropertyKey_RealTime: @YES,
    (__bridge NSString *)kVTCompressionPropertyKey_ExpectedFrameRate: @15,
     __bridge NSString *)kVTCompressionPropertyKey_AverageBitRate: @600000
};
```

### Showing a New View Controller
Simply update `sdlManager.streamManager.rootViewController` to the new view controller. This will also update the [haptic parser](Video Streaming for Navigation Apps/Supporting Haptic Input).

### Mirroring the Device Screen vs. Off-Screen UI
It is recommended that you use an off-screen view controller for your UI. This view controller will appear on-screen in the car, while remaining off-screen on the device. It is possible to mirror your device screen, however we strongly recommend against this course of action.

!!! NOTE
If you are using off-screen rendering, it is recommended that your on-screen view controller not rotate. If it does, the lock screen will also rotate. Nothing will break in this case, but the UI won't look good if it rotates while your app is streaming.
!!!

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


### Supporting Different Video Streaming View Sizes (SDL v7.1+, RPC v7.1+)
Some HMIs support multiple view sizes and may resize your SDL app's view during video streaming (i.e. to a collapsed view, split screen, preview mode or picture-in-picture). By default, your app will support all the view sizes and the `CarWindow` will resize the view controller's frame when the HMI notifies the app of the updated screen size. If you you wish to support only some screen sizes, you can configure the `supportedPortraitStreamingRange` and `supportedLandscapeStreamingRange` properties via the `SDLStreamingMediaConfiguration` before starting the video stream. This will allow you to limit support to one or a combination of minimum/maximum resolutions, minimum diagonal, or minimum/maximum aspect ratios. If you want to support all possible landscape or portrait sizes you can simply set `nil` for the streaming range. If you wish to disable support for all possible landscape or portrait orientations you can disable the streaming range using the `SDLVideoStreamingRange.disabled` configuration.

#### Creating the Video Streaming Ranges
Below are some examples of how to configure a supported video streaming range:

##### Objective-C
```objc
// Use if you wish to disable support for all landscape orientations or all portrait orientations
SDLVideoStreamingRange *disabledStreamingRange = SDLVideoStreamingRange.disabled;

// Use if you wish to only support landscape image resolutions between widths: 500-800 and heights: 200-400. All aspect ratios and diagonal screen sizes will be supported.
SDLVideoStreamingRange *streamingRange = [[SDLVideoStreamingRange alloc] initWithMinimumResolution:[[SDLImageResolution alloc] initWithWidth:500 height:200] maximumResolution:[[SDLImageResolution alloc] initWithWidth:800 height:400]];

// Use if you wish to only support aspect ratios between 1.0 and 2.5. All image resolutions and diagonal screen sizes will be supported.
SDLVideoStreamingRange *streamingRange = [[SDLVideoStreamingRange alloc] init];
streamingRange.minimumAspectRatio = 1.0;
streamingRange.maximumAspectRatio = 2.5;
```

##### Swift
```swift
// Use if you wish to disable support for all landscape orientations or all portrait orientations
let disabledStreamingRange = SDLVideoStreamingRange.disabled()

// Use if you wish to only support landscape image resolutions between widths: 500-800 and heights: 200-400. All aspect ratios and diagonal screen sizes will be supported.
let streamingRange = SDLVideoStreamingRange(minimumResolution: SDLImageResolution(width: 500, height: 200), maximumResolution: SDLImageResolution(width: 800, height: 400))

// Use if you wish to only support aspect ratios between 1.0 and 2.5. All image resolutions and diagonal screen sizes will be supported.
let streamingRange = SDLVideoStreamingRange()
streamingRange.minimumAspectRatio = 1.0
streamingRange.maximumAspectRatio = 2.5
```

#### Setting the Video Streaming Ranges
Once you have configured a supported video streaming range, you can use it to set the `supportedPortraitStreamingRange` or `supportedLandscapeStreamingRange` properties when you are configuring the `SDLStreamingMediaConfiguration`.

##### Objective-C
```objc
streamingMediaConfig.supportedPortraitStreamingRange = disabledStreamingRange;
streamingMediaConfig.supportedLandscapeStreamingRange = streamingRange;
```

##### Swift
```swift
streamingMediaConfig.supportedPortraitStreamingRange = disabledStreamingRange
streamingMediaConfig.supportedLandscapeStreamingRange = streamingRange
```

!!! NOTE
If you disable both the `supportedLandscapeStreamingRange` and `supportedPortraitStreamingRange`, video will not stream.
!!!

#### Getting the Updated Screen Size
If the HMI resizes the view during streaming, the video stream will automatically restart with the new size. If desired, you can subscribe to screen size updates via the `SDLStreamingVideoDelegate`.

##### Objective-C
```objc
streamingMediaConfig.delegate = self;

- (void)videoStreamingSizeDidUpdate:(CGSize)displaySize {
    <#Use displaySize.width and displaySize.height#>
}
```

##### Swift
```swift
streamingMediaConfig.delegate = self

extension ProxyManager: SDLStreamingVideoDelegate {
    func videoStreamingSizeDidUpdate(toSize displaySize: CGSize) {
        <#Use displaySize.width and displaySize.height#>
    }
}
```

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
```objc
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

### Handling HMI Scaling (RPC v6.0+)
If the HMI scales the video stream, you will have to handle scaling the projected view, touches and haptic rectangles yourself (this is all handled for you behind the scenes in the `CarWindow` API). To find out if the HMI scales the video stream, you must for query and check the `SDLVideoStreamingCapability` for the `scale` property. Please check the [Adaptive Interface Capabilities](Displaying a User Interface/Adaptive Interface Capabilities) section for more information on how to query for this property using the system capability manager.  
