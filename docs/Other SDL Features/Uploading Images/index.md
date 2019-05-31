# Uploading Images

!!! NOTE
If you are looking to upload images for use in template graphics, soft buttons, or the menu, you can use the [ScreenManager](Displaying a User Interface/Text Images and Buttons). Other situations, such as VR help lists and turn by turn directions, are not currently covered by the `ScreenManager`.
!!!

You should be aware of these four things when using images in your SDL app:

1. You may be connected to a head unit that does not have the ability to display images.
2. You must upload images from your mobile device to the head unit before using them in a template.
3. Persistant images are stored on a head unit between sessions. Ephemeral images are destroyed when a session ends (i.e. when the user turns off their vehicle).
4. Images can not be uploaded when the app's `hmiLevel` is `NONE`. For more information about permissions, please review [Understanding Permissions](Getting Started/Understanding Permissions).

To learn how to use images once they are uploaded, please see [Text, Images, and Buttons](Displaying a User Interface/Text Images and Buttons).

## Checking if Graphics are Supported
Before uploading images to a head unit you should first check if the head unit supports graphics. If not, you should avoid uploading unneccessary image data. To check if graphics are supported look at the `SDLManager`'s `registerResponse` property once the `SDLManager` has started successfully.

##### Objective-C
```objc
__weak typeof (self) weakSelf = self;
[self.sdlManager startWithReadyHandler:^(BOOL success, NSError * _Nullable error) {
    if (!success) {
        NSLog(@"SDL errored starting up: %@", error);
        return;
    } 

    SDLDisplayCapabilities *displayCapabilities = weakSelf.sdlManager.registerResponse.displayCapabilities;
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
        print("SDL errored starting up: \(error.debugDescription)")
        return
    }
    
    var areGraphicsSupported = false
    if let displayCapabilities = self?.sdlManager.registerResponse?.displayCapabilities {
        areGraphicsSupported = displayCapabilities.graphicSupported.boolValue
    }
}
```

## Uploading an Image Using SDLFileManager
The `SDLFileManager` uploads files and keeps track of all the uploaded files names during a session. To send data with the `SDLFileManager`, you need to create either a `SDLFile` or `SDLArtwork` object. `SDLFile` objects are created with a local `NSURL` or `NSData`; `SDLArtwork` a `UIImage`.

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

### Batch File Uploads, Persistence, etc.
Similar to other files, artworks can be persistent, batched, overwrite, etc. See [Uploading Files](Other SDL Features/Uploading Files)
