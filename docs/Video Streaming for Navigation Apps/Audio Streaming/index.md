# Audio Streaming
Navigation apps are allowed to stream raw audio to be played by the head unit. The audio received this way is played immediately, and the current audio source will be attenuated. The raw audio has to be played with the following parameters:

* **Format**: PCM
* **Sample Rate**: 16k
* **Number of Channels**: 1
* **Bits Per Second (BPS)**: 16 bits per sample / 2 bytes per sample

In order to stream audio from a SDL app, we focus on the @![iOS]`SDLStreamingMediaManager`!@@![android]`AudioStreamingManager`!@ class. @![iOS]A reference to this class is available from an `SDLManager` property `streamManager`!@@![android]The `AudioStreamingManager` object can we obtained from `SdlManager`s `getAudioStreamManager()`!@.

@![iOS]
## Audio Stream Lifecycle
Like the lifecycle of the video stream, the lifecycle of the audio stream is maintained by the SDL library. When you receive the `SDLAudioStreamDidStartNotification`, you can begin streaming audio.

### SDLAudioStreamManager
If you do not already have raw PCM data ready at hand, the `SDLAudioStreamManager` can help. The `SDLAudioStreamManager` will help you to do on-the-fly transcoding and streaming of your files in mp3 or other formats.

##### Objective-C
```objc
[self.sdlManager.streamManager.audioManager pushWithFileURL:audioFileURL];
[self.sdlManager.streamManager.audioManager playNextWhenReady];
```

##### Swift
```swift
self.sdlManager.streamManager?.audioManager.push(withFileURL: url)
self.sdlManager.streamManager?.audioManager.playNextWhenReady()
```

#### Implementing the Delegate

##### Objective-C
```objc
- (void)audioStreamManager:(SDLAudioStreamManager *)audioManager errorDidOccurForFile:(NSURL *)fileURL error:(NSError *)error {
}

- (void)audioStreamManager:(SDLAudioStreamManager *)audioManager fileDidFinishPlaying:(NSURL *)fileURL successfully:(BOOL)successfully {
    if (audioManager.queue.count != 0) {
        [audioManager playNextWhenReady];
    }
}
```

##### Swift
```swift
public func audioStreamManager(_ audioManager: SDLAudioStreamManager, errorDidOccurForFile fileURL: URL, error: Error) {

}

public func audioStreamManager(_ audioManager: SDLAudioStreamManager, fileDidFinishPlaying fileURL: URL, successfully: Bool) {
    if audioManager.queue.count != 0 {
        audioManager.playNextWhenReady()
    }
}
```

### Manually Sending Data
Once the audio stream is connected, data may be easily passed to the Head Unit. The function `sendAudioData:` provides us with whether or not the PCM Audio Data was successfully transferred to the Head Unit. If your app is in a state that it is unable to send audio data, this method will return a failure. If successful playback will begin immediately.

##### Objective-C
```objective-c

NSData* audioData = <#Acquire Audio Data#>;

if ([self.sdlManager.streamManager sendAudioData:audioData] == NO) {
  NSLog(@"Could not send Audio Data");
}
```

##### Swift
```swift
let audioData = <#Acquire Audio Data#>;

guard let streamManager = self.sdlManager.streamManager, streamManager.isAudioConnected else { return }

if streamManager.sendAudioData(audioData) == false {
    print("Could not send Audio Data")
}
```
!@

@![android]
To stream audio, we call `sdlManager.getAudioStreamManager().start()` which will start the manager. When that callback returns successful, you call `sdlManager.getAudioStreamManager().startAudioStream()`. When the callback for that is successful, you can push the audio source using `sdlManager.getAudioStreamManager().pushAudioSource()`. Below is an example of playing an `mp3` file that we have in our resource directory:

```java
if (sdlManager.getAudioStreamManager() != null) {
    Log.i(TAG, "Trying to start audio streaming");
    sdlManager.getAudioStreamManager().start(new CompletionListener() {
        @Override
        public void onComplete(boolean success) {
            if (success) {
                sdlManager.getAudioStreamManager().startAudioStream(false, new CompletionListener() {
                    @Override
                    public void onComplete(boolean success) {
                        if (success) {
                            Resources resources = getApplicationContext().getResources();
                            int resourceId = R.raw.exampleMp3;
                            Uri uri = new Uri.Builder()
                            .scheme(ContentResolver.SCHEME_ANDROID_RESOURCE)
                            .authority(resources.getResourcePackageName(resourceId))
                            .appendPath(resources.getResourceTypeName(resourceId))
                            .appendPath(resources.getResourceEntryName(resourceId))
                            .build();
                            sdlManager.getAudioStreamManager().pushAudioSource(uri, new CompletionListener() {
                                @Override
                                public void onComplete(boolean success) {
                                    if (success) {
                                        Log.i(TAG, "Audio file played successfully!");
                                    } else {
                                        Log.i(TAG, "Audio file failed to play!");
                                    }
                                }
                            });
                        } else {
                            Log.d(TAG, "Audio stream failed to start!");
                        }
                    }
                });
            } else {
                Log.i(TAG, "Failed to start audio streaming manager");
            }
        }
    });
}
```

#### Stopping the Audio Stream
When the stream is complete, or you receive HMI_NONE, you should stop the stream by calling:

```java
sdlManager.getAudioStreamManager().stopAudioStream(new CompletionListener() {
    @Override
    public void onComplete(boolean success) {

    }
});
```
!@
