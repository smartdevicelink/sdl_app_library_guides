# Slider
A @![iOS]`SDLSlider`!@@![android,javaSE,javaEE,javascript]`Slider`!@ creates a full screen or pop-up overlay (depending on platform) that a user can control. There are two main @![iOS]`SDLSlider`!@@![android,javaSE,javaEE,javascript]`Slider`!@ layouts, one with a static footer and one with a dynamic footer.

!!! NOTE
The slider will persist on the screen until the timeout has elapsed or the user dismisses the slider by selecting a position or canceling.
!!!

## Slider
A slider popup with a static footer displays a single, optional, footer message below the slider UI. A dynamic footer can show a different message for each slider position.

### Slider UI
![Slider with Static Footer 1](assets/StaticFooter.png)

##### Dynamic Slider in Position 1
![Slider with Dynamic Footer 1](assets/DynamicFooter1.png)

##### Dynamic Slider in Position 2
![Slider with Dynamic Footer 2](assets/DynamicFooter2.png)

### Creating the Slider
@![iOS]
##### Objective-C
```objc
// Create the slider
SDLSlider *sdlSlider = [[SDLSlider alloc] init];
```
##### Swift
```swift
// Create the slider
let sdlSlider = SDLSlider()
```
!@

@![android,javaSE,javaEE]
```java
Slider slider = new Slider();
```
!@

@![javascript]
```js
const slider = new SDL.rpc.messages.Slider();
```
!@

### Ticks
The number of selectable items on a horizontal axis.
@![iOS]
##### Objective-C
```objc
// Must be a number between 2 and 26
sdlSlider.numTicks = @(5);
```
##### Swift
```swift
// Must be a number between 2 and 26
sdlSlider.numTicks = NSNumber(5)
```
!@

@![android,javaSE,javaEE]
```java
// Must be a number between 2 and 26
slider.setNumTicks(5);
```
!@

@![javascript]
```js
// Must be a number between 2 and 26
slider.setNumTicks(5);
```
!@

### Position 
The initial position of slider control (cannot exceed numTicks).
@![iOS]
##### Objective-C
```objc
// Must be a number between 1 and 26
sdlSlider.position = @(1);
```
##### Swift
```swift
// Must be a number between 1 and 26
sdlSlider.position = NSNumber(1)
```
!@

@![android,javaSE,javaEE]
```java
// Must be a number between 1 and 26
slider.setPosition(1);
```
!@

@![javascript]
```js
// Must be a number between 1 and 26
slider.setPosition(1);
```
!@

### Header 
The header to display.
@![iOS]
##### Objective-C
```objc
// Max length 500 chars
sdlSlider.sliderHeader = @"This is a Header";
```
##### Swift
```swift
// Max length 500 chars
sdlSlider.sliderHeader = "This is a Header"
```
!@

@![android,javaSE,javaEE]
```java
// Max length 500 chars
slider.setSliderHeader("This is a Header");
```
!@

@![javascript]
```js
// Max length 500 chars
slider.setSliderHeader("This is a Header");
```
!@

### Static Footer
The footer will have the same message across all positions of the slider.
@![iOS]
##### Objective-C
```objc
// Max length 500 chars
sdlSlider.sliderFooter = @[@"Static Footer"];
```
##### Swift
```swift
// Max length 500 chars
sdlSlider.sliderFooter = ["Static Footer"]
```
!@

@![android,javaSE,javaEE]
```java
// Max length 500 chars
slider.setSliderFooter(Collections.singletonList("Static Footer"));
```
!@

@![javascript]
```js
// Max length 500 chars
slider.setSliderFooter(["Static Footer"]);
```
!@

### Dynamic Footer
This type of footer will have a different message displayed for each position of the slider. The footer is an optional paramater. The footer message displayed will be based off of the slider's current position. The footer array should be the same length as `numTicks` because each footer must correspond to a tick value. Or, you can pass @![iOS]`nil`!@@![android,javaSE,javaEE]`null`!@ to have no footer at all.

@![iOS]
##### Objective-C
```objc
// Array length 1 - 26, Max length 500 chars
NSArray<NSString *> *footers = @[@"Footer 1", @"Footer 2", @"Footer 3"];
sdlSlider.sliderFooter = footers;
```
##### Swift
```swift
// Array length 1 - 26, Max length 500 chars
let footers = ["Footer 1", "Footer 2", "Footer 3"]
sdlSlider.sliderFooter = footers
```
!@

@![android,javaSE,javaEE]
```java
// Array length 1 - 26, Max length 500 chars
slider.setSliderFooter(Arrays.asList("Footer 1","Footer 2","Footer 3"));
```
!@

@![javascript]
```js
// Array length 1 - 26, Max length 500 chars
slider.setSliderFooter(["Footer 1","Footer 2","Footer 3"]);
```
!@

### Cancel ID
An ID for this specific slider to allow cancellation through the `CancelInteraction` RPC.
@![iOS]
##### Objective-C
```objc
sdlSlider.cancelID = @(45);
```
##### Swift
```swift
sdlSlider.cancelID = NSNumber(45)
```
!@

@![android,javaSE,javaEE]
```java
slider.setCancelID(45);
```
!@

@![javascript]
```js
slider.setCancelID(45);
```
!@

## Show the Slider
@![iOS]
##### Objective-C
```objc
[self.sdlManager sendRequest:sdlSlider withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if (!response || !response.success.boolValue) {
        SDLLogE(@"Error getting the SDLSlider response");
        return;
    }

    // Create a SDLSlider response object from the handler response
    SDLSliderResponse *sdlSliderResponse = (SDLSliderResponse *)response;
    NSUInteger position = sdlSliderResponse.sliderPosition.unsignedIntegerValue;

    <#Use the slider position#>
}];
```
##### Swift
```swift
manager.send(request: sdlSlider, responseHandler: { (req, res, err) in
    // Create a SDLSlider response object from the handler response
    guard let response = res as? SDLSliderResponse, response.success.boolValue == true, let position = response.sliderPosition?.intValue else { return }

    <#Use the slider position#>
})
```
!@

