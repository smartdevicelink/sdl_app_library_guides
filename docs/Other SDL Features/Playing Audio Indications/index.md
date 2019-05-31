# Playing Audio Indications
As of SDL v6.1, you can pass an uploaded Audio File's name to `TTSChunk`, allowing any API that takes a `TTSChunk` to pass and play your audio file. This can be used, for example, to play a distinctive audio chime or indication unique to your application, letting the user know that something has occurred. A sports app, for example, could use this to notify the user of a score update alongside an `Alert` request.

!!! NOTE
Only SDL systems v.5.0+ support this feature.
!!!

## Uploading the Audio File
The first step is to make sure the audio file is available on the remote system. To do this, you use the `SDLFileManager`.

##### Objective-C
```objc
SDLFile *audioFile = [[SDLFile alloc] initWithFileURL:<#(File location on disk)#> name:<#(Audio file's reference for usage)#> persistent:<#(True if the file is generic beyond just this session)#>];
[self.sdlManager.fileManager uploadFile:audioFile completionHandler:^(BOOL success, NSUInteger bytesAvailable, NSError * _Nullable error) {
    <#(audio file is ready if success is true)#>
}];
```

##### Swift
```swift
let audioFile = SDLFile(fileURL: <#File Location on disk#>, name: <#Audio file's reference for usage#>, persistent: <#True if the file is generic beyond just this session#>)
sdlManager.fileManager.upload(file: audioFile) { (success, bytesAvailable, error) in
    <#audio file is ready if success is true#>
}
```

For more information about uploading files, see [the relevant guide](Other SDL Features/Uploading Files).

## Using the Audio File in an Alert
Now that the file is uploaded to the remote system, it can be used in various APIs, such as `Speak`, `Alert`, `AlertManeuver`, `PerformInteraction`. To use the audio file in an alert, you simply need to construct a `TTSChunk` referring to the file's name.

##### Objective-C
```objc
SDLAlert *alert = [[SDLAlert alloc] initWithAlertText1:<#(nullable NSString *)#> alertText2:<#(nullable NSString *)#> duration:<#(UInt16)#>];
alert.ttsChunks = [SDLTTSChunk fileChunksWithName:<#(File's name)#>];
[self.sdlManager sendRequest:alert];
```

##### Swift
```swift
let alert = SDLAlert(alertText1: <#T##String?#>, alertText2: <#T##String?#>, duration: <#T##UInt16#>)
alert.ttsChunks = SDLTTSChunk.fileChunks(withName: <#File's name#>)
sdlManager.send(alert)
```