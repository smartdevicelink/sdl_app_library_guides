# Main Screen Templates
Each head unit manufacturer supports a set of user interface templates. These templates determine the position and size of the text, images, and buttons on the screen. Once the app has connected successfully with an SDL enabled head unit, a list of supported templates is available on @![iOS]`SDLManager.systemCapabilityManager.defaultMainWindowCapability.templatesAvailable`!@@![android, javaSE, javaEE, javascript]`sdlManager.getSystemCapabilityManager().getDefaultMainWindowCapability().getTemplatesAvailable()`!@.

## Change the Template
To change a template at any time, use @![iOS]`[SDLScreenManager changeLayout:]`!@@![android, javaSE, javaEE, javascript]`ScreenManager.changeLayout()`!@. This guide requires SDL @![android, javaSE, javaEE]Java Suite version 5.0!@@![iOS]iOS version 7.0!@ @![javascript]JavaScript Suite version 1.2!@. If using an older version, use the `SetDisplayLayout` RPC.

!!! NOTE
When changing the layout, you may get an error or failure if the update is "superseded." This isn't technically a failure, because changing the layout has not yet been attempted. The layout or batched operation was cancelled before it could be completed because another operation was requested. The layout change will then be inserted into the future operation and completed then.
!!!

@![iOS]
|~
```objc
[self.sdlManager.screenManager changeLayout:[[SDLTemplateConfiguration alloc] initWithTemplate:SDLPredefinedLayoutGraphicWithText] withCompletionHandler:^(NSError * _Nullable error) {
    if (error != nil) {
        // Print out the error if there is one and return early
        return;
    }
    // The template has been set successfully
}];
```
```swift
sdlManager.screenManager.changeLayout(SDLTemplateConfiguration(predefinedLayout: .graphicWithText)) { err in
    if let error = err {
        // Print out the error if there is one and return early
        return
    }
    // The template has been set successfully
}
```
~|
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
|~
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
~|
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
sdlManager.getScreenManager().setPrimaryGraphic(artwork);
const success = await sdlManager.getScreenManager().commit();
if (success) {
    console.log('Text, Graphic, and Template changed successful');
}
```
!@

@![iOS]
When changing screen layouts and template data (for example, to show a weather hourly data screen vs. a daily weather screen), it is recommended to encapsulate these updates into a class or method. Doing so is a good way to keep SDL UI changes organized. A fully-formed example of this can be seen in the [example weather app](https://github.com/SmartDeviceLink-Examples/example_weather_app_ios). Below is a generic example.
!@

@![android,javaEE,javaSE]
When changing screen layouts and template data (for example, to show a weather hourly data screen vs. a daily weather screen), it is recommended to encapsulate these updates into a class or method. Doing so is a good way to keep SDL UI changes organized. A fully-formed example of this can be seen in the [example weather app](https://github.com/SmartDeviceLink-Examples/example_weather_app_android). Below is a generic example.
!@

@![javascript]
When changing screen layouts and template data (for example, to show a weather hourly data screen vs. a daily weather screen), it is recommended to encapsulate these updates into a class or method. Doing so is a good way to keep SDL UI changes organized. Below is a generic example.
!@

## Screen Change Example Code
This example code creates an interface that can be implemented by various "screens" of your SDL app. This is a recommended design pattern so that you can separate your code to only involve the data models you need. This is just a simple example and your own needs may be different.

### Screen Change Example Interface
All screens will need to have access to the @![iOS]`SDLScreenManager`!@@![android, javaEE, javaSE, javascript]`ScreenManager`!@ object and a function to display the screen. Therefore, it is recommended to create a generic interface for all screens to follow. For the example below, the `CustomSDLScreen` protocol requires an initializer with the parameters `SDLManager` and a `showScreen` method.

@![iOS]
|~
```objc
// CustomSDLScreen.h
@class SDLManager;

@protocol CustomSDLScreen <NSObject>

@required
- (instancetype)initWithManager:(SDLManager *)sdlManager;
- (void)showScreen;

@end
```
```swift
protocol CustomSDLScreen {
    init(sdlManager: SDLManager)
    func showScreen()
}
```
~|
!@

@![android,javaSE,javaEE]
```java
public class CustomSdlScreen {
    protected SdlManager sdlManager;

    public CustomSdlScreen(SdlManager sdlManager) {
        this.sdlManager = sdlManager;
    }

