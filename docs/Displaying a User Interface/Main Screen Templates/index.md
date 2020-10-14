# Main Screen Templates
Each head unit manufacturer supports a set of user interface templates. These templates determine the position and size of the text, images, and buttons on the screen. Once the app has connected successfully with an SDL enabled head unit, a list of supported templates is available on @![iOS]`SDLManager.systemCapabilityManager.defaultMainWindowCapability.templatesAvailable`!@@![android, javaSE, javaEE, javascript]`sdlManager.getSystemCapabilityManager().getDefaultMainWindowCapability().getTemplatesAvailable()`!@.

## Change the Template
To change a template at any time, use @![iOS]`[SDLScreenManager changeLayout:]`!@@![android, javaSE, javaEE, javascript]`ScreenManager.changeLayout()`!@. This guide requires SDL @![android, javaSE, javaEE]Java Suite version 5.0!@@![iOS]iOS version 7.0!@ @![javascript]JavaScript Suite version 1.2!@. If using an older version, use the `SetDisplayLayout` RPC.

!!! NOTE
When changing the layout, you may get an error or failure if the update is "superseded." This isn't technically a failure, because changing the layout has not yet been attempted. The layout or batched operation was cancelled before it could be completed because another operation was requested. The layout change will then be inserted into the future operation and completed then.
!!!

@![iOS]
##### Objective-C
```objc
[self.sdlManager.screenManager changeLayout:[[SDLTemplateConfiguration alloc] initWithTemplate:SDLPredefinedLayoutGraphicWithText] withCompletionHandler:^(NSError * _Nullable error) {
    if (error != nil) {
        // Print out the error if there is one and return early
        return;
    }
    // The template has been set successfully
}];
```

##### Swift
```swift
sdlManager.screenManager.changeLayout(SDLTemplateConfiguration(predefinedLayout: .graphicWithText)) { err in
    if let error = err {
        // Print out the error if there is one and return early
        return
    }
    // The template has been set successfully
}
```
!@

@![android, javaSE, javaEE]
```java
TemplateConfiguration templateConfiguration = new TemplateConfiguration().setTemplate(PredefinedLayout.GRAPHIC_WITH_TEXT.toString());
sdlManager.getScreenManager().changeLayout(templateConfiguration, new CompletionListener() {
    @Override
    public void onComplete(boolean success) {
        if (success) {
            DebugTool.logInfo(TAG, "Layout set successfully");
        } else {
            DebugTool.logInfo(TAG, "Layout not set successfully");
        }
    }
});
```
!@

@![javascript]
```js
const templateConfiguration = new SDL.rpc.structs.TemplateConfiguration()
    .setTemplate(SDL.rpc.enums.PredefinedLayout.GRAPHIC_WITH_TEXT);
    
const success = await sdlManager.getScreenManager().changeLayout(templateConfiguration);
if (success) {
    console.log('Layout set successfully');
} else {
    console.log('Layout not set successfully');
}
```
!@

Template changes can also be batched with text and graphics updates:

@![iOS]
##### Objective-C
```objc
[self.sdlManager.screenManager beginUpdates];
self.sdlManager.screenManager.textField1 = "Line of Text";
[self.sdlManager.screenManager changeLayout:[[SDLTemplateConfiguration alloc] initWithTemplate:SDLPredefinedLayoutGraphicWithText] withCompletionHandler:^(NSError * _Nullable error) {
    // This listener will be ignored, and will use the handler sent in endUpdatesWithCompletionHandler.
}];
self.sdlManager.screenManager.primaryGraphic = <#SDLArtwork#>;
[self.sdlManager.screenManager endUpdatesWithCompletionHandler:^(NSError * _Nullable error) {
    if (error != nil) {
        // Print out the error if there is one and return early
        return
    }
    // The data and template has been set successfully
}];
```

##### Swift
```swift
sdlManager.screenManager.beginUpdates()
sdlManager.screenManager.textField1 = "Line of Text"
sdlManager.screenManager.changeLayout(SDLTemplateConfiguration(predefinedLayout: .graphicWithText)) { err in
    // This listener will be ignored, and will use the handler set in the endUpdates call.
}
sdlManager.screenManager.primaryGraphic = <#SDLArtwork#>
sdlManager.screenManager.endUpdates { err in
    if let error = err {
        // Print out the error if there is one and return early
        return
    }
    // The data and template has been set successfully
}
```
!@

