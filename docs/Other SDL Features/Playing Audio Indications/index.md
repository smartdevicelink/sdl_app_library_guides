# Playing Audio Indications
As of library v.@![iOS]6.1!@ @![android, javaSE, javaEE]4.7!@ and SDL Core v.5.0+, you can pass an uploaded audio file's name to @![iOS]`SDLTTSChunk`!@ @![android, javaSE, javaEE]`TTSChunk`!@, allowing any API that takes a text-to-speech parameter to pass and play your audio file. This can be used, for example, to play a distinctive audio chime that lets the user know that an action has occurred. A sports app for example, could use this to notify the user of a score update alongside an `Alert` request.

## Uploading the Audio File
The first step is to make sure the audio file is available on the remote system. To upload the file use the @![iOS]`SDLFileManager`!@ @![android, javaSE, javaEE]`FileManager`!@.

@![iOS]
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
!@

@![android, javaSE, javaEE]
```java
SdlFile audioFile = new SdlFile("Audio File Name", FileType.AUDIO_MP3, Uri.parse("File Location"), true);
sdlManager.getFileManager().uploadFile(audioFile, new CompletionListener() {
	@Override
	public void onComplete(boolean success) {

	}
});
```
!@

For more information about uploading files, see the [Uploading Files guide](Other SDL Features/Uploading Files).

## Using the Audio File in an Alert
Now that the file is uploaded to the remote system, it can be used in various APIs, such as `Speak`, `Alert`, `AlertManeuver`, and `PerformInteraction`. To use the audio file in an alert, you simply need to construct a @![iOS]`SDLTTSChunk`!@ @![android, javaSE, javaEE]`TTSChunk`!@ referring to the file's name.

![iOS]
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
!@

@![android, javaSE, javaEE]
```java
Alert alert = new Alert();
alert.setAlertText1("Alert Text 1");
alert.setAlertText2("Alert Text 2");
alert.setDuration(5000);
alert.setTtsChunks(Arrays.asList(new TTSChunk("Audio File Name", SpeechCapabilities.FILE)));
```
!@