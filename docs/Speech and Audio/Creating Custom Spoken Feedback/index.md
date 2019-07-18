# Playing Spoken Feedback
Since your user will be driving while interacting with your SDL app, speech phrases can provide important feedback to your user. At any time during your app's lifecycle you can send a speech phrase using the @![iOS]`SDLSpeak`!@@![android,javaSE,javaEE]`Speak`!@ request and SDL's text-to-speech (TTS) engine will produce synthesized speech from your provided text.

When using the `SDLSpeak` RPC, you will receive a response from the head unit once the operation has completed. From the response you will be able to tell if the speech was completed, interrupted, rejected or aborted. It is important to keep in mind that a speech request can interrupt another on-going speech request. If you want to chain speech requests you must wait for for the current speech request to finish before sending the next speech request. 

## Creating the Speak Request
The speech request you send can simply be a text phrase or it can consist of phoneme specifications to direct SDL’s TTS engine to speak a speech-sculpted phrase. It is also possible to play a pre-recorded sound file using the speech request. For more information on how to play a sound file please refer to [Playing Audio Indications](Speech and Audio/Playing Audio Indications). 

### Text Phrase
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
// TODO
```
!@

### SAPI Phonemes Phrase
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
// TODO
```
!@

## Sending the Speak Request
@![iOS]
##### Objective-C
```objc
[self.sdlManager sendRequest:speak withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if (![response.resultCode isEqualToEnum:SDLResultSuccess]) {
        if ([response.resultCode isEqualToEnum:SDLResultDisallowed]) {
            <#The app does not have permission to use the speech request#>
        } else if ([response.resultCode isEqualToEnum:SDLResultRejected]) {
            <#The request was rejected because a higher priority request is in progress#>
        } else if ([response.resultCode isEqualToEnum:SDLResultAborted]) {
            <#The request was aborted by another higher priority request#>
        } else {
            <#Some other error occured#>
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
    guard response.resultCode == .success else {
        switch response.resultCode {
        case .disallowed:
            <#The app does not have permission to use the speech request#>
        case .rejected:
            <#The request was rejected because a higher priority request is in progress#>
        case .aborted:
            <#The request was aborted by another higher priority request#>
        default:
            <#Some other error occured#>
        }
        return
    }

    <#Speech was successfully spoken#>
}
```
!@

@![android,javaSE,javaEE]
```java
// TODO
```
!@
