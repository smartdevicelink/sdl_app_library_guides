# Uploading Images

!!! NOTE
If you use the @![iOS]`SDLScreenManager`!@@![android, javaSE, javaEE]`ScreenManager`!@, [image uploading for template graphics](Displaying a User Interface/Template Images), [soft buttons](Displaying a User Interface/Template Custom Buttons), and [menu items](Displaying a User Interface/Main Menu) is handled for you behind the scenes. However, you will still need to manually upload your images if you need images in an alert, VR help lists, turn-by-turn directions, or other features not currently covered by the @![iOS]`SDLScreenManager`!@ @![android, javaSE, javaEE]`ScreenManager`!@.
!!!

You should be aware of these four things when using images in your SDL app:

1. You may be connected to a head unit that does not have the ability to display images.
2. You must upload images from your mobile device to the head unit before using them in a template.
3. Persistent images are stored on a head unit between sessions. Ephemeral images are destroyed when a session ends (i.e. when the user turns off their vehicle).
4. Images can not be uploaded when the app's `hmiLevel` is `NONE`. For more information about permissions, please review [Understanding Permissions](Getting Started/Understanding Permissions).

## Checking if Graphics are Supported
Before uploading images to a head unit you should first check if the head unit supports graphics. If not, you should avoid uploading unnecessary image data. To check if graphics are supported, @![iOS]check the `SDLManager.systemCapabilityManager.defaultMainWindowCapability` property once the `SDLManager` has started successfully.!@@![android,javaSE,javaEE] check the `getCapability()` method of a valid `SystemCapabilityManager` obtained from `sdlManager.getSystemCapabilityManager()` to find out the display capabilities of the head unit.!@

@![iOS]
##### Objective-C
```objc
__weak typeof (self) weakSelf = self;
[self.sdlManager startWithReadyHandler:^(BOOL success, NSError * _Nullable error) {
    if (!success) {
        <#Manager errored while starting up#>
        return;
    }

    SDLWindowCapability *mainWindowCapability = weakSelf.sdlManager.systemCapabilityManager.defaultMainWindowCapability;
    BOOL graphicsSupported = (mainWindowCapability.imageFields.count > 0);
}];
```

##### Swift
```swift
sdlManager.start { [weak self] (success, error) in
    guard let self = self else { return }

    guard success else {
        <#Manager errored while starting up#>
        return
    }

    let mainWindowCapability = self.sdlManager.systemCapabilityManager.defaultMainWindowCapability
    let graphicsSupported = (mainWindowCapability.count > 0)
}
```
!@

@![android,javaSE,javaEE]
```java
List<ImageField> imageFields = sdlManager.getSystemCapabilityManager().getDefaultMainWindowCapability().getImageFields();
boolean imagesSuported = (imageFields.size() > 0);
```
!@

## Uploading an Image Using the File Manager
The @![iOS]`SDLFileManager`!@ @![android, javaSE, javaEE]`FileManager`!@ uploads files and keeps track of all the uploaded files names during a session. To send data with the @![iOS]`SDLFileManager`!@ @![android, javaSE, javaEE]`FileManager`!@, you need to create either a @![iOS]`SDLFile`!@ @![android, javaSE, javaEE]`SdlFile`!@ or @![iOS]`SDLArtwork`!@ @![android, javaSE, javaEE]`SdlArtwork`!@object. @![iOS]`SDLFile` objects are created with a local `NSURL` or `NSData`; `SDLArtwork` a `UIImage`.!@ @![android]Both `SdlFile`s and `SdlArtwork`s can be created with a `Uri`, `byte[]`, or `resourceId`.!@ @![javaSE, javaEE]Both `SdlFile`s and `SdlArtwork`s can be created with using `filePath`, or `byte[]`.!@


@![iOS]
##### Objective-C
```objc
UIImage* image = [UIImage imageNamed:@"<#Image Name#>"];
if (!image) {
    <#Error reading from assets#>
    return;
}

SDLArtwork *artwork = [SDLArtwork persistentArtworkWithImage:image asImageFormat:<#SDLArtworkImageFormat#>];

[self.sdlManager.fileManager uploadArtwork:artwork completionHandler:^(BOOL success, NSString * _Nonnull artworkName, NSUInteger bytesAvailable, NSError * _Nullable error) {
    if (error != nil) { return; }
    <#Image Upload Successful#>
    // To send the image as part of a show request, create a SDLImage object using the artworkName
    SDLImage *image = [[SDLImage alloc] initWithName:artworkName isTemplate:<#BOOL#>];
}];
```

##### Swift
```swift
guard let image = UIImage(named: "<#Image Name#>") else {
	<#Error reading from assets#>
	return
}
let artwork = SDLArtwork(image: image, persistent: <#Bool#>, as: <#SDLArtworkImageFormat#>)

sdlManager.fileManager.upload(artwork: artwork) { (success, artworkName, bytesAvailable, error) in
    guard error == nil else { return }
    <#Image Upload Successful#>
    // To send the image as part of a show request, create a SDLImage object using the artworkName
    let graphic = SDLImage(name: artworkName, isTemplate: <#Bool#>)
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
