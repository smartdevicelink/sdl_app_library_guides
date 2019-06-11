# In-Car Microphone Audio
Capturing in-car audio allows developers to interact with users by requesting raw audio data provided to them from the car's microphones. In order to gather the raw audio from the vehicle, we must leverage the @![iOS][`SDLPerformAudioPassThru`](https://smartdevicelink.com/en/docs/iOS/master/Classes/SDLPerformAudioPassThru/)!@ @![android,javaSE,javaEE]`PerformAudioPassThru` !@ RPC.

!!! NOTE
PerformAudioPassThru does not support automatic speech cancellation detection, so if this feature is desired, it is up to the developer to implement. The user may press an OK or Cancel button, the dialog may timeout, or you may close the dialog with @![iOS]`SDLEndAudioPassThru`!@ @![android,javaSE,javaEE] `EndAudioPassThru`!@.
!!!

In order to know the currently supported audio capture capabilities of the connected head unit, please refer to the @![iOS]`SDLSystemCapabilityManager.audioPassThruCapabilities` [documentation](https://smartdevicelink.com/en/docs/iOS/master/Classes/SDLRegisterAppInterfaceResponse/)!@ @![android,javaSE,javaEE] `SystemCapabilityManager`. It can retrieve the `AudioPassThruCapabilities` that the head unit supports.

@![android,javaSE,javaEE] `// TODO Android : add link to the documentation like iOS does above: Please see .md file for an example.`!@

!!! NOTE
SDL does not support an open microphone. However, SDL is working on wake-word support in the future. You may implement a voice command and start an audio pass thru session when that voice command occurs.
!!!

## Starting Audio Capture
To initiate audio capture, we must construct a @![iOS]`SDLPerformAudioPassThru`!@ @![android,javaSE,javaEE]`PerformAudioPassThru`!@ request. The properties we will set in this object's constructor relate to how we wish to gather the audio data from the vehicle we are connected to.

!!! NOTE
Currently, SDL only supports Sampling Rates of 16 khz and Bit Rates of 16 bit.
!!!

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

###### Ford HMI
![Ford Audio Pass Thru](assets/Ford_AudioPassThruPrompt.png)

### Gathering Audio Data
@![iOS]SDL provides audio data as fast as it can gather it, and sends it to the developer in chunks. In order to retrieve this audio data, the developer must add a handler to the `SDLPerformAudioPassThru`.!@ @![android,javaSE,javaEE] Before starting audio capture, the app has to subscribe to AudioPassThru notification. SDL provides audio data as fast as it can gather it, and sends it to the developer in chunks. In order to retrieve this audio data, observe the OnAudioPassThru notification.!@

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
- For bit rates of 8 bits, the audio samples are unsigned. For bit rates of 16 bits, the audio samples are signed and are in little endian.

## Ending Audio Capture
Perform Audio Pass Thru is a request that works in a different way than other RPCs. For most RPCs, a request is followed by an immediate response, with whether that RPC was successful or not. This RPC, however, will only send out the response when the PerformAudioPassThru is ended.

Audio Capture can be ended in 4 ways:

1. Audio Pass Thru has timed out.

    If the Audio Pass Thru has proceeded longer than the requested timeout duration, Core will end this request with a @![iOS]`resultCode`!@ @![android,javaSE,javaEE]`Result`!@ of `SUCCESS`. You should expect to handle this Audio Pass Thru as though it was successful.

2. Audio Pass Thru was closed due to user pressing "Cancel".

    If the Audio Pass Thru was displayed, and the user pressed the "Cancel" button, you will receive a @![iOS]`resultCode`!@ @![android,javaSE,javaEE]`Result!`!@ of `ABORTED`. You should expect to ignore this Audio Pass Thru.

3. Audio Pass Thru was closed due to user pressing "Done".

    If the Audio Pass Thru was displayed, and the user pressed the "Done" button, you will receive a @![iOS]`resultCode`!@ @![android,javaSE,javaEE]`Result!`!@ of `SUCCESS`. You should expect to handle this Audio Pass Thru as though it was successful.

4. Audio Pass Thru was ended due to the developer ending the request.

    If the Audio Pass Thru was displayed, but you have established on your own that you no longer need to capture audio data, you can send an @![iOS]`SDLEndAudioPassThru`!@ @![android,javaSE,javaEE] `EndAudioPassThru`!@ RPC.

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

You will receive a @![iOS]`resultCode`!@ @![android,javaSE,javaEE]`Result!`!@ of `SUCCESS`, and should expect to handle this audio pass thru as though it was successful.

## Handling the Response
To process the response that we received from an ended audio capture, we @![iOS]use the `withResponseHandler` property in `SDLManager`'s `send(_ :)` function!@ @![android,javaSE,javaEE] monitor the PerformAudioPassThruResponse by adding a listener to the PerformAudioPassThru RPC before sending it. If the response has a successful Result, all of the audio data for the passthrough has been received and is ready for processing !@.

@![iOS]
##### Objective-C
```objc
[self.sdlManager sendRequest:performAudioPassThru withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if (error || ![response isKindOfClass:SDLPerformAudioPassThruResponse.class]) {
        NSLog(@"Encountered Error sending Perform Audio Pass Thru: %@", error);
        return;
    }

    SDLPerformAudioPassThruResponse *audioPassThruResponse = (SDLPerformAudioPassThruResponse *)response;
    SDLResult *resultCode = audioPassThruResponse.resultCode;
    if (![resultCode isEqualToEnum:SDLResultSuccess]) {
        // Cancel any usage of the audio data
    }

    // Process audio data
}];
```

##### Swift
```swift
sdlManager.send(request: performAudioPassThru) { (request, response, error) in
    guard let response = response else { return }

    guard response.resultCode == .success else {
        // Cancel any usage of the audio data.
        return
    }

    // Process audio data
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
});
```
!@