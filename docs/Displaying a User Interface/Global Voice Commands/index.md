# Global Voice Commands
Voice commands are global commands available anywhere on the head unit to users of your app. Once the user has opened your SDL app (i.e. your SDL app has left the HMI state of `NONE`) they have access to the voice commands you have setup. Your app will be notified when a voice command has been triggered even if the SDL app has been backgrounded.

!!! NOTE
The head unit manufacturer will determine how these voice commands are triggered and some head units will not support voice commands.
!!!

You have the ability to create voice command shortcuts to your [Main Menu](Displaying a User Interface/Main Menu) cells which we highly recommended that you implement. Global voice commands should be created for functions that you wish to make available as voice commands that are **not** available as menu cells. We recommend creating global voice commands for common actions such as the actions performed by your [Soft Buttons](Displaying a User Interface/Text Images and Buttons).


## Creating Voice Commands
To create voice commands, you simply create and set @![iOS]`SDLVoiceCommand`!@ @![android, javaSE, javaEE]`SdlVoiceCommand`!@ objects to the `voiceCommands` array on the screen manager.

@![iOS]
##### Objective-C
```objc
// Create the voice command
SDLVoiceCommand *voiceCommand = [[SDLVoiceCommand alloc] initWithVoiceCommands:<#(nonnull NSArray<NSString *> *)#> handler:<#^(void)handler#>];

self.sdlManager.screenManager.voiceCommands = @[voiceCommand];
```

##### Swift
```swift
// Create the voice command
let voiceCommand = SDLVoiceCommand(voiceCommands: <#T##[String]#>) {
    <#code#>
}

self.sdlManager.screenManager.voiceCommands = [voiceCommand]
```
!@

@![android, javaSE, javaEE]
```java
VoiceCommand voiceCommand = new VoiceCommand(Collections.singletonList("Command One"), new VoiceCommandSelectionListener() {
    @Override
    public void onVoiceCommandSelected() {
        // <#Handle the VoiceCommand's Selection#>
    }
});


sdlManager.getScreenManager().setVoiceCommands(Collections.singletonList(voiceCommand));
```
!@

## Using RPCs
If you wish to do this without the aid of the screen manager, you can create @![iOS]`SDLAddCommand`!@ @![android, javaSE, javaEE]`AddCommand`!@ objects without the `menuParams` parameter to create global voice commands.