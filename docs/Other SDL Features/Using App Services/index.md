# Using App Services
App services is a powerful feature enabling both a new kind of vehicle-to-app communication and app-to-app communication via SDL.

App services are used to publish navigation, weather and media data (such as temperature, navigation waypoints, or the current playlist name). This data can then be used by both the vehicle head unit and, if the publisher of the app service desires, other SDL apps. Creating an app service is covered [in another guide](Other SDL Features/Creating an App Service).

Vehicle head units may use these services in various ways. One app service for each type will be the "active" service to the module. For media, for example, this will be the media app that the user is currently using or listening to. For navigation, it would be a navigation app that the user is using to navigate. For weather, it may be the last used weather app, or a user-selected default. The system may then use that service's data to perform various actions (such as navigating to an address with the active service or to display the temperature as provided from the active weather service).

An SDL app can also subscribe to a published app service. Once subscribed, the app will be sent the new data when the app service publisher updates its data. This guide will cover subscribing to a service. Subscribed apps can also send certain RPCs and generic URI-based actions (see the section [Sending an Action to a Service Provider](#4-sending-an-action-to-a-service-provider), below) to your service.

Currently, there is no high-level API support for using an app service, so you will have to use raw RPCs for all app service related APIs.

## Getting and Subscribing to Services
Once your app has connected to the head unit, you will first want to be notified of all available services and updates to the metadata of all services on the head unit. Second, you will narrow down your app to subscribe to an individual app service and subscribe to its data. Third, you may want to interact with that service through RPCs, or fourth, through service actions.

@![iOS]
!!! NOTE
Please note that if you are integrating an sdl_ios version less than v6.3, the example code in this guide will not work. We recommend updating to the latest release version.
!!!
!@

### 1. Getting and Subscribing to Available Services
To get information on all services published on the system, as well as on changes to published services, you will use the @![iOS]`GetSystemCapability` request / response as well as the `OnSystemCapabilityUpdated` notification. !@ @![android,javaSE,javaEE] `SystemCapabilityManager` to get the information. Because this information is initially available asynchronously, we have to attach an `OnSystemCapabilityListener` to the `getCapability` request.!@@![javascript] `SystemCapabilityManager` and await the `updateCapability` method to get the information!@

@![iOS]
##### Objective-C
```objc
id subscribedObserver = [self.sdlManager.systemCapabilityManager subscribeToCapabilityType:SDLSystemCapabilityTypeAppServices withUpdateHandler:^(SDLSystemCapability * _Nullable capability, BOOL subscribed, NSError * _Nullable error) {
    NSArray<SDLAppServiceCapability *> *appServices = capability.appServicesCapabilities.appServices;
    <#Use the app services records#>
}];
```

##### Swift
```swift
let subscribedObserver = sdlManager.systemCapabilityManager.subscribe(capabilityType: .appServices) { (capability, subscribed, error) in
    let appServices = capability?.appServicesCapabilities?.appServices
    <#Use the app services records#>
}
```
!@

@![android,javaSE,javaEE]
##### Java
```java
// Grab the capability once
sdlManager.getSystemCapabilityManager().getCapability(SystemCapabilityType.APP_SERVICES, new OnSystemCapabilityListener() {
    @Override
    public void onCapabilityRetrieved(Object capability) {
        AppServicesCapabilities servicesCapabilities = (AppServicesCapabilities) capability;
    }

    @Override
    public void onError(String info) {
        // Handle Error
    }
}, false);
...

// Subscribe to capability updates
sdlManager.getSystemCapabilityManager().addOnSystemCapabilityListener(SystemCapabilityType.APP_SERVICES, new OnSystemCapabilityListener() {
    @Override
    public void onCapabilityRetrieved(Object capability) {
        AppServicesCapabilities servicesCapabilities = (AppServicesCapabilities) capability;
    }

    @Override
    public void onError(String info) {
        // Handle Error
    }
});
```
!@

@![javascript]
```js
// Grab the capability once
const servicesCapabilities = await sdlManager.getSystemCapabilityManager().updateCapability(SDL.rpc.enums.SystemCapabilityType.APP_SERVICES);

...

// Subscribe to updates
sdlManager.getSystemCapabilityManager().addOnSystemCapabilityListener(SDL.rpc.enums.SystemCapabilityType.APP_SERVICES, (servicesCapabilities) => {

});
```
!@

#### Checking the App Service Capability
Once you've retrieved the initial list of app service capabilities (in the `GetSystemCapability` response), or an updated list of app service capabilities (from the `OnSystemCapabilityUpdated` notification), you may want to inspect the data to find what you are looking for. Below is example code with comments explaining what each part of the app service capability is used for.

@![iOS]
##### Objective-C
```objc
// From GetSystemCapabilityResponse
SDLGetSystemCapabilityResponse *getResponse = <#From wherever you got it#>;
SDLAppServicesCapabilities *capabilities = getResponse.systemCapability.appServicesCapabilities;

// This array contains all currently available app services on the system
NSArray<SDLAppServiceCapability *> *appServices = capabilities.appServices;
SDLAppServiceCapability *aCapability = appServices.firstObject;

// This will be nil since it's the first update
SDLServiceUpdateReason capabilityReason = aCapability.updateReason;

// The app service record will give you access to a service's generated id, which can be used to address the service directly (see below), it's manifest, used to see what data it supports, whether or not the service is published (it always will be here), and whether or not the service is the active service for its service type (only one service can be active for each type)
SDLAppServiceRecord *serviceRecord = aCapability.updatedAppServiceRecord;

// From OnSystemCapabilityUpdated
SDLOnSystemCapabilityUpdated *serviceNotification = <#From wherever you got it#>;
SDLAppServicesCapabilities *capabilities = serviceNotification.systemCapability.appServicesCapabilities;

// This array contains all recently updated services
NSArray<SDLAppServiceCapability *> *appServices = capabilities.appServices;
SDLAppServiceCapability *aCapability = appServices.firstObject;

// This won't be nil. It will tell you why a service is in the list of updates
SDLServiceUpdateReason capabilityReason = aCapability.updateReason;

// The app service record will give you access to a service's generated id, which can be used to address the service directly (see below), it's manifest, used to see what data it supports, whether or not the service is published (if it's not, it was just removed and should not be addressed), and whether or not the service is the active service for its service type (only one service can be active for each type)
SDLAppServiceRecord *serviceRecord = aCapability.updatedAppServiceRecord;
```

##### Swift
```swift
// From GetSystemCapabilityResponse
let getResponse: SDLGetSystemCapabilityResponse = <#From wherever you got it#>
let capabilities = getResponse.systemCapability.appServicesCapabilities

// This array contains all currently available app services on the system
let appServices: [SDLAppServiceCapability] = capabilities.appServices
let aCapability = appServices.first

// This will be nil since it's the first update
let capabilityReason = aCapability.updateReason

// The app service record will give you access to a service's generated id, which can be used to address the service directly (see below), it's manifest, used to see what data it supports, whether or not the service is published (it always will be here), and whether or not the service is the active service for its service type (only one service can be active for each type)
let serviceRecord = aCapability.updatedAppServiceRecord

// From OnSystemCapabilityUpdated
let serviceNotification: SDLOnSystemCapabilityUpdated = <#From wherever you got it#>
let capabilities = serviceNotification.systemCapability.appServicesCapabilities

// This array contains all recently updated services
let appServices: [SDLAppServiceCapability] = capabilities.appServices
let aCapability = appServices.first

// This won't be nil. It will tell you why a service is in the list of updates
let capabilityReason = aCapability.updateReason

// The app service record will give you access to a service's generated id, which can be used to address the service directly (see below), it's manifest, used to see what data it supports, whether or not the service is published (if it's not, it was just removed and should not be addressed), and whether or not the service is the active service for its service type (only one service can be active for each type)
let serviceRecord = aCapability.updatedAppServiceRecord
```
!@

@![android,javaSE,javaEE]
##### Java
```java
// This array contains all currently available app services on the system
List<AppServiceCapability> appServices = servicesCapabilities.getAppServices();

if (appServices!= null && appServices.size() > 0) {
    for (AppServiceCapability anAppServiceCapability : appServices) {
        // This will tell you why a service is in the list of updates
        ServiceUpdateReason updateReason = anAppServiceCapability.getUpdateReason();

        // The app service record will give you access to a service's generated id, which can be used to address the service directly (see below), it's manifest, used to see what data it supports, whether or not the service is published (it always will be here), and whether or not the service is the active service for its service type (only one service can be active for each type)
        AppServiceRecord serviceRecord = anAppServiceCapability.getUpdatedAppServiceRecord();
    }
}
```
!@

@![javascript]
```js
// This array contains all currently available app services on the system
const appServices = servicesCapabilities.getAppServices();

if (appServices !== null) {
    appServices.forEach(anAppServiceCapability => {
        // This will tell you why a service is in the list of updates
        const updateReason = anAppServiceCapability.getUpdateReason();
        // The app service record will give you access to a service's generated id, which can be used to address the service directly (see below), it's manifest, used to see what data it supports, whether or not the service is published (it always will be here), and whether or not the service is the active service for its service type (only one service can be active for each type)
        const serviceRecord = anAppServiceCapability.getUpdatedAppServiceRecord();
    });
}
```
!@

### 2. Getting and Subscribing to a Service Type's Data
Once you have information about all of the services available, you may want to view or subscribe to a service type's data. To do so, you will use the `GetAppServiceData` RPC.

Note that you will currently only be able to get data for the *active* service of the service type. You can attempt to make another service the active service by using the `PerformAppServiceInteraction` RPC, discussed below in [Sending an Action to a Service Provider](#4-sending-an-action-to-a-service-provider).

@![iOS]
##### Objective-C
```objc
// Get service data once
SDLGetAppServiceData *getServiceData = [[SDLGetAppServiceData alloc] initWithAppServiceType:SDLAppServiceTypeMedia];

// Subscribe to service data in perpetuity via `OnAppServiceData` notifications.
SDLGetAppServiceData *subscribeServiceData = [[SDLGetAppServiceData alloc] initAndSubscribeToAppServiceType:SDLAppServiceTypeMedia];

// Unsubscribe to service data previously subscribed
SDLGetAppServiceData *unsubscribeServiceData = [[SDLGetAppServiceData alloc] initWithAppServiceType:SDLAppServiceTypeMedia];
unsubscribeServiceData.subscribe = @NO;

// Get the service's data
[self.sdlManager sendRequest:getServiceData withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if (!response || !response.success.boolValue) {
        SDLLogE(@"Error sending get system capability: Req %@, Res %@, err %@", request, response, error);
        return;
    }

    SDLGetAppServiceDataResponse *serviceResponse = (SDLGetAppServiceDataResponse *)response;
    SDLMediaServiceData *mediaData = serviceResponse.serviceData.mediaServiceData;

    <#Use the mediaData#>
}];
```

##### Swift
```swift
// Get service data once
let getServiceData = SDLGetAppServiceData(appServiceType: .media)

// Subscribe to service data in perpetuity via `OnAppServiceData` notifications.
let subscribeServiceData = SDLGetAppServiceData(andSubscribeToAppServiceType: .media)

// Unsubscribe to service data previously subscribed
let unsubscribeServiceData = SDLGetAppServiceData(appServiceType: .media)
unsubscribeServiceData.subscribe = NSNumber(false)

// Get the service's data
let getServiceData = SDLGetAppServiceData(appServiceType: .media)
sdlManager.send(request: getServiceData) { (req, res, err) in
    guard let response = res as? SDLGetAppServiceDataResponse, response.success.boolValue == true, let mediaData = response.serviceData?.mediaServiceData else { return }

    <#Use the mediaData#>
}
```
!@

@![android,javaEE,javaSE]
##### Java
```java
// Get service data once
GetAppServiceData getAppServiceData = new GetAppServiceData(AppServiceType.MEDIA.toString());

// Subscribe to future updates if you want them
getAppServiceData.setSubscribe(true);

getAppServiceData.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse response) {
        if (response != null){
            GetAppServiceDataResponse serviceResponse = (GetAppServiceDataResponse) response;
            MediaServiceData mediaServiceData = serviceResponse.getServiceData().getMediaServiceData();
        }
    }
    @Override
    public void onError(int correlationId, Result resultCode, String info){
        // Handle Error
    }
});
sdlManager.sendRPC(getAppServiceData);

...

// Unsubscribe from updates
GetAppServiceData unsubscribeServiceData = new GetAppServiceData(AppServiceType.MEDIA.toString());
unsubscribeServiceData.setSubscribe(false);
sdlManager.sendRPC(unsubscribeServiceData);
```
!@

@![javascript]
```js
// Get service data once
const getAppServiceData = new SDL.rpc.messages.GetAppServiceData()
    .setServiceType(SDL.rpc.enums.AppServiceType.MEDIA);

// Subscribe to future updates if you want them
getAppServiceData.setSubscribe(true);

const response = await sdlManager.sendRpc(getAppServiceData).catch(error => error);
if (response !== null && ) {
    const mediaServiceData = response.getServiceData().getMediaServiceData();
}
...

// Unsubscribe from updates
const unsubscribeServiceData = new SDL.rpc.messages.GetAppServiceData(SDL.rpc.enums.AppServiceType.MEDIA);
unsubscribeServiceData.setSubscribe(false);

sdlManager.sendRpc(unsubscribeServiceData);
```
!@

## Interacting with a Service Provider
Once you have a service's data, you may want to interact with a service provider by sending RPCs or actions.

### 3. Sending RPCs to a Service Provider
Only certain RPCs are available to be passed to the service provider based on their service type. See the [Creating an App Service](Other SDL Features/Creating an App Service#supporting-service-rpcs-and-actions) guide (under the **Supporting Service RPCs and Actions** section) for a chart detailing which RPCs work with which service types. The RPC can only be sent to the active service of a specific service type, not to any inactive service.

Sending an RPC works exactly the same as if you were sending the RPC to the head unit system. The head unit will simply route your RPC to the appropriate app automatically.

!!! NOTE
Your app may need special permissions to use the RPCs that route to app service providers.
!!!

@![iOS]
##### Objective-C
```objc
SDLButtonPress *buttonPress = [[SDLButtonPress alloc] initWithButtonName:SDLButtonNameOk moduleType:SDLModuleTypeAudio moduleId:nil buttonPressMode:SDLButtonPressModeShort];
[self.sdlManager sendRequest:buttonPress withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if (!response || !response.success.boolValue) {
        SDLLogE(@"Error sending button press: Req %@, Res %@, err %@", request, response, error);
        return;
    }

    SDLButtonPressResponse *pressResponse = (SDLButtonPressResponse *)response;
    <#Use the response#>
}];
```

##### Swift
```swift
let buttonPress = SDLButtonPress(buttonName: .ok, moduleType: .audio, moduleId: nil, buttonPressMode: .short)
sdlManager.send(request: buttonPress) { (req, res, err) in
    guard let response = res as? SDLButtonPressResponse, response.success.boolValue == true else { return }

    <#Use the response#>
}
```
!@

@![android,javaSE,javaEE]
##### Java
```java
ButtonPress buttonPress = new ButtonPress();
buttonPress.setButtonPressMode(ButtonPressMode.SHORT);
buttonPress.setButtonName(ButtonName.OK);
buttonPress.setModuleType(ModuleType.AUDIO);
buttonPress.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse response) {
        <#Use the response#>
    }
    @Override
    public void onError(int correlationId, Result resultCode, String info){
        <#Handle the Error#>
    }
});
sdlManager.sendRPC(buttonPress);
```
!@

@![javascript]
```js
const buttonPress = new SDL.rpc.messages.ButtonPress()
    .setButtonPressMode(SDL.rpc.enums.ButtonPressMode.SHORT)
    .setButtonName(SDL.rpc.enums.ButtonName.OK)
    .setModuleType(SDL.rpc.enums.ModuleType.AUDIO);

const response = await sdlManager.sendRpc(buttonPress).catch(error => error);
```
!@

### 4. Sending an Action to a Service Provider
Actions are generic URI-based strings sent to any app service (active or not). You can also use actions to request to the system that they make the service the active service for that service type. Service actions are *schema-less*, i.e. there is no way to define the appropriate URIs through SDL. The service provider must document their list of available actions elsewhere (such as their website).

@![iOS]
##### Objective-C
```objc
SDLPerformAppServiceInteraction *performAction = [[SDLPerformAppServiceInteraction alloc] initWithServiceUri:@"<#sdlexample://x-callback-url/showText?x-source=MyApp&text=My%20Custom%20String#>" serviceID:@"<#Previously Retrived ServiceID#>" originApp:@"<#Your App Id#>" requestServiceActive:NO];
[self.sdlManager sendRequest:performAction withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if (!response || !response.success.boolValue) {
        SDLLogE(@"Error sending perform action: Req %@, Res %@, err %@", request, response, error);
        return;
    }

    SDLPerformAppServiceInteractionResponse *actionResponse = (SDLPerformAppServiceInteractionResponse *)response;
    <#Use the response#>
}];
```

##### Swift
```swift
let performAction = SDLPerformAppServiceInteraction(serviceUri: "<#sdlexample://x-callback-url/showText?x-source=MyApp&text=My%20Custom%20String#>", serviceID: "<#Previously Retrived ServiceID#>", originApp: "<#Your App Id#>", requestServiceActive: false)
sdlManager.send(request: performAction) { (req, res, err) in
    guard let response = res as? SDLPerformAppServiceInteractionResponse else { return }
    <#Check the error and response#>
}
```
!@

@![android,javaSE,javaEE]
##### Java
```java
PerformAppServiceInteraction performAppServiceInteraction = new PerformAppServiceInteraction("sdlexample://x-callback-url/showText?x-source=MyApp&text=My%20Custom%20String","<#Previously Retrieved ServiceID#>","<#Your App Id#>");
performAppServiceInteraction.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse response) {
        <#Use the response#>
    }
    @Override
    public void onError(int correlationId, Result resultCode, String info){
        <#Handle the Error#>
    }
});
sdlManager.sendRPC(performAppServiceInteraction);
```
!@

@![javascript]
```js
const performAppServiceInteraction = new SDL.rpc.messages.PerformAppServiceInteraction()
    .setServiceUri("sdlexample://x-callback-url/showText?x-source=MyApp&text=My%20Custom%20String")
    .setServiceID("<#Previously Retrieved ServiceID#>")
    .setOriginApp("<#Your App Id#>");

const response = await sdlManager.sendRpc(performAppServiceInteraction).catch(error => error);
```
!@

### 5. Getting a File from a Service Provider
In some cases, a service may upload an image that can then be retrieved from the module. First, you will need to get the image name from the @![iOS]`SDLAppServiceData`!@@![android,javaSE,javaEE,javascript]`AppServiceData`!@ (see [point 2](#2-getting-and-subscribing-to-a-service-types-data) above). Then you will use the image name to retrieve the image data. 

@![iOS]
##### Objective-C
```objc
SDLAppServiceData *data = <#Get the App Service Data#>;
SDLWeatherServiceData *weatherData = data.weatherServiceData;
SDLImage *currentForecastImage = weatherData.currentForecast.weatherIcon;
NSString *currentForecastImageName = currentForecastImage.value;
SDLGetFile *getCurrentForecastImage = [[SDLGetFile alloc] initWithFileName:currentForecastImageName];

__block NSUInteger imageDataLength = 0;
__block NSUInteger imageDataLengthReceived = 0;
NSMutableData *imageData = [[NSMutableData alloc] init];
[self.sdlManager sendRequest:getCurrentForecastImage withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    SDLGetFileResponse *getFileResponse = response;
    if (getFileResponse == nil || !response.success) {
        // Something went wrong, examine the resultCode and info
        return;
    }

    NSData *rpcImageData = response.bulkData;
    if (imageData == nil) {
        // There's no image data
        return;
    }

    [imageData appendData:rpcImageData];
    imageDataLengthReceived += rpcImageData.length;
    if (getFileResponse.offset == 0 && getFileResponse.length != nil) {
        imageDataLength = getFileResponse.length.unsignedIntegerValue;
    }

    if (imageDataLengthReceived < imageDataLength) {
        // Send additional GetFile requests to get the rest of the data using the offset parameter
    } else {
        // The file is complete, turn the file data into an image and use it.
    }
}];
```

##### Swift
```swift
let data: SDLAppServiceData = <#Get the App Service Data#>
let weatherData: SDLWeatherServiceData = data.weatherServiceData
guard let currentForecastImage = weatherData.currentForecast?.weatherIcon else {
    // The image doesn't exist, exit early
    return
}
let currentForecastImageName = currentForecastImage.value
let getCurrentForecastImage = SDLGetFile(fileName: currentForecastImageName)

var imageDataLength = 0
var imageDataLengthReceived = 0
var imageData = Data()
sdlManager.send(request: getCurrentForecastImage) { (req, res, err) in
    guard let response = res as? SDLGetFileResponse, response.success.boolValue == true, let rpcImageData = response.bulkData else {
        // Something went wrong, examine the resultCode and info
        return;
    }

    imageData.append(rpcImageData)
    imageDataLengthReceived += rpcImageData.count
    if response.offset?.intValue == 0, let rpcImageLength = response.length?.intValue {
        imageDataLength = rpcImageLength
    }

    if imageDataLengthReceived < imageDataLength {
        // Send additional GetFile requests to get the rest of the data using the offset parameter
    }
}
```

!@
@![android, javaSE, javaEE]
```java
AppServiceData appServiceData = <#Get the App Service Data#>;
WeatherServiceData weatherServiceData = appServiceData.getWeatherServiceData();
if (weatherServiceData == null || weatherServiceData.getCurrentForecast() == null || weatherServiceData.getCurrentForecast().getWeatherIcon() == null) {
    // The image doesn't exist, exit early
    return;
}
String currentForecastImageName = weatherServiceData.getCurrentForecast().getWeatherIcon().getValue();

GetFile getFile = new GetFile(currentForecastImageName);
getFile.setAppServiceId(<#Service ID>);
getFile.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse response) {
        GetFileResponse getFileResponse = (GetFileResponse) response;
        byte[] fileData = getFileResponse.getBulkData();
        SdlArtwork sdlArtwork = new SdlArtwork(fileName, FileType.GRAPHIC_PNG, fileData, false);
        // Use the sdlArtwork 
    }

    @Override
    public void onError(int correlationId, Result resultCode, String info) {
        // Something went wrong, examine the resultCode and info
    }
});
sdlManager.sendRPC(getFile);
```
!@

@![javascript]
```js
const appServiceData = <#Get the App Service Data#>;
const weatherServiceData = appServiceData.getWeatherServiceData();

if (weatherServiceData === null || weatherServiceData.getCurrentForecast() === null || weatherServiceData.getCurrentForecast().getWeatherIcon() === null) {
    // The image doesn't exist, exit early
    return;
}
const currentForecastImageName = weatherServiceData.getCurrentForecast().getWeatherIcon().getValueParam();

const getFile = new SDL.rpc.messages.GetFile()
    .setFileName(currentForecastImageName)
    .setAppServiceId(<#Service ID>);

const getFileResponse = await sdlManager.sendRpc(getFile).catch(error => error);
const fileData = getFileResponse.getBulkData();
const sdlArtwork = new SDL.manager.file.filetypes.SdlArtwork(fileName, FileType.GRAPHIC_PNG, fileData, false);
// Use the sdlArtwork 
```
!@