    public void showScreen() {
        // stub
    }
}
```
!@

@![javascript]
```js
class CustomSdlScreen {
    constructor (sdlManager) {
        this.sdlManager = sdlManager;
    }
    showScreen () {
        // stub
    }
}
```
!@

### Screen Change Example Implementations
The following example code shows a few implementations of the example screen changing protocol. A good practice for screen classes is to keep screen data in a view model. Doing so will add a layer of abstraction for exposing public properties and commands to the screen.

For the example below, the `HomeScreen` class will inherit the `CustomSDLScreen` interface and will have a property of type `HomeDataViewModel`. The screen manager will change its text fields based on the view model's data. In addition, the home screen will also create a navigation button to open the `ButtonSDLScreen` when pressed.

@![iOS]
|~
```objc
// HomeSDLScreen.h
#import <Foundation/Foundation.h>
#import "CustomSDLScreen.h"

@class HomeDataViewModel;
@class SDLManager;
@class SDLScreenManager;
@class SDLSoftButtonObject;

NS_ASSUME_NONNULL_BEGIN

@interface HomeSDLScreen : NSObject<CustomSDLScreen>

@end

NS_ASSUME_NONNULL_END

/************************************************************************/
// HomeSDLScreen.m
#import "HomeSDLScreen.h"

#import "ButtonSDLScreen.h"
#import "CustomSDLScreen.h"
#import "HomeDataViewModel.h"
#import "SmartDeviceLink.h"

@interface HomeSDLScreen()

@property (weak, nonatomic) SDLManager *sdlManager;
// A button to navigate to the ButtonSDLScreen
@property (strong, nonatomic) ButtonSDLScreen *buttonScreen;
// An example of your data model that will feed data to the SDL screen's UI
@property (strong, nonatomic) HomeDataViewModel *homeDataViewModel;

@end

@implementation HomeSDLScreen

- (instancetype)initWithManager:(SDLManager *)sdlManager {
    self = [super init];
    if (!self) { return nil; }

    _sdlManager = sdlManager;
    _buttonScreen = [[ButtonSDLScreen alloc] initWithManager:sdlManager];
    _homeDataViewModel = [[HomeDataViewModel alloc] init];

    return self;
}

- (void)showScreen {
    // Batch updates
    [self.sdlManager.screenManager beginUpdates];
    // Change template to Graphics With Text and SoftButtons
    [self.sdlManager.screenManager changeLayout:[[SDLTemplateConfiguration alloc] initWithTemplate:SDLPredefinedLayoutGraphicWithTextAndSoftButtons] withCompletionHandler:nil];
    // Assign text fields to view model data
    self.sdlManager.screenManager.textField1 = self.homeDataViewModel.text1;
    self.sdlManager.screenManager.textField2 = self.homeDataViewModel.text2;
    self.sdlManager.screenManager.textField3 = self.homeDataViewModel.text3;
    self.sdlManager.screenManager.textField4 = self.homeDataViewModel.text4;
    // Create and assign a button to navigate to the ButtonSDLScreen
    SDLSoftButtonObject *navigationButton = [[SDLSoftButtonObject alloc] initWithName:@"ButtonSDLScreen" text:@"Button Screen" artwork:nil handler:^(SDLOnButtonPress * _Nullable buttonPress, SDLOnButtonEvent * _Nullable buttonEvent) {
        if (buttonPress == nil) { return; }
        [self.buttonScreen showScreen];
    }];
    self.sdlManager.screenManager.softButtonObjects = @[navigationButton];
    // Change Primary Graphic
    self.sdlManager.screenManager.primaryGraphic = <#SDLArtwork#>;
    [self.sdlManager.screenManager endUpdates];
}

@end
```
```swift
struct HomeSDLScreen: CustomSDLScreen {
    let sdlManager: SDLManager
    let buttonScreen: ButtonSDLScreen
    // An example of your data model that will feed data to the SDL screen's UI
    let homeDataViewModel = HomeDataViewModel()

    init(sdlManager: SDLManager) {
        self.sdlManager = sdlManager
        self.buttonScreen = ButtonSDLScreen(sdlManager: sdlManager)
    }

    func showScreen() {
        // Batch updates
        self.sdlManager.screenManager.beginUpdates()
        // Change template to Graphics With Text and Soft Buttons
        self.sdlManager.screenManager.changeLayout(SDLTemplateConfiguration(predefinedLayout: .graphicWithTextAndSoftButtons))
        // Assign text fields to view model data
        self.sdlManager.screenManager.textField1 = self.homeDataViewModel.text1
        self.sdlManager.screenManager.textField2 = self.homeDataViewModel.text2
        self.sdlManager.screenManager.textField3 = self.homeDataViewModel.text3
        self.sdlManager.screenManager.textField4 = self.homeDataViewModel.text4
        // Create and assign a button to navigate to the ButtonSDLScreen
        let navigationButton = SDLSoftButtonObject(name: "ButtonSDLScreen", text: "Button Screen", artwork: nil) { (buttonPress, buttonEvent) in
            guard buttonPress != nil else { return }
            self.buttonScreen.showScreen()
        }
        self.sdlManager.screenManager.softButtonObjects = [navigationButton]
        self.sdlManager.screenManager.endUpdates()
    }
}
```
~|
!@

@![android,javaSE,javaEE]
```java
public class HomeSdlScreen extends CustomSdlScreen {
    private ButtonSdlScreen buttonScreen;
    // An example of your data model that will feed data to the SDL screen's UI
    private HomeDataViewModel homeDataViewModel;

