# Displaying Turn Directions
While your app is navigating the user, you will also want to send turn by turn directions. This is useful for if your app is in the background or if the user is in the middle of a phone call, and gives the system additional information about the next maneuver the user must make.
When your navigation app is guiding the user to a specific destination, you can provide the user with visual and audio turn-by-turn prompts. These prompts will be presented even when your SDL app is backgrounded or a phone call is ongoing.
While your app is navigating the user, you will also want to send turn by turn directions. This is useful if your app is in the background or if the user is in the middle of a phone call, and gives the system additional information about the next maneuver the user must make.

To create a turn-by-turn direction that provides both a visual and audio cues, a combination of the @![iOS]`SDLShowConstantTBT`!@@![android]`ShowConstantTBT`!@ and @![iOS]`SDLAlertManeuver`!@@![android]`AlertManeuver`!@ RPCs must should be sent to the head unit.

!!! NOTE
If the connected device has received a phone call in the vehicle, the @![iOS]`SDLAlertManeuver`!@@![android]`AlertManeuver`!@ is the only way for your app to inform the user of the next turn.
!!!

### Visual Turn Directions 
The visual data is sent using the @![iOS]`SDLShowConstantTBT`!@@![android]`ShowConstantTBT`!@ RPC. The main properties that should be set are `navigationText1`, `navigationText2`, and `turnIcon`. A best practice for navigation apps is to use the `navigationText1` as the direction to give (i.e. turn right) and `navigationText2` to provide the distance to that direction (i.e. 3 mi.). 
 
### Audio Turn Directions
The audio data is sent using the @![iOS]`SDLAlertManeuver`!@@![android]`AlertManeuver`!@ RPC. When sent, the head unit will speak the text you provide (e.g. In 3 miles turn right).

## Sending Both Audio and Visual Turn Directions
@![iOS]
#### Objective-C
```objc
// Create SDLImage object for turnIcon.
SDLImage* turnIcon = <#Create SDLImage#>;

SDLShowConstantTBT* turnByTurn = [[SDLShowConstantTBT alloc] init];
turnByTurn.navigationText1 = @"Turn Right";
turnByTurn.navigationText2 = @"3 mi";
turnByTurn.turnIcon = turnIcon;

__weak typeof(self) weakSelf = self;
[self.sdlManager sendRequest:turnByTurn withResponseHandler:^(SDLRPCRequest *request, SDLRPCResponse *response, NSError *error) {
    if (!response.success.boolValue) {
        <#Error sending ShowConstantTBT#>
        return;
    }

    typeof(weakSelf) strongSelf = weakSelf;
    SDLAlertManeuver* alertManeuver = [[SDLAlertManeuver alloc] initWithTTS:@"In 3 miles turn right" softButtons:nil];
    [strongSelf.sdlManager sendRequest:alertManeuver withResponseHandler:^(SDLRPCRequest *request, SDLRPCResponse *response, NSError *error) {
        if (!response.success.boolValue) {
            <#Error sending AlertManeuver#>
            return;
        }

        <#Both ShowConstantTBT and AlertManeuver were sent successfully#>
    }];
}];
```

##### Swift
```swift
// Create SDLImage object for turnIcon.
let turnIcon = <#Create SDLImage#>

let turnByTurn = SDLShowConstantTBT()
turnByTurn.navigationText1 = "Turn Right"
turnByTurn.navigationText2 = "3 mi"
turnByTurn.turnIcon = turnIcon

sdlManager.send(request: turnByTurn) { (request, response, error) in
    guard response?.success.boolValue == true else {
        <#Error sending ShowConstantTBT#>
        return
    }

    let alertManeuver = SDLAlertManeuver(tts: "In 3 miles turn right", softButtons: nil)
    self.sdlManager.send(request: alertManeuver, responseHandler: { (request, response, error) in
        guard response?.success.boolValue == true else { 
            <#Error sending AlertManeuver#>
            return 
        }

        <#Both ShowConstantTBT and AlertManeuver were sent successfully#>
    })
}
```
!@

@![android]
```java
ShowConstantTbt turnByTurn = new ShowConstantTbt()
    .setNavigationText1("Turn Right")
    .setNavigationText2("3 mi")
    .setTurnIcon(turnIcon);
turnByTurn.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse response) {
        if (!response.getSuccess()){
            DebugTool.logError(TAG, "onResponse: Error sending TBT");
            return;
        }

            AlertManeuver alertManeuver = new AlertManeuver()
                .setTtsChunks(Collections.singletonList(new TTSChunk("In 3 miles turn right", SpeechCapabilities.TEXT)));
            alertManeuver.setOnRPCResponseListener(new OnRPCResponseListener() {
                @Override
                public void onResponse(int correlationId, RPCResponse response) {
                    if (!response.getSuccess()){
                        DebugTool.logError(TAG, "onResponse: Error sending AlertManeuver");
                    }
                }
            });
            sdlManager.sendRPC(alertManeuver);
        }
    });
sdlManager.sendRPC(turnByTurn);
```
!@

Remember when sending a @![iOS]`SDLImage`!@@![android,javaSE,javaEE]`Image`!@, that the image must first be uploaded to the head unit with the @![iOS]`SDLFileManager`!@@![android,javaSE,javaEE]`FileManager`!@.

## Clearing the Turn Directions
To clear a navigation direction from the screen, send a @![iOS]`SDLShowConstantTBT`!@@![android,javaSE,javaEE]`ShowConstantTbt`!@ with the `maneuverComplete` property set to true. This will also clear the accompanying @![iOS]`SDLAlertManeuver`!@@![android,javaSE,javaEE]`AlertManeuver`!@.

@![iOS]
##### Objective-C
```objc
SDLShowConstantTBT* clearTurnByTurn = [[SDLShowConstantTBT alloc] init];
clearTurnByTurn.maneuverComplete = @YES;

[self.sdlManager sendRequest:clearTurnByTurn withResponseHandler:^(SDLRPCRequest *request, SDLRPCResponse *response, NSError *error) {
    if (!response.success.boolValue) {
        <#Error sending TBT#>
        return;
    }

    <#TBT successfully cleared#>
}];
```

##### Swift
```swift
let clearTurnByTurn = SDLShowConstantTBT()
clearTurnByTurn.maneuverComplete = NSNumber(true)

sdlManager.send(request: clearTurnByTurn) { (request, response, error) in
    guard response?.success.boolValue == true else {
        <#Error sending TBT#>
        return
    }

    <#TBT successfully cleared#>
}
```
!@

@![android]
```java
ShowConstantTbt turnByTurn = new ShowConstantTbt()
    .setManeuverComplete(true);
turnByTurn.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse response) {
        if (!response.getSuccess()){
            DebugTool.logError(TAG, "onResponse: Error sending TBT");
        }
    }
});
sdlManager.sendRPC(turnByTurn);
```
!@
