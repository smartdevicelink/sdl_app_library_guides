# Audio Streaming
A navigation app can stream raw audio to the head unit. This audio data is played immediately. If audio is already playing, the current audio source will be attenuated and your audio will play. Raw audio must be played with the following parameters:

* **Format**: PCM
* **Sample Rate**: 16k
* **Number of Channels**: 1
* **Bits Per Second (BPS)**: 16 bits per sample / 2 bytes per sample

To stream audio from a SDL app, use the @![iOS]`SDLStreamingMediaManager`!@@![android]`AudioStreamingManager`!@ class. A reference to this class is available from the @![iOS]`SDLManager`'s `streamManager`!@@![android]`SdlManager`s `audioStreamManager`!@ property.

@![iOS]
## Audio Stream Lifecycle
Like the lifecycle of the video stream, the lifecycle of the audio stream is maintained by the SDL library. When you receive the `SDLAudioStreamDidStartNotification`, you can begin streaming audio.

### Audio Stream Manager
The `SDLAudioStreamManager` will help you to do on-the-fly transcoding and streaming of your files in mp3 or other formats, or prepare raw PCM data to be queued and played.

#### Playing from File
##### Objective-C
```objc
[self.sdlManager.streamManager.audioManager pushWithFileURL:audioFileURL];
[self.sdlManager.streamManager.audioManager playNextWhenReady];
```

##### Swift
```swift
sdlManager.streamManager?.audioManager.push(withFileURL: url)
sdlManager.streamManager?.audioManager.playNextWhenReady()
```

#### Playing from Data
##### Objective-C
```objc
[self.sdlManager.streamManager.audioManager pushWithData:audioData];
[self.sdlManager.streamManager.audioManager playNextWhenReady];
```

##### Swift
```swift
sdlManager.streamManager?.audioManager.push(with: audioData)
sdlManager.streamManager?.audioManager.playNextWhenReady()
```

#### Implementing the Delegate
##### Objective-C
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

##### Swift
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

### Manually Sending Data
Once the audio stream is connected, data may be easily passed to the Head Unit. The function `sendAudioData:` provides us with whether or not the PCM Audio Data was successfully transferred to the Head Unit. If your app is in a state that it is unable to send audio data, this method will return a failure. If successful playback will begin immediately.

##### Objective-C
```objective-c

NSData *audioData = <#Acquire Audio Data#>;

if (![self.sdlManager.streamManager sendAudioData:audioData]) {
    <#Could not send audio data#>
}
```

##### Swift
```swift
let audioData = <#Acquire Audio Data#>

guard let streamManager = self.sdlManager.streamManager, streamManager.isAudioConnected else { return }

if !streamManager.sendAudioData(audioData) {
    <#Could not send audio data#>
}
```
!@

@![android]
To stream audio, we call `sdlManager.getAudioStreamManager().start()` which will start the manager. When that callback returns successful, you call `sdlManager.getAudioStreamManager().startAudioStream()`. When the callback for that is successful, you can push the audio source using `sdlManager.getAudioStreamManager().pushResource()`. Below is an example of playing an `mp3` file that we have in our resource directory:

```java
if (sdlManager.getAudioStreamManager() != null) {
    DebugTool.logInfo(TAG, "Trying to start audio streaming");
    sdlManager.getAudioStreamManager().start(new CompletionListener() {
        @Override
        public void onComplete(boolean success) {
            if (success) {
                sdlManager.getAudioStreamManager().startAudioStream(false, new CompletionListener() {
                    @Override
                    public void onComplete(boolean success) {
                        if (success) {
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
                        } else {
                            DebugTool.logInfo(TAG, "Audio stream failed to start!");
                        }
                    }
                });
            } else {
                DebugTool.logInfo(TAG, "Failed to start audio streaming manager");
            }
        }
    });
}
```

#### Using a Buffer

You can also send `ByteBuffer`s to the `AudioStreamManager` to be played. To use it, replace the `pushResource` call in the example above to the `pushBuffer` call shown below:

```java
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
