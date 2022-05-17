# Setting Up Voice Commands
Voice commands are global commands available anywhere on the head unit to users of your app. Once the user has opened your SDL app (i.e. your SDL app has left the HMI state of `NONE`) they have access to the voice commands you have setup. Your app will be notified when a voice command has been triggered even if the SDL app has been backgrounded.

!!! NOTE
The head unit manufacturer will determine how these voice commands are triggered, and some head units will not support voice commands.
!!!

!!! NOTE
On [Manticore](https://smartdevicelink.com/resources/manticore/), voice commands are viewed and activated by a tab in the right hand section, not through a microphone.
!!!

You have the ability to create voice command shortcuts to your [Main Menu](Displaying a User Interface/Main Menu) cells which we highly recommended that you implement. Global voice commands should be created for functions that you wish to make available as voice commands that are **not** available as menu cells. We recommend creating global voice commands for common actions such as the actions performed by your [Soft Buttons](Displaying a User Interface/Template Custom Buttons).

## Creating Voice Commands
To create voice commands, you simply create and set @![iOS]`SDLVoiceCommand`!@ @![android, javaSE, javaEE, javascript]`VoiceCommand`!@ objects to the `voiceCommands`  @![iOS, javascript]array!@ @![android, javaSE, javaEE]List!@ on the screen manager.

@![iOS]
|~
```objc
SDLVoiceCommand *voiceCommand = [[SDLVoiceCommand alloc] initWithVoiceCommands:@[<#NSString#>] handler:^{
    <#Voice command selected#>
}];

self.sdlManager.screenManager.voiceCommands = @[voiceCommand];
```
```swift
let voiceCommand = SDLVoiceCommand(voiceCommands: [<#String#>]) {
    <#Voice command triggered#>
}

sdlManager.screenManager.voiceCommands = [voiceCommand]
```
~|
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
    // Handle the VoiceCommand's Selection
});
sdlManager.getScreenManager().setVoiceCommands([voiceCommand]);
```
!@

### Unsupported Voice Commands
The library automatically filters out empty strings and whitespace-only strings from a voice command's @![iOS, javascript]array!@@![android, javaSE, javaEE]list!@ of strings. For example, if a voice command has the following @![iOS, javascript]array!@@![android, javaSE, javaEE]list!@ values: `[" ", "CommandA", "", "Command A"]` the library will filter it to: `["CommandA", "Command A"]`.

If you provide @![iOS, javascript]an array!@@![android, javaSE, javaEE]a list!@ of voice commands which only contains empty string and whitespace-only strings across all of the voice commands, the upload request will be aborted and the previous voice commands will remain available.

### Duplicate Strings in Voice Commands

#### Duplicates Between Different Commands
Voice commands that are sent with duplicate strings in different voice commands, such as:
```
{
        Command1: ["Command A", "Command B"],
        Command2: ["Command B", "Command C"],
        Command3: ["Command D", "Command E"]
}
```
Then the manager will abort the upload request. The previous voice commands will remain available.

#### Duplicates in The Same Command
If any individual voice command contains duplicate strings, they will be reduced to one. For example, if the voice commands to be sent are:
```
{
        Command1: ["Command A", "Command A", "Command B"],
        Command2: ["Command C", "Command D"]
}
```

Then the manager will strip the duplicates to:
```
{
        Command1: ["Command A", "Command B"],
        Command2: ["Command C", "Command D"]
}
```

## Deleting Voice Commands
To delete previously set voice commands, you just have to set an empty @![iOS, javascript]array!@ @![android, javaSE, javaEE]List!@ to the `voiceCommands` @![iOS, javascript]array!@ @![android, javaSE, javaEE]List!@ on the screen manager.

@![iOS]
|~
```objc
self.sdlManager.screenManager.voiceCommands = [];
```
```swift
sdlManager.screenManager.voiceCommands = []
```
~|
!@

@![android, javaSE, javaEE]
```java
sdlManager.getScreenManager().setVoiceCommands(Collections.<VoiceCommand>emptyList());
```
!@

@![javascript]
```js
sdlManager.getScreenManager().setVoiceCommands([]);
```
!@

!!! NOTE
Setting voice command strings composed only of whitespace characters will be considered invalid (e.g.  `" "`) and your request will be aborted by the module.
!!!

## Using RPCs
If you wish to do this without the aid of the screen manager, you can create @![iOS]`SDLAddCommand`!@ @![android, javaSE, javaEE, javascript]`AddCommand`!@ objects without the `menuParams` parameter to create global voice commands.
