# Template Images
You can easily display text, images, and buttons using the  @![iOS]`SDLScreenManager`!@ @![android, javaSE, javaEE]`ScreenManager`!@. To update the UI, simply give the manager your new data and sandwich the update between the manager's @![iOS]`beginUpdates`!@ @![android, javaSE, javaEE]`beginTransaction()`!@ and @![iOS]`endUpdatesWithCompletionHandler`!@ @![android, javaSE, javaEE]`commit()`!@ methods.

### Image Fields
| @![iOS]SDLScreenManager!@ @![android, javaSE, javaEE]`ScreenManager!@ Parameter Name  | Description |
|:--------------------------------------------|:--------------|
| primaryGraphic | The primary image in a template that supports images |
| secondaryGraphic | The second image in a template that supports multiple images |

### Showing Images
@![iOS]
##### Objective-C
```objc
[self.sdlManager.screenManager beginUpdates];
self.sdlManager.screenManager.primaryGraphic = <#SDLArtwork#>;
[self.sdlManager.screenManager endUpdatesWithCompletionHandler:^(NSError * _Nullable error) {
    if (error != nil) {
        <#Error Updating UI#>
    } else {
        <#Update to UI was Successful#>
    }
}];
```

##### Swift
```swift
sdlManager.screenManager.beginUpdates()
sdlManager.screenManager.primaryGraphic = <#SDLArtwork#>
sdlManager.screenManager.endUpdates { (error) in
    if error != nil {
        <#Error Updating UI#>
    } else {
        <#Update to UI was Successful#>
    }
}
```
!@

@![android, javaSE, javaEE]

```java
sdlManager.getScreenManager().beginTransaction();
sdlManager.getScreenManager().setPrimaryGraphic(<#SDLArtwork#>);
sdlManager.getScreenManager().commit(new CompletionListener() {
	@Override
	public void onComplete(boolean success) {
		Log.i(TAG, "ScreenManager update complete: " + success);
	}
});
```
!@

### Removing Images
To remove an image from the screen you just need to set the screen manager property to @![iOS]`nil`!@ @![android, javaSE, javaEE]`null`!@.

@![iOS]
##### Objective-C
```objc
self.sdlManager.screenManager.primaryGraphic = nil;
```

##### Swift
```swift
sdlManager.screenManager.primaryGraphic = nil
```
!@

@![android, javaSE, javaEE]
```java
sdlManager.getScreenManager().setPrimaryGraphic(null);
```
!@

## Templating Images (RPC v5.0+)
Templated images are tinted by Core so the image is visible regardless of whether your user has set the head unit to day or night mode. For example, if a head unit is in night mode with a dark theme (see [Customizing the Template](Customizing Look and Functionality/Customizing the Template) section for more details on how to customize theme colors), then your templated images will be displayed as white. In the day theme, the image will automatically change to black.

Soft buttons, menu icons, and primary / secondary graphics can all be templated. @![iOS]A template image works [very much like it does on iOS](https://developer.apple.com/documentation/uikit/uiimage/1624153-imagewithrenderingmode) and in fact, it uses the same API as iOS. Any `SDLArtwork` created with a `UIImage` that has a `renderingMode` of `alwaysTemplate` will be templated via SDL as well.!@ Images that you wish to template must be PNGs with a transparent background and only one color for the icon. Therefore, templating is only useful for things like icons and not for images that must be rendered in a specific color. 

### Templated Images Example
In the screenshots below, the shuffle and repeat icons have been templated. In night mode, the icons are tinted white and in day mode the icons are tinted black.

##### Night Mode
![Generic - Template Images Dark Mode](assets/Generic_template_media_dark.png)

##### Day Mode
![Generic - Template Images Light Mode](assets/Generic_template_media_light.png)

@![iOS]
##### Objective-C
```objc
UIImage *image = [[UIImage imageNamed:@"<#String#>"] imageWithRenderingMode:UIImageRenderingModeAlwaysTemplate];
SDLArtwork *artwork = [SDLArtwork artworkWithImage:image asImageFormat:SDLArtworkImageFormatPNG];
```

##### Swift
```swift
let image = UIImage(named: "<#String#>")?.withRenderingMode(.alwaysTemplate)
let artwork = SDLArtwork(image: image, persistent: true, as: .PNG)
```
!@

@![android, javaSE, javaEE]
```java
SdlArtwork image = new SdlArtwork("ArtworkName", FileType.GRAPHIC_PNG, R.drawable.artworkName, true);
image.setTemplateImage(true);
```
!@

## Static Icons
Static icons are pre-existing images on the remote system that you may reference and use in your own application. Static icons are fully supported by the screen manager via an @![iOS]`SDLArtwork`!@ @![android, javaSE, javaEE]`SdlArtwork`!@ initializer. Static icons can be used in primary and secondary graphic fields, soft button image fields, and menu icon fields.

@![iOS]
##### Objective-C
```objc
SDLArtwork *staticIconArt = [[SDLArtwork alloc] initWithStaticIcon:SDLStaticIconNameAlbum];;
SDLSoftButtonState *softButtonState1 = [[SDLSoftButtonState alloc] initWithStateName:@"<#Soft Button State Name#>" text:@"<#Button Label Text#>" artwork:staticIconArt];

<#Set the state into an `SDLSoftButtonObject` and then set the screen manager's array of soft buttons#>
```

##### Swift
```swift
let staticIconArt = SDLArtwork(staticIcon: .album)
let softButtonState1 = SDLSoftButtonState(stateName: "<#Soft Button State Name#>", text: "<#Button Label Text#>", artwork: staticIconArt)

<#Set the state into an `SDLSoftButtonObject` and then set the screen manager's array of soft buttons#>
```
!@

@![android, javaSE, javaEE]
```java
SdlArtwork sdlArtwork = new SdlArtwork(StaticIconName.ALBUM);
sdlManager.getScreenManager().setPrimaryGraphic(sdlArtwork);

<#Set the state into an `SoftButtonObject` and then set the screen manager's array of soft buttons#>
```
!@