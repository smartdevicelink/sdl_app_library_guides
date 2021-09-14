# Supporting Haptic Input (RPC v4.5+)
SDL now supports "haptic" input: input from something other than a touch screen. This could include trackpads, click-wheels, etc. These kinds of inputs work by knowing which views on the screen are touchable and focusing / highlighting on those areas when the user moves the trackpad or click wheel. When the user selects within a view, the center of that area will be "touched".

!!! NOTE
Currently, there are no RPCs for knowing which view is highlighted, so your UI will have to remain static (i.e. you should not create a scrolling menu in your SDL app).
!!!

@![iOS]
You will also need to implement [touch input support](Video Streaming for Navigation Apps/Touch Input) in order to receive touches on the views. In addition, you must support the automatic focusable item manager in order to receive a touched `UIView` in the `SDLTouchManagerDelegate` in addition to the `CGPoint`.
!@

## Automatic Focusable Rectangles
SDL has support for automatically detecting focusable views within your UI and sending that data to the head unit. You will still need to tell SDL when your UI changes so that it can re-scan and detect the views to be sent.

@![iOS]
In order to use the automatic focusable item locator, you must set the `UIWindow` of your streaming content on `SDLStreamingMediaConfiguration.window`. So long as the window is set, the focusable item locator will start running. Whenever your app UI updates, you will need to send a notification:

|~
```objc
[[NSNotificationCenter defaultCenter] postNotificationName:SDLDidUpdateProjectionView object:nil];
```
```swift
NotificationCenter.default.post(name: SDLDidUpdateProjectionView, object: nil)
```
~|

!!! NOTE
When your renderingType is `SDLCarWindowRenderingTypeLayer`, the `SDLDidUpdateProjectionView` notification should only be sent in the overridden `viewDidLayoutSubviews` method of your `rootViewController`. If you do not, your haptic rects may not update as you expect.

SDL can only automatically detect `UIButton`s and anything else that responds `true` to `canBecomeFocused`. This means that custom `UIView` objects will *not* be found. You must send these objects manually, see "Manual Focusable Rects".

Before Xcode 12.5, some built-in `UIView` subclasses, such as `UITextField`, responded `true` to `canBecomeFocused`. That is not longer true, and you must subclass these built-in views and implement `canBecomeFocused` to return `true`.
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

                return false;
            }
        });
    }
}
```


This will go through your view that was passed in and then find and send the rects to the head unit for use. When your UI changes, call `invalidate()` from your class that extends `SdlRemoteDisplay`.
!@

## Manual Focusable Rects
@![iOS]
If you need to supplement the automatic focusable item locator, or do all of the location yourself (e.g. views that are not focusable such as custom UIViews or OpenGL views), then you will have to manually send and update the focusable rects using `SDLSendHapticData`. This request, when sent replaces all current rects with new rects; so, if you want to clear all of the rects, you would send the RPC with an empty array. Or, if you want to add a single rect, you must re-send all previous rects in the same request.

Usage is simple, you create the rects using `SDLHapticRect`, add a unique id, and send all the rects using `SDLSendHapticData`.

|~
```objc
SDLRectange *viewRect = [[SDLRectangle alloc] initWithCGRect:view.bounds];
SDLHapticRect *hapticRect = [[SDLHapticRect alloc] initWithId:1 rect:viewRect];
SDLSendHapticData *hapticData = [[SDLSendHapticData alloc] initWithHapticRectData:@[hapticRect]];

[self.sdlManager sendRequest:hapticData];
```
```swift
guard let viewRect = SDLRectangle(cgRect: view.bounds) else { return }
let hapticRect = SDLHapticRect(id: 1, rect: viewRect)
let hapticData = SDLSendHapticData(hapticRectData: [hapticRect])

self.sdlManager.send(hapticData)
```
~|
!@

@![android]
It is also possible that you may want to create your own rects instead of using the automated methods in the Presentation class. It is important that if sending this data yourself that you also use the `SystemCapabilityManager` to check if you are on a head unit that supports this feature. If the capability is available, it is easy to build the area you want to become selectable:

```java
public void sendHapticData() {
	Rectangle rectangle = new Rectangle()
	    .setX((float) 1.0)
	    .setY((float) 1.0)
	    .setWidth((float) 1.0)
	    .setHeight((float) 1.0);

	HapticRect hapticRect = new HapticRect()
	    .setId(123)
	    .setRect(rectangle);

	ArrayList<HapticRect> hapticArray = new ArrayList<HapticRect>();
	hapticArray.add(0, hapticRect);

	SendHapticData sendHapticData = new SendHapticData();
	sendHapticData.setHapticRectData(hapticArray);

	sdlManager.sendRPC(sendHapticData);
}
```
Each `SendHapticData` RPC should contain the entirety of all clickable areas to be accessed via haptic controls.
!@
