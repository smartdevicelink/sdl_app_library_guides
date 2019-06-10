# Displaying Turn Directions
While your app is navigating the user, you will also want to send turn by turn directions. This is useful for if your app is in the background or if the user is in the middle of a phone call, and gives the system additional information about the next maneuver the user must make.

To display a Turn by Turn direction, a combination of the @![iOS]`SDLShowConstantTBT`!@  @![android,javaSE,javaEE] `Add Property`!@ and @![iOS]`SDLAlertManeuver`!@  @![android,javaSE,javaEE] `Add Property`!@ RPCs must be used. The @![iOS]`SDLShowConstantTBT`!@  @![android,javaSE,javaEE] `Add Property`!@ RPC involves the data that will be shown on the head unit. The main properties of this object to set are `navigationText1`, `navigationText2`, and `turnIcon`. A best practice for navigation applications is to use the `navigationText1` as the direction to give (Turn Right) and `navigationText2` to provide the distance to that direction (3 mi). When an @![iOS]`SDLAlertManeuver`!@  @![android,javaSE,javaEE] `Add Property`!@ is sent, you may also include accompanying text that you would like the head unit to speak when an direction is displayed on screen (e.g. In 3 miles turn right.).

!!! NOTE
If the connected device has received a phone call in the vehicle, the Alert Maneuver is the only way for your app to inform the user of the next turn.
!!!

## Sending a Maneuver

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
    if (![response.resultCode isEqualToEnum:SDLResultSuccess]) {
        NSLog(@"Error sending TBT");
        return;
    }

    typeof(weakSelf) strongSelf = weakSelf;
    SDLAlertManeuver* alertManeuver = [[SDLAlertManeuver alloc] initWithTTS:@"In 3 miles turn right" softButtons:nil];
    [strongSelf.sdlManager sendRequest:alertManeuver withResponseHandler:^(SDLRPCRequest *request, SDLRPCResponse *response, NSError *error) {
        if (![response.resultCode isEqualToEnum:SDLResultSuccess]) {
            NSLog(@"Error sending AlertManeuver.");
            return;
        }
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

sdlManager.send(request: turnByTurn) { [weak self] (request, response, error) in
    if response?.resultCode.isEqual(to: .success) == false {
        print("Error sending TBT.")
        return
    }

    let alertManeuver = SDLAlertManeuver(tts: "In 3 miles turn right", softButtons: nil)
    self.sdlManager.send(alertManeuver) { (request, response, error) in
        if response?.resultCode.isEqual(to: .success) == false {
            print("Error sending AlertManeuver.")
            return
        }
    }
}
```
!@

@![android,javaSE,javaEE]
`// TODO Add Code Example`
!@

Remember when sending a SDLImage, that the image must first be uploaded to the head unit with the FileManager.

## Clearing the Maneuver
To clear a navigation direction from the screen, we send an @![iOS]`SDLShowConstantTBT`!@  @![android,javaSE,javaEE]`Add Property` @! with the `maneuverComplete` property as `YES`. This specific RPC does not require an accompanying @![iOS]`SDLAlertManeuver`!@ @![android,javaSE,javaEE] `Add Property`.

##### Objective-C
```objc
SDLShowConstantTBT* turnByTurn = [[SDLShowConstantTBT alloc] init];
turnByTurn.maneuverComplete = @(YES);

[self.sdlManager sendRequest:turnByTurn withResponseHandler:^(SDLRPCRequest *request, SDLRPCResponse *response, NSError *error) {
    if (![response.resultCode isEqualToEnum:SDLResultSuccess]) {
        NSLog(@"Error sending TBT.");
        return;
    }

    // Successfully cleared
}];
```

##### Swift
```swift
let turnByTurn = SDLShowConstantTBT()
turnByTurn.maneuverComplete = true

sdlManager.send(request: turnByTurn) { (request, response, error) in
    if response?.resultCode.isEqual(to: .success) == false {
        print("Error sending TBT.")
        return
    }
}
```
@![android,javaSE,javaEE]
`// TODO Add Code Example`
!@