# Uploading Files
In almost all cases, you will not need to handle uploading images because the screen manager API will do that for you. There are some situations, such as VR help-lists and turn-by-turn directions, that are not currently covered by the screen manager so you will have manually upload the image yourself in those cases. For more information about uploading images, see the [Uploading Images](Other SDL Features/Uploading Images) guide.


## Uploading an MP3 Using the File Manager
The @![iOS]`SDLFileManager`!@ @![android, javaSE, javaEE, javascript]`FileManager`!@ uploads files and keeps track of all the uploaded files names during a session. To send data with the file manager you need to create either a @![iOS]`SDLFile`!@ @![android, javaSE, javaEE, javascript]`SdlFile`!@ or @![iOS]`SDLArtwork`!@ @![android, javaSE, javaEE, javascript]`SdlArtwork`!@ object. @![iOS]`SDLFile` objects are created with a local `NSURL` or `NSData`; `SDLArtwork` uses a `UIImage`.!@ @![android]Both `SdlFile`s and `SdlArtwork`s can be created with a `Uri`, `byte[]`, or `resourceId`.!@ @![javaSE, javaEE]Both `SdlFile`s and `SdlArtwork`s can be created with using `filePath`, or `byte[]`.!@@![javascript]Both `SdlFile`s and `SdlArtwork`s can be created with using `filePath`, or `String`.!@

@![iOS]
##### Objective-C
```objc
NSData *mp3Data = <#Get the File Data#>;
SDLFile *audioFile = [SDLFile fileWithData:mp3Data name:<#File name#> fileExtension:<#File Extension#>];

[self.sdlManager.fileManager uploadFile:audioFile completionHandler:^(BOOL success, NSUInteger bytesAvailable, NSError * _Nullable error) {
    if (error != nil) { return; }
    <#File upload successful#>
}];]
```

##### Swift
```swift
let mp3Data = <#Get MP3 Data#>
let audioFile = SDLFile(data: mp3Data, name: <#File name#>, fileExtension: <#File Extension#>)

sdlManager.fileManager.upload(file: audioFile) { (success, bytesAvailable, error) in
    guard error == nil else { return }
    <#File upload successful#>
}
```
!@

@![android, javaSE, javaEE]
```java
byte[] mp3Data = <#Get the File Data#>;
SdlFile audioFile = new SdlFile("File Name", FileType.AUDIO_MP3, mp3Data, true);
sdlManager.getFileManager().uploadFile(audioFile, new CompletionListener() {
    @Override
    public void onComplete(boolean success) {
        if (success) {
            <#File upload successful#>
        }
    }
});
```
!@

@![javascript]
```js
const audioFile = new SDL.manager.file.filetypes.SdlFile('File Name', SDL.rpc.enums.FileType.AUDIO_MP3, <#Audio byte array data as a string#>, true);
const success = await sdlManager.getFileManager().uploadFile(audioFile)
    .catch(error => {
        // handle errors here
        return false;
    });
if (success) {
    <#File upload successful#>
}
```
!@

## Batching File Uploads
If you want to upload a group of files, you can use the @![iOS]`SDLFileManager`'s!@ @![android, javaSE, javaEE, javascript]`FileManager`!@ batch upload methods. Once all of the uploads have completed you will be notified if any of the uploads failed. @![iOS]If desired, you can also track the progress of each file in the group.!@

@![iOS]
##### Objective-C
```objc
SDLFile *file1 = [SDLFile fileWithData:<#Data#> name:<#File name to be referenced later#> fileExtension:<#File Extension#>];
SDLFile *file2 = [SDLFile fileWithData:<#Data#> name:<#File name#> fileExtension:<#File Extension#>];

[self.sdlManager.fileManager uploadFiles:@[file1, file2] progressHandler:^BOOL(NSString * _Nonnull fileName, float uploadPercentage, NSError * _Nullable error) {
    <#Called as each upload completes#>
    // Return true to continue sending files. Return false to cancel any files that have not yet been sent.
    return YES;
} completionHandler:^(NSArray<NSString *> * _Nonnull fileNames, NSError * _Nullable error) {
    <#Called when all uploads complete#>
}];
```

##### Swift
```swift
let file1 = SDLFile(data: <#File Data#>, name: <#File name#> fileExtension: <#File Extension#>)
let file2 = SDLFile(data: <#File Data#>, name: <#File name#> fileExtension: <#File Extension#>)

sdlManager.fileManager.upload(files: [file1, file2], progressHandler: { (fileName, uploadPercentage, error) -> Bool in
    <#Called as each upload completes#>
    // Return true to continue sending files. Return false to cancel any files that have not yet been sent.
    return true
}) { (fileNames, error) in
    <#Called when all uploads complete#>
}
```
!@

@![android, javaSE, javaEE]
```java
sdlManager.getFileManager().uploadFiles(sdlFileList, new MultipleFileCompletionListener() {
    @Override
    public void onComplete(Map<String, String> errors) {

    }
});
```
!@

@![javascript]
```js
const files = await sdlManager.getFileManager().uploadFiles(sdlFileList)
    .catch(err => {
        // handle errors here
    });
```
!@

