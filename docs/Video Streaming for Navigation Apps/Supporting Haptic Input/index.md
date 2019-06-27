# Supporting Haptic Input
SDL now supports "haptic" input: input from something other than a touch screen. This could include trackpads, click-wheels, etc. These kinds of inputs work by knowing which views on the screen are touchable and focusing / highlighting on those areas when the user moves the trackpad or click wheel. When the user selects within a view, the center of that area will be "touched".

!!! NOTE
Currently, there are no RPCs for knowing which view is highlighted, so your UI will have to remain static (i.e. you should not create a scrolling menu in your SDL app).
!!!

You will also need to implement [touch input support](Video Streaming for Navigation Apps/Touch Input) in order to receive touches on the views. @![iOS]In addition, you must support the automatic focusable item manager in order to receive a touched `UIView` in the `SDLTouchManagerDelegate` in addition to the `CGPoint`.!@

## Automatic Focusable Rects
SDL has support for automatically detecting focusable views within your UI and sending that data to the head unit. You will still need to tell SDL when your UI changes so that it can re-scan and detect the views to be sent.

@![iOS]
!!! IMPORTANT
This is only supported on iOS 9 devices and above. If you want to support this feature on iOS 8, see "Manual Focusable Rects" below.
!!!

In order to use the automatic focusable item locator, you must set the `UIWindow` of your streaming content on `SDLStreamingMediaConfiguration.window`. So long as the device is on iOS 9+ and the window is set, the focusable item locator will start running. Whenever your app UI updates, you will need to send a notification:

##### Objective-C
```objc
[[NSNotificationCenter defaultCenter] postNotificationName:SDLDidUpdateProjectionView object:nil];
```

##### Swift
```swift
NotificationCenter.default.post(name: SDLDidUpdateProjectionView, object: nil)
```

!!! NOTE
SDL can only automatically detect `UIButton`s and anything else that responds `true` to `canBecomeFocused`. This means that custom `UIView` objects will *not* be found. You must send these objects manually, see "Manual Focusable Rects".
!!!
!@

@![android]
The easiest way to use this is by taking advantage of SDL's Presentation class. This will automatically check if the capability is available and instantiate the manager for you. All you have to do is set your layout:

```java
public static class MyPresentation extends SdlRemoteDisplay {

    public MyPresentation(Context context, Display display) {
        super(context, display);
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.haptic_layout);
        LinearLayout videoView = (LinearLayout) findViewById(R.id.cat_view);
        videoView.setOnTouchListener(new View.OnTouchListener() {
            @Override
            public boolean onTouch(View view, MotionEvent motionEvent) {
                // ...Update something on the ui

                MyPresentation.this.invalidate();
            }
        });
    }
}
```

This will go through your view that was passed in and then find and send the rects to the head unit for use. When your UI changes, call `invalidate()` from your class that extends `SdlRemoteDisplay`.
!@

## Manual Focusable Rects
@![iOS]
If you need to supplement the automatic focusable item locator, or do all of the location yourself (e.g. devices lower than iOS 9, or views that are not focusable such as custom UIViews or OpenGL views), then you will have to manually send and update the focusable rects using `SDLSendHapticData`. This request, when sent replaces all current rects with new rects; so, if you want to clear all of the rects, you would send the RPC with an empty array. Or, if you want to add a single rect, you must re-send all previous rects in the same request.

Usage is simple, you create the rects using `SDLHapticRect`, add a unique id, and send all the rects using `SDLSendHapticData`.

##### Objective-C
```objc
SDLRectange *viewRect = [[SDLRectangle alloc] initWithCGRect:view.bounds];
SDLHapticRect *hapticRect = [[SDLHapticRect alloc] initWithId:1 rect:viewRect];
SDLSendHapticData *hapticData = [[SDLSendHapticData alloc] initWithHapticRectData:@[hapticRect]];

[self.sdlManager.sendRequest:hapticData];
```

##### Swift
```swift
guard let viewRect = SDLRectangle(cgRect: view.bounds) else { return }
let hapticRect = SDLHapticRect(id: 1, rect: viewRect)
let hapticData = SDLSendHapticData(hapticRectData: [hapticRect])

self.sdlManager.send(hapticData)
```
!@

@![android]
It is also possible that you may want to create your own rects instead of using the automated methods in the Presentation class. It is important that if sending this data yourself that you also use the `SystemCapabilityManager` to check if you are on a head unit that supports this feature. If the capability is available, it is easy to build the area you want to become selectable:

```java
public void sendHapticData() {

	Rectangle rectangle = new Rectangle();
	rectangle.setX((float) 1.0);
	rectangle.setY((float) 1.0);
	rectangle.setWidth((float) 1.0);
	rectangle.setHeight((float) 1.0);

	HapticRect hapticRect = new HapticRect();
	hapticRect.setId(123);
	hapticRect.setRect(rec);

	ArrayList<HapticRect> hapticArray = new ArrayList<HapticRect>();
	hapticArray.add(0, hr);

	SendHapticData sendHapticData = new SendHapticData();
	sendHapticData.setHapticRectData(hapticArray);

	sdlManager.sendRPC(sendHapticData);

}
```
Each SendHapticData rpc should contain the entirety of all clickable areas to be accessed via haptic controls.
!@
