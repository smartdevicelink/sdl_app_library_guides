# Alerts
An alert is a pop-up window showing a short message with optional buttons. When an alert is activated, it will abort any SDL operation that is in-progress, except the already-in-progress alert. If an alert is issued while another alert is still in progress, the newest alert will simply be ignored.

Depending the platform, an alert can have up to three lines of text, a progress indicator (e.g. a spinning wheel or hourglass), and up to four soft buttons.

## Alert Layouts
###### Alert With No Soft Buttons

![Generic - Alert](assets/Generic_alert.png)

!!! NOTE
If no soft buttons are added to an alert some OEMs may add a default "cancel" or "close" button.
!!!

###### Alert With Soft Buttons

![Generic - Alert](assets/Generic_alert_buttons.png)

## Creating the Alert

### Text
@![iOS]
##### Objective-C
```objc
SDLAlert *alert = [[SDLAlert alloc] initWithAlertText:<#NSString#> softButtons:<#[SDLSoftButton]#> playTone:<#BOOL#> ttsChunks:<#[SDLTTSChunk]#> alertIcon:<#SDLImage#> cancelID:<#UInt32#>];
```

##### Swift
```swift
let alert = SDLAlert(alertText: <#String?#>, softButtons: <#[SDLSoftButton]?#>, playTone: <#Bool#>, ttsChunks: <#[SDLTTSChunk]?#>, alertIcon: <#SDLImage?#>, cancelID: <#UInt32#>)
```
!@

@![android,javaSE,javaEE]
```java
Alert alert = new Alert();
alert.setAlertText1("Line 1")
     .setAlertText2("Line 2")
     .setAlertText3("Line 3")
     .setCancelID(<#Integer>);
```
!@

@![javascript]
```js
const alert = new SDL.rpc.messages.Alert()
    .setAlertText1('Line 1')
    .setAlertText2('Line 2')
    .setAlertText3('Line 3')
    .setCancelID(integer);
```
!@

### Buttons

@![iOS]
##### Objective-C
```objc
SDLSoftButton *button1 = [[SDLSoftButton alloc] initWithType:SDLSoftButtonTypeText text:@"<#Button Text#>" image:nil highlighted:false buttonId:<#Soft Button Id#> systemAction:SDLSystemActionDefaultAction handler:^(SDLOnButtonPress *_Nullable buttonPress, SDLOnButtonEvent *_Nullable buttonEvent) {
    if (buttonPress == nil) {
        return;
    }
    <#Button has been pressed#>
}];

SDLSoftButton *button2 = [[SDLSoftButton alloc] initWithType:SDLSoftButtonTypeText text:<#Button Text#> image:nil highlighted:false buttonId:<#Soft Button Id#> systemAction:SDLSystemActionDefaultAction handler:^(SDLOnButtonPress *_Nullable buttonPress, SDLOnButtonEvent *_Nullable buttonEvent) {
    if (buttonPress == nil) {
        return;
    }
    <#Button has been pressed#>
}];

alert.softButtons = @[button1, button2];
```

##### Swift
```swift
let button1 = SDLSoftButton(type: .text, text: <#Button Text#>, image: nil, highlighted: false, buttonId: <#Soft Button Id#>, systemAction: .defaultAction, handler: { buttonPress, buttonEvent in
    guard buttonPress != nil else { return }
    <#Button has been pressed#>
})

let button2 = SDLSoftButton(type: .text, text: <#Button Text#>, image: nil, highlighted: false, buttonId: <#Soft Button Id#>, systemAction: .defaultAction, handler: { buttonPress, buttonEvent in
    guard buttonPress != nil else { return }
    <#Button has been pressed#>
})

alert.softButtons = [button1, button2]
```
!@

@![android,javaSE,javaEE]
```java
// Soft buttons
final int softButtonId = 123; // Set it to any unique ID
SoftButton okButton = new SoftButton(SoftButtonType.SBT_TEXT, softButtonId);
okButton.setText("OK");

// Set the softbuttons(s) to the alert
alert.setSoftButtons(Collections.singletonList(okButton));

// This listener is only needed once, and will work for all of soft buttons you send with your alert
sdlManager.addOnRPCNotificationListener(FunctionID.ON_BUTTON_PRESS, new OnRPCNotificationListener() {
      @Override
      public void onNotified(RPCNotification notification) {
          OnButtonPress onButtonPress = (OnButtonPress) notification;
          if (onButtonPress.getCustomButtonID() == softButtonId){
               Log.i(TAG, "Ok button pressed");
          }
      }
});
```
!@

@![javascript]
```js
// Soft buttons
const softButtonId = 123; // Set it to any unique ID
const okButton = new SDL.rpc.structs.SoftButton()
    .setType(SDL.rpc.enums.SoftButtonType.SBT_TEXT)
    .setSoftButtonID(softButtonId)
    .setText('OK');

// Set the softbuttons(s) to the alert
alert.setSoftButtons([okButton]);

// This listener is only needed once, and will work for all of soft buttons you send with your alert
sdlManager.addRpcListener(SDL.rpc.enums.FunctionID.ON_BUTTON_PRESS, function (onButtonPress) {
    if (onButtonPress.getCustomButtonId() === softButtonId) {
        console.log("OK button pressed");
    }
})
```
!@

### Alert Icon
An alert can include a custom or static (built-in) image that will be displayed within the alert. Before you add the image to the alert make sure the image is uploaded to the head unit using the @![iOS]`SDLFileManager`!@@![android,javaSE,javaEE,javascript]FileManager!@. If the image is already uploaded, you can set the `alertIcon` property.

![Generic - Alert](assets/Generic_alertIcon.png)

@![iOS]
##### Objective-C
```objc
alert.alertIcon = [[SDLImage alloc] initWithName:<#artworkName#> isTemplate:YES];
```
##### Swift
```swift
alert.alertIcon = SDLImage(name: <#artworkName#>, isTemplate: true)
```
!@

@![android,javaSE,javaEE]
```java
alert.setAlertIcon(new Image(<#artworkName#>, ImageType.DYNAMIC));
```
!@

@![javascript]
```js
alert.setAlertIcon(new SDL.rpc.structs.Image(<#artworkName#>, SDL.rpc.enums.ImageType.DYNAMIC));
```
!@

### Timeouts
An optional timeout can be added that will dismiss the alert when the duration is over. Typical timeouts are between 3 and 10 seconds. If omitted a default of 5 seconds is used.

@![iOS]
##### Objective-C
```objc
// Duration timeout is in milliseconds
alert.duration = @(4000);
```

##### Swift
```swift
// Duration timeout is in milliseconds
alert.duration = NSNumber(4000)
```
!@

@![android,javaSE,javaEE]
```java
alert.setDuration(5000);
```
!@

@![javascript]
```js
alert.setDuration(5000);
```
!@

### Progress Indicator
Not all OEMs support a progress indicator. If supported, the alert will show an animation that indicates that the user must wait (e.g. a spinning wheel or hourglass, etc). If omitted, no progress indicator will be shown.

@![iOS]
##### Objective-C
```objc
alert.progressIndicator = @YES;
```

##### Swift
```swift
alert.progressIndicator = NSNumber(true)
```
!@

@![android,javaSE,javaEE]
```java
alert.setProgressIndicator(true);
```
!@

@![javascript]
```js
alert.setProgressIndicator(true);
```
!@

### Text-To-Speech
An alert can also speak a prompt or play a sound file when the alert appears on the screen. This is done by setting the `ttsChunks` parameter.

#### Text
@![iOS]
##### Objective-C
```objc
alert.ttsChunks = [SDLTTSChunk textChunksFromString:@"<#Text to speak#>"];
```

##### Swift
```swift
alert.ttsChunks = SDLTTSChunk.textChunks(from: "<#Text to speak#>")
```
!@

@![android,javaSE,javaEE]
```java
alert.setTtsChunks(TTSChunkFactory.createSimpleTTSChunks("Text to Speak"));
```
!@

