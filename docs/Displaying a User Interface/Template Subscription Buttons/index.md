# Template Subscription Buttons
This guide shows you how to subscribe and react to "subscription" buttons. Subscription buttons are used to detect when the user has interacted with buttons located in the car's center console or steering wheel. A subscription button may also show up as part of your template, however, the text and/or image used in the button is determined by the template and is (usually) not customizable. 

In the screenshot below, the pause, seek left and seek right icons are subscription buttons. Once subscribed to, for example, the seek left button, you will be notified when the user selects the seek left button on the HMI or when they select the seek left button on the car's center console and/or steering wheel. 

![Generic - Media Template with subscribe buttons](assets/Generic_template_media_light.png)

## Types of Subscription Buttons
There are three general types of subscriptions buttons: audio related buttons only used for media apps, navigation related buttons only used for navigation apps, and general buttons, like preset buttons and the OK button, that can be used with all apps. Please note that if your app type is not `MEDIA` or `NAVIGATION`, your attempt to subscribe to media-only or navigation-only buttons will be rejected.

| Button  | App Type | RPC Version |
| ------------- | ------------- | ------------- |
| Ok | All | v1.0+ |
| Preset 0-9 | All | v1.0+ |
| Search | All | v1.0+ |
| Play / Pause | Media only | v5.0+ |
| Seek left | Media only | v1.0+ |
| Seek right | Media only | v1.0+ |
| Tune up | Media only | v1.0+ |
| Tune down | Media only | v1.0+ |
| Center Location | Navigation only | v6.0+ |
| Zoom In | Navigation only | v6.0+ |
| Zoom Out | Navigation only | v6.0+ |
| Pan Up | Navigation only | v6.0+ |
| Pan Up-Right | Navigation only | v6.0+ |
| Pan Right | Navigation only | v6.0+ |
| Pan Down-Right | Navigation only | v6.0+ |
| Pan Down | Navigation only | v6.0+ |
| Pan Down-Left | Navigation only | v6.0+ |
| Pan Left | Navigation only | v6.0+ |
| Pan Up-Left | Navigation only | v6.0+ |
| Toggle Tilt | Navigation only | v6.0+ |
| Rotate Clockwise | Navigation only | v6.0+ |
| Rotate Counter-Clockwise | Navigation only | v6.0+ |
| Toggle Heading | Navigation only | v6.0+ |

@![iOS,android,javaSE,javaEE]
## Subscribing to Subscription Buttons
You can easily subscribe to subscription buttons using the !@@![iOS]`SDLScreenManager`!@@![android, javaSE, javaEE]`ScreenManager`!@@![iOS,android,javaSE,javaEE]. Simply tell the manager which button to subscribe and you will be notified when the user selects the button.
!@

@![iOS]
There are two different ways to receive button press notifications. The first is to pass a block handler that will get called when the button is selected. The second is to pass a selector that will be notified when the button is selected.

### Subscribe with a Block Handler
Once you have subscribed to the button with a block handler, the handler will be called whenever the button has been selected. If an error occurs attempting to subscribe to the button, the error will be returned in the `error` parameter.

##### Objective-C
```objc
NSObject *observer = [self.sdlManager.screenManager subscribeButton:SDLButtonNamePlayPause withUpdateHandler:^(SDLOnButtonPress * _Nullable buttonPress, SDLOnButtonEvent * _Nullable buttonEvent, NSError * _Nullable error) {
    if (error != nil) {
        // There was an error subscribing to the button
        return;
    }

    if (buttonPress != nil) {
        // Contains information about whether the button was short or long pressed
    }

    if (buttonEvent != nil) {
        // Contains information about when the button is depressed or released
    }
}];
```

##### Swift
```swift
let observer = sdlManager.screenManager.subscribeButton(.playPause) { (buttonPress, buttonEvent, error) in
    guard error == nil else {
        // There was an error subscribing to the button
        return
    }

    if let buttonPress = buttonPress {
        // Contains information about whether the button was short or long pressed
    }

    if let buttonPress = buttonPress {
        // Contains information about when the button is depressed or released
    }
}
```

