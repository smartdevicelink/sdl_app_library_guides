# Custom Text-To-Speech
To have the vehicle audio system speak a phrase, you can send a @![iOS]`SDLSpeak`!@@![android,javaSE,javaEE]`Speak`!@ request with the desired phrase. The provided can be simply a text phrase, or it can consist of phoneme specifications to direct SDL’s TTS engine to speak a speech-sculpted phrase. 

When using the `SDLSpeak` RPC, you will only be able to find out if Core got the request successfully. You will not be able to find out if the speech request was completed, interrupted or aborted.

## Chaining Speech Requests
It is important to keep in mind that a speech request can interrupt another on-going speech request. If you want to chain speech requests, you must wait for for the current speech request to finish before sending the next speech request. 

## Creating the Speak Command
@![iOS]
##### Objective-C
```objc

```

##### Swift
```swift

```
!@

@![android,javaSE,javaEE]
```java

```
!@