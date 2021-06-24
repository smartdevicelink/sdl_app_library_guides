# Playing Audio Indications (RPC v5.0+)
You can pass an uploaded audio file's name to @![iOS]`SDLTTSChunk`!@@![android, javaSE, javaEE, javascript]`TTSChunk`!@, allowing any API that takes a text-to-speech parameter to pass and play your audio file. A sports app, for example, could play a distinctive audio chime to notify the user of a score update alongside an `Alert` request.

## Uploading the Audio File
The first step is to make sure the audio file is available on the remote system. To upload the file use the @![iOS]`SDLFileManager`!@@![android, javaSE, javaEE, javascript]`FileManager`!@.

@![iOS]
|~
```objc
SDLFile *audioFile = [[SDLFile alloc] initWithFileURL:<#File location on disk#> name:<#Audio file name#> persistent:<#True if the file will be used beyond just this session#>];
[self.sdlManager.fileManager uploadFile:audioFile completionHandler:^(BOOL success, NSUInteger bytesAvailable, NSError * _Nullable error) {
    <#audio file is ready if success is true#>
}];
```
```swift
let audioFile = SDLFile(fileURL: <#File location on disk#>, name: <#Audio file name#>, persistent: <#True if the file will be used beyond just this session#>)
sdlManager.fileManager.upload(file: audioFile) { (success, bytesAvailable, error) in
    <#audio file is ready if success is true#>
}
```
~|
!@

@![android, javaSE, javaEE]
```java
SdlFile audioFile = new SdlFile("Audio file name", FileType.AUDIO_MP3, fileUri, true);
sdlManager.getFileManager().uploadFile(audioFile, new CompletionListener() {
	@Override
	public void onComplete(boolean success) {

	}
});
```
!@

@![javascript]
```js
const audioFile = new SDL.manager.file.filetypes.SdlFile('Audio file name', SDL.rpc.enums.FileType.AUDIO_MP3, <#File Data#>, true);
const success = await sdlManager.getFileManager().uploadFile(audioFile)
```
!@

For more information about uploading files, see the [Uploading Files guide](Other SDL Features/Uploading Files).

## Using the Audio File
Now that the file is uploaded to the remote system, it can be used in various RPCs, such as `Speak`, `Alert`, and `AlertManeuver`. To use the audio file in an alert, you simply need to construct a @![iOS]`SDLTTSChunk`!@@![android, javaSE, javaEE, javascript]`TTSChunk`!@ referring to the file's name.

@![iOS]
|~
```objc
SDLAlert *alert = [[SDLAlert alloc] init];
alert.ttsChunks = [SDLTTSChunk fileChunksWithName:<#Audio file name#>];
[self.sdlManager sendRequest:alert];
```
```swift
let alert = SDLAlert()
alert.ttsChunks = SDLTTSChunk.fileChunks(withName: <#Audio file name#>)
sdlManager.send(alert)
```
~|
!@

@![android, javaSE, javaEE]
```java
Alert alert = new Alert()
    .setAlertText1("Alert Text 1")
    .setAlertText2("Alert Text 2")
    .setDuration(5000)
    .setTtsChunks(Arrays.asList(new TTSChunk("Audio file name", SpeechCapabilities.FILE)));
sdlManager.sendRPC(alert);
```
!@

@![javascript]
```js
const alert = new SDL.rpc.messages.Alert();
alert.setAlertText1('Alert Text 1');
alert.setAlertText2('Alert Text 2');
alert.setDuration(5000);
alert.setTtsChunks([new SDL.rpc.structs.TTSChunk().setText('Audio file name').setType(SDL.rpc.enums.SpeechCapabilities.FILE)]);
// sdl_javascript_suite v1.1+
sdlManager.sendRpcResolve(alert);
// Pre sdl_javascript_suite v1.1
sdlManager.sendRpc(alert);
```
!@
