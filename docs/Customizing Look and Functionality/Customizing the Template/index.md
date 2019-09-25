# Customizing the Template
There is some ability to customize the look and feel of the template. How much customization is available depends on the version of Core you are connected with and the design of the head unit.

## Customizing Template Colors (Core 5.0+)
You can customize the color scheme of your app using template coloring APIs.

### Customizing the Default Layout
You can change the template colors of the initial template layout in the `lifecycleConfiguration`.

![Template Coloring from Above](assets/template-colors-example.png)

##### Objective-C
```objc
SDLRGBColor *green = [[SDLRGBColor alloc] initWithRed:126 green:188 blue:121];
SDLRGBColor *white = [[SDLRGBColor alloc] initWithRed:249 green:251 blue:254];
SDLRGBColor *darkGrey = [[SDLRGBColor alloc] initWithRed:57 green:78 blue:96];
SDLRGBColor *grey = [[SDLRGBColor alloc] initWithRed:186 green:198 blue:210];
lifecycleConfiguration.dayColorScheme = [[SDLTemplateColorScheme alloc] initWithPrimaryRGBColor:green secondaryRGBColor:grey backgroundRGBColor:white];
lifecycleConfiguration.nightColorScheme = [[SDLTemplateColorScheme alloc] initWithPrimaryRGBColor:green secondaryRGBColor:grey backgroundRGBColor:darkGrey];
```

##### Swift
```swift
let green = SDLRGBColor(red: 126, green: 188, blue: 121)
let white = SDLRGBColor(red: 249, green: 251, blue: 254)
let grey = SDLRGBColor(red: 186, green: 198, blue: 210)
let darkGrey = SDLRGBColor(red: 57, green: 78, blue: 96)
lifecycleConfiguration.dayColorScheme = SDLTemplateColorScheme(primaryRGBColor: green, secondaryRGBColor: grey, backgroundRGBColor: white)
lifecycleConfiguration.nightColorScheme = SDLTemplateColorScheme(primaryRGBColor: green, secondaryRGBColor: grey, backgroundRGBColor: darkGrey)
```

!!! NOTE
You may change the template coloring in the `lifecycleConfiguration` and the `SetDisplayLayout`, if connecting to a head unit with Core v.5.0+,  or with the `Show` request if connecting to Core v.6.0+. You may only change the template coloring once per template; that is, you cannot call `SetDisplayLayout` or `Show` for the template you are already on and expect the color scheme to update.
!!!

### Customizing Future Layouts
You can also change template coloring when you change layouts by altering `SDLSetDisplayLayout`.

@![iOS]
##### Objective-C
```objc
SDLRGBColor *green = [[SDLRGBColor alloc] initWithRed:126 green:188 blue:121];
SDLRGBColor *white = [[SDLRGBColor alloc] initWithRed:249 green:251 blue:254];
SDLRGBColor *darkGrey = [[SDLRGBColor alloc] initWithRed:57 green:78 blue:96];
SDLRGBColor *grey = [[SDLRGBColor alloc] initWithRed:186 green:198 blue:210];

SDLSetDisplayLayout *setLayout = [[SDLSetDisplayLayout alloc] initWithPredefinedLayout:SDLPredefinedLayoutGraphicWithText];
setLayout.dayColorScheme = [[SDLTemplateColorScheme alloc] initWithPrimaryRGBColor:green secondaryRGBColor:grey backgroundRGBColor:white];
setLayout.nightColorScheme = [[SDLTemplateColorScheme alloc] initWithPrimaryRGBColor:green secondaryRGBColor:grey backgroundRGBColor:darkGrey];
```

##### Swift
```swift
let green = SDLRGBColor(red: 126, green: 188, blue: 121)
let white = SDLRGBColor(red: 249, green: 251, blue: 254)
let grey = SDLRGBColor(red: 186, green: 198, blue: 210)
let darkGrey = SDLRGBColor(red: 57, green: 78, blue: 96)

let setLayout = SDLSetDisplayLayout(predefinedLayout: .graphicWithText)
setLayout.dayColorScheme = SDLTemplateColorScheme(primaryRGBColor: green, secondaryRGBColor: grey, backgroundRGBColor: white)
setLayout.nightColorScheme = SDLTemplateColorScheme(primaryRGBColor: green, secondaryRGBColor: grey, backgroundRGBColor: darkGrey)
```
!@

@![android, javaSE, javaEE]
```java
// TODO
```
!@

## Customizing the Menu Title and Icon
You can also customize the title and icon of the main menu button that appears on your template layouts. The menu icon must already be uploaded with a specific name through the file manager; see the [Uploading Images guide](Other SDL Features/Uploading Images) for more information on how to upload your image.

@![iOS]
##### Objective-C
```objc
SDLSetGlobalProperties *setGlobals = [[SDLSetGlobalProperties alloc] init];
setGlobals.menuTitle = @"Custom Title";

// This must be uploaded before referencing the name here
setGlobals.menuIcon = [[SDLImage alloc] initWithName:@"Custom Icon Name" isTemplate:YES];

[self.sdlManager sendRequest:setGlobals withResponseHandler:^(SDLRPCRequest *request, SDLRPCResponse *response, NSError *error) {
    if (error != nil) {
        // Something went wrong
    }

    // The menu title and icon should be updated
}];
```

##### Swift
```swift
let setGlobals = SDLSetGlobalProperties()
setGlobals.menuTitle = "Custom Title"

// This must be uploaded before referencing the name here
setGlobals.menuIcon = SDLImage(name: "Custom Icon Name", isTemplate: true)

sdlManager.send(request: setGlobals) { (request, response, error) in
    if error != nil {
        // Something went wrong
    }

    // The menu title and icon should be updated
}
```
!@

@![android, javaSE, javaEE]
```java
// TODO
```
!@