    public HomeSdlScreen(SdlManager sdlManager) {
        super(sdlManager);

        buttonScreen = new ButtonSdlScreen(sdlManager);
        homeDataViewModel = new HomeDataViewModel();
    }

    public void showScreen() {
        // Batch Updates
        sdlManager.getScreenManager().beginTransaction();
        // Change template to Graphics With Text and Soft Buttons
        TemplateConfiguration templateConfiguration = new TemplateConfiguration().setTemplate(PredefinedLayout.GRAPHIC_WITH_TEXT.toString());
        sdlManager.getScreenManager().changeLayout(templateConfiguration, new CompletionListener() {
            @Override
            public void onComplete(boolean success) {}
        });

        // Assign text fields to view model data
        sdlManager.getScreenManager().setTextField1(homeDataViewModel.getText1());
        sdlManager.getScreenManager().setTextField2(homeDataViewModel.getText2());
        sdlManager.getScreenManager().setTextField3(homeDataViewModel.getText3());
        sdlManager.getScreenManager().setTextField4(homeDataViewModel.getText4());
        // Create and assign a button to navigate to the ButtonSdlScreen
        SoftButtonState textState = new SoftButtonState("ButtonSdlScreenState", "Button Screen", null);
        SoftButtonObject navigationButton = new SoftButtonObject("ButtonSdlScreen", Collections.singletonList(textState), textState.getName(), new SoftButtonObject.OnEventListener() {
            @Override
            public void onPress(SoftButtonObject softButtonObject, OnButtonPress onButtonPress) {
                buttonScreen.showScreen();
            }

            @Override
            public void onEvent(SoftButtonObject softButtonObject, OnButtonEvent onButtonEvent) {
            }
        });
        sdlManager.getScreenManager().setSoftButtonObjects(Collections.singletonList(navigationButton));
        sdlManager.getScreenManager().commit(new CompletionListener() {
            @Override
            public void onComplete(boolean success) {}
        });
    }
}
```
!@

@![javascript]
```js
class HomeSdlScreen extends CustomSdlScreen {
    constructor (sdlManager) {
        super(sdlManager);
        this.buttonScreen = new ButtonSdlScreen(sdlManager);
        this.homeDataViewModel = new HomeDataViewModel(); // holds plain object data
    }

    showScreen () {
        const screenManager = this.sdlManager.getScreenManager();
        // Batch Updates
        screenManager.beginTransaction();
        // Change template to Graphics With Text and Soft Buttons
        screenManager.changeLayout(new SDL.rpc.structs.TemplateConfiguration()
            .setTemplate(SDL.rpc.enums.PredefinedLayout.GRAPHIC_WITH_TEXT_AND_SOFTBUTTONS));
        // Assign text fields to view model data
        screenManager.setTextField1(this.homeDataViewModel.text1);
        screenManager.setTextField2(this.homeDataViewModel.text2);
        screenManager.setTextField3(this.homeDataViewModel.text3);
        screenManager.setTextField4(this.homeDataViewModel.text4);
        // Create and assign a button to navigate to the ButtonSdlScreen
        const navigationButton = new SDL.manager.screen.utils.SoftButtonObject('ButtonSdlScreen', [new SDL.manager.screen.utils.SoftButtonState('ButtonSdlScreen', 'Button Screen')], 'ButtonSdlScreen', (id, rpc) => {
            if (rpc instanceof SDL.rpc.messages.OnButtonPress) {
                this.buttonScreen.showScreen();
            }
        });
        screenManager.setSoftButtonObjects([navigationButton]);
        screenManager.commit();
    }
}
```
!@

The `ButtonSDLScreen` follows the same patterns as the `HomeSDLScreen` but has minor implementation differences. The screen's view model `ButtonDataViewModel` contains properties unique to the `ButtonSDLScreen` such as text fields and an array of soft button objects. It also changes the template configuration to tiles only.

@![iOS]
|~
```objc
// ButtonSDLScreen.h
#import <Foundation/Foundation.h>
#import "CustomSDLScreen.h"

@class ButtonDataViewModel;
@class SDLManager;
@class SDLScreenManager;

NS_ASSUME_NONNULL_BEGIN

@interface ButtonSDLScreen : NSObject<CustomSDLScreen>

@end

NS_ASSUME_NONNULL_END

