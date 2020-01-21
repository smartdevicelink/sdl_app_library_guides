# Template Text
You can easily display and update text, images, and buttons using the  @![iOS]`SDLScreenManager`!@ @![android, javaSE, javaEE]`ScreenManager`!@. To update the UI, simply give the manager your new data and sandwich the update between the manager's @![iOS]`beginUpdates`!@ @![android, javaSE, javaEE]`beginTransaction()`!@ and @![iOS]`endUpdatesWithCompletionHandler:`!@ @![android, javaSE, javaEE]`commit()`!@ methods.

### Template Text Fields
| @![iOS]`SDLScreenManager`!@ @![android, javaSE, javaEE]`ScreenManager`!@ Parameter Name | Description |
|:--------------------------------------------|:--------------|
| textField1 | The text displayed in a single-line display, or in the upper display line of a multi-line display |
| textField2 | The text displayed on the second display line of a multi-line display |
| textField3 | The text displayed on the third display line of a multi-line display |
| textField4 | The text displayed on the bottom display line of a multi-line display |
| mediaTrackTextField | The text displayed in the in the track field. This field is only valid for media applications |
| textAlignment | The text justification for the text fields. The text alignment can be left, center, or right  |
| textField1Type | The type of data provided in `textField1` |
| textField2Type | The type of data provided in `textField2` |
| textField3Type | The type of data provided in `textField3` |
| textField4Type | The type of data provided in `textField4` |
| title | The title of the displayed template |

### Showing Text
@![iOS]
##### Objective-C
```objc
[self.sdlManager.screenManager beginUpdates];
self.sdlManager.screenManager.textField1 = @"<#Line 1 of Text#>";
self.sdlManager.screenManager.textField2 = @"<#Line 2 of Text#>";
[self.sdlManager.screenManager endUpdatesWithCompletionHandler:^(NSError * _Nullable error) {
    if (error != nil) {
        <#Error Updating UI#>
    } else {
        <#Update to UI was Successful#>
    }
}];
```

##### Swift
```swift
sdlManager.screenManager.beginUpdates()
sdlManager.screenManager.textField1 = "<#Line 1 of Text#>"
sdlManager.screenManager.textField2 = "<#Line 2 of Text#>"
sdlManager.screenManager.endUpdates { (error) in
    if error != nil {
        <#Error Updating UI#>
    } else {
        <#Update to UI was Successful#>
    }
}
```
!@

@![android, javaSE, javaEE]

```java
sdlManager.getScreenManager().beginTransaction();
sdlManager.getScreenManager().setTextField1("Hello, this is MainField1.");
sdlManager.getScreenManager().setTextField2("Hello, this is MainField2.");
sdlManager.getScreenManager().setTextField3("Hello, this is MainField3.");
sdlManager.getScreenManager().setTextField4("Hello, this is MainField4.");
sdlManager.getScreenManager().commit(new CompletionListener() {
	@Override
	public void onComplete(boolean success) {
		Log.i(TAG, "ScreenManager update complete: " + success);
	}
});
```
!@

### Removing Text
To remove text from the screen you just need to set the screen manager property to @![iOS]`nil`!@ @![android, javaSE, javaEE]`null`!@.

@![iOS]
##### Objective-C
```objc
self.sdlManager.screenManager.textField1 = nil;
self.sdlManager.screenManager.textField2 = nil;
```

##### Swift
```swift
sdlManager.screenManager.textField1 = nil
sdlManager.screenManager.textField2 = nil
```
!@

@![android, javaSE, javaEE]
```java
sdlManager.getScreenManager().setTextField1(null);
sdlManager.getScreenManager().setTextField2(null);
```
!@