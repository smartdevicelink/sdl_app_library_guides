## Main Screen Templates
Each car manufacturer supports a set of templates for the user interface. These templates determine the position and size of the text, images, and buttons on the screen. A list of supported templates is sent in `SDLManager.systemCapabilityManager.displayCapabilities.templatesAvailable`.

To change a template at any time, send a `SDLSetDisplayLayout` RPC to the SDL Core. If you want to ensure that the new template is used, wait for a response from the SDL Core before sending any more user interface RPCs.

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

### Available Templates
There are fifteen standard templates to choose from, however some head units may only support a subset of these templates. Please check `SystemCapabilityManager` for the supported templates. The following examples show how templates will appear on the [Generic HMI](https://github.com/smartdevicelink/generic_hmi) and [Ford's SYNC 3 HMI](https://developer.ford.com). 

##### Media - with and without progress bar
###### Generic HMI
![Media](assets/generic_Media.png)

###### Ford HMI
![Media - with progress bar](assets/ford_MediaWithProgressBar.png)

![Media - without progress bar](assets/ford_MediaWithoutProgressBar.png)

##### Non-Media - with and without soft buttons
###### Generic HMI
![Non-Media](assets/generic_NonMedia.png)

###### Ford HMI
![Non-Media - with soft buttons](assets/ford_NonMediaWithSoftButtons.png)

![Non-Media - without soft buttons](assets/ford_NonMediaWithoutSoftButtons.png)

##### GRAPHIC_WITH_TEXT
###### Ford HMI
![Graphic with Text](assets/ford_GraphicWithText.png)

##### TEXT_WITH_GRAPHIC
###### Ford HMI
![Text with Graphic](assets/ford_TextWithGraphic.png)

##### TILES_ONLY
###### Ford HMI
![Tiles Only](assets/ford_TilesOnly.png)

##### GRAPHIC_WITH_TILES
###### Ford HMI
![Graphic with Tiles](assets/ford_GraphicWithTiles.png)

##### TILES_WITH_GRAPHIC
###### Ford HMI
![Tiles with Graphic](assets/ford_TilesWithGraphic.png)

##### GRAPHIC_WITH_TEXT_AND_SOFTBUTTONS
###### Ford HMI
![Graphic with Text and Softbuttons](assets/ford_GraphicWithTextAndSoftButtons.png)

##### TEXT_AND_SOFTBUTTONS_WITH_GRAPHIC
###### Ford HMI
![Text and Softbuttons with Graphic](assets/ford_TextAndSoftButtonsWithGraphic.png)

##### GRAPHIC_WITH_TEXTBUTTONS
###### Ford HMI
![Graphic with Textbuttons](assets/ford_GraphicWithTextButtons.png)

##### DOUBLE_GRAPHIC_SOFTBUTTONS
###### Ford HMI
![Double Graphic and Softbuttons](assets/ford_DoubleGraphicSoftButtons.png)

##### TEXTBUTTONS_WITH_GRAPHIC
###### Ford HMI
![Textbuttons with Graphic](assets/ford_TextButtonsWithGraphic.png)

##### TEXTBUTTONS_ONLY
###### Ford HMI
![Textbuttons Only](assets/ford_TextButtonsOnly.png)

##### LARGE_GRAPHIC_WITH_SOFTBUTTONS
###### Generic HMI
![Large Graphic with Softbuttons](assets/generic_LargeGraphicWithSoftButtons.png)

###### Ford HMI
![Large Graphic with Softbuttons](assets/ford_LargeGraphicWithSoftButtons.png)

##### LARGE_GRAPHIC_ONLY
###### Generic HMI
![Large Graphic Only](assets/generic_LargeGraphicOnly.png)

###### Ford HMI
![Large Graphic Only](assets/ford_LargeGraphicOnly.png)