@![android,javaSE,javaEE]
```java
slider.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse response) {
        if (response.getSuccess()) {
            SliderResponse sliderResponse = (SliderResponse) response;
            DebugTool.logInfo(TAG, "Slider Position Set: " + sliderResponse.getSliderPosition());
        }
    }
});
sdlManager.sendRPC(slider);
```
!@

@![javascript]
```js
// sdl_javascript_suite v1.1+
const sliderResponse = await sdlManager.sendRpcResolve(slider);
if (sliderResponse.getSuccess()) {
    console.log('Slider Position Set: ' + sliderResponse.getSliderPosition());
}
// thrown exceptions should be caught by a parent function via .catch()

// Pre sdl_javascript_suite v1.1
const sliderResponse = await sdlManager.sendRpc(slider).catch(function (error) {
    // Handle Error
});
if (sliderResponse.getSuccess()) {
    console.log('Slider Position Set: ' + sliderResponse.getSliderPosition());
}
```
!@

## Dismissing a Slider (RPC v6.0+)
You can dismiss a displayed slider before the timeout has elapsed by dismissing either a specific slider or the current slider.

!!! NOTE
If connected to older head units that do not support this feature, the cancel request will be ignored, and the slider will persist on the screen until the timeout has elapsed or the user dismisses by selecting a position or canceling.
!!!

### Dismissing a Specific Slider

@![iOS]
##### Objective-C
```objc
// `cancelID` is the ID that you assigned when creating the slider
SDLCancelInteraction *cancelInteraction = [[SDLCancelInteraction alloc] initWithSliderCancelID:cancelID];
[self.sdlManager sendRequest:cancelInteraction withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if (!response.success.boolValue) { 
        // Print out the error if there is one and return early
        return;
    }
    <#The slider was canceled successfully#>
}];
```

##### Swift
```swift
// `cancelID` is the ID that you assigned when creating the slider
let cancelInteraction = SDLCancelInteraction(sliderCancelID: cancelID)
sdlManager.send(request: cancelInteraction) { (request, response, error) in
    guard response?.success.boolValue == true else { return }
    <#The slider was canceled successfully#>
}
```
!@

@![android,javaSE,javaEE]
```java
// `cancelID` is the ID that you assigned when creating the slider
CancelInteraction cancelInteraction = new CancelInteraction(FunctionID.SLIDER.getId(), cancelID);
cancelInteraction.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse response) {
        if (response.getSuccess()){
            DebugTool.logInfo(TAG, "Slider was dismissed successfully");
        }
    }
});
sdlManager.sendRPC(cancelInteraction);
```
!@

@![javascript]
```js
// sdl_javascript_suite v1.1+
// `cancelID` is the ID that you assigned when creating the slider
const cancelInteraction = new SDL.rpc.messages.CancelInteraction()
    .setFunctionIDParam(SDL.rpc.enums.FunctionID.Slider)
    .setCancelID(cancelID);
const response = await sdlManager.sendRpcResolve(cancelInteraction);
if (response.getSuccess()) {
    console.log('Slider was dismissed successfully');
}
// thrown exceptions should be caught by a parent function via .catch()

// Pre sdl_javascript_suite v1.1
// `cancelID` is the ID that you assigned when creating the slider
const cancelInteraction = new SDL.rpc.messages.CancelInteraction()
    .setFunctionIDParam(SDL.rpc.enums.FunctionID.Slider)
    .setCancelID(cancelID);
const response = await sdlManager.sendRpc(cancelInteraction).catch(function (error) {
    // Handle Error
});
if (response.getSuccess()) {
    console.log('Slider was dismissed successfully');
}
```
!@

### Dismissing the Current Slider

@![iOS]
##### Objective-C
```objc
SDLCancelInteraction *cancelInteraction = [SDLCancelInteraction slider];
[self.sdlManager sendRequest:cancelInteraction withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if (!response.success.boolValue) { 
        // Print out the error if there is one and return early
        return;
    }
    <#The slider was canceled successfully#>
}];
```

##### Swift
```swift
let cancelInteraction = SDLCancelInteraction.slider()
sdlManager.send(request: cancelInteraction) { (request, response, error) in
    guard response?.success.boolValue == true else { return }
    <#The slider was canceled successfully#>
}
```
!@

@![android,javaSE,javaEE]
```java
CancelInteraction cancelInteraction = new CancelInteraction(FunctionID.SLIDER.getId());
cancelInteraction.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse response) {
        if (response.getSuccess()){
            DebugTool.logInfo(TAG, "Slider was dismissed successfully");
        }
    }
});
sdlManager.sendRPC(cancelInteraction);
```
!@  

@![javascript]
```js
// sdl_javascript_suite v1.1+
const cancelInteraction = new SDL.rpc.messages.CancelInteraction().setFunctionIDParam(SDL.rpc.enums.FunctionID.Slider);
const response = await sdlManager.sendRpcResolve(cancelInteraction);
if (response.getSuccess()) {
    console.log('Slider was dismissed successfully');
}
// thrown exceptions should be caught by a parent function via .catch()

// Pre sdl_javascript_suite v1.1
const cancelInteraction = new SDL.rpc.messages.CancelInteraction().setFunctionIDParam(SDL.rpc.enums.FunctionID.Slider);
const response = await sdlManager.sendRpc(cancelInteraction).catch(function (error) {
    // Handle Error
});
if (response.getSuccess()) {
    console.log('Slider was dismissed successfully');
}
```
!@
