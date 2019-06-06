## Main Screen Templates
Each car manufacturer supports a set of user interface templates. These templates determine the position and size of the text, images, and buttons on the screen. Once the app has connected successfully with a SDL enabled accessory, a list of supported templates is available in `SDLManager.systemCapabilityManager.displayCapabilities.templatesAvailable`.

To change a template at any time, send a `SetDisplayLayout` RPC to core.

@![iOS]
##### Objective-C
```objc
SDLSetDisplayLayout* display = [[SDLSetDisplayLayout alloc] initWithPredefinedLayout:SDLPredefinedLayoutGraphicWithText];
[self.sdlManager sendRequest:display withResponseHandler:^(SDLRPCRequest *request, SDLRPCResponse *response, NSError *error) {
    if ([response.resultCode isEqualToEnum:SDLResultSuccess]) {
      // The template has been set successfully
    }
}];
```

##### Swift
```swift
let display = SDLSetDisplayLayout(predefinedLayout: .graphicWithText)
sdlManager.send(request: display) { (request, response, error) in
    if response?.resultCode == .success {
        // The template has been set successfully
    }
}
```
!@

@![android, javaSE, javaEE]
`// TODO: Android / Java content`
!@

### Available Templates
There are fifteen standard templates to choose from, however some head units may only support a subset of these templates. Please check `SystemCapabilityManager` for the supported templates. The following examples show how templates will appear on the [Generic HMI](https://github.com/smartdevicelink/generic_hmi) and [Ford's SYNC 3 HMI](https://developer.ford.com). 

#### Media
###### Generic HMI
![Generic - Media without progress bar](assets/GenericHMI/Generic_Default_Media.png)
###### Ford HMI
![SYNC 3 - Media without progress bar](assets/SYNC3HMI/SYNC3_Default_Media.jpg)

#### Media (with a Progress Bar)
###### Generic HMI
![Generic - Media with progress bar](assets/GenericHMI/Generic_media_dark.png)
###### Ford HMI
![SYNC 3 - Media with progress bar](assets/SYNC3HMI/SYNC3_media.jpg)

#### Non-Media
###### Generic HMI
![Generic - Non-Media](assets/GenericHMI/Generic_non_media.png)
###### Ford HMI
![SYNC 3 - Non-Media](assets/SYNC3HMI/SYNC3_non_media.jpg)

#### Graphic with Text
###### Generic HMI
![Generic - Graphic with Text](assets/GenericHMI/Generic_graphic_with_text.png)
###### Ford HMI
![SYNC 3 - Graphic with Text](assets/SYNC3HMI/SYNC3_graphic_with_text.jpg)

#### Text with Graphic
###### Generic HMI
![Generic - Text with Graphic](assets/GenericHMI/Generic_text_with_graphic.png)
###### Ford HMI
![SYNC 3 - Text with Graphic](assets/SYNC3HMI/SYNC3_text_with_graphic.jpg)

#### Tiles Only
###### Generic HMI
![Generic - Tiles Only](assets/GenericHMI/Generic_tiles_only.png)
###### Ford HMI
![SYNC 3 - Tiles Only](assets/SYNC3HMI/SYNC3_tiles_only.jpg)

#### Graphic with Tiles
###### Generic HMI
Currently not implemented
###### Ford HMI
![SYNC 3 - Graphic with Tiles](assets/SYNC3HMI/SYNC3_graphic_with_tiles.jpg)

#### Tiles with Graphic
###### Generic HMI
Currently not implemented
###### Ford HMI
![SYNC 3 - Tiles with Graphic](assets/SYNC3HMI/SYNC3_tiles_with_graphic.jpg)

#### Graphic with Text and Soft Buttons 
###### Generic HMI
Currently not implemented
###### Ford HMI
![SYNC 3 - Graphic with Text and Soft Buttons](assets/SYNC3HMI/SYNC3_graphic_with_text_and_soft_buttons.jpg)

#### Text and Soft Buttons with Graphic 
###### Generic HMI
Currently not implemented
###### Ford HMI
![SYNC 3 Text and Softbuttons with Graphic](assets/SYNC3HMI/SYNC3_text_and_soft_buttons_with_graphic.jpg)

#### Graphic with Text Buttons
###### Generic HMI
![Generic - Graphic with Text Buttons](assets/GenericHMI/Generic_graphic_with_text_buttons.png)
###### Ford HMI
![SYNC 3 - Graphic with Text Buttons](assets/SYNC3HMI/SYNC3_graphic_with_text_buttons.jpg)

#### Double Graphic with Soft Buttons
###### Generic HMI
![Generic - Double Graphic with Softbuttons](assets/GenericHMI/Generic_double_graphic_with_soft_buttons.png)
###### Ford HMI
![SYNC 3 - Double Graphic with Softbuttons](assets/SYNC3HMI/SYNC3_double_graphic_with_soft_buttons.jpg)

#### Text Buttons with Graphic
###### Generic HMI
![Generic - Text Buttons with Graphic](assets/GenericHMI/Generic_text_buttons_with_graphic.png)
###### Ford HMI
![SYNC 3 - Text Buttons with Graphic](assets/SYNC3HMI/SYNC3_text_buttons_with_graphic.jpg)

#### Text Buttons Only
###### Generic HMI
![Generic - Text Buttons Only](assets/GenericHMI/Generic_text_buttons_only.png)
###### Ford HMI
![SYNC 3 - Text Buttons Only](assets/SYNC3HMI/SYNC3_text_buttons_only.jpg)

#### Large Graphic with Soft Buttons
###### Generic HMI
![Generic - Large Graphic with Softbuttons](assets/GenericHMI/Generic_large_graphic_with_soft_buttons.png)
###### Ford HMI
![SYNC 3 - Large Graphic with Softbuttons](assets/SYNC3HMI/SYNC3_large_graphic_with_soft_buttons.jpg)

#### Large Graphic Only
###### Generic HMI
![Generic - Large Graphic Only](assets/GenericHMI/Generic_large_graphic_only.png)
###### Ford HMI
![SYNC 3 - Large Graphic Only](assets/SYNC3HMI/SYNC3_large_graphic_only.jpg)