@![javascript]
```js
const chunk = new SDL.rpc.structs.TTSChunk()
    .setType(SDL.rpc.enums.SpeechCapabilities.TEXT)
    .setText('Text to Speak');
alert.setTtsChunks([chunk]);
```
!@

#### Sound File
The `ttsChunks` parameter can also take a file to play/speak. For more information on how to upload the file please refer to the [Playing Audio Indications](Speech and Audio/Playing Audio Indications) guide.

@![iOS]
##### Objective-C
```objc
 alert.ttsChunks = [SDLTTSChunk fileChunksWithName:@"<#Name#>"];
```

##### Swift
```swift
 alert.ttsChunks = SDLTTSChunk.fileChunks(withName: "<#Name#>")
```
!@

@![android,javaSE,javaEE]
```java
TTSChunk ttsChunk = new TTSChunk(sdlFile.getName(), SpeechCapabilities.FILE);
alert.setTtsChunks(Collections.singletonList(ttsChunk));
```
!@

@![javascript]
```js
const ttsChunk = new SDL.rpc.structs.TTSChunk()
    .setText(sdlFile.getName())
    .setType(SDL.rpc.enums.SpeechCapabilities.FILE);
alert.setTtsChunk([ttsChunk]);
```
!@

### Play Tone
To play the alert tone when the alert appears and before the text-to-speech is spoken, set `playTone` to `true`.

@![iOS]
##### Objective-C
```objc
alert.playTone = @YES;
```

##### Swift
```swift
alert.playTone = NSNumber(true)
```
!@

@![android,javaSE,javaEE]
```java
alert.setPlayTone(true);
```
!@

@![javascript]
```js
alert.setPlayTone(true);
```
!@

## Showing the Alert

@![iOS]
##### Objective-C
```objc
[self.sdlManager sendRequest:alert withResponseHandler:^(SDLRPCRequest *request, SDLRPCResponse *response, NSError *error) {
    if (!response.success.boolValue) { 
        // Print out the error if there is one and return early
        return;
    }
    <#Alert was shown successfully#>
}];
```

##### Swift
```swift
sdlManager.send(request: alert) { (request, response, error) in
    guard response?.success.boolValue == true else { return }
    <#Alert was shown successfully#>
}
```
!@

@![android,javaSE,javaEE]
```java
// Handle RPC response
alert.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse response) {
      if (response.getSuccess()){
        Log.i(TAG, "Alert was shown successfully");
      }
    }
});
sdlManager.sendRPC(alert);
```
!@

@![javascript]
```js
// sdl_javascript_suite v1.1+
const response = await sdlManager.sendRpcResolve(alert);
if (response.getSuccess()) {
    console.log('Alert was shown successfully');
}
// thrown exceptions should be caught by a parent function via .catch()

// Pre sdl_javascript_suite v1.1
// Handle RPC Response
const response = await sdlManager.sendRpc(alert).catch(function (error) {
    // Handle Error
});
if (response.getSuccess()) {
    console.log('Alert was shown successfully');
}
```

## Dismissing the Alert (RPC v6.0+)
You can dismiss a displayed alert before the timeout has elapsed. This feature is useful if you want to show users a loading screen while performing a task, such as searching for a list for nearby coffee shops. As soon as you have the search results, you can cancel the alert and show the results. 

!!! NOTE
If connected to older head units that do not support this feature, the cancel request will be ignored, and the alert will persist on the screen until the timeout has elapsed or the user dismisses the alert by selecting a button.
!!!

Please note that canceling the alert will only dismiss the displayed alert. If you have set the `ttsChunk` property, the speech will play in its entirety even when the displayed alert has been dismissed. If you know you will cancel an alert, consider setting a short `ttsChunk` like "searching" instead of "searching for coffee shops, please wait."

There are two ways to dismiss an alert. The first way is to dismiss a specific alert using a unique `cancelID` assigned to the alert. The second way is to dismiss whichever alert is currently on-screen.

### Dismissing a Specific Alert