## File Persistence
@![iOS]`SDLFile`!@ @![android, javaSE, javaEE, javascript]`SdlFile`!@ and its subclass @![iOS]`SDLArtwork`!@ @![android, javaSE, javaEE, javascript]`SdlArtwork`!@ support uploading persistent files, i.e. files that are not deleted when the car turns off. Persistence should be used for files that will be used every time the user opens the app. If the file is only displayed for short time the file should not be persistent because it will take up unnecessary space on the head unit. You can check the persistence via:

@![iOS]
##### Objective-C
```objc
BOOL isPersistent = file.isPersistent;
```

##### Swift
```swift
let isPersistent = file.isPersistent;
```
!@

@![android, javaSE, javaEE]
```java
Boolean isPersistent = file.isPersistent();
```
!@

@![javascript]
```js
const isPersistent = file.isPersistent();
```
!@

!!! NOTE
Be aware that persistence will not work if space on the head unit is limited. The @![iOS]`SDLFileManager`!@ @![android, javaSE, javaEE, javascript]`FileManager`!@ will always handle uploading images if they are non-existent.
!!!


## Overwriting Stored Files
@![iOS, android, javaEE, javaSE]
If a file being uploaded has the same name as an already uploaded file, the new file will be ignored. To override this setting, set the !@@![iOS]`SDLFile`!@@![android, javaSE, javaEE]`SdlFile`!@@![iOS, android, javaEE, javaSE]'s `overwrite` property to `true`.
!@

@![iOS]
##### Objective-C
```objc
file.overwrite = YES;
```

##### Swift
```swift
file.overwrite = true
```
!@

@![android, javaSE, javaEE]
```java
file.setOverwrite(true);
```
!@

@![javascript]
If a file being uploaded has the same name as an already uploaded file, the existing file will be overwritten. Changing overwriting properties is not currently supported.
!@

## Checking the Amount of File Storage Left
To find the amount of file storage left for your app on the head unit, use the @![iOS]`SDLFileManager`’s `bytesAvailable` property!@ @![android, javaSE, javaEE, javascript]`ListFiles` RPC!@.

@![iOS]
##### Objective-C
```objc
NSUInteger bytesAvailable = self.sdlManager.fileManager.bytesAvailable;
```

##### Swift
```swift
let bytesAvailable = sdlManager.fileManager.bytesAvailable
```
!@

@![android, javaSE, javaEE]
```java
int bytesAvailable = sdlManager.getFileManager().getBytesAvailable();
```
!@

@![javascript]
```js
const bytesAvailable = sdlManager.getFileManager().getBytesAvailable();
```
!@

## Checking if a File Has Already Been Uploaded
You can check out if an image has already been uploaded to the head unit via the @![iOS]`SDLFileManager`'s!@@![android, javaSE, javaEE, javascript]`FileManager`'s!@ `remoteFileNames` property.

@![iOS]
##### Objective-C
```objc
BOOL isFileOnHeadUnit = [self.sdlManager.fileManager.remoteFileNames containsObject:<#Name Uploaded As#>];
```

##### Swift
```swift
let isFileOnHeadUnit = sdlManager.fileManager.remoteFileNames.contains(<#Name Uploaded As#>)
```
!@

@![android, javaSE, javaEE]
```java
Boolean fileIsOnHeadUnit = sdlManager.getFileManager().getRemoteFileNames().contains("Name Uploaded As");
```
!@

@![javascript]
```js
const fileIsOnHeadUnit = sdlManager.getFileManager().getRemoteFileNames().includes('<#File Name#>');
```
!@

## Deleting Stored Files
Use the file manager’s delete request to delete a file associated with a file name.

@![iOS]
##### Objective-C
```objc
[self.sdlManager.fileManager deleteRemoteFileWithName:@"<#Name Uploaded As#>" completionHandler:^(BOOL success, NSUInteger bytesAvailable, NSError *error) {
    if (success) {
        <#File was deleted successfully#>
    }
}];
```

##### Swift
```swift
sdlManager.fileManager.delete(fileName: "<#Name Uploaded As#>") { (success, bytesAvailable, error) in
    if success {
        <#File was deleted successfully#>
    }
}
```
!@

@![android, javaSE, javaEE]
```java
sdlManager.getFileManager().deleteRemoteFileWithName("Name Uploaded As", new CompletionListener() {
	@Override
	public void onComplete(boolean success) {

	}
});
```
!@

@![javascript]
```js
const success = await sdlManager.getFileManager().deleteRemoteFileWithName('<#File Name#>');
```
!@

## Batch Deleting Files
@![iOS]
##### Objective-C
```objc
[self.sdlManager.fileManager deleteRemoteFileWithNames:@[@"<#Name Uploaded As#>", @"<#Name Uploaded As 2#>"] completionHandler:^(NSError *error) {
    if (error == nil) {
        <#Images were deleted successfully#>
    }
}];
```

##### Swift
```swift
sdlManager.fileManager.delete(fileNames: ["<#Name Uploaded As#>", "<#Name Uploaded As 2#>"]) { (error) in
    if (error == nil) {
        <#Files were deleted successfully#>
    }
}
```
!@

@![android, javaSE, javaEE]
```java
sdlManager.getFileManager().deleteRemoteFilesWithNames(remoteFiles, new MultipleFileCompletionListener() {
	@Override
	public void onComplete(Map<String, String> errors) {

	}
});
```
!@

@![javascript]
```js
const successes = await sdlManager.getFileManager().deleteRemoteFilesWithNames(remoteFiles);
```
!@
