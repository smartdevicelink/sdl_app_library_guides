# Subscribing to System Buttons
Subscribe buttons are used to detect changes to hard buttons located in the car's center console or steering wheel. You can subscribe to the following hard buttons:

| Button  | Template | Button Type |
| ------------- | ------------- | ------------- |
| Play / Pause | media only | soft button and hard button |
| Ok | media only | soft button and hard button |
| Seek left | media only | soft button and hard button |
| Seek right | media only | soft button and hard button |
| Tune up | media only | hard button |
| Tune down | media only | hard button |
| Preset 0-9 | any | hard button |
| Search | any | hard button |
| Center Location | NavigationFullscreenMap only | soft and hard button |
| Zoom In | NavigationFullscreenMap only | soft and hard button |
| Zoom Out | NavigationFullscreenMap only | soft and hard button |
| Pan Up | NavigationFullscreenMap only | soft and hard button |
| Pan Up-Right | NavigationFullscreenMap only | soft and hard button |
| Pan Right | NavigationFullscreenMap only | soft and hard button |
| Pan Down-Right | NavigationFullscreenMap only | soft and hard button |
| Pan Down | NavigationFullscreenMap only | soft and hard button |
| Pan Down-Left | NavigationFullscreenMap only | soft and hard button |
| Pan Left | NavigationFullscreenMap only | soft and hard button |
| Pan Up-Left | NavigationFullscreenMap only | soft and hard button |
| Toggle Tilt | NavigationFullscreenMap only | soft and hard button |
| Rotate Clockwise | NavigationFullscreenMap only | soft and hard button |
| Rotate Counter-Clockwise | NavigationFullscreenMap only | soft and hard button |
| Toggle Heading | NavigationFullscreenMap only | soft and hard button |

### Subscribe Buttons HMI
In the screenshot below, the pause, seek left and seek right icons are subscribe buttons.
![Generic - Media Template with subscribe buttons](assets/Generic_template_media_light.png)

!!! NOTE
There is no way to customize a subscribe button's image or text.
!!!

## Audio-Related Buttons
The play/pause, seek left, seek right, tune up, and tune down subscribe buttons can only be used in the `MEDIA` template. Depending on the manufacturer of the head unit, the subscribe button might also show up as a soft button in the media template. For example, the SYNC 3 HMI will add the ok, seek right, and seek left soft buttons to the media template when you subscribe to those buttons. You will automatically be assigned the media template if you set your app's `appType` to `MEDIA`.

!!! NOTE
Before library v.@![iOS]6.1!@@![android, javaSE, javaEE]4.7!@ and SDL Core v.5.0, `Ok` and `PlayPause` were combined into `Ok`. Subscribing to `Ok` will, in v.@![iOS]6.1!@@![android, javaSE, javaEE]4.7!@, also subscribe you to `PlayPause`. This means that for the time being, *you should not simultaneously subscribe to `Ok` and `PlayPause`*. In a future major version, this will change. For now, only subscribe to either `Ok` or `PlayPause` and the library will execute the right action based on the version of Core to which you are connected.
!!!

@![iOS]
##### Objective-C
```objc
SDLSubscribeButton *subscribeButton = [[SDLSubscribeButton alloc] initWithButtonName:SDLButtonNamePlayPause handler:^(SDLOnButtonPress * _Nullable buttonPress, SDLOnButtonEvent * _Nullable buttonEvent) {
    <#subscribe button selected#>
}];
[manager sendRequest:subscribeButton withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if (error != nil) { return; }
    <#subscribe button sent successfully#>
}];
```

##### Swift
```swift
let subscribeButton = SDLSubscribeButton(buttonName: .ok) { (buttonPress, buttonEvent) in
    <#subscribe button selected#>
}
sdlManager.send(request: subscribeButton) { (request, response, error) in
    guard error == nil else { return }
    <#subscribe button sent successfully#>
}
```
!@

@![android, javaSE, javaEE]
```java
sdlManager.addOnRPCNotificationListener(FunctionID.ON_BUTTON_EVENT, new OnRPCNotificationListener() {
    @Override
    public void onNotified(RPCNotification notification) {
        OnButtonPress onButtonPressNotification = (OnButtonPress) notification;
        switch (onButtonPressNotification.getButtonName()) {
            case OK:
                break;
            case PLAY_PAUSE:
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
            case PLAY_PAUSE:
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
subscribeButtonRequest.setButtonName(ButtonName.OK);
sdlManager.sendRPC(subscribeButtonRequest);
```
!@