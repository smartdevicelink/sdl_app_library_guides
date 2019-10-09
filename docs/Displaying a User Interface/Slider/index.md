# Slider
A @![iOS]`SDLSlider`!@@![android,javaSE,javaEE]`Slider`!@ creates a full screen or pop-up overlay (depending on platform) that a user can control. There are two main @![iOS]`SDLSlider`!@@![android,javaSE,javaEE]`Slider`!@ layouts, one with a static footer and one with a dynamic footer.

!!! NOTE
The slider will persist on the screen until the timeout has elapsed or the user dismisses the slider by selecting a position or canceling.
!!!

## Slider with Static Footer
A slider popup with a static footer displays a single, optional, footer message below the slider UI.

### Slider UI
![Slider with Static Footer 1](assets/StaticFooter.png)

### Creating the Slider

@![iOS]
##### Objective-C
```objc
// Create a slider with number of ticks, starting position 'tick number', a header message, an optional footer message, and a timeout of 30 seconds
SDLSlider *sdlSlider = [[SDLSlider alloc] initWithNumTicks:5 position:1 sliderHeader:@"This is a Header" sliderFooter:@"Static Footer" timeout:30000 cancelID:45];

// Send the slider RPC request with handler
[manager sendRequest:sdlSlider withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
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
// Create a slider with number of ticks, starting position 'tick number', a header message, an optional footer message, and a timeout of 30 seconds
let slider =  SDLSlider(numTicks: 5, position: 1, sliderHeader: "This is a Header", sliderFooter: "Static Footer", timeout: 30000, cancelID: 45)

// Send the slider RPC request with handler
manager.send(request: slider, responseHandler: { (req, res, err) in
    // Create a SDLSlider response object from the handler response
    guard let response = res as? SDLSliderResponse, response.resultCode == .success, let position = response.sliderPosition.intValue else { return }

    <#Use the slider position#>
})
```
!@
@![android,javaSE,javaEE]

```java

//Create a slider
Slider slider = new Slider(5, 1, "This is a Header");

List<String> footer = Collections.singletonList("Static Footer");
slider.setSliderFooter(footer);
slider.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse response) {
        SliderResponse sliderResponse = (SliderResponse) response;
        Log.i(TAG, "Slider Position Set: "+ sliderResponse.getSliderPosition());
    }

    @Override
    public void onError(int correlationId, Result resultCode, String info){
        Log.e(TAG, "onError: "+ resultCode+ " | Info: "+ info );
    }
});

//Send Request
sdlManager.sendRPC(slider);
```
!@

## Slider with Dynamic Footer
This type of slider will have a different footer message displayed for each position of the slider. The footer is an optional paramater. The footer message displayed will be based off of the slider's current position. The footer array should be the same length as `numTicks` because each footer must correspond to a tick value. Or, you can pass @![iOS]`nil`!@@![android,javaSE,javaEE]`null`!@ to have no footer at all.

### Slider UI

##### Dynamic Slider in Position 1
![Slider with Dynamic Footer 1](assets/DynamicFooter1.png)

##### Dynamic Slider in Position 2
![Slider with Dynamic Footer 2](assets/DynamicFooter2.png)

### Creating the Slider

@![iOS]
##### Objective-C
```objc
// Create an array of footers to display to the user. This array *must* be equal in length to the number of ticks of the slider you create below.
NSArray<NSString *> *footers = @[@"Footer 1", @"Footer 2", @"Footer 3"];

// Create a slider with number of ticks, starting position 'tick number', a header message, and an optional footer array, and a timeout of 30 seconds
SDLSlider *sdlSlider = [[SDLSlider alloc] initWithNumTicks:3 position:1 sliderHeader:@"This is a Header" sliderFooters:footers timeout:30000 cancelID:45];

// Send the slider RPC request with handler
[manager sendRequest:sdlSlider withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
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
// Create an array of footers to display to the user. This array *must* be equal in length to the number of ticks of the slider you create below.
let footers = ["Footer 1", "Footer 2", "Footer 3"]

// Create a slider with number of ticks, starting position 'tick number', a header message, and an optional footer array, and a timeout of 30 seconds
let slider =  SDLSlider(numTicks: 3, position: 1, sliderHeader: "This is a Header", sliderFooters: footers, timeout: 30000, cancelID: 45)
manager.send(request: slider, responseHandler: { (req, res, err) in
    // Create a SDLSlider response object from the handler response
    guard let response = res as? SDLSliderResponse, response.resultCode == .success, let position = response.sliderPosition.intValue else { return }

    <#Use the slider position#>
})
```
!@
@![android,javaSE,javaEE]

