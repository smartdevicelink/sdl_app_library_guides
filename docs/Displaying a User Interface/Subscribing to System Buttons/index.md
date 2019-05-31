# Subscribing to System Buttons
Subscribe buttons are used to detect changes to hard buttons located in the car's center console or steering wheel. You can subscribe to the following hard buttons:

| Button  | Template | Button Type |
| ------------- | ------------- | ------------- |
| Play / Pause | media template only | soft button and hard button |
| Ok | any template | soft button and hard button |
| Seek left | media template only | soft button and hard button |
| Seek right | media template only | soft button and hard button |
| Tune up | media template only | hard button |
| Tune down | media template only | hard button |
| Preset 0-9 | any template | hard button |
| Search | any template | hard button |

!!! NOTE
There is no way to customize a subscribe button's image or text.
!!!

## Audio-Related Buttons
The play/pause, seek left, seek right, tune up, and tune down subscribe buttons can only be used in the `MEDIA` template. Depending on the manufacturer of the head unit, the subscribe button might also show up as a soft button in the media template. For example, the SYNC 3 HMI will add the ok, seek right, and seek left soft buttons to the media template when you subscribe to those buttons. You will automatically be assigned the media template if you set your app's configuration `appType` to `MEDIA`.

!!! NOTE
Before iOS library v6.1 and SDL Core 5.0, `Ok` and `PlayPause` were combined into `Ok`. Subscribing to `Ok` will, in v6.1, also subscribe you to `PlayPause`. This means that for the time being, *you should not simultaneously subscribe to `Ok` and `PlayPause`*. In a future major version, this will change. For now, only subscribe to either `Ok` or `PlayPause` and the library will do the right thing based on the version of Core to which you are subscribed.
!!!

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

