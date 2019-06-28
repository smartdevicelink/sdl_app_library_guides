# Upgrading to 4.9

## Overview

This guide is to help developers get setup with the SDL Android library version 4.9. It is assumed that the developer is already updated to at least version 4.7 of the library.

The full release notes are published [here](https://github.com/smartdevicelink/sdl_java_suite/releases).

The main differences between the previous release and this are mainly additive features, including 3 new managers which we will describe briefly. Additionally, we have fixed an issue where symlinks were not working on Windows machines by creating a gradle task that builds them for you. Additionally, we have added the ability to pass a buffer to the AudioStreamManager to play buffers.

## Voice Command Manager

The voice command manager is accessed via the `ScreenManager`. It allows for an easy way to create global voice commands for your application. These are not supposed to be a replacement for menu voice commands, but rather an easy way to trigger main events in your application, similar to something you might use a `SoftButton` for.

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

Previously it required a lot of code to use `PerformInteraction`s with SDL. To alleviate some of this pain, we have introduced the Choice Set Manager, which is accessible via the `ScreenManager`. Because the Choice Set Manager covers so many items, we will do a brief overview here, and you may continue to the "Popup Menus and Keyboards" guide within the "Displaying a User Interface" folding menu.

There are 2 main use cases for using this manager, one is to display a choice set, and the other is to display a keyboard. 

### Choice Set

Displaying a choice set is achieved by creating some `ChoiceCell`s. If you know what your choices will be, we recommend using the `preloadChoices` method. This will ensure your `ChoiceSet` is ready to be displayed when you want to display it, and your user is not kept waiting. You can preload cells as follows:

```java
// create some choice cells
ChoiceCell cell1 = new ChoiceCell("Item 1");
ChoiceCell cell2 = new ChoiceCell("Item 2");
ChoiceCell cell3 = new ChoiceCell("Item 3");

// create the array of choice cells
choiceCellList = new ArrayList<>(Arrays.asList(cell1,cell2,cell3));

// pre-load the cells on the head unit
sdlManager.getScreenManager().preloadChoices(choiceCellList, null);
```

!!! NOTE
You will want to reference this array of cells when presenting your choice set later (even if you add more cells). This is why we are setting this list to a variable for now.
!!!