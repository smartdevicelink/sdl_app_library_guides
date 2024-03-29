# Audio Streaming
A navigation app can stream raw audio to the head unit. This audio data is played immediately. If audio is already playing, the current audio source will be attenuated and your audio will play. Raw audio must be played with the following parameters:

* **Format**: PCM
* **Sample Rate**: 16k
* **Number of Channels**: 1
* **Bits Per Second (BPS)**: 16 bits per sample / 2 bytes per sample

To stream audio from a SDL app, use the @![iOS]`SDLStreamingMediaManager`!@@![android]`AudioStreamingManager`!@ class. A reference to this class is available from the @![iOS]`SDLManager`'s `streamManager`!@@![android]`SdlManager`s `audioStreamManager`!@ property.

### Audio Stream Manager
The @![iOS]`SDLAudioStreamManager`!@@![android]`AudioStreamManager`!@ will help you to do on-the-fly transcoding and streaming of your files in mp3 or other formats, or prepare raw PCM data to be queued and played.

### Starting the Audio Manager
@![iOS]
Like the lifecycle of the video stream, the lifecycle of the audio stream is maintained by the SDL library, therefore, you do not need to start the audio stream if you've set a streaming configuration when starting your SDLManager. When you receive the SDLAudioStreamDidStartNotification, you can begin streaming audio.
!@ 
@![android]
To stream audio, we call `sdlManager.getAudioStreamManager().start()` which will start the manager. When that callback returns with a success, call `sdlManager.getAudioStreamManager().startAudioStream()`. Once this callback returns successfully you can send and play audio.

```java
if (sdlManager.getAudioStreamManager() == null) {
    // Handle the failure
    return;
}

sdlManager.getAudioStreamManager().start(new CompletionListener() {
    @Override
    public void onComplete(boolean success) {
        if (!success) {
            // Failed to start audio streaming manager
            return;
        }
        sdlManager.getAudioStreamManager().startAudioStream(false, new CompletionListener() {
            @Override
            public void onComplete(boolean success) {
                if (!success) {
                    // Failed to start audio stream
                    return;
                }
                // Push Audio Source
            }
        });
    }
});
```
!@

#### Playing from File
@![iOS]
|~
```objc
[self.sdlManager.streamManager.audioManager pushWithFileURL:audioFileURL];
[self.sdlManager.streamManager.audioManager playNextWhenReady];
```
```swift
sdlManager.streamManager?.audioManager.push(withFileURL: url)
sdlManager.streamManager?.audioManager.playNextWhenReady()
```
~|
!@

@![android]
```java
//Push from Uri Audio Source
sdlManager.getAudioStreamManager().pushAudioSource(audioSourceUri, new CompletionListener() {
    @Override
    public void onComplete(boolean success) {
        if (success) {
            DebugTool.logInfo(TAG, "Audio Uri played successfully!");
        } else {
            DebugTool.logInfo(TAG, "Audio Uri failed to play!");
        }
    }
});

//Push from Raw Audio Source
sdlManager.getAudioStreamManager().pushResource(R.raw.exampleMp3, new CompletionListener() {
    @Override
    public void onComplete(boolean success) {
        if (success) {
            DebugTool.logInfo(TAG, "Audio file played successfully!");
        } else {
            DebugTool.logInfo(TAG, "Audio file failed to play!");
        }
    }
});
```
!@

#### Playing from Data
@![iOS]
|~
```objc
[self.sdlManager.streamManager.audioManager pushWithData:audioData];
[self.sdlManager.streamManager.audioManager playNextWhenReady];
```
```swift
sdlManager.streamManager?.audioManager.push(with: audioData)
sdlManager.streamManager?.audioManager.playNextWhenReady()
```
~|
!@

@![android]
```java
//Push from ByteBuffer Audio Source
sdlManager.getAudioStreamManager().pushBuffer(byteBuffer, new CompletionListener() {
    @Override
    public void onComplete(boolean success) {
        if (success) {
            DebugTool.logInfo(TAG, "Buffer played successfully!");
        } else {
            DebugTool.logInfo(TAG, "Buffer failed to play!");
        }
    }
});
```
!@

@![iOS]
#### Implementing the Delegate
|~
```objc
- (void)audioStreamManager:(SDLAudioStreamManager *)audioManager errorDidOccurForFile:(NSURL *)fileURL error:(NSError *)error {

}

- (void)audioStreamManager:(SDLAudioStreamManager *)audioManager errorDidOccurForDataBuffer:(NSError *)error {

}

- (void)audioStreamManager:(SDLAudioStreamManager *)audioManager fileDidFinishPlaying:(NSURL *)fileURL successfully:(BOOL)successfully {
    if (audioManager.queue.count != 0) {
        [audioManager playNextWhenReady];
    }
}

- (void)audioStreamManager:(SDLAudioStreamManager *)audioManager dataBufferDidFinishPlayingSuccessfully:(BOOL)successfully {
    if (audioManager.queue.count != 0) {
        [audioManager playNextWhenReady];
    }
}
```
```swift
func audioStreamManager(_ audioManager: SDLAudioStreamManager, errorDidOccurForFile fileURL: URL, error: Error) {

}

func audioStreamManager(_ audioManager: SDLAudioStreamManager, errorDidOccurForDataBuffer error: Error) {

}

func audioStreamManager(_ audioManager: SDLAudioStreamManager, fileDidFinishPlaying fileURL: URL, successfully: Bool) {
    if audioManager.queue.count != 0 {
        audioManager.playNextWhenReady()
    }
}

func audioStreamManager(_ audioManager: SDLAudioStreamManager, dataBufferDidFinishPlayingSuccessfully successfully: Bool) {
    if audioManager.queue.count != 0 {
        audioManager.playNextWhenReady()
    }
}
```
~|

### Manually Sending Data
Once the audio stream is connected, data may be easily passed to the Head Unit. The function `sendAudioData:` provides us with whether or not the PCM Audio Data was successfully transferred to the Head Unit. If your app is in a state that it is unable to send audio data, this method will return a failure. If successful playback will begin immediately.

|~
```objc
NSData *audioData = <#Acquire Audio Data#>;

if (![self.sdlManager.streamManager sendAudioData:audioData]) {
    <#Could not send audio data#>
}
```
```swift
let audioData = <#Acquire Audio Data#>

guard let streamManager = self.sdlManager.streamManager, streamManager.isAudioConnected else { return }

if !streamManager.sendAudioData(audioData) {
    <#Could not send audio data#>
}
```
~|
!@

@![android]
#### Stopping the Audio Stream
When the stream is complete, or you receive `HMI_NONE`, you should stop the stream by calling:

```java
sdlManager.getAudioStreamManager().stopAudioStream(new CompletionListener() {
    @Override
    public void onComplete(boolean success) {
        // do something once the stream is stopped
    }
});
```
!@
