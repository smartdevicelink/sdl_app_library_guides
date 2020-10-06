# Upgrading to 4.9

## Overview

This guide is to help developers get setup with the SDL Java library version 4.9. It is assumed that the developer is already updated to at least version 4.7 or 4.8 of the library.

The full release notes are published [here](https://github.com/smartdevicelink/sdl_java_suite/releases).

The main differences between the previous release and this are mainly additive, including 3 new managers which we will describe briefly. Additionally, we have fixed an issue where symlinks were not working on Windows machines by creating a gradle task that builds them for you. Additionally, we have added the ability to pass a buffer to the AudioStreamManager to play raw data.

## Voice Command Manager

The voice command manager is accessed via the `ScreenManager`. It allows for an easy way to create global voice commands for your application. These are not supposed to be a replacement for menu voice commands, but rather an easy way to trigger main events in your application, similar to something you might use a `SoftButton` for. These commands, once sent, will be available on the system as voice commands for the duration of the session.

An example is as follows:

```java
List<String> list1 = Collections.singletonList("Command One");
List<String> list2 = Collections.singletonList("Command two");

VoiceCommand voiceCommand1 = new VoiceCommand(list1, new VoiceCommandSelectionListener() {
	@Override
	public void onVoiceCommandSelected() {
		Log.i(TAG, "Voice Command 1 triggered");
	}
});

VoiceCommand voiceCommand2 = new VoiceCommand(list2, new VoiceCommandSelectionListener() {
	@Override
	public void onVoiceCommandSelected() {
		Log.i(TAG, "Voice Command 2 triggered");
	}
});

sdlManager.getScreenManager().setVoiceCommands(Arrays.asList(voiceCommand1,voiceCommand2));
```

## Menu Manager

Menus have now become simpler with the `MenuManager`, which is accessed via the `ScreenManager`. The cells, called `MenuCell`'s contain 2 constructors. One is for a cell itself, and the other is a cell that contains a sub-menu. Note that currently SmartDeviceLink (SDL) only supports sub-menus to the depth of 1.

`MenuCell`s contain a `MenuSelectionListener` which informs you that the cell has been triggered, so that you might perform an action based on the cell selected. Note that you can add images and voice commands to menu cells.

!!! NOTE
When submitting a list of Menu cells, or adding a list of sub cells to a menu cell, the order in which the cells will appear from top to bottom will be the order in which they are in the list.
!!!

Example use:

```java
// SUB MENU CELLS FOR MAIN MENU CELL 2

// Sub cells are just normal cells
MenuCell subCell1 = new MenuCell("SubCell 1",null, null, new MenuSelectionListener() {
	@Override
	public void onTriggered(TriggerSource trigger) {
		Log.i(TAG, "Sub cell 1 triggered. Source: "+ trigger.toString());
	}
});

MenuCell subCell2 = new MenuCell("SubCell 2",null, null, new MenuSelectionListener() {
	@Override
	public void onTriggered(TriggerSource trigger) {
		Log.i(TAG, "Sub cell 2 triggered. Source: "+ trigger.toString());
	}
});

// THE MAIN MENU CELLS

// normal cell
MenuCell mainCell1 = new MenuCell("Test Cell 1 (speak)", null, null, new MenuSelectionListener() {
	@Override
	public void onTriggered(TriggerSource trigger) {
		Log.i(TAG, "Test cell 1 triggered. Source: "+ trigger.toString());
	}
});

// sub menu parent cell
MenuCell mainCell2 = new MenuCell("Test Cell 3 (sub menu)", null, Arrays.asList(subCell1,subCell2));

// Send the entire menu off to be created
sdlManager.getScreenManager().setMenu(Arrays.asList(mainCell1, mainCell2));
```

## Choice Set Manager

Previously it required a lot of code to use `PerformInteraction`s with SDL. To alleviate some of this pain, we have introduced the Choice Set Manager which is accessible via the `ScreenManager`. Because the Choice Set Manager covers so many items, we will do a brief overview here. You may continue to the [Popup Menus and Keyboards](Displaying a User Interface/Popup Menus and Keyboards) section for more detailed information.

There are 2 main use cases for using this manager, one is to display a choice set, and the other is to display a keyboard.

### Choice Set

Displaying a choice set is achieved by creating some `ChoiceCell`s. If you know what your choices will be, we recommend using the `preloadChoices` method. This will ensure your `ChoiceSet` is ready to be displayed when you want to display it, and your user is not kept waiting. You can preload cells as follows:

```java
// create some choice cells
ChoiceCell cell1 = new ChoiceCell("Item 1");
ChoiceCell cell2 = new ChoiceCell("Item 2");
ChoiceCell cell3 = new ChoiceCell("Item 3");

// create the array of choice cells
choiceCellList = Arrays.asList(cell1,cell2,cell3);

// pre-load the cells on the head unit
sdlManager.getScreenManager().preloadChoices(choiceCellList, null);
```

!!! NOTE
You will want to reference this array of cells when presenting your choice set later (even if you add more cells). This is why we are setting this list to a variable for now.
!!!

Once you are ready to present the Choice Set, you can do so by:

```java
ChoiceSet choiceSet = new ChoiceSet("Choose an Item from the list", choiceCellList, new ChoiceSetSelectionListener() {
	@Override
	public void onChoiceSelected(ChoiceCell choiceCell, TriggerSource triggerSource, int rowIndex) {
		// do something with the selection
	}

	@Override
	public void onError(String error) {
		Log.e(TAG, "There was an error showing the perform interaction: "+ error);
	}
});
sdlManager.getScreenManager().presentChoiceSet(choiceSet, InteractionMode.MANUAL_ONLY);
```

### Displaying A Keyboard

There is now also an easy way to display a keyboard, and listen for key events. You simply need a `KeyboardListener` object.

```java
KeyboardListener keyboardListener = new KeyboardListener() {
	@Override
	public void onUserDidSubmitInput(String inputText, KeyboardEvent event) {

	}

	@Override
	public void onKeyboardDidAbortWithReason(KeyboardEvent event) {

	}

	@Override
	public void updateAutocompleteWithInput(String currentInputText, KeyboardAutocompleteCompletionListener keyboardAutocompleteCompletionListener) {

	}

	@Override
	public void updateCharacterSetWithInput(String currentInputText, KeyboardCharacterSetCompletionListener keyboardCharacterSetCompletionListener) {

	}

	@Override
	public void onKeyboardDidSendEvent(KeyboardEvent event, String currentInputText) {

	}
};
```

You can note that two of the methods contain a `KeyboardAutocompleteCompletionListener` and a `KeyboardCharacterSetCompletionListener`. These listeners allow you to show auto completion text and to modify the available keys, respectively, on supported head units.

To actually display the keyboard, call:

```java
sdlManager.getScreenManager().presentKeyboard("initialText", null, keyboardListener);
```

The `null` parameter in this example is a `KeyboardProperties` object that you can optionally pass in to modify the keyboard for this request.

@![android]
## Audio Stream Buffer

We now have the option to send `ByteBuffer`s to the `AudioStreamManager` to be played.

```java
sdlManager.getAudioStreamManager().pushBuffer(byteBuffer, new CompletionListener() {
	@Override
	public void onComplete(boolean success) {
		// do something once the buffer is played
	}
});
```

## Symlinks in Windows

With the creation of the Java Suite, we had the need to share base files between the Android and the JavaSE and JavaEE projects. To allow the Android project to read these base files, we created symlinks to allow the files to be seen from within the project. However, symlinks work differently on Mac / Linux machines than they do on Windows.

To fix this, we created a gradle task to create the Windows symlinks. Simply call:

```java
gradle buildWindowSymLinks
```

from Android Studio's terminal.

!!! NOTE
You will need administrator privileges and Python installed to execute this task.
!!!

!@



