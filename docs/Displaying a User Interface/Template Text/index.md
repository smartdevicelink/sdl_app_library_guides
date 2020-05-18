# Template Text
You can easily display text, images, and buttons using the  @![iOS]`SDLScreenManager`!@@![android, javaSE, javaEE, javascript]`ScreenManager`!@. To update the UI, simply give the manager your new data and (optionally) sandwich the update between the manager's @![iOS]`beginUpdates`!@@![android, javaSE, javaEE, javascript]`beginTransaction()`!@ and @![iOS]`endUpdatesWithCompletionHandler`!@@![android, javaSE, javaEE, javascript]`commit()`!@ methods.

### Text Fields
| @![iOS]SDLScreenManager!@@![android, javaSE, javaEE, javascript]ScreenManager!@ Parameter Name | Description |
|:--------------------------------------------|:--------------|
| textField1 | The text displayed in a single-line display, or in the upper display line of a multi-line display |
| textField2 | The text displayed on the second display line of a multi-line display |
| textField3 | The text displayed on the third display line of a multi-line display |
| textField4 | The text displayed on the bottom display line of a multi-line display |
| mediaTrackTextField | The text displayed in the in the track field; this field is only valid for media applications |
| textAlignment | The text justification for the text fields; the text alignment can be left, center, or right  |
| textField1Type | The type of data provided in textField1 |
| textField2Type | The type of data provided in textField2 |
| textField3Type | The type of data provided in textField3 |
| textField4Type | The type of data provided in textField4 |
| title | The title of the displayed template |

## Showing Text
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
sdlManager.getScreenManager().setTextField1("Line 1 of Text");
sdlManager.getScreenManager().setTextField2("Line 2 of Text");
sdlManager.getScreenManager().commit(new CompletionListener() {
	@Override
	public void onComplete(boolean success) {
		Log.i(TAG, "ScreenManager update complete: " + success);
	}
});
```
!@

@![javascript]
```js
sdlManager.getScreenManager().beginTransaction();
sdlManager.getScreenManager().setTextField1('Line 1 of Text');
sdlManager.getScreenManager().setTextField2('Line 2 of Text');
// Commit the updates and catch any errors
const success = await sdlManager.getScreenManager().commit().catch(error => error);
console.log('ScreenManager update complete:', success);
if (success === true) {
    // Update complete
} else {
    // Something went wrong
}
```
!@

### Removing Text
To remove text from the screen simply set the screen manager property to @![iOS]`nil`!@@![android, javaSE, javaEE]`null`!@.

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

@![javascript]
```js
sdlManager.getScreenManager().setTextField1(null);
sdlManager.getScreenManager().setTextField2(null);
```
!@