# Customizing Help Prompts
You are able to customize the head unit's help system with your own information.

## Help Menu
If the head unit supports displaying a custom help menu, you can customize this menu with your own title and menu options. This menu is commonly used to assist users with knowing what voice commands are available. If you don't customize these options, then the head unit's default title and/or items will be used.

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

    // The help prompt is updated
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

    // The help prompt is updated
}
```
!@

@![android, javaSE, javaEE]
```java
// TODO
```
!@

## Help Prompt
In SDL, the user is able to request assistance with using your app by saying "Help" on head units that support voice recognition. This will display the help menu discussed above, but it can also speak a custom spoken text-to-speech response to the user.

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

## Timeout Prompt
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

    // The help prompt is updated
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

    // The help prompt is updated
}
```
!@

@![android, javaSE, javaEE]
```java
// TODO
```
!@