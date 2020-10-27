# Customizing the Keyboard
@![iOS, android, javaEE, javaSE]
If you present keyboards in your app – such as in searchable interactions or another custom keyboard – you may wish to customize the keyboard for your users. The best way to do this is through the !@@![iOS]`SDLScreenManager`!@@![android, javaSE, javaEE]`ScreenManager`!@@![iOS, android, javaEE, javaSE]. For more information presenting keyboards, see the [Popup Menus and Keyboards guide](Displaying a User Interface/Popup Keyboards).
!@

@![javascript]
The `ChoiceSetManager` used to customize the keyboard is currently missing from the SDL JavaScript Suite. This will be addressed in a future release. However you can still customize the keyboard through the `SetGlobalProperties` RPC.
!@

## Setting Keyboard Properties
You can modify the language of the keyboard to change the characters that are displayed.

@![iOS]
##### Objective-C
```objc
SDLKeyboardProperties *keyboardConfig = [[SDLKeyboardProperties alloc] init];
keyboardConfig.language = SDLLanguageHeIl; // Set to Israeli Hebrew
keyboardConfig.keyboardLayout = SDLKeyboardLayoutAZERTY; // Set to AZERTY

self.sdlManager.screenManager.keyboardConfiguration = keyboardConfig;
```

##### Swift
```swift
let keyboardConfig = SDLKeyboardProperties()
keyboardConfig.language = .heIl // Set to Israeli Hebrew
keyboardConfig.keyboardLayout = .AZERTY; // Set to AZERTY

sdlManager.screenManager.keyboardConfiguration = keyboardConfig
```
!@

@![android, javaSE, javaEE]
```java
KeyboardProperties keyboardProperties = new KeyboardProperties()
    .setLanguage(Language.HE_IL) // Set to Israeli Hebrew
    .setKeyboardLayout(KeyboardLayout.AZERTY); // Set to AZERTY

sdlManager.getScreenManager().setKeyboardConfiguration(keyboardProperties);
```
!@

@![javascript]
```js
const keyboardProperties = new SDL.rpc.structs.KeyboardProperties().setLanguage(SDL.rpc.enums.Language.HE_IL).setKeyboardLayout(SDL.rpc.enums.KeyboardLayout.AZERTY);
const setGlobalProperties = new SDL.rpc.messages.SetGlobalProperties().setKeyboardProperties(keyboardProperties);
const response = await sdlManager.sendRpc(setGlobalProperties).catch(error => error); // If there's an error, catch it and return it
if(response instanceof SDL.rpc.RpcResponse && response.getSuccess()) {
    //keyboard properties updated
} else {
    // Handle Error here
}
```
!@

@![iOS, android, javaSE, javaEE]
## Other Properties
!@
@![iOS]While there are other keyboard properties available on `SDLKeyboardProperties`, these will be overridden by the screen manager. The `keypressMode` must be a specific configuration for the screen manager's callbacks to work properly. The `limitedCharacterList`, `autoCompleteText`, and `autoCompleteList` will be set on a per-keyboard basis in the `SDLKeyboardDelegate` which is set on the `presentKeyboard` and `presentSearchableChoiceSet` methods.!@


@![android, javaSE, javaEE]While there are other keyboard properties available on `KeyboardProperties`, these will be overridden by the screen manager. The `keypressMode` must be a specific configuration for the screen manager's callbacks to work properly. The `limitedCharacterList`, `autoCompleteText`, and `autoCompleteList` will be set on a per-keyboard basis when calling `sdlManager.getScreenManager.presentKeyboard(...)`, should custom keyboard properties be set. !@
