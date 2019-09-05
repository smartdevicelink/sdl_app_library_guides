# Video Streaming Menu
When building a video-streaming navigation application, you can choose to create a custom menu using your own UI or use the built-in SDL menu system. The SDL menu allows you to display a menu structure so users can select menu options or submenus. For more information about the SDL menu system, see [menus](https://smartdevicelink.com/en/guides/iOS/displaying-a-user-interface/main-menu/i). It's recommended to use the built-in SDL menu system to have better performance, automatic driver distraction support - such as list limitations and text sizing, and more.

To open the SDL built-in menu from your video streaming UI, see 'Opening the Built-In Menu' below.

## Opening the Built-In Menu
The Show Menu RPC allows you to open the menu programmatically. That way, you can open the menu from your own UI.

### Show Top Level Menu
To show the top level menu use `sdlManager.screenManager.openMenu`.

@![iOS]
##### Objective-C
```objc
[self.sdlManager.screenManager openMenu];
```

##### Swift
```swift
self.sdlManager.screenManager.openMenu()
```
!@

@![android, javaSE, javaEE]
```java
// TODO - add example 
```
!@

### Show Sub-Menu
You can also open the menu directly to a sub-menu. This is further down the tree than the top-level menu. To open a sub-menu, pass a cell that contains sub-cells. If the cell has no sub-cells the method call will fail. 

@![iOS]
##### Objective-C
```objc
[self.sdlManager.screenManager openSubmenu:(<#CellWithSubCells#>)];
```

##### Swift
```swift
self.sdlManager.screenManager.openSubmenu(<#CellWithSubCells#>)
```
!@

@![android, javaSE, javaEE]
```java
// TODO - add example 
```
!@

## Close Application
If you choose to not use the built-in SDL menu system and instead to use your own menu UI, you need to have a way for users to close your application. This should be done through a menu option in your UI that sends the `CloseApplication` RPC. 

This RPC is unnecessary if you are using `OpenMenu` because OEMs will take care of providing a close button into your menu themselves.

@![iOS]
##### Objective-C
```objc
SDLCloseApplication *closeRPC = [[SDLCloseApplication alloc] init];
[self.sdlManager sendRequest:closeRPC];
```

##### Swift
```swift
let closeRPC = SDLCloseApplication()
self.sdlManager.send(closeRPC)
```
!@

@![android, javaSE, javaEE]
```java
//TODO - add example 
```
!@


