# Uploading Files
In almost all cases, graphics are uploaded using the `ScreenManager`. You can find out about setting images in templates, soft buttons, and menus in the [Text Images and Buttons](Displaying a User Interface/Text Images and Buttons) guide. Other situations, such as `PerformInteraction`s, VR help lists, and turn by turn directions, are not currently covered by the `ScreenManager`. To upload an image, see the [Uploading Images](Other SDL Features/Uploading Images) guide.

## Uploading an mp3 Using the File Manager
@![iOS]
The `FileManager` uploads files and keeps track of all the uploaded files names during a session. To send data with the file manager you need to create either a @![iOS]`SDLFile`!@ @![android, javaSE, javaEE]`SdlFile`!@ or @![iOS]`SDLArtwork`!@ @![android, javaSE, javaEE]`SdlArtwork`!@ object. @![iOS]`SDLFile` objects are created with a local `NSURL` or `NSData`; `SDLArtwork` uses a `UIImage`.!@
!@

@![iOS]
##### Objective-C
```objc
NSData *mp3Data = <#Get the File Data#>;
SDLFile *file = [SDLFile fileWithData:mp3Data name:<#File name to be referenced later#> fileExtension:<#File Extension#>];

[self.sdlManager.fileManager uploadFile:file completionHandler:^(BOOL success, NSUInteger bytesAvailable, NSError * _Nullable error) {
    if (error != nil) { return; }
    <#File Upload Successful#>
}];]
```

##### Swift
```swift
let mp3Data = <#Get MP3 Data#>
let file = SDLFile(data: mp3Data, name: <#File name#> fileExtension: <#File Extension#>)

sdlManager.fileManager.upload(file: file) { (success, bytesAvailable, error) in
    guard error == nil else { return }
    <#File Upload Successful#>
}
```
!@

@![android, javaSE, javaEE]
`// TODO: Android / Java content`
!@

## Batching File Uploads
If you want to upload a group of files, you can use the @![iOS]`SDLFileManager`'s!@ @![android, javaSE, javaEE]`FileManager`!@ batch upload methods. Once all of the uploads have completed you will be notified if any of the uploads failed. If desired, you can also track the progress of each file in the group.

@![iOS]
##### Objective-C
```objc
SDLFile *file1 = [SDLFile fileWithData:<#Data#> name:<#File name to be referenced later#> fileExtension:<#File Extension#>];
SDLFile *file2 = [SDLFile fileWithData:<#Data#> name:<#File name to be referenced later#> fileExtension:<#File Extension#>];

[self.sdlManager.fileManager uploadFiles:@[file1, file2] progressHandler:^BOOL(NSString * _Nonnull fileName, float uploadPercentage, NSError * _Nullable error) {
    // A single file has finished uploading. Use this to check for individual errors, to use an file as soon as its uploaded, or to check the progress of the upload
    // The upload percentage is calculated as the total file size of all attempted file uploads (regardless of the successfulness of the upload) divided by the sum of the data in all the files
    // Return YES to continue sending files. Return NO to cancel any files that have not yet been sent.
} completionHandler:^(NSArray<NSString *> * _Nonnull fileNames, NSError * _Nullable error) {
    // All files have completed uploading.
    // If all files were uploaded successfully, the error will be nil
    // The error's userInfo parameter is of type [fileName: error message]
}];
```

##### Swift
```swift
let file1 = SDLFile(data: <#File Data#>, name: <#File name#> fileExtension: <#File Extension#>)
let file2 = SDLFile(data: <#File Data#>, name: <#File name#> fileExtension: <#File Extension#>)

sdlManager.fileManager.upload(files: [file1, file2], progressHandler: { (fileName, uploadPercentage, error) -> Bool in
    // A single file has finished uploading. Use this to check for individual errors, to use an file as soon as its uploaded, or to check the progress of the upload
    // The upload percentage is calculated as the total file size of all attempted file uploads (regardless of the successfulness of the upload) divided by the sum of the data in all the files
    // Return true to continue sending files. Return false to cancel any files that have not yet been sent.
}) { (fileNames, error) in
    // All files have completed uploading.
    // If all files were uploaded successfully, the error will be nil
    // The error's userInfo parameter is of type [fileName: error message]
}
```
!@

@![android, javaSE, javaEE]
The first step in uploading files to the connected module is creating an instance of `SdlFile`. There are a few different constructors that can be used based on the source of the file. The following can be used to instantiate `SdlFile`:

##### Resource ID
```java
new SdlFile(@NonNull String fileName, @NonNull FileType fileType, int id, boolean persistentFile)
```
##### URI
```java
new SdlFile(@NonNull String fileName, @NonNull FileType fileType, Uri uri, boolean persistentFile)
```

##### Byte Array
```java
new SdlFile(@NonNull String fileName, @NonNull FileType fileType, byte[] data, boolean persistentFile)
```

To upload more than one file simply create a `List<SdlFile>` object, add your files, and then call:

```java
sdlManager.getFileManager().uploadFiles(sdlFileList, new MultipleFileCompletionListener() {
    @Override
    public void onComplete(Map<String, String> errors) {
                            
    }
});
```
!@

## File Persistance
@![iOS]`SDLFile`!@ @![android, javaSE, javaEE]`SdlFile`!@ and its subclass @![iOS]`SDLArtwork`!@ @![android, javaSE, javaEE]`SdlArtwork`!@ support uploading persistant files, i.e. files that are not deleted when the car turns off. Persistance should be used for files that will be used every time the user opens the app. If the file is only displayed for short time the file should not be persistant because it will take up unnecessary space on the head unit. You can check the persistence via:

@![iOS]
##### Objective-C
```objc
if (file.isPersistent) {
    <#File was initialized as persistent#>
}
```

##### Swift
```swift
if file.isPersistent {
    <#File was initialized as persistent#>
}
```
!@

@![android, javaSE, javaEE]
`// TODO: Android / Java content`
!@

!!! NOTE
Be aware that persistance will not work if space on the head unit is limited. @![iOS]`SDLFileManager`!@ @![android, javaSE, javaEE]`FileManager`!@ will always handle uploading images if they are non-existent.
!!!

## Overwriting Stored Files
If a file being uploaded has the same name as an already uploaded file, the new file will be ignored. To override this setting, set the @![iOS]`SDLFile`’s!@ @![android, javaSE, javaEE]`SdlFile`!@ `overwrite` property to true.

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
`// TODO: Android / Java content`
!@

## Checking the Amount of File Storage
To find the amount of file storage left for your app on the head unit, use the @![iOS]`SDLFileManager`’s!@ @![android, javaSE, javaEE]`FileManager`'s!@ `bytesAvailable` property.

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
`// TODO: Android / Java content`
!@

## Checking if a File Has Already Been Uploaded
You can check out if an image has already been uploaded to the head unit via the @![iOS]`SDLFileManager`'s!@ @![android, javaSE, javaEE]`FileManager`'s!@ `remoteFileNames` property.

@![iOS]
##### Objective-C
```objc
BOOL isFileOnHeadUnit = [self.sdlManager.fileManager.remoteFileNames containsObject:@"<#Name#>"];
```

##### Swift
```swift
if let fileIsOnHeadUnit = sdlManager.fileManager.remoteFileNames.contains("<#Name Uploaded As#>") {
    if fileIsOnHeadUnit {
        <#File exists#>
    } else {
        <#File does not exist#>
    }
}
```
!@

@![android, javaSE, javaEE]
`// TODO: Android / Java content`
!@

## Deleting Stored Files
Use the file manager’s delete request to delete a file associated with a file name.

@![iOS]
##### Objective-C
```objc
[self.sdlManager.fileManager deleteRemoteFileWithName:@"<#Save As Name#>" completionHandler:^(BOOL success, NSUInteger bytesAvailable, NSError *error) {
    if (success) {
        <#Image was deleted successfully#>
    }
}];
```

##### Swift
```swift
sdlManager.fileManager.delete(fileName: "<#Save As Name#>") { (success, bytesAvailable, error) in
    if success {
        <#Image was deleted successfully#>
    }
}
```
!@

@![android, javaSE, javaEE]
sdlManager.getFileManager().deleteRemoteFileWithName("testFile", new CompletionListener() {
	@Override
	public void onComplete(boolean success) {
				
	}
});
!@

## Batch Deleting Files
@![iOS]
##### Objective-C
```objc
[self.sdlManager.fileManager deleteRemoteFileWithNames:@[@"<#Save As Name#>", @"<#Save As Name 2#>"] completionHandler:^(NSError *error) {
    if (error == nil) {
        <#Images were deleted successfully#>
    }
}];
```

##### Swift
```swift
sdlManager.fileManager.delete(fileNames: ["<#Save As Name#>", "<#Save as Name 2#>"]) { (error) in
    if (error == nil) {
        <#Images were deleted successfully#>
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
