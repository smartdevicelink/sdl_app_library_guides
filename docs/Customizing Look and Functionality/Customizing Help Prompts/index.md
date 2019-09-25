# Customizing Help Prompts
On some head units it is possible to display a customized help menu or speak a custom command if the user asks for help while using your app. The help menu is commonly used to let users know what voice commands are available however it can also be customized to help your user navigate the app or let them know what features are available. 

## Configuring the Help Menu
You can customize the help menu with your own title and/or menu options. If you don't customize these options, then the head unit's default menu will be used.

If you wish to use an image, you should check the @![iOS]`sdlManager.systemCapabilityManager.defaultMainWindowCapability.imageFields`!@@![android, javaSE, javaEE]`// TODO`!@ for an `imageField.name` of `vrHelpItem` to see if that image is supported. If `vrHelpItem` is in the `imageFields` array, then it can be used. You will need to then upload the image using the file manager before using it in the RPC below. See the [Uploading Images guide](Other SDL Features/Uploading Images) for more information.

@![iOS]
##### Objective-C
```objc
SDLSetGlobalProperties *setGlobals = [[SDLSetGlobalProperties alloc] init];
setGlobals.vrHelpTitle = <#Custom help title string such as: "What Can I Say?"#>

// Set up the menu items
SDLVRHelpItem *item1 = [[SDLVRHelpItem alloc] initWithText:<#Help item name such as "Show Artists"#> image: <#A previously uploaded image or nil#> position: 1];
SDLVRHelpItem *item2 = [[SDLVRHelpItem alloc] initWithText:<#Help item name such as "Shuffle All"#> image: <#A previously uploaded image or nil#> position: 2];
setGlobals.vrHelp = @[item1, item2];

[self.sdlManager sendRequest:setGlobals withResponseHandler:^(SDLRPCRequest *request, SDLRPCResponse *response, NSError *error) {
    if (error != nil) {
        // Something went wrong
    }

    // The help menu is updated
}];
```

##### Swift
```swift
let setGlobals = SDLSetGlobalProperties()
setGlobals.vrHelpTitle = <#Custom help title string such as: "What Can I Say?"#>

// Set up the menu items
let item1 = SDLVRHelpItem(text:<#Help item name such as "Show Artists"#>, image: <#A previously uploaded image or nil#>, position: 1)
let item2 = SDLVRHelpItem(text:<#Help item name such as "Shuffle All"#>, image: <#A previously uploaded image or nil#>, position: 2)
setGlobals.vrHelp = [item1, item2];

sdlManager.send(request: setGlobals) { (request, response, error) in
    if error != nil {
        // Something went wrong
    }

    // The help menu is updated
}
```
!@

@![android, javaSE, javaEE]
```java
// TODO
```
!@

## Configuring the  Help Prompt
On head units that support voice recognition, a user can request assistance by saying "Help." In addition to displaying the help menu discussed above a custom spoken text-to-speech response can be spoken to the user.

@![iOS]
##### Objective-C
```objc
SDLSetGlobalProperties *setGlobals = [[SDLSetGlobalProperties alloc] init];
setGlobals.helpPrompt = SDLTTSChunk *response = [SDLTTSChunk textChunksFromString:<#Your custom help prompt#>];

[self.sdlManager sendRequest:setGlobals withResponseHandler:^(SDLRPCRequest *request, SDLRPCResponse *response, NSError *error) {
    if (error != nil) {
        // Something went wrong
    }

    // The help prompt is updated
}];
```

##### Swift
```swift
let setGlobals = SDLSetGlobalProperties()
setGlobals.helpPrompt = SDLTTSChunk.textChunks(from: <#Your custom help prompt#>)

sdlManager.send(request: setGlobals) { (request, response, error) in
    if error != nil {
        // Something went wrong
    }

    // The help prompt is updated
}
```
!@

@![android, javaSE, javaEE]
```java
// TODO
```
!@

## Configuring the Timeout Prompt
If a popup menu you display times out, you can create a custom text-to-speech response that will be spoken to the user.

@![iOS]
##### Objective-C
```objc
SDLSetGlobalProperties *setGlobals = [[SDLSetGlobalProperties alloc] init];
setGlobals.timeoutPrompt = SDLTTSChunk *response = [SDLTTSChunk textChunksFromString:<#Your custom help prompt#>];

[self.sdlManager sendRequest:setGlobals withResponseHandler:^(SDLRPCRequest *request, SDLRPCResponse *response, NSError *error) {
    if (error != nil) {
        // Something went wrong
    }

    // The timeout prompt is updated
}];
```

##### Swift
```swift
let setGlobals = SDLSetGlobalProperties()
setGlobals.timeoutPrompt = SDLTTSChunk.textChunks(from: <#Your custom help prompt#>)

sdlManager.send(request: setGlobals) { (request, response, error) in
    if error != nil {
        // Something went wrong
    }

    // The timeout prompt is updated
}
```
!@

@![android, javaSE, javaEE]
```java
// TODO
```
!@