/************************************************************************/
// ButtonSDLScreen.m
#import "ButtonSDLScreen.h"

#import "ButtonDataViewModel.h"
#import "CustomSDLScreen.h"
#import "SmartDeviceLink.h"

@interface ButtonSDLScreen()

@property (strong, nonatomic) SDLManager *sdlManager;
// An example of your data model that will feed data to the SDL screen's UI
@property (strong, nonatomic) ButtonDataViewModel *buttonDataViewModel;

@end

@implementation ButtonSDLScreen

- (instancetype)initWithManager:(SDLManager *)sdlManager {
    self = [super init];
    if (!self) { return nil; }

    _sdlManager = sdlManager;
    _buttonDataViewModel = [[ButtonDataViewModel alloc] init];

    return self;
}

- (void)showScreen {
    // Batch Updates
    [self.sdlManager.screenManager beginUpdates];
    // Change template to Tiles Only
    [self.sdlManager.screenManager changeLayout:[[SDLTemplateConfiguration alloc] initWithTemplate:SDLPredefinedLayoutTilesOnly] withCompletionHandler:nil];
    // Assign soft button objects to view model buttons array
    self.sdlManager.screenManager.softButtonObjects = self.buttonDataViewModel.buttons;
    [self.sdlManager.screenManager endUpdates];
}

@end
```
```swift
struct ButtonSDLScreen: CustomSDLScreen {
    let sdlManager: SDLManager
    // An example of your data model that will feed data to the SDL screen's UI
    let buttonDataViewModel = ButtonDataViewModel()

    init(sdlManager: SDLManager) {
        self.sdlManager = sdlManager
    }

    func showScreen() {
        // Batch Updates
        self.sdlManager.screenManager.beginUpdates()
        // Change template to Tiles Only
        self.sdlManager.screenManager.changeLayout(SDLTemplateConfiguration(predefinedLayout: .tilesOnly))
        // Assign soft button objects to view model buttons array
        self.sdlManager.screenManager.softButtonObjects = buttonDataViewModel.buttons
        self.sdlManager.screenManager.endUpdates()
    }
}
```
~|
!@

@![android,javaSE,javaEE]
```java
public class ButtonSdlScreen extends CustomSdlScreen {
    private ButtonDataViewModel buttonDataViewModel;

    public ButtonSdlScreen(SdlManager sdlManager) {
        super(sdlManager);


    }

    public void showScreen() {
        sdlManager.getScreenManager().beginTransaction();
        TemplateConfiguration templateConfiguration = new TemplateConfiguration().setTemplate(PredefinedLayout.TILES_ONLY.toString());
        sdlManager.getScreenManager().changeLayout(templateConfiguration, new CompletionListener() {
            @Override
            public void onComplete(boolean success) {}
        });
        sdlManager.getScreenManager().setSoftButtonObjects(buttonDataViewModel.getButtonObjects());
        sdlManager.getScreenManager().commit(new CompletionListener() {
            @Override
            public void onComplete(boolean success) {}
        });
    }
}
```
!@

@![javascript]
```js
class ButtonSdlScreen extends CustomSdlScreen {
    constructor (sdlManager) {
        super(sdlManager);
        this.buttonDataViewModel = new ButtonDataViewModel(); // holds plain object data
    }
    showScreen () {
        const screenManager = this.sdlManager.getScreenManager();
        // Batch Updates
        screenManager.beginTransaction();
        // Change template to Tiles Only
        screenManager.changeLayout(new SDL.rpc.structs.TemplateConfiguration()
            .setTemplate(SDL.rpc.enums.PredefinedLayout.TILES_ONLY));
        // Assign soft button objects to view model buttons array
        screenManager.setSoftButtonObjects(this.buttonDataViewModel.buttons);
        screenManager.commit();
    }
}
```
!@

## Available Templates
There are fifteen standard templates to choose from, however some head units may only support a subset of these templates. The following examples show how templates will appear on the [Generic HMI](https://github.com/smartdevicelink/generic_hmi) and [Ford's SYNC® 3 HMI](https://developer.ford.com).

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
![SYNC® 3 - Graphic with Tiles](assets/SYNC3HMI/SYNC3_graphic_with_tiles.bmp)

#### Tiles with Graphic
![SYNC® 3 - Tiles with Graphic](assets/SYNC3HMI/SYNC3_tiles_with_graphic.bmp)

#### Graphic with Text and Soft Buttons
![SYNC® 3 - Graphic with Text and Soft Buttons](assets/SYNC3HMI/SYNC3_graphic_with_text_and_soft_buttons.bmp)

#### Text and Soft Buttons with Graphic
![SYNC® 3 Text and Soft Buttons with Graphic](assets/SYNC3HMI/SYNC3_text_and_soft_buttons_with_graphic.bmp)

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