```java
//Create a slider
Slider slider = new Slider(3, 1, "This is a Header");

// Each footer corresponds with the slider's position
List<String> footer = Arrays.asList("Footer 1","Footer 2","Footer 3");
slider.setSliderFooter(footer);
slider.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse response) {
        SliderResponse sliderResponse = (SliderResponse) response;
        Log.i(TAG, "Slider Position Set: "+ sliderResponse.getSliderPosition());
    }

    @Override
    public void onError(int correlationId, Result resultCode, String info){
        Log.e(TAG, "onError: "+ resultCode+ " | Info: "+ info );
    }
});

//Send Request
sdlManager.sendRPC(slider);
```
!@

## Canceling a Specific Slider
If you are connected to a head unit with SDL Core v6.0+, you can specificly dismiss a displayed slider before the timeout has elapsed.

If connected to older head units that do not support this feature, the cancel request will be ignored, and the slider will persist on the screen until the timeout has elapsed or the user dismisses by selecting a position or canceling.

### Dismissing a Specific Slider

@![iOS]
##### Objective-C
```objc
// Assign a unique cancel id to the slider
UInt32 cancelID = 45;
slider.cancelID = @(cancelID);

// Use the cancel id to dismiss the slider
SDLCancelInteraction *cancelInteraction = [[SDLCancelInteraction alloc] initWithSliderCancelID:cancelID];
[self.sdlManager sendRequest:cancelInteraction withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if (![response.resultCode isEqualToEnum:SDLResultSuccess]) { return; }
    <#The slider was canceled successfully#>
}];
```

##### Swift
```swift
// Assign a unique cancel id to the slider
let cancelID: UInt32 = 45
slider.cancelID = cancelID as NSNumber

// Use the cancel id to dismiss the slider
let cancelInteraction = SDLCancelInteraction(sliderCancelID: cancelID)
sdlManager.send(request: cancelInteraction) { (request, response, error) in
    guard response?.resultCode == .success else { return }
    <#The slider was canceled successfully#>
}
```
!@

@![android,javaSE,javaEE]
```java
// Assign a unique cancel id to the slider
final Integer cancelID = 45;
slider.setCancelID(cancelID);

// Use the cancel id to dismiss the slider
CancelInteraction cancelInteraction = new CancelInteraction(FunctionID.SLIDER.getId(), cancelID);
cancelInteraction.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse response) {
        if (response.getSuccess()){
            Log.i(TAG, "Slider was dismissed successfully");
        }
    }

    @Override
    public void onError(int correlationId, Result resultCode, String info) {
        Log.e(TAG, "onError: "+ resultCode+ " | Info: "+ info );
    }
});
sdlManager.sendRPC(cancelInteraction);
```
!@

### Dismissing Current Slider

@![iOS]
##### Objective-C
```objc
SDLCancelInteraction *cancelInteraction = [SDLCancelInteraction slider];
[self.sdlManager sendRequest:cancelInteraction withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if (![response.resultCode isEqualToEnum:SDLResultSuccess]) { return; }
    <#The slider was canceled successfully#>
}];
```

##### Swift
```swift
let cancelInteraction = SDLCancelInteraction.slider()
sdlManager.send(request: cancelInteraction) { (request, response, error) in
    guard response?.resultCode == .success else { return }
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
            Log.i(TAG, "Slider was dismissed successfully");
        }
    }

    @Override
    public void onError(int correlationId, Result resultCode, String info) {
        Log.e(TAG, "onError: "+ resultCode+ " | Info: "+ info );
    }
});
sdlManager.sendRPC(cancelInteraction);
```
!@  

