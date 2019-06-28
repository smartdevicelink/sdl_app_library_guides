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

## Menus

Menus have now become simpler with the `MenuManager`, which is accessed via the `ScreenManager`. The cells, called `MenuCell`'s contain 2 constructors. One is for a cell itself, and the other is a cell that contains a sub-menu. Note that currently SmartDeviceLink (SDL) only supports sub-menus to the depth of 1.

`MenuCell`s contain a `MenuSelectionListener` which informs you that the cell has been triggered, so that you might perform an action based on the cell selected.

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