@![iOS]
##### Objective-C
```objc
// `cancelID` is the ID that you assigned when creating and sending the alert
SDLCancelInteraction *cancelInteraction = [[SDLCancelInteraction alloc] initWithAlertCancelID:cancelID];
[self.sdlManager sendRequest:cancelInteraction withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if (!response.success.boolValue) { 
        // Print out the error if there is one and return early
        return;
    }
    <#The alert was canceled successfully#>
}];
```

##### Swift
```swift
// `cancelID` is the ID that you assigned when creating and sending the alert
let cancelInteraction = SDLCancelInteraction(alertCancelID: cancelID)
sdlManager.send(request: cancelInteraction) { (request, response, error) in
    guard response?.success.boolValue == true else { return }
    <#The alert was canceled successfully#>
}
```
!@

@![android,javaSE,javaEE]
```java
// `cancelID` is the ID that you assigned when creating and sending the alert
CancelInteraction cancelInteraction = new CancelInteraction(FunctionID.ALERT.getId(), cancelID);
cancelInteraction.setOnRPCResponseListener(new OnRPCResponseListener() {
	@Override
	public void onResponse(int correlationId, RPCResponse response) {
		if (response.getSuccess()){
			Log.i(TAG, "Alert was dismissed successfully");
		}
	}
});
sdlManager.sendRPC(cancelInteraction);
```
!@

@![javascript]
```js
// sdl_javascript_suite v1.1+
const cancelInteraction = new SDL.rpc.messages.CancelInteraction()
    .setFunctionIDParam(SDL.rpc.enums.FunctionID.Alert)
    .setCancelID(cancelID);
const response = await sdlManager.sendRpcResolve(cancelInteraction);
if (response.getSuccess()) {
    console.log('Alert was dismissed successfully');
}
// thrown exceptions should be caught by a parent function via .catch()

// Pre sdl_javascript_suite v1.1
const cancelInteraction = new SDL.rpc.messages.CancelInteraction()
    .setFunctionIDParam(SDL.rpc.enums.FunctionID.Alert)
    .setCancelID(cancelID);
const response = await sdlManager.sendRpc(cancelInteraction).catch(function (error) {
    // Handle Error
});
if (response.getSuccess()) {
    console.log('Alert was dismissed successfully');
}
```
!@

### Dismissing the Current Alert

@![iOS]
##### Objective-C
```objc
SDLCancelInteraction *cancelInteraction = [SDLCancelInteraction alert];
[self.sdlManager sendRequest:cancelInteraction withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if (!response.success.boolValue) { 
        // Print out the error if there is one and return early
        return;
    }
    <#The alert was canceled successfully#>
}];
```

##### Swift
```swift
let cancelInteraction = SDLCancelInteraction.alert()
sdlManager.send(request: cancelInteraction) { (request, response, error) in
    guard response?.success.boolValue == true else { return }
    <#The alert was canceled successfully#>
}
```
!@

@![android,javaSE,javaEE]
```java
CancelInteraction cancelInteraction = new CancelInteraction(FunctionID.ALERT.getId());
cancelInteraction.setOnRPCResponseListener(new OnRPCResponseListener() {
	@Override
	public void onResponse(int correlationId, RPCResponse response) {
		if (response.getSuccess()){
			Log.i(TAG, "Alert was dismissed successfully");
		}
	}
});
sdlManager.sendRPC(cancelInteraction);
```
!@  

@![javascript]
```js
// sdl_javascript_suite v1.1+
const cancelInteraction = new SDL.rpc.messages.CancelInteraction().setFunctionIDParam(SDL.rpc.enums.FunctionID.Alert);
const response = await sdlManager.sendRpcResolve(cancelInteraction);
if (response.getSuccess()) {
    console.log('Alert was dismissed successfully');
}
// thrown exceptions should be caught by a parent function via .catch()

// Pre sdl_javascript_suite v1.1
const cancelInteraction = new SDL.rpc.messages.CancelInteraction().setFunctionIDParam(SDL.rpc.enums.FunctionID.Alert);
const response = await sdlManager.sendRpc(cancelInteraction).catch(function (error) {
    // Handle Error
});
if (response.getSuccess()) {
    console.log('Alert was dismissed successfully');
}
```
!@