# Alerts
An alert is a pop-up window that can show a short message with optional buttons. When an alert is activated, it will abort any SDL operation that is in-progress, except the already-in-progress alert. If an alert is issued while another alert is still in progress, the newest alert will simply be ignored.

Depending the platform, an alert can have up to three lines of text, a progress indicator (e.g. a spinning wheel or hourglass), and up to four soft buttons.

!!! NOTE
The alert will persist on the screen until the timeout has elapsed, or the user dismisses the alert by selecting a button. There is no way to dismiss the alert programmatically other than to set the timeout length.
!!!

## Creating an Alert
### Alert With No Soft Buttons
!!! NOTE
If no soft buttons are added to an alert some OEMs may add a default "cancel" or "close" button.
!!!

#### Alert HMI
![Generic - Alert](assets/Generic_alert.png)

@![iOS]
#####  Objective-C
```objc
SDLAlert *alert =  [SDLAlert alloc] initWithAlertText1:@"<#Line 1#>" alertText2:@"<#Line 2#>" alertText3:@"<#Line 3#>"];
```

##### Swift
```swift
let alert = SDLAlert(alertText1: "<#Line 1#>", alertText2: "<#Line 2#>", alertText3: "<#Line 3#>")
```
!@

@![android,javaSE,javaEE]
` TODO - code example `
!@

### Alert With Soft Buttons

#### Alert HMI
![Generic - Alert](assets/Generic_alert_buttons.png)

@![iOS]
##### Objective-C
```objc
SDLAlert *alert = [[SDLAlert alloc] initWithAlertText1:@"<#Line 1#>" alertText2:@"<#Line 2#>" alertText3:@"<#Line 3#>"];

NSMutableArray<SDLSoftButton *> *softButtons = [[NSMutableArray alloc] init];

SDLSoftButton *button1 = [[SDLSoftButton alloc] initWithType:SDLSoftButtonTypeText text:<#Button Text#> image:nil highlighted:@NO buttonId:<#Soft Button Id#> systemAction:SDLSystemActionDefaultAction handler:^(SDLOnButtonPress *_Nullable buttonPress, SDLOnButtonEvent *_Nullable buttonEvent) {
    if (buttonPress == nil) {
        return;
    }
    <#Code for when the button is pressed#>
}];

SDLSoftButton *button2 = [[SDLSoftButton alloc] initWithType:SDLSoftButtonTypeText text:<#Button Text#> image:nil highlighted:@NO buttonId:<#Soft Button Id#> systemAction:SDLSystemActionDefaultAction handler:^(SDLOnButtonPress *_Nullable buttonPress, SDLOnButtonEvent *_Nullable buttonEvent) {
    if (buttonPress == nil) {
        return;
    }
    <#Code for when the button is pressed#>
}];

[softButtons addObject: button1];
[softButtons addObject: button2];

alert.softButtons = softButtons;
```

#### Swift
```swift
let alert = SDLAlert(alertText1: "<#Line 1#>", alertText2: "<#Line 2#>", alertText3: "<#Line 3#>")

var softButtons = [SDLSoftButton]()

let button1 = SDLSoftButton(type: .text, text: <#Button Text#>, image: nil, highlighted: false, buttonId: <#Soft Button Id#>, systemAction: .defaultAction, handler: { buttonPress, buttonEvent in
    guard buttonPress != nil else { return }
    <#Code for when the button is pressed#>
})

let button2 = SDLSoftButton(type: .text, text: <#Button Text#>, image: nil, highlighted: false, buttonId: <#Soft Button Id#>, systemAction: .defaultAction, handler: { buttonPress, buttonEvent in
    guard buttonPress != nil else { return }
    <#Code for when the button is pressed#>
})

softButtons.append(button1)
softButtons.append(button2)

alert.softButtons = softButtons;
```
!@

@![android,javaSE,javaEE]
` TODO - code example `
!@

## Additonal Alert Paramaters

### Timeouts
An optional timeout can be added that will dimiss the alert when the duration is over.  Typical timeouts are between 3 and 10 seconds. If omitted a default of 5 second is used.

@![iOS]
##### Objective-C
```objc
// Duration timeout is in milliseconds
alert.duration = @(4000);
```

##### Swift
```swift
// Duration timeout is in milliseconds
alert.duration = 4000
```
!@

@![android,javaSE,javaEE]
`//TODO - code example `
!@

### Progress Indicator
Not all OEMs support a progress indicator. If supported, the alert will show an animation that indicates that the user must wait (e.g. a spinning wheel or hourglass, etc). If omitted, no progress indicator will be shown.

@![iOS]
#### Objective-C
```objc
alert.progressIndicator = @YES;
```

#### Swift
```swift
alert.progressIndicator = true
```
!@

@![android,javaSE,javaEE]
` TODO - code example `
!@

### Text-To-Speach
The alert can also be formatted to speak a prompt or play a file when the alert appears on the screen. Do this by setting the `ttsChunks` parameter.

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
` TODO - code example `
!@

### Text-To-Speach With File
The `ttsChunks` can also take a file to play/speak. First you must [upload](https://smartdevicelink.com/en/guides/iOS/other-sdl-features/uploading-files/) a file to the head unit.

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
` TODO - code example `
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
alert.playTone = true
```
!@

@![android,javaSE,javaEE]
` TODO - code example `
!@

### Showing the Alert

@![iOS]
##### Objective-C
```objc
[self.sdlManager sendRequest:alert withResponseHandler:^(SDLRPCRequest *request, SDLRPCResponse *response, NSError *error) {
    if ([response.resultCode isEqualToEnum:SDLResultSuccess]) {
        // alert was dismissed successfully
    }
}];
```

##### Swift
```swift
sdlManager.send(request: alert) { (request, response, error) in
    guard error == nil else { return }
        // alert was dismissed successfully
    }    
}
```
!@

@![android,javaSE,javaEE]
` TODO - code example `
!@
