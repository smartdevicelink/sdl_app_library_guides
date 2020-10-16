# Customizing Help Prompts
On some head units it is possible to display a customized help menu or speak a custom command if the user asks for help while using your app. The help menu is commonly used to let users know what voice commands are available, however, it can also be customized to help your user navigate the app or let them know what features are available. 

## Configuring the Help Menu
You can customize the help menu with your own title and/or menu options. If you don't customize these options, then the head unit's default menu will be used.

If you wish to use an image, you should check the @![iOS]`sdlManager.systemCapabilityManager.defaultMainWindowCapability.imageFields`!@@![android, javaSE, javaEE, javascript]`sdlManager.getSystemCapabilityManager().getDefaultMainWindowCapability().getImageFields();`!@ for an `imageField.name` of `vrHelpItem` to see if that image is supported. If `vrHelpItem` is in the `imageFields` array, then it can be used. You will then need to upload the image using the file manager before using it in the request. See the [Uploading Images](Other SDL Features/Uploading Images) section for more information.

@![iOS]
##### Objective-C
```objc
SDLSetGlobalProperties *setGlobals = [[SDLSetGlobalProperties alloc] init];
setGlobals.vrHelpTitle = <#Custom help title string such as: "What Can I Say?"#>;

// Set up the menu items
SDLVRHelpItem *item1 = [[SDLVRHelpItem alloc] initWithText:<#Help item name such as "Show Artists"#> image: <#A previously uploaded image or nil#> position: 1];
SDLVRHelpItem *item2 = [[SDLVRHelpItem alloc] initWithText:<#Help item name such as "Shuffle All"#> image: <#A previously uploaded image or nil#> position: 2];
setGlobals.vrHelp = @[item1, item2];

[self.sdlManager sendRequest:setGlobals withResponseHandler:^(SDLRPCRequest *request, SDLRPCResponse *response, NSError *error) {
    if (error != nil) {
        // Something went wrong
        return;
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
    if let error = error {
        // Something went wrong
        return;
    }

    // The help menu is updated
}
```
!@

@![android, javaSE, javaEE]
```java
SetGlobalProperties setGlobalProperties = new SetGlobalProperties();
setGlobalProperties.setVrHelpTitle("What Can I Say?");

VrHelpItem item1 = new VrHelpItem("Show Artists", 1);
item1.setImage(image); // a previously uploaded image or null

VrHelpItem item2 = new VrHelpItem("Show Albums", 2);
item2.setImage(image); // a previously uploaded image or null

setGlobalProperties.setVrHelp(Arrays.asList(item1, item2));
setGlobalProperties.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse response) {
        // The help menu is updated
    }
});
sdlManager.sendRPC(setGlobalProperties);
```
!@

@![javascript]
```js
const setGlobalProperties = new SDL.rpc.messages.SetGlobalProperties();
setGlobalProperties.setVrHelpTitle('What Can I Say?');

const item1 = new SDL.rpc.structs.VrHelpItem().setText("Show Artists").setPosition(1).setImage(<#image#>); // a previously uploaded image or null

const item2 = new SDL.rpc.structs.VrHelpItem().setText("Show Albums").setPosition(2).setImage(<#image#>); // a previously uploaded image or null

setGlobalProperties.setVrHelp([item1, item2]);
sdlManager.sendRpc(setGlobalProperties).catch(err => err); // If there was an error, catch it and return it
if (response instanceof SDL.rpc.RpcRespone && response.getSuccess()) {
    // The help menu is updated
} else {
    // Handle Error
}
```
!@

## Configuring the  Help Prompt
On head units that support voice recognition, a user can request assistance by saying "Help." In addition to displaying the help menu discussed above a custom spoken text-to-speech response can be spoken to the user.

@![iOS]
##### Objective-C
```objc
SDLSetGlobalProperties *setGlobals = [[SDLSetGlobalProperties alloc] init];
setGlobals.helpPrompt = [SDLTTSChunk textChunksFromString:<#Your custom help prompt#>];

[self.sdlManager sendRequest:setGlobals withResponseHandler:^(SDLRPCRequest *request, SDLRPCResponse *response, NSError *error) {
    if (error != nil) {
        // Something went wrong
        return;
    }

    // The help prompt is updated
}];
```

##### Swift
```swift
let setGlobals = SDLSetGlobalProperties()
setGlobals.helpPrompt = SDLTTSChunk.textChunks(from: <#Your custom help prompt#>)

sdlManager.send(request: setGlobals) { (request, response, error) in
    if let error = error {
        // Something went wrong
        return
    }

    // The help prompt is updated
}
```
!@

@![android, javaSE, javaEE]
```java
SetGlobalProperties setGlobalProperties = new SetGlobalProperties();
setGlobalProperties.setHelpPrompt(Collections.singletonList(new TTSChunk("Your custom help prompt", SpeechCapabilities.TEXT)));
setGlobalProperties.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse response) {
        if (response.getSuccess()) {
            // The help prompt is updated
        } else {
            // Handle Error
        }
    }
});
sdlManager.sendRPC(setGlobalProperties);
```
!@

@![javascript]
```js
const setGlobalProperties = new SDL.rpc.messages.SetGlobalProperties();
const chunk = new SDL.rpc.structs.TTSChunk().setText('Your custom help prompt').setType(SDL.rpc.enums.SpeechCapabilities.TEXT);
setGlobalProperties.setHelpPrompt([chunk]);
const response = await sdlManager.sendRpc(setGlobalProperties).catch(err => err); // If there was an error, catch it and return it
if (response instanceof SDL.rpc.RpcRespone && response.getSuccess()) {
    // The help prompt is updated
} else {
    // Handle Error
}
```
!@

