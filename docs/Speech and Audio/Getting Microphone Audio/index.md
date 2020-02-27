# Getting Microphone Audio
Capturing in-car audio allows developers to interact with users by requesting raw audio data provided to them from the car's microphones. In order to gather the raw audio from the vehicle, you must leverage the @![iOS][`SDLPerformAudioPassThru`](https://smartdevicelink.com/en/docs/iOS/master/Classes/SDLPerformAudioPassThru/)!@@![android,javaSE,javaEE]`PerformAudioPassThru`!@ RPC.

SDL does not support automatic speech cancellation detection, so if this feature is desired, it is up to the developer to implement. The user may press an "OK" or "Cancel" button, the dialog may timeout, or you may close the dialog with @![iOS]`SDLEndAudioPassThru`!@@![android,javaSE,javaEE]`EndAudioPassThru`!@.

!!! NOTE
SDL does not support an open microphone. However, SDL is working on wake-word support in the future. You may implement a voice command and start an audio pass thru session when that voice command occurs.
!!!

## Starting Audio Capture
To initiate audio capture, first construct a @![iOS]`SDLPerformAudioPassThru`!@@![android,javaSE,javaEE]`PerformAudioPassThru`!@ request. You must use a sampling rate, bit rate, and audio type supported by the head unit. To get the head unit's supported audio capture capabilities, check the @![iOS]`SDLSystemCapabilityManager.audioPassThruCapabilities`!@@![android,javaSE,javaEE] `sdlManager.getSystemCapabilityManager().getCapability(SystemCapabilityType.AUDIO_PASSTHROUGH)`!@ after successfully connecting to the head unit.

| Audio Pass Thru Capability | Parameter Name  |  Description |
| ------------- | ------------- | ------------- |
| Sampling Rate | samplingRate | The sampling rate |
| Bits Per Sample | bitsPerSample | The sample depth in bits |
| Audio Type | audioType | The audio type |

@![iOS]
##### Objective-C
```objc
SDLPerformAudioPassThru *audioPassThru = [[SDLPerformAudioPassThru alloc] initWithInitialPrompt:@"<#A speech prompt when the dialog appears#>" audioPassThruDisplayText1:@"<#Ask me \"What's the weather?\"#>" audioPassThruDisplayText2:@"<#or \"What is 1 + 2?\"#>" samplingRate:SDLSamplingRate16KHZ bitsPerSample:SDLBitsPerSample16Bit audioType:SDLAudioTypePCM maxDuration:<#Time in milliseconds to keep the dialog open#> muteAudio:YES];

[self.sdlManager sendRequest:audioPassThru];
```

##### Swift
```swift
let audioPassThru = SDLPerformAudioPassThru(initialPrompt: "<#A speech prompt when the dialog appears#>", audioPassThruDisplayText1: "<#Ask me \"What's the weather?\"#>", audioPassThruDisplayText2: "<#or \"What is 1 + 2?\"#>", samplingRate: .rate16KHZ, bitsPerSample: .sample16Bit, audioType: .PCM, maxDuration: <#Time in milliseconds to keep the dialog open#>, muteAudio: true)

sdlManager.send(audioPassThru)
```
!@

@![android,javaSE,javaEE]
```java
PerformAudioPassThru performAPT = new PerformAudioPassThru();
performAPT.setAudioPassThruDisplayText1("Ask me \"What's the weather?\"");
performAPT.setAudioPassThruDisplayText2("or \"What's 1 + 2?\"");

performAPT.setInitialPrompt(TTSChunkFactory.createSimpleTTSChunks("Ask me What's the weather? or What's 1 plus 2?"));
performAPT.setSamplingRate(SamplingRate._22KHZ);
performAPT.setMaxDuration(7000);
performAPT.setBitsPerSample(BitsPerSample._16_BIT);
performAPT.setAudioType(AudioType.PCM);
performAPT.setMuteAudio(false);

sdlManager.sendRPC(performAPT);
```
!@

![Ford Audio Pass Thru](assets/Ford_AudioPassThruPrompt.png)

### Gathering Audio Data
SDL provides audio data as fast as it can gather it, and sends it to the developer in chunks. In order to retrieve this audio data, the developer must @![iOS]add a handler to the `SDLPerformAudioPassThru`.!@
@![android,javaSE,javaEE]observe the `OnAudioPassThru` notification.!@

!!! NOTE
This audio data is only the current chunk of audio data, so the developer must be in charge of managing previously retrieved audio data.
!!!

@![iOS]
##### Objective-C
```objc
SDLPerformAudioPassThru *audioPassThru = [[SDLPerformAudioPassThru alloc] initWithInitialPrompt:@"<#A speech prompt when the dialog appears#>" audioPassThruDisplayText1:@"<#Ask me \"What's the weather?\"#>" audioPassThruDisplayText2:@"<#or \"What is 1 + 2?\"#>" samplingRate:SDLSamplingRate16KHZ bitsPerSample:SDLBitsPerSample16Bit audioType:SDLAudioTypePCM maxDuration:<#Time in milliseconds to keep the dialog open#> muteAudio:YES];

audioPassThru.audioDataHandler = ^(NSData * _Nullable audioData) {
    // Do something with current audio data.
    if (audioData.length == 0) { return; }
    <#code#>
}

[self.sdlManager sendRequest:audioPassThru];
```

##### Swift
```swift
let audioPassThru = SDLPerformAudioPassThru(initialPrompt: "<#A speech prompt when the dialog appears#>", audioPassThruDisplayText1: "<#Ask me \"What's the weather?\"#>", audioPassThruDisplayText2: "<#or \"What is 1 + 2?\"#>", samplingRate: .rate16KHZ, bitsPerSample: .sample16Bit, audioType: .PCM, maxDuration: <#Time in milliseconds to keep the dialog open#>, muteAudio: true)

audioPassThru.audioDataHandler = { (data) in
    // Do something with current audio data.
    guard let audioData = data else { return }
    <#code#>
}

sdlManager.send(audioPassThru)
```
!@

@![android,javaSE,javaEE]
```java
sdlManager.addOnRPCNotificationListener(FunctionID.ON_AUDIO_PASS_THRU, new OnRPCNotificationListener() {
    @Override
    public void onNotified(RPCNotification notification) {
        OnAudioPassThru onAudioPassThru = (OnAudioPassThru) notification;
        byte[] dataRcvd = onAudioPassThru.getAPTData();
        processAPTData(dataRcvd); // Do something with audio data
    }
});
```
!@

#### Format of Audio Data
The format of audio data is described as follows:
- It does not include a header (such as a RIFF header) at the beginning.
- The audio sample is in linear PCM format.
- The audio data includes only one channel (i.e. monaural).
- For bit rates of 8 bits, the audio samples are unsigned. For bit rates of 16 bits, the audio samples are signed and are in little-endian.

## Ending Audio Capture
@![iOS]`SDLPerformAudioPassThru`!@@![android,javaSE,javaEE]`PerformAudioPassThru`!@ is a request that works in a different way than other RPCs. For most RPCs, a request is followed by an immediate response, with whether that RPC was successful or not. This RPC, however, will only send out the response when the audio pass thru has ended.

Audio capture can be ended in 4 ways:

1. Audio pass thru has timed out.

    * If the audio pass thru has proceeded longer than the requested timeout duration, Core will end this request with a `resultCode` of `SUCCESS`. You should handle the audio pass thru though it was successful.

2. Audio pass thru was closed due to user pressing "Cancel".

    * If the audio pass thru was displayed, and the user pressed the "Cancel" button, you will receive a `resultCode` of `ABORTED`. You should ignore the audio pass thru.

3. Audio pass thru was closed due to user pressing "Done".

    * If the audio pass thru was displayed and the user pressed the "Done" button, you will receive a `resultCode` of `SUCCESS`. You should handle the audio pass thru as though it was successful.

4. Audio pass thru was ended due to the developer ending the request.

    * If the audio pass thru was displayed, but you have established on your own that you no longer need to capture audio data, you can send an @![iOS]`SDLEndAudioPassThru`!@@![android,javaSE,javaEE]`EndAudioPassThru`!@ RPC. You will receive a `resultCode` of `SUCCESS`, and should handle the audio pass thru as though it was successful.

@![iOS]
##### Objective-C
```objc
SDLEndAudioPassThru *endAudioPassThru = [[SDLEndAudioPassThru alloc] init];
[self.sdlManager sendRequest:endAudioPassThru];
```

##### Swift
```swift
let endAudioPassThru = SDLEndAudioPassThru()
sdlManager.send(endAudioPassThru)
```
!@

@![android,javaSE,javaEE]
```java
EndAudioPassThru endAPT = new EndAudioPassThru();
sdlManager.sendRPC(endAPT);
```
!@

## Handling the Response
To process the response received from an ended audio capture, @![iOS]use the `withResponseHandler` property in `SDLManager`'s `send(_ :)` function!@@![android,javaSE,javaEE] monitor the `PerformAudioPassThruResponse` by adding a listener to the `PerformAudioPassThru` RPC before sending it. If the response has a successful result, all of the audio data for the passthrough has been received and is ready for processing!@.

@![iOS]
##### Objective-C
```objc
[self.sdlManager sendRequest:audioPassThru withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if (error || ![response isKindOfClass:SDLPerformAudioPassThruResponse.class]) {
        return;
    }

    SDLPerformAudioPassThruResponse *audioPassThruResponse = (SDLPerformAudioPassThruResponse *)response;
    if (!response.success.boolValue) {
        <#Cancel any usage of the audio data#>
        return;
    }

    <#Process audio data#>
}];
```

##### Swift
```swift
sdlManager.send(request: audioPassThru) { (request, response, error) in
    guard let response = response else { return }

    guard response?.success.boolValue == true else {
        <#Cancel any usage of the audio data#>
        return
    }

    <#Process audio data#>
}
```
!@

@![android,javaSE,javaEE]
```java
performAPT.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse response) {
        Result result = response.getResultCode();

        if(result.equals(Result.SUCCESS)){
            // We can use the data
        }else{
            // Cancel any usage of the data
            Log.e("SdlService", "Audio pass thru attempt failed.");
        }
    }
    @Override
    public void onError(int correlationId, Result resultCode, String info){
        Log.e(TAG, "onError: "+ resultCode+ " | Info: "+ info );
    }
});
```
!@
