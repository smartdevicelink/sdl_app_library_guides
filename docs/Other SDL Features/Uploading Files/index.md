# Uploading Files
In almost all cases, you will not need to handle uploading images because the screen manager API will do that for you. There are some situations, such as VR help-lists and turn-by-turn directions, that are not currently covered by the screen manager so you will have manually upload the image yourself in those cases. For more information about uploading images, see the [Uploading Images](Other SDL Features/Uploading Images) guide.


## Uploading an MP3 Using the File Manager
The @![iOS]`SDLFileManager`!@ @![android, javaSE, javaEE]`FileManager`!@ uploads files and keeps track of all the uploaded files names during a session. To send data with the file manager you need to create either a @![iOS]`SDLFile`!@ @![android, javaSE, javaEE]`SdlFile`!@ or @![iOS]`SDLArtwork`!@ @![android, javaSE, javaEE]`SdlArtwork`!@ object. @![iOS]`SDLFile` objects are created with a local `NSURL` or `NSData`; `SDLArtwork` uses a `UIImage`.!@ @![android]Both `SdlFile`s and `SdlArtwork`s can be created with a `Uri`, `byte[]`, or `resourceId`.!@ @![javaSE, javaEE]Both `SdlFile`s and `SdlArtwork`s can be created with using `filePath`, or `byte[]`.!@

@![iOS]
##### Objective-C
```objc
NSData *mp3Data = <#Get the File Data#>;
SDLFile *audioFile = [SDLFile fileWithData:mp3Data name:<#File name to be referenced later#> fileExtension:<#File Extension#>];

[self.sdlManager.fileManager uploadFile:audioFile completionHandler:^(BOOL success, NSUInteger bytesAvailable, NSError * _Nullable error) {
    if (error != nil) { return; }
    <#File Upload Successful#>
}];]
```

##### Swift
```swift
let mp3Data = <#Get MP3 Data#>
let audioFile = SDLFile(data: mp3Data, name: <#File name#> fileExtension: <#File Extension#>)

sdlManager.fileManager.upload(file: audioFile) { (success, bytesAvailable, error) in
    guard error == nil else { return }
    <#File Upload Successful#>
}
```
!@

@![android, javaSE, javaEE]
```java
byte[] mp3Data = Get the file data;
SdlFile audioFile = new SdlFile("File Name", FileType.AUDIO_MP3, mp3Data, true);
sdlManager.sendRPC(audioFile);
```
!@

## Batching File Uploads
If you want to upload a group of files, you can use the @![iOS]`SDLFileManager`'s!@ @![android, javaSE, javaEE]`FileManager`!@ batch upload methods. Once all of the uploads have completed you will be notified if any of the uploads failed. @![iOS]If desired, you can also track the progress of each file in the group.!@

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
```java
sdlManager.getFileManager().uploadFiles(sdlFileList, new MultipleFileCompletionListener() {
    @Override
    public void onComplete(Map<String, String> errors) {
                            
    }
});
```
!@

## File Persistance
@![iOS]`SDLFile`!@ @![android, javaSE, javaEE]`SdlFile`!@ and its subclass @![iOS]`SDLArtwork`!@ @![android, javaSE, javaEE]`SdlArtwork`!@ support uploading persistant files, i.e. files that are not deleted when the car turns off. Persistance should be used for files that will be used every time the user opens the app. If the file is only displayed for short time the file should not be persistent because it will take up unnecessary space on the head unit. You can check the persistence via:

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

!!! NOTE
Be aware that persistance will not work if space on the head unit is limited. The @![iOS]`SDLFileManager`!@ @![android, javaSE, javaEE]`FileManager`!@ will always handle uploading images if they are non-existent.
!!!

@![iOS]
## Overwriting Stored Files
If a file being uploaded has the same name as an already uploaded file, the new file will be ignored. To override this setting, set the `SDLFile`'s `overwrite` property to true.

##### Objective-C
```objc
file.overwrite = YES;
```

##### Swift
```swift
file.overwrite = true
```
!@

## Checking the Amount of File Storage Left
To find the amount of file storage left for your app on the head unit, use the @![iOS]`SDLFileManager`’s `bytesAvailable` property!@ @![android, javaSE, javaEE]`ListFiles` RPC!@.

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
ListFiles listFiles = new ListFiles();
listFiles.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse response) {
        if(response.getSuccess()){
            Integer spaceAvailable = ((ListFilesResponse) response).getSpaceAvailable();
            Log.i("SdlService", "Space available on Core = " + spaceAvailable);
        }else{
            Log.i("SdlService", "Failed to request list of uploaded files.");
        }
    }
});

sdlManager.sendRPC(listFiles);
```
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
let fileIsOnHeadUnit = sdlManager.fileManager.remoteFileNames.contains("<#Name#>")
```
!@

@![android, javaSE, javaEE]
```java
Boolean fileIsOnHeadUnit = sdlManager.getFileManager().getRemoteFileNames().contains("Name")
```
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
```java
sdlManager.getFileManager().deleteRemoteFileWithName("testFile", new CompletionListener() {
	@Override
	public void onComplete(boolean success) {
				
	}
});
```
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