## Configuring the Timeout Prompt
If you display any sort of popup menu or modal interaction that has a timeout – such as an alert, interaction, or slider – you can create a custom text-to-speech response that will be spoken to the user in the event that a timeout occurs.

@![iOS]
##### Objective-C
```objc
SDLSetGlobalProperties *setGlobals = [[SDLSetGlobalProperties alloc] init];
setGlobals.timeoutPrompt = [SDLTTSChunk textChunksFromString:<#Your custom help prompt#>];

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
    if let error = error {
        // Something went wrong
    }

    // The timeout prompt is updated
}
```
!@

@![android, javaSE, javaEE]
```java
SetGlobalProperties setGlobalProperties = new SetGlobalProperties();
setGlobalProperties.setTimeoutPrompt(Collections.singletonList(new TTSChunk("Your custom help prompt", SpeechCapabilities.TEXT)));
setGlobalProperties.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse response) {
        if (response.getSuccess()) {
            // The timeout prompt is updated
        } else {
            // Handle Error
        }
    }
});
sdlManager.sendRPC(setGlobalProperties);
```
!@

@![javascript]
```js
const setGlobalProperties = new SDL.rpc.messages.SetGlobalProperties();
const chunk = new SDL.rpc.structs.TTSChunk().setText('Your custom help prompt').setType(SDL.rpc.enums.SpeechCapabilities.TEXT);
setGlobalProperties.setTimeoutPrompt([chunk]);
const response = await sdlManager.sendRpc(setGlobalProperties).catch(err => err); // If there was an error, catch it and return it
if (response instanceof SDL.rpc.RpcRespone && response.getSuccess()) {
    // The timeout prompt is updated
} else {
    // Handle Error
}
```
!@

## Clearing Help Menu and Prompt Customizations
You can also reset your customizations to the help menu or spoken prompts. To do so, you will send a `ResetGlobalProperties` RPC with the fields that you wish to clear.

@![iOS]
##### Objective-C
```objc
// Reset the help menu
SDLResetGlobalProperties *resetGlobals = [[SDLResetGlobalProperties alloc] initWithProperties:@[SDLGlobalPropertyVoiceRecognitionHelpItems, SDLGlobalPropertyVoiceRecognitionHelpTitle]];

// Reset the menu icon and title
SDLResetGlobalProperties *resetGlobals = [[SDLResetGlobalProperties alloc] initWithProperties:@[SDLGlobalPropertyMenuIcon, SDLGlobalPropertyMenuName]];


// Reset the spoken prompts
SDLResetGlobalProperties *resetGlobals = [[SDLResetGlobalProperties alloc] initWithProperties:@[SDLGlobalPropertyHelpPrompt, SDLGlobalPropertyTimeoutPrompt]];

[self.sdlManager sendRequest:resetGlobals withResponseHandler:^(SDLRPCRequest *request, SDLRPCResponse *response, NSError *error) {
    if (error != nil) {
        // Something went wrong
    }

    // The global properties are reset
}];
```

##### Swift
```swift
// Reset the help menu
let resetGlobals = SDLResetGlobalProperties(properties: [.voiceRecognitionHelpItems, .voiceRecognitionHelpTitle])

// Reset the menu icon and title
let resetGlobals = SDLResetGlobalProperties(properties: [.menuIcon, .menuName])

// Reset the spoken prompts
let resetGlobals = SDLResetGlobalProperties(properties: [.helpPrompt, .timeoutPrompt])
sdlManager.send(request: resetGlobals) { (req, res, err) in
    if let error = error {
        // Something went wrong
    }

    // The global properties are reset
}
```
!@

@![android, javaSE, javaEE]
```java
// Reset the help menu
ResetGlobalProperties resetGlobalProperties = new ResetGlobalProperties(Arrays.asList(GlobalProperty.VRHELPITEMS, GlobalProperty.VRHELPTITLE));

// Reset the menu icon and title
ResetGlobalProperties resetGlobalProperties = new ResetGlobalProperties(Arrays.asList(GlobalProperty.MENUICON, GlobalProperty.MENUNAME));

// Reset spoken prompts
ResetGlobalProperties resetGlobalProperties = new ResetGlobalProperties(Arrays.asList(GlobalProperty.HELPPROMPT, GlobalProperty.TIMEOUTPROMPT));

// To send any one of these, use the typical format:
resetGlobalProperties.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse response) {
        if (response.getSuccess()) {
            // The global properties are reset
        } else {
            // Handle Error
        }
    }
});
sdlManager.sendRPC(resetGlobalProperties);
```
!@

@![javascript]
```js
// Reset the help menu
const resetGlobalProperties = new SDL.rpc.messages.ResetGlobalProperties().setProperties([SDL.rpc.enums.GlobalProperty.VRHELPITEMS, SDL.rpc.enums.GlobalProperty.VRHELPTITLE]);

// Reset the menu icon and title
const resetGlobalProperties = new SDL.rpc.messages.ResetGlobalProperties().setProperties([SDL.rpc.enums.GlobalProperty.MENUICON, SDL.rpc.enums.GlobalProperty.MENUNAME]);

// Reset spoken prompts
const resetGlobalProperties = new SDL.rpc.messages.ResetGlobalProperties().setProperties([SDL.rpc.enums.GlobalProperty.HELPPROMPT, SDL.rpc.enums.GlobalProperty.TIMEOUTPROMPT]);

// To send any one of these, use the typical format:
const response = await sdlManager.sendRpc(setGlobalProperties).catch(err => err); // If there was an error, catch it and return it
if (response instanceof SDL.rpc.RpcRespone && response.getSuccess()) {
    // The global properties are reset
} else {
    // Handle Error
}
```
!@