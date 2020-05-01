# Playing Spoken Feedback
Since your user will be driving while interacting with your SDL app, speech phrases can provide important feedback to your user. At any time during your app's lifecycle you can send a speech phrase using the @![iOS]`SDLSpeak`!@@![android,javaSE,javaEE]`Speak`!@ request and the head unit's text-to-speech (TTS) engine will produce synthesized speech from your provided text.

When using the @![iOS]`SDLSpeak`!@@![android,javaSE,javaEE]`Speak`!@ RPC, you will receive a response from the head unit once the operation has completed. From the response you will be able to tell if the speech was completed, interrupted, rejected or aborted. It is important to keep in mind that a speech request can interrupt another on-going speech request. If you want to chain speech requests you must wait for the current speech request to finish before sending the next speech request. 

## Creating the Speak Request
The speech request you send can simply be a text phrase, which will be played back in accordance with the user's current language settings, or it can consist of phoneme specifications to direct SDL’s TTS engine to speak a language-independent, speech-sculpted phrase. It is also possible to play a pre-recorded sound file (such as an MP3) using the speech request. For more information on how to play a sound file please refer to [Playing Audio Indications](Speech and Audio/Playing Audio Indications). 

### Getting the Supported Capabilities
Once you have successfully connecting to the module, you can access supported speech capabilities properties on the @![iOS]`SDLManager.systemCapabilityManager`!@@![android, javaSE, javaEE]`sdlManager.getSystemCapabilityManager`!@ instance.

@![iOS]
##### Objective-C
```objc
NSArray<SDLSpeechCapabilities> *speechCapabilities = self.sdlManager.systemCapabilityManager.speechCapabilities;
```

##### Swift
```swift
let speechCapabilities = sdlManager.systemCapabilityManager.speechCapabilities
```
!@

@![android, javaSE, javaEE]
```java
sdlManager.getSystemCapabilityManager().getCapability(SystemCapabilityType.SPEECH, new OnSystemCapabilityListener() {
	@Override
	public void onCapabilityRetrieved(Object capability) {
		List<SpeechCapabilities> speechCapabilities = (List<SpeechCapabilities>) capability;
	}

	@Override
	public void onError(String info) {
        <#Handle Error#>
	}
}, false);
```
!@

Below is a list of commonly supported speech capabilities.

| Speech Capability |  Description |
| ------------- | ------------- |
| Text | Text phrases |
| SAPI Phonemes | Microsoft speech synthesis API |
| File | A pre-recorded sound file |

### Creating Different Types of Speak Requests
#### Text Phrase
@![iOS]
##### Objective-C
```objc
SDLSpeak *speak = [[SDLSpeak alloc] initWithTTS:@"hello"];
```

##### Swift
```swift
let speech = SDLSpeak(tts: "hello")
```
!@

@![android,javaSE,javaEE]
```java
TTSChunk ttsChunk = new TTSChunk("hello", SpeechCapabilities.TEXT);
List<TTSChunk> ttsChunkList = Collections.singletonList(ttsChunk);
Speak speak = new Speak(ttsChunkList);
```
!@

#### SAPI Phonemes Phrase
@![iOS]
##### Objective-C
```objc
NSArray<SDLTTSChunk *> *sapiPhonemesTTSChunks = [SDLTTSChunk sapiChunksFromString:@"h eh - l ow 1"];
SDLSpeak *speak = [[SDLSpeak alloc] initWithTTSChunks:sapiPhonemesTTSChunks];
```
##### Swift
```swift
let sapiPhonemesTTSChunks = SDLTTSChunk.sapiChunks(from: "h eh - l ow 1")
let speech = SDLSpeak(ttsChunks: sapiPhonemesTTSChunk)
```
!@

@![android,javaSE,javaEE]
```java
TTSChunk ttsChunk = new TTSChunk("h eh - l ow 1", SpeechCapabilities.SAPI_PHONEMES);
List<TTSChunk> ttsChunkList = Collections.singletonList(ttsChunk);
Speak speak = new Speak(ttsChunkList);
```
!@

## Sending the Speak Request
@![iOS]
##### Objective-C
```objc
[self.sdlManager sendRequest:speak withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
     if (!response.success.boolValue) { 
        if ([response.resultCode isEqualToEnum:SDLResultDisallowed]) {
            <#The app does not have permission to use the speech request#>
        } else if ([response.resultCode isEqualToEnum:SDLResultRejected]) {
            <#The request was rejected because a higher priority request is in progress#>
        } else if ([response.resultCode isEqualToEnum:SDLResultAborted]) {
            <#The request was aborted by another higher priority request#>
        } else {
            <#Some other error occurred#>
        }

        return;
    }

    <#Speech was successfully spoken#>
}];
```
##### Swift
```swift
sdlManager.send(request: speech) { (request, response, error) in
    guard let response = response as? SDLSpeakResponse else { return }
    guard response?.success.boolValue == true else {
        switch response.resultCode {
        case .disallowed:
            <#The app does not have permission to use the speech request#>
        case .rejected:
            <#The request was rejected because a higher priority request is in progress#>
        case .aborted:
            <#The request was aborted by another higher priority request#>
        default:
            <#Some other error occurred#>
        }
        return
    }

    <#Speech was successfully spoken#>
}
```
!@

@![android,javaSE,javaEE]
```java
speak.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse response) {
        SpeakResponse speakResponse = (SpeakResponse) response;
        if (!speakResponse.getSuccess()){
            switch (speakResponse.getResultCode()){
                case DISALLOWED:
                    Log.i(TAG, "The app does not have permission to use the speech request");
                    break;
                case REJECTED:
                    Log.i(TAG, "The request was rejected because a higher priority request is in progress");
                    break;
                case ABORTED:
                    Log.i(TAG, "The request was aborted by another higher priority request");
                    break;
                default:
                    Log.i(TAG, "Some other error occurred");
            }
            return;
        }
        Log.i(TAG, "Speech was successfully spoken");
    }

    @Override
    public void onError(int correlationId, Result resultCode, String info) {
        Log.i(TAG, "onError: " + info);
    }
});
sdlManager.sendRPC(speak);
```
!@
