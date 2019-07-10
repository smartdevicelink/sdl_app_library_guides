# Uploading Images

!!! NOTE
If you are looking to upload images for use in template graphics, soft buttons, or the menu, you can use the [ScreenManager](Displaying a User Interface/Text Images and Buttons). Other situations, such as VR help lists and turn by turn directions, are not currently covered by the `ScreenManager`.
!!!

You should be aware of these four things when using images in your SDL app:

1. You may be connected to a head unit that does not have the ability to display images.
2. You must upload images from your mobile device to the head unit before using them in a template.
3. Persistent images are stored on a head unit between sessions. Ephemeral images are destroyed when a session ends (i.e. when the user turns off their vehicle).
4. Images can not be uploaded when the app's `hmiLevel` is `NONE`. For more information about permissions, please review [Understanding Permissions](Getting Started/Understanding Permissions).

To learn how to use images once they are uploaded, please see [Text, Images, and Buttons](Displaying a User Interface/Text Images and Buttons).

## Checking if Graphics are Supported
Before uploading images to a head unit you should first check if the head unit supports graphics. If not, you should avoid uploading unnecessary image data. To check if graphics are supported, @![iOS]look at the `SDLManager`'s `systemCapabilityManager`'s ,`displayCapabilities` property once the `SDLManager` has started successfully.!@ @![android,javaSE,javaEE] use the `getCapability()` method of a valid `SystemCapabilityManager` obtained from `sdlManager.getSystemCapabilityManager()` to find out the display capabilities of the head unit.!@

@![iOS]
##### Objective-C
```objc
__weak typeof (self) weakSelf = self;
[self.sdlManager startWithReadyHandler:^(BOOL success, NSError * _Nullable error) {
    if (!success) {
        NSLog(@"SDL encountered an error starting up: %@", error);
        return;
    }

    SDLDisplayCapabilities *displayCapabilities = weakSelf.sdlManager.systemCapabilityManager.displayCapabilities;
    BOOL areGraphicsSupported = NO;
    if (displayCapabilities != nil) {
        areGraphicsSupported = displayCapabilities.graphicSupported.boolValue;
    }
}];
```

##### Swift
```swift
sdlManager.start { [weak self] (success, error) in
    if !success {
        print("SDL encountered an error starting up: \(error.debugDescription)")
        return
    }
    
    var areGraphicsSupported = false
    if let displayCapabilities = self?.sdlManager.systemCapabilityManager?.displayCapabilities {
        areGraphicsSupported = displayCapabilities.graphicSupported.boolValue
    }
}
```
!@

@![android,javaSE,javaEE]
```java
sdlManager.getSystemCapabilityManager().getCapability(SystemCapabilityType.DISPLAY, new OnSystemCapabilityListener(){

   @Override
   public void onCapabilityRetrieved(Object capability){
      DisplayCapabilities dispCapability = (DisplayCapabilities) capability;
      boolean graphicsSupported = dispCapability.getGraphicSupported();
   }

   @Override
   public void onError(String info){
      Log.i(TAG, "Capability could not be retrieved: "+ info);
   }
 });
```
!@

## Uploading an Image Using SDL FileManager
The @![iOS]`SDLFileManager`!@ @![android, javaSE, javaEE]`FileManager`!@ uploads files and keeps track of all the uploaded files names during a session. To send data with the @![iOS]`SDLFileManager`!@ @![android, javaSE, javaEE]`FileManager`!@, you need to create either a @![iOS]`SDLFile`!@ @![android, javaSE, javaEE]`SdlFile`!@ or @![iOS]`SDLArtwork`!@ @![android, javaSE, javaEE]`SdlArtwork`!@object. @![iOS]`SDLFile` objects are created with a local `NSURL` or `NSData`; `SDLArtwork` a `UIImage`.!@ @![android]Both `SdlFile`s and `SdlArtwork`s can be created with a `Uri`, `byte[]`, or `resourceId`.!@ @![javaSE, javaEE]Both `SdlFile`s and `SdlArtwork`s can be created with using `filePath`, or `byte[]`.!@


@![iOS]
##### Objective-C
```objc
UIImage* image = [UIImage imageNamed:@"<#Image Name#>"];
if (!image) {
    <#Error Reading from Assets#>
    return;
}

SDLArtwork* artwork = [SDLArtwork artworkWithImage:image asImageFormat:<#SDLArtworkImageFormat#>];

[self.sdlManager.fileManager uploadArtwork:artwork completionHandler:^(BOOL success, NSString * _Nonnull artworkName, NSUInteger bytesAvailable, NSError * _Nullable error) {
    if (error != nil) { return; }
    <#Image Upload Successful#>
    // To send the image as part of a show request, create a SDLImage object using the artworkName
    SDLImage *image = [[SDLImage alloc] initWithName:artworkName];
}];
```

##### Swift
```swift
guard let image = UIImage(named: "<#Image Name#>") else {
	<#Error Reading from Assets#>
	return
}
let artwork = SDLArtwork(image: image, persistent: <#Bool#>, as: <#SDLArtworkImageFormat#>)

sdlManager.fileManager.upload(artwork: artwork) { (success, artworkName, bytesAvailable, error) in
    guard error == nil else { return }
    <#Image Upload Successful#>
    // To send the image as part of a show request, create a SDLImage object using the artworkName
    let graphic = SDLImage(name: artworkName)
}
```
!@

@![android,javaSE,javaEE]
```java
SdlArtwork artwork = new SdlArtwork("image_name", FileType.GRAPHIC_PNG, <image byte[]>, false);
sdlManager.getFileManager().uploadFile(artwork, new CompletionListener() {
    @Override
    public void onComplete(boolean success) {
        if (success){
            <#Image Upload Successful#>
        }
    }
});
```
!@

### Batch File Uploads, Persistence, etc.
Similar to other files, artworks can be persistent, batched, overwrite, etc. See [Uploading Files](Other SDL Features/Uploading Files) for more information.