### Subscribe with a Selector
Once you have subscribed to the button, the selector will be called when the button has been selected. If there is an error subscribing to the subscribe button it will be returned in the `error` parameter.

The selector can be created with between zero and four parameters of types in the following order: `SDLButtonName`, `NSError`, `SDLOnButtonPress`, and `SDLOnButtonEvent`. When the fourth parameter, `SDLOnButtonEvent`, is omitted from the selector, then you will only be notified when a button press occurs. When the third parameter, `SDLOnButtonPress` is omitted from the selector, you will be unable to distinguish between short and long button presses.

##### Objective-C
```objc
[self.sdlManager.screenManager subscribeButton:SDLButtonNamePlayPause withObserver:self selector:@selector(buttonPressEventWithButtonName:error:buttonPress:buttonEvent:)];

- (void)buttonPressEventWithButtonName:(SDLButtonName)buttonName error:(NSError *)error buttonPress:(SDLOnButtonPress *)buttonPress buttonEvent:(SDLOnButtonEvent *)buttonEvent {
    if (error != nil) {
        // There was an error subscribing to the button
        return;
    }

    if (buttonPress != nil) {
        // Contains information about whether the button was short or long pressed
    }

    if (buttonEvent != nil) {
        // Contains information about when the button is depressed or released
    }
}
```

##### Swift
```swift
sdlManager.screenManager.subscribeButton(.playPause, withObserver: self, selector: #selector(buttonPressEvent(buttonName:error:buttonPress:buttonEvent:)))

@objc private func buttonPressEvent(buttonName: SDLButtonName, error: Error?, buttonPress: SDLOnButtonPress?, buttonEvent: SDLOnButtonEvent?) {
    if let error = error {
        // There was an error subscribing to the button
        return
    }

    if let buttonPress = buttonPress {
        // Contains information about whether the button was short or long pressed
    }

    if let buttonPress = buttonPress {
        // Contains information about when the button is depressed or released
    }
}
```
!@

@![android,javaSE,javaEE]
### Subscribe with a Listener
Once you have subscribed to the button, the listener will be called when the button has been selected. If there is an error subscribing to the button the error message will be returned in the `error` parameter.

```java
OnButtonListener playPauseButtonListener = new OnButtonListener() {
    @Override
    public void onPress(ButtonName buttonName, OnButtonPress buttonPress) {

    }

    @Override
    public void onEvent(ButtonName buttonName, OnButtonEvent buttonEvent) {

    }

    @Override
    public void onError(String info) {

    }
};

sdlManager.getScreenManager().addButtonListener(ButtonName.PLAY_PAUSE, playPauseButtonListener);
```
!@

@![iOS,android,javaSE,javaEE]
## Unsubscribing from Subscription Buttons
!@

@![iOS]
When unsubscribing, you will need to pass the observer object and which button name that you want to unsubscribe. If you subscribed using a handler, use the observer object returned when you subscribed. If you subscribed using a selector, use the same observer object you passed when subscribing. 

##### Objective-C
```objc
[self.sdlManager.screenManager unsubscribeButton:SDLButtonNamePlayPause withObserver:<#Your observer object#> withCompletionHandler:^(NSError * _Nullable error) {
    if (error != nil) {
        // There was an error unsubscribing to the button
        return;
    }

    // The button was unsubscribed successfully
}];
```

##### Swift
```swift
sdlManager.screenManager.unsubscribeButton(.playPause, withObserver: <#Your observer object#>) { (error) in
    if let error = error {
        // There was an error unsubscribing to the button
        return
    }

    // The button was unsubscribed successfully
}
```
!@

@![android,javaSE,javaEE]
To unsubscribe to a subscription button, simply tell the `ScreenManager` which button name and listener object to unsubscribe.
```java
sdlManager.getScreenManager().removeButtonListener(ButtonName.PLAY_PAUSE, playPauseButtonListener);
```
!@

## Media Buttons
The play/pause, seek left, seek right, tune up, and tune down subscribe buttons can only be used if the app type is `MEDIA`. Depending on the OEM, the subscribed button could show up as an on-screen button in the `MEDIA` template, work as a physical button on the car console or steering wheel, or both. For example, Ford's SYNC 3 HMI will add the play/pause, seek right, and seek left soft buttons to the media template when you subscribe to those buttons. However, those buttons will also trigger when the user uses the seek left / seek right buttons on the steering wheel.

If desired, you can toggle the play/pause button image between a play, stop, or pause icon by updating the audio streaming state as described in the [Media Clock](Displaying a User Interface/Media Clock#pausing-resuming) guide. 

!!! NOTE
Before library v.@![iOS]6.1!@@![android, javaSE, javaEE]4.7!@ and RPC v5.0, `Ok` and `PlayPause` were combined into `Ok`. Subscribing to `Ok` will, in v@![iOS]6.1+!@@![android, javaSE, javaEE]4.7+!@, also subscribe you to `PlayPause`. This means that for the time being, *you should not simultaneously subscribe to `Ok` and `PlayPause`*. In a future major version, this will change. For now, only subscribe to either `Ok` or `PlayPause` and the library will execute the right action based on the connected head unit.
!!!

@![iOS]
##### Objective-C
```objc
[self.sdlManager.screenManager subscribeButton:SDLButtonNamePlayPause withUpdateHandler:^(SDLOnButtonPress * _Nullable buttonPress, SDLOnButtonEvent * _Nullable buttonEvent, NSError * _Nullable error) {
    if (error != nil) {
        // There was an error subscribing to the button
        return;
    }

    if (buttonPress == nil) { return; }

    if ([buttonPress.buttonPressMode isEqualToEnum:SDLButtonPressModeShort]) {
        // The user short pressed the button
    } else if ([buttonPress.buttonPressMode isEqualToEnum:SDLButtonPressModeLong]) {
        // The user long pressed the button
    }
}];
```

##### Swift
```swift
sdlManager.screenManager.subscribeButton(.playPause) { (buttonPress, buttonEvent, error) in
    if let error = error {
        // There was an error subscribing to the button
        return
    }

    guard let buttonPress = buttonPress else { return }

    switch buttonPress.buttonPressMode {
    case .short:
        <#The user short pressed the button#>
    case .long:
        <#The user long pressed the button#>
    default:
        <#code#>
    }
}
```
!@

@![android, javaSE, javaEE]
```java
sdlManager.getScreenManager().addButtonListener(ButtonName.PLAY_PAUSE, new OnButtonListener() {
    @Override
    public void onPress (ButtonName buttonName, OnButtonPress buttonPress) {
        switch (buttonPress.getButtonPressMode()) {
            case SHORT:
                // The user short pressed the button
            case LONG:
                // The user long pressed the button
        }
    }

    @Override
    public void onEvent (ButtonName buttonName, OnButtonEvent buttonEvent) { }

    @Override
    public void onError (String info) {
        // There was an error subscribing to the button
    }
});
```
!@

@![javascript]
```js
sdlManager.addRpcListener(SDL.rpc.enums.FunctionID.OnButtonPress, function (onButtonPress) {
    if (onButtonPress instanceof SDL.rpc.messages.OnButtonPress) {
        switch (onButtonPress.getButtonName()) {
            case SDL.rpc.enums.ButtonName.PLAY_PAUSE:
                // PLAY_PAUSE subscribe button selected
                break;
        }
    }
});

const subscribeButtonRequest = new SDL.rpc.messages.SubscribeButton();
subscribeButtonRequest.setButtonName(SDL.rpc.enums.ButtonName.PLAY_PAUSE);
// sdl_javascript_suite v1.1+
sdlManager.sendRpcResolve(subscribeButtonRequest);
// Pre sdl_javascript_suite v1.1
sdlManager.sendRpc(subscribeButtonRequest);
```
!@

## Preset Buttons
All app types can subscribe to preset buttons. Depending on the OEM, the preset buttons may be added to the template when subscription occurs. Preset buttons can also be physical buttons on the console that will notify the subscriber when selected. An OEM may support only template buttons or only hard buttons or they may support both template and hard buttons. The screenshot below shows how the Ford SYNC 3 HMI displays the preset buttons on the HMI. 

![Ford - Preset Soft Button Menu Button](assets/ford_sync_presetMenu.bmp)
![Ford - Preset Soft Buttons List](assets/ford_sync_presetOptions.png)

### Checking if Preset Buttons are Supported
You can check if a HMI supports subscribing to preset buttons, and if so, how many preset buttons are supported, by checking the system capability manager.

@![iOS]
##### Objective-C
```objc
NSInteger numberOfCustomPresetsAvailable = self.sdlManager.systemCapabilityManager.defaultMainWindowCapability.numCustomPresetsAvailable.integerValue;
```

##### Swift
```swift
let numberOfCustomPresetsAvailable = sdlManager.systemCapabilityManager.defaultMainWindowCapability?.numCustomPresetsAvailable?.intValue
```
!@

@![android,javaSE,javaEE]
```java
Integer numOfCustomPresetsAvailable = sdlManager.getSystemCapabilityManager().getDefaultMainWindowCapability().getNumCustomPresetsAvailable();
```
!@

@![javascript]
```js
const numOfCustomPresetsAvailable = sdlManager.getSystemCapabilityManager().getDefaultMainWindowCapability().getNumCustomPresetsAvailable();
```
!@

### Subscribing to Preset Buttons
@![iOS]
##### Objective-C
```objc
[self.sdlManager.screenManager subscribeButton:SDLButtonNamePreset1 withObserver:self selector:@selector(buttonPressEventWithButtonName:error:buttonPress:)];
[self.sdlManager.screenManager subscribeButton:SDLButtonNamePreset2 withObserver:self selector:@selector(buttonPressEventWithButtonName:error:buttonPress:)];

- (void)buttonPressEventWithButtonName:(SDLButtonName)buttonName error:(NSError *)error buttonPress:(SDLOnButtonPress *)buttonPress {
    if (error != nil) {
        // There was an error subscribing to the button
        return;
    }

    if ([buttonName isEqualToEnum:SDLButtonNamePreset1]) {
        // The user short or long pressed the preset 1 button
    } else if ([buttonName isEqualToEnum:SDLButtonNamePreset2]) {
        // The user short or long pressed the preset 2 button
    }
}
```

##### Swift
```swift
sdlManager.screenManager.subscribeButton(.preset1, withObserver: self, selector: #selector(buttonPressEvent(buttonName:error:buttonPress:)))
sdlManager.screenManager.subscribeButton(.preset2, withObserver: self, selector: #selector(buttonPressEvent(buttonName:error:buttonPress:)))

@objc private func buttonPressEvent(buttonName: SDLButtonName, error: Error?, buttonPress: SDLOnButtonPress?) {
    if let error = error {
        // There was an error subscribing to the button
        return
    }

    guard let buttonPress = buttonPress else { return }

    switch buttonName {
    case .preset1:
        <#The user short or long pressed the preset 1 button#>
    case .preset2:
        <#The user short or long pressed the preset 2 button#>
    default:
        <#The user pressed another preset button#>
    }
}
```
!@

@![android,javaSE,javaEE]
```java
OnButtonListener onButtonListener = new OnButtonListener() {
    @Override
    public void onPress(ButtonName buttonName, OnButtonPress buttonPress) {
        switch (buttonName) {
            case PRESET_1:
                // The user short or long pressed the preset 1 button
                break;
            case PRESET_2:
                // The user short or long pressed the preset 2 button
                break;
        }
    }

    @Override
    public void onEvent (ButtonName buttonName, OnButtonEvent buttonEvent) { }

    @Override
    public void onError (String info) {
        // There was an error subscribing to the button
    }
};

sdlManager.getScreenManager().addButtonListener(ButtonName.PRESET_1, onButtonListener);
sdlManager.getScreenManager().addButtonListener(ButtonName.PRESET_2, onButtonListener);
```
!@

@![javascript]
```js
sdlManager.addRpcListener(SDL.rpc.enums.FunctionID.OnButtonPress, function (onButtonPress) {
    if (onButtonPress instanceof SDL.rpc.messages.OnButtonPress) {
        switch (onButtonPress.getButtonName()) {
            case SDL.rpc.enums.ButtonName.PRESET_1:
                // PRESET_1 subscribe button selected
                break;
            case SDL.rpc.enums.ButtonName.PRESET_2:
                // PRESET_2 subscribe button selected
                break;
        }
    }
});

const preset1 = new SDL.rpc.messages.SubscribeButton(ButtonName.PRESET_1);
const preset2 = new SDL.rpc.messages.SubscribeButton(ButtonName.PRESET_2);
// sdl_javascript_suite v1.1+
sdlManager.sendRpcsResolve([preset1, preset2]);
// Pre sdl_javascript_suite v1.1
sdlManager.sendRpcs([preset1, preset2]);
```
!@

## Navigation Buttons
Head units supporting RPC v6.0+ may support subscription buttons that allow your user to drag and scale the map using hard buttons located on car's center console or steering wheel. Subscriptions to navigation buttons will only succeed if your app's type is `NAVIGATION`. If subscribing to these buttons succeeds, you can remove any buttons of your own from your map screen. If subscribing to these buttons fails, you can display buttons of your own on your map screen.

### Subscribing to Navigation Buttons
@![iOS]
##### Objective-C
```objc
[self.sdlManager.screenManager subscribeButton:SDLButtonNameNavPanUp withUpdateHandler:^(SDLOnButtonPress * _Nullable buttonPress, SDLOnButtonEvent * _Nullable buttonEvent, NSError * _Nullable error) {
    if (error != nil) {
        // There was an error subscribing to the button
        return;
    }

    if (buttonPress == nil) { return; }

    if ([buttonPress.buttonPressMode isEqualToEnum:SDLButtonPressModeShort]) {
        // The user short pressed the button
    } else if ([buttonPress.buttonPressMode isEqualToEnum:SDLButtonPressModeLong]) {
        // The user long pressed the button
    }
}];
```

##### Swift
```swift
sdlManager.screenManager.subscribeButton(.navPanUp) { (buttonPress, buttonEvent, error) in
    if let error = error {
        // There was an error subscribing to the button
        return
    }

    guard let buttonPress = buttonPress else { return }

    switch buttonPress.buttonPressMode {
    case .short:
        <#The user short pressed the button#>
    case .long:
        <#The user long pressed the button#>
    default:
        <#code#>
    }
}
```
!@

@![android,javaSE,javaEE]
```java
sdlManager.getScreenManager().addButtonListener(ButtonName.NAV_PAN_UP, new OnButtonListener() {
    @Override
    public void onPress (ButtonName buttonName, OnButtonPress buttonPress) {
        switch (buttonPress.getButtonPressMode()) {
            case SHORT:
                // The user short pressed the button
            case LONG:
                // The user long pressed the button
        }
    }

    @Override
    public void onEvent (ButtonName buttonName, OnButtonEvent buttonEvent) { }

    @Override
    public void onError (String info) {
        // There was an error subscribing to the button
    }
});
```
!@

@![javascript]
```js
sdlManager.addRpcListener(SDL.rpc.enums.FunctionID.OnButtonPress, function (onButtonPress) {
    if (onButtonPress instanceof SDL.rpc.messages.OnButtonPress) {
        switch (onButtonPress.getButtonName()) {
            case SDL.rpc.enums.ButtonName.NAV_PAN_UP:
                break;
        }
    }
});

const subscribeButtonRequest = new SDL.rpc.messages.SubscribeButton();
subscribeButtonRequest.setButtonName(SDL.rpc.enums.ButtonName.NAV_PAN_UP);
// sdl_javascript_suite v1.1+
sdlManager.sendRpcResolve(subscribeButtonRequest);
// Pre sdl_javascript_suite v1.1
sdlManager.sendRpc(subscribeButtonRequest);
```
!@
