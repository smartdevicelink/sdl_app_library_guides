# Voice Commands
Voice commands are global commands available anywhere on the head unit to users of your app. If your app has left the HMI state of `NONE` because the user has interacted with your app, they may speak the commands you have setup and trigger actions in your app. How these commands are triggered (and whether they are supported at all) will depend on the head unit you connect to, but you don't have to worry about those intracacies when setting up your global commands. 

You have the ability to create voice command shortcuts to your [Main Menu](Displaying a User Interface/Main Menu) cells, and it is recommended that you do so. If you have additional functions that you wish to make available as voice commands that are not available as menu cells, you can create pure global voice commands.

!!! NOTE
We recommend creating global voice commands for common actions such as the actions performed by your [Soft Buttons](Displaying a User Interface/Text Images and Buttons).
!!!

You simply must create and set `SDLVoiceCommand` objects to the `voiceCommands` array on the screen manager.

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

## Using RPCs
If you wish to do this without the aid of the screen manager, you can create `SDLAddCommand` objects without the `menuParams` parameter to create global voice commands.