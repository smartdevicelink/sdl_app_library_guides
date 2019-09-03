# Video Streaming Menu

## Open Built-In Menu
The Show Menu RPC allows you to open the menu programmatically. When creating a projection applicaiton it is recommended to still use the built-in menu rather then creating a custom one. Since there is no guaranteed the built-in menu button will be visble in a projection application we recommned creating a custom button and call the `ShowMenu` RPC.

### Show Top Level Menu
To show the top level menu use the `screenManger`s `openMenu` function.

@![iOS]
##### Objective-C
```objc
[self.sdlManger.screenManager openMenu];
```
##### Swift
```swift
self.sdlManager.screenManager.openMenu()
```
!@

@![android, javaSE, javaEE]
```java
toDO - add example 
```
!@

### Show Sub Menu
Showing a sub menu is also possible with the new Show App Menu RPC. To open a certain sub menu simply pass the parent cell that contains sub cells. If the cell has no sub cells the RPC will fail. 

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
toDO - add example 
```
!@

## Close Application
Video Streaming / Projection application need a way to close the application if not utilizing  `ShowMenu` . If you are utizling the `ShowMenu` RPC it is highly reccomned to not use the `CloseApplication` RPC.  If you chose to create a custom menu the  `CloseApplication`  may be useful for your users.

@![iOS]
##### Objective-C
```objc
SDLCloseApplication *closeApp = [[SDLCloseApplication alloc] init];
[self.sdlManager sendRequest:closeApp];
```
##### Swift
```swift
let closeApp = SDLCloseApplication()
self.sdlManager.send(closeApp)
```
!@

@![android, javaSE, javaEE]
```java
toDO - add example 
```
!@


