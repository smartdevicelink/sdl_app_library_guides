# Customizing the Keyboard
If you present keyboards in your app – such as in searchable interactions or another custom keyboard – you may wish to customize the keyboard for your users. The best way to do this is through the @![iOS]`SDLScreenManager`!@@![android, javaSE, javaEE]`ScreenManager`!@. For more information presenting keyboards, see the [Popup Menus and Keyboards guide](Displaying a User Interface/Popup Menus and Keyboards).

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
// TODO
```
!@

## Other Properties
While there are other keyboard properties available on @![iOS]`SDLKeyboardProperties`!@@![android, javaSE, javaEE]`KeyboardProperties`!@, these will be overridden by the screen manager. The `keypressMode` must be a specific configuration for the screen manager's callbacks to work properly. The `limitedCharacterList`, `autoCompleteText`, and `autoCompleteList` will be set on a per-keyboard basis in the @![iOS]`SDLKeyboardDelegate`!@@![android, javaSE, javaEE]`// TODO`!@ which is set on the @![iOS]`presentKeyboard` and `presentSearchableChoiceSet`!@@![android, javaSE, javaEE]`// TODO`!@ methods.
