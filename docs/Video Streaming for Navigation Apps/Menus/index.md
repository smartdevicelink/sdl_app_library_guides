# Video Streaming Menu

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
[self.sdlManager sendRequest:closeApp];
```

##### Swift
```swift
let closeRPC = SDLCloseApplication()
self.sdlManager.send(closeApp)
```
!@

@![android, javaSE, javaEE]
```java
toDO - add example 
```
!@


