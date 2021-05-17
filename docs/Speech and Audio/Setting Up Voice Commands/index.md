# Setting Up Voice Commands
Voice commands are global commands available anywhere on the head unit to users of your app. Once the user has opened your SDL app (i.e. your SDL app has left the HMI state of `NONE`) they have access to the voice commands you have setup. Your app will be notified when a voice command has been triggered even if the SDL app has been backgrounded.

!!! NOTE
The head unit manufacturer will determine how these voice commands are triggered, and some head units will not support voice commands.
!!!

You have the ability to create voice command shortcuts to your [Main Menu](Displaying a User Interface/Main Menu) cells which we highly recommended that you implement. Global voice commands should be created for functions that you wish to make available as voice commands that are **not** available as menu cells. We recommend creating global voice commands for common actions such as the actions performed by your [Soft Buttons](Displaying a User Interface/Template Custom Buttons).


## Creating Voice Commands
To create voice commands, you simply create and set @![iOS]`SDLVoiceCommand`!@ @![android, javaSE, javaEE, javascript]`VoiceCommand`!@ objects to the `voiceCommands`  @![iOS, javascript]array!@ @![android, javaSE, javaEE]List!@ on the screen manager.

@![iOS]
##### Objective-C
```objc
SDLVoiceCommand *voiceCommand = [[SDLVoiceCommand alloc] initWithVoiceCommands:@[<#NSString#>] handler:^{
    <#Voice command selected#>
}];

self.sdlManager.screenManager.voiceCommands = @[voiceCommand];
```

##### Swift
```swift
let voiceCommand = SDLVoiceCommand(voiceCommands: [<#String#>]) {
    <#Voice command triggered#>
}

sdlManager.screenManager.voiceCommands = [voiceCommand]
```
!@

@![android, javaSE, javaEE]
```java
VoiceCommand voiceCommand = new VoiceCommand(Collections.singletonList("Command One"), new VoiceCommandSelectionListener() {
    @Override
    public void onVoiceCommandSelected() {
        // Handle the VoiceCommand's Selection
    }
});

sdlManager.getScreenManager().setVoiceCommands(Collections.singletonList(voiceCommand));
```
!@

@![javascript]
```js
const voiceCommand = new SDL.manager.screen.utils.VoiceCommand(['Command One'], function () {
    // <#Handle the VoiceCommand's Selection#>
});
sdlManager.getScreenManager().setVoiceCommands([voiceCommand]);
```
!@

### Unsupported Voice Commands
A voice command that contains empty strings (or strings only containing whitespace) in its array will have those empty strings filtered out by the library. For example, if a voice command has the following array values: `[" ", "First", "", "Voice Command"]` will become `["First", "Voice Command"]`.

If all the voice commands contain only empty strings, the upload request will be aborted and the previous voice commands will remain available.

## Deleting Voice Commands
@![iOS, android, javaSE, javaEE]To delete previously set voice commands, you just have to set an empty !@ @![iOS]array!@ @![android, javaSE, javaEE]List!@ @![iOS, android, javaSE, javaEE] to the `voiceCommands` !@ @![iOS]array!@ @![android, javaSE, javaEE]List!@ @![iOS, android, javaSE, javaEE] on the screen manager.!@ @![javascript] The JavaScript Suite currently does not support clearing previously set voice commands without setting new voice commands.!@

@![iOS]
##### Objective-C
```objc
self.sdlManager.screenManager.voiceCommands = [];
```

##### Swift
```swift
sdlManager.screenManager.voiceCommands = []
```
!@

@![android, javaSE, javaEE]
```java
sdlManager.getScreenManager().setVoiceCommands(Collections.<VoiceCommand>emptyList());
```
!@

!!! NOTE
Setting voice command strings composed only of whitespace characters will be considered invalid (e.g.  `" "`) and your request will be aborted by the module.
!!!

## Using RPCs
If you wish to do this without the aid of the screen manager, you can create @![iOS]`SDLAddCommand`!@ @![android, javaSE, javaEE, javascript]`AddCommand`!@ objects without the `menuParams` parameter to create global voice commands.
