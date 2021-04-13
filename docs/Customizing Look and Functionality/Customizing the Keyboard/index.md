# Customizing the Keyboard (RPC v3.0+)
If you present keyboards in your app – such as in searchable interactions or another custom keyboard – you may wish to customize the keyboard for your users. The best way to do this is through the @![iOS]`SDLScreenManager`!@@![android, javaSE, javaEE, javascript]`ScreenManager`!@. For more information presenting keyboards, see the [Popup Keyboards](Displaying a User Interface/Popup Keyboards) section.


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
const keyboardProperties = new SDL.rpc.structs.KeyboardProperties()
    .setLanguage(SDL.rpc.enums.Language.HE_IL) // Set to Israeli Hebrew
    .setKeyboardLayout(SDL.rpc.enums.KeyboardLayout.AZERTY); // Set to AZERTY

sdlManager.getScreenManager().setKeyboardConfiguration(keyboardProperties);
```
!@

## Other Properties
@![iOS]While there are other keyboard properties available on `SDLKeyboardProperties`, these will be overridden by the screen manager. The `keypressMode` must be a specific configuration for the screen manager's callbacks to work properly. The `limitedCharacterList`, `autoCompleteText`, and `autoCompleteList` will be set on a per-keyboard basis in the `SDLKeyboardDelegate` which is set on the `presentKeyboard` and `presentSearchableChoiceSet` methods.!@


@![android, javaSE, javaEE, javascript]While there are other keyboard properties available on `KeyboardProperties`, these will be overridden by the screen manager. The `keypressMode` must be a specific configuration for the screen manager's callbacks to work properly. The `limitedCharacterList`, `autoCompleteText`, and `autoCompleteList` will be set on a per-keyboard basis when calling `sdlManager.getScreenManager.presentKeyboard(...)`, should custom keyboard properties be set. !@
