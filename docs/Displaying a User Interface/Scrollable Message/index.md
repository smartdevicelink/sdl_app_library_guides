# Scrollable Message
An @![iOS]`SDLScrollableMessage`!@@![android,javaSE,javaEE]`ScrollableMessage`!@ creates an overlay containing a large block of formatted text that can be scrolled. @![iOS]`SDLScrollableMessage`!@@![android,javaSE,javaEE]`ScrollableMessage`!@ contains a body of text, a message timeout, and up to 8 soft buttons depending on head unit. You must check the `DisplayCapabilities` to get the max number of `SoftButtons` allowed by the head unit for a `ScrollableMessage`.

You simply create @![iOS]an `SDLScrollableMessage`!@@![android,javaSE,javaEE]a `ScrollableMessage`!@ RPC request and send it to display the Scrollable Message.

!!! NOTE
The message will persist on the screen until the timeout has elapsed or the user dismisses the message by selecting a soft button or cancelling (if the head unit provides cancel UI).
!!!

##  Scrollable Message UI
![Scrollable Message](assets/ScrollableMessage.png)

## Creating the Scrollable Message
@![iOS]
##### Objective-C
```objc
// Create SoftButton Array
NSMutableArray<SDLSoftButton *> *softButtons = [[NSMutableArray alloc] init];

// Create Message To Display
NSString *scrollableMessageString = [NSString stringWithFormat:@"Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.\n\n\nVestibulum mattis ullamcorper velit sed ullamcorper morbi tincidunt ornare. Purus in massa tempor nec feugiat nisl pretium fusce id.\n\n\nPharetra convallis posuere morbi leo urna molestie at elementum eu. Dictum sit amet justo donec enim diam."];

// Create a timeout of 50 seconds
UInt16 scrollableMessageTimeout = 50000;

// Create SoftButtons
SDLSoftButton *scrollableSoftButton = [[SDLSoftButton alloc] initWithType:SDLSoftButtonTypeText text:@"Button 1" image:nil highlighted:NO buttonId:111 systemAction:nil handler:^(SDLOnButtonPress * _Nullable buttonPress, SDLOnButtonEvent * _Nullable buttonEvent) {
    if (buttonPress == nil) { return;
    // Create a custom action for the selected button
}];
SDLSoftButton *scrollableSoftButton2 = [[SDLSoftButton alloc] initWithType:SDLSoftButtonTypeText text:@"Button 2" image:nil highlighted:NO buttonId:222 systemAction:nil handler:^(SDLOnButtonPress * _Nullable buttonPress, SDLOnButtonEvent * _Nullable buttonEvent) {
    if (buttonPress == nil) { return; }
    // Create a custom action for the selected button
}];

[softButtons addObject:scrollableSoftButton];
[softButtons addObject:scrollableSoftButton2];

// Create SDLScrollableMessage Object
SDLScrollableMessage *scrollableMessage = [[SDLScrollableMessage alloc] initWithMessage:scrollableMessageString timeout:scrollableMessageTimeout softButtons:[softButtons copy]];

// Send the scrollable message
[sdlManager sendRequest:scrollableMessage];
```

##### Swift
```swift
// Create SoftButton Array
var softButtons = [SDLSoftButton]()

// Create Message To Display
let scrollableMessageText = """
Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.

Vestibulum mattis ullamcorper velit sed ullamcorper morbi tincidunt ornare. Purus in massa tempor nec feugiat nisl pretium fusce id.

Pharetra convallis posuere morbi leo urna molestie at elementum eu. Dictum sit amet justo donec enim diam.
"""

// Create a timeout of 50 seconds
let scrollableTimeout: UInt16 = 50000

// Create SoftButtons
let scrollableSoftButton = SDLSoftButton(type: .text, text: "Button 1", image: nil, highlighted: false, buttonId: 111, systemAction: .defaultAction, handler: { (buttonPress, buttonEvent) in
    guard let press = buttonPress else { return }

    // Create a custom action for the selected button
})
let scrollableSoftButton2 = SDLSoftButton(type: .text, text: "Button 2", image: nil, highlighted: false, buttonId: 222, systemAction: .defaultAction, handler: { (buttonPress, buttonEvent) in
    guard let press = buttonPress else { return }

    // Create a custom action for the selected button
})

softButtons.append(scrollableSoftButton)
softButtons.append(scrollableSoftButton2)

// Create SDLScrollableMessage Object
let scrollableMessage = SDLScrollableMessage(message: scrollableMessageText, timeout: scrollableTimeout, softButtons: softButtons)

// Send the scrollable message
sdlManager.send(scrollableMessage)
```
!@

@![android,javaSE,javaEE]
```java
// Create SoftButton Array
List<SoftButtonObject> softButtonObjects = new ArrayList<SoftButtonObject>;

// Create Message To Display
String scrollableMessageText = "Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.Vestibulum mattis ullamcorper velit sed ullamcorper morbi tincidunt ornare. Purus in massa tempor nec feugiat nisl pretium fusce id. Pharetra convallis posuere morbi leo urna molestie at elementum eu. Dictum sit amet justo donec enim diam."

// Create SoftButtons
SoftButtonState softButtonState1 = new SoftButtonState("state1", "state1", new SdlArtwork("state1.png", FileType.GRAPHIC_PNG, R.drawable.state1, true));
SoftButtonState softButtonState2 = new SoftButtonState("state2", "state2", new SdlArtwork("state2.png", FileType.GRAPHIC_PNG, R.drawable.state2, true));

SoftButtonObject softButtonObject1 = new SoftButtonObject("Button 1", softButtonState1, null);
SoftButtonObject softButtonObject2 = new SoftButtonObject("Button 2", softButtonState2, null);

// Add Event Listeners for buttons
softButtonObject1.setOnEventListener(new SoftButtonObject.OnEventListener() {
@Override
public void onPress(SoftButtonObject softButtonObject, OnButtonPress onButtonPress) {
softButtonObject.transitionToNextState();
}

@Override
public void onEvent(SoftButtonObject softButtonObject, OnButtonEvent onButtonEvent) {

}
});

softButtonObject2.setOnEventListener(new SoftButtonObject.OnEventListener() {
@Override
public void onPress(SoftButtonObject softButtonObject, OnButtonPress onButtonPress) {
softButtonObject.transitionToNextState();
}

@Override
public void onEvent(SoftButtonObject softButtonObject, OnButtonEvent onButtonEvent) {

}
});

//Add to button array 
softButtonObjects.add(softButtonObject1);
softButtonObjects.add(softButtonObject2);

// Create ScrollableMessage Object
ScrollableMessage scrollableMessage = new ScrollableMessage();
scrollableMessage.setScrollableMessageBody(scrollableMessageText);
scrollableMessage.setTimeout(50000)
scrollableMessage.setSoftButtons(softButtonObjects)

// Send the scrollable message
sdlManager.sendRPC(scrollableMessage);
```
!@