@![android, javaSE, javaEE]
```java
sdlManager.getScreenManager().beginTransaction();
sdlManager.getScreenManager().setTextField1("Line of Text");
sdlManager.getScreenManager().changeLayout(templateConfiguration, new CompletionListener() {
    @Override
    public void onComplete(boolean success) {
        // This listener will be ignored, and will use the CompletionListener sent in commit.
    }
});
sdlManager.getScreenManager().setPrimaryGraphic(sdlArtwork);
sdlManager.getScreenManager().commit(new CompletionListener() {
    @Override
    public void onComplete(boolean success) {
        if (success) {
            DebugTool.logInfo(TAG, "The data and template have been set successfully");
        }
    }
});
```
!@

@![javascript]
```js
sdlManager.getScreenManager().beginTransaction();
sdlManager.getScreenManager().setTextField1('Line of Text');
// The promise returned by changeLayout will not resolve because it is part of a batch update, and the await operator should be avoided as a result
sdlManager.getScreenManager().changeLayout(templateConfiguration);
sdlManager.getScreenManager().setPrimaryGraphic(<#SDLArtwork#>);
const success = await sdlManager.getScreenManager().commit();
if (success) {
    console.log('Text, Graphic, and Template changed successful');
}
```
!@

## Available Templates
There are fifteen standard templates to choose from, however some head units may only support a subset of these templates. The following examples show how templates will appear on the [Generic HMI](https://github.com/smartdevicelink/generic_hmi) and [Ford's SYNC 3 HMI](https://developer.ford.com).

#### Media
![Generic - Media without progress bar](assets/GenericHMI/Generic_Default_Media.png)

#### Media (with a Progress Bar)
![Generic - Media with progress bar](assets/GenericHMI/Generic_media_dark.png)

#### Non-Media
![Generic - Non-Media](assets/GenericHMI/Generic_non_media.png)

#### Graphic with Text
![Generic - Graphic with Text](assets/GenericHMI/Generic_graphic_with_text.png)

#### Text with Graphic
![Generic - Text with Graphic](assets/GenericHMI/Generic_text_with_graphic.png)

#### Tiles Only
![Generic - Tiles Only](assets/GenericHMI/Generic_tiles_only.png)

#### Graphic with Tiles
![SYNC 3 - Graphic with Tiles](assets/SYNC3HMI/SYNC3_graphic_with_tiles.bmp)

#### Tiles with Graphic
![SYNC 3 - Tiles with Graphic](assets/SYNC3HMI/SYNC3_tiles_with_graphic.bmp)

#### Graphic with Text and Soft Buttons
![SYNC 3 - Graphic with Text and Soft Buttons](assets/SYNC3HMI/SYNC3_graphic_with_text_and_soft_buttons.bmp)

#### Text and Soft Buttons with Graphic
![SYNC 3 Text and Soft Buttons with Graphic](assets/SYNC3HMI/SYNC3_text_and_soft_buttons_with_graphic.bmp)

#### Graphic with Text Buttons
![Generic - Graphic with Text Buttons](assets/GenericHMI/Generic_graphic_with_text_buttons.png)

#### Double Graphic with Soft Buttons
![Generic - Double Graphic with Softbuttons](assets/GenericHMI/Generic_double_graphic_with_soft_buttons.png)

#### Text Buttons with Graphic
![Generic - Text Buttons with Graphic](assets/GenericHMI/Generic_text_buttons_with_graphic.png)

#### Text Buttons Only
![Generic - Text Buttons Only](assets/GenericHMI/Generic_text_buttons_only.png)

#### Large Graphic with Soft Buttons
![Generic - Large Graphic with Softbuttons](assets/GenericHMI/Generic_large_graphic_with_soft_buttons.png)

#### Large Graphic Only
![Generic - Large Graphic Only](assets/GenericHMI/Generic_large_graphic_only.png)
