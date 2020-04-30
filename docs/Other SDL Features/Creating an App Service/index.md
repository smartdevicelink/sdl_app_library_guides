# Creating an App Service (RPC v5.1+)
App services is a powerful feature enabling both a new kind of vehicle-to-app communication and app-to-app communication via SDL.

App services are used to publish navigation, weather and media data (such as temperature, navigation waypoints, or the current playlist name). This data can then be used by both the vehicle head unit and, if the publisher of the app service desires, other SDL apps.

Vehicle head units may use these services in various ways. One app service for each type will be the "active" service to the module. For media, for example, this will be the media app that the user is currently using or listening to. For navigation, it would be a navigation app that the user is using to navigate. For weather, it may be the last used weather app, or a user-selected default. The system may then use that service's data to perform various actions (such as navigating to an address with the active service or to display the temperature as provided from the active weather service).

An SDL app can also subscribe to a published app service. Once subscribed, the app will be sent the new data when the app service publisher updates its data. To find out more about how to subscribe to an app service check out the [Using App Services](Other SDL Features/Using App Services) section. Subscribed apps can also send certain RPCs and generic URI-based actions (see the section Supporting App Actions, below) to your service.

Currently, there is no high-level API support for publishing an app service, so you will have to use raw RPCs for all app service related APIs.

Using an app service is covered [in another guide](Other SDL Features/Using App Services).

## App Service Types
Apps are able to declare that they provide an app service by publishing an app service manifest. Three types of app services are currently available and more will be made available over time. The currently available types are: Media, Navigation, and Weather. An app may publish multiple services (one for each of the different service types) if desired.

## Publishing an App Service
Publishing a service is a multi-step process. First, you need to create your app service manifest. Second, you will publish your app service to the module. Third, you will publish the service data using `OnAppServiceData`. Fourth, you must listen for data requests and respond accordingly. Fifth, if your app service supports handling of RPCs related to your service you must listen for these RPC requests and handle them accordingly. Sixth, optionally, you can support URI-based app actions. Finally, if necessary, you can you update or delete your app service manifest.

@![iOS]
!!! NOTE
Please note that if you are integrating an sdl_ios version less than v6.3, the example code in this guide will not work. We recommend updating to the latest release version.
!!!
!@

### 1. Creating an App Service Manifest
The first step to publishing an app service is to create an @![iOS]`SDLAppServiceManifest`!@ @![android,javaSE,javaEE]`AppServiceManifest`!@ object. There is a set of generic parameters you will need to fill out as well as service type specific parameters based on the app service type you are creating.

@![iOS]
##### Objective-C
```objc
SDLAppServiceManifest *manifest = [[SDLAppServiceManifest alloc] initWithAppServiceType:SDLAppServiceTypeMedia];
manifest.serviceName = @"My Media App"; // Must be unique across app services.
manifest.serviceIcon = [[SDLImage alloc] initWithName:@"Service Icon Name" isTemplate:NO]; // Previously uploaded service icon. This could be the same as your app icon.
manifest.allowAppConsumers = @YES; // Whether or not other apps can view your data in addition to the head unit. If set to `NO` only the head unit will have access to this data.
manifest.maxRPCSpecVersion = [[SDLMsgVersion alloc] initWithMajorVersion:5 minorVersion:0 patchVersion:0]; // An *optional* parameter that limits the RPC spec versions you can understand to the provided version *or below*.
manifest.handledRPCs = @[]; // If you add function ids to this *optional* parameter, you can support newer RPCs on older head units (that don't support those RPCs natively) when those RPCs are sent from other connected applications.
manifest.mediaServiceManifest = <#Code#> // Covered below
```

##### Swift
```swift
let manifest = SDLAppServiceManifest(appServiceType: .media)
manifest.serviceIcon = SDLImage(name:"Service Icon Name", isTemplate: false) // Previously uploaded service icon. This could be the same as your app icon.
manifest.allowAppConsumers = true; // Whether or not other apps can view your data in addition to the head unit. If set to `NO` only the head unit will have access to this data.
manifest.maxRPCSpecVersion = SDLMsgVersion(majorVersion: 5, minorVersion: 0, patchVersion: 0) // An *optional* parameter that limits the RPC spec versions you can understand to the provided version *or below*.
manifest.handledRPCs = []; // If you add function ids to this *optional* parameter, you can support newer RPCs on older head units (that don't support those RPCs natively) when those RPCs are sent from other connected applications.
manifest.mediaServiceManifest = <#Code#> // Covered below
```
!@

@![android,javaSE,javaEE]
```java
AppServiceManifest manifest = new AppServiceManifest(AppServiceType.MEDIA.toString());
manifest.setServiceName("My Media App"); // Must be unique across app services.
manifest.setServiceIcon(new Image("Service Icon Name", ImageType.DYNAMIC)); // Previously uploaded service icon. This could be the same as your app icon.
manifest.setAllowAppConsumers(true); // Whether or not other apps can view your data in addition to the head unit. If set to `false` only the head unit will have access to this data.
manifest.setRpcSpecVersion(new SdlMsgVersion(5,0)); // An *optional* parameter that limits the RPC spec versions you can understand to the provided version *or below*.
manifest.setHandledRpcs(List<FunctionID>); // If you add function ids to this *optional* parameter, you can support newer RPCs on older head units (that don't support those RPCs natively) when those RPCs are sent from other connected applications.
manifest.setMediaServiceManifest(<#Code#>); // Covered Below
```
!@

#### Creating a Media Service Manifest
Currently, there's no information you have to provide in your media service manifest! You'll just have to create an empty media service manifest and set it into your general app service manifest.

@![iOS]

##### Objective-C
```objc
SDLMediaServiceManifest *mediaManifest = [[SDLMediaServiceManifest alloc] init];
manifest.mediaServiceManifest = mediaManifest;
```

##### Swift
```swift
let mediaManifest = SDLMediaServiceManifest()
manifest.mediaServiceManifest = mediaManifest
```
!@

@![android,javaSE,javaEE]
```java
MediaServiceManifest mediaManifest = new MediaServiceManifest();
manifest.setMediaServiceManifest(mediaManifest);
```
!@

#### Creating a Navigation Service Manifest
You will need to create a navigation manifest if you want to publish a navigation service. You will declare whether or not your navigation app will accept waypoints. That is, if your app will support receiving _multiple_ points of navigation (e.g. go to this McDonalds, then this Walmart, then home).

@![iOS]

##### Objective-C
```objc
SDLNavigationServiceManifest *navigationManifest = [[SDLNavigationServiceManifest alloc] initWithAcceptsWayPoints:YES];
manifest.navigationServiceManifest = navigationManifest;
```

##### Swift
```swift
let navigationManifest = SDLNavigationServiceManifest(acceptsWayPoints: true)
manifest.navigationServiceManifest = navigationManifest
```
!@

@![android,javaSE,javaEE]
```java
NavigationServiceManifest navigationManifest = new NavigationServiceManifest();
navigationManifest.setAcceptsWayPoints(true);
manifest.setNavigationServiceManifest(navigationManifest);
```
!@

#### Creating a Weather Service Manifest
You will need to create a weather service manifest if you want to publish a weather service. You will declare the types of data your service provides in its @![iOS]`SDLWeatherServiceData`!@ @![android,javaSE,javaEE] `WeatherServiceData` !@.

@![iOS]

##### Objective-C
```objc
SDLWeatherServiceManifest *weatherManifest = [[SDLWeatherServiceManifest alloc] initWithCurrentForecastSupported:YES maxMultidayForecastAmount:10 maxHourlyForecastAmount:24 maxMinutelyForecastAmount:60 weatherForLocationSupported:YES];
manifest.weatherServiceManifest = weatherManifest;
```

##### Swift
```swift
let weatherManifest = SDLWeatherServiceManifest(currentForecastSupported: true, maxMultidayForecastAmount: 10, maxHourlyForecastAmount: 24, maxMinutelyForecastAmount: 60, weatherForLocationSupported: true)
manifest.weatherServiceManifest = weatherManifest
```
!@

@![android,javaSE,javaEE]
```java
WeatherServiceManifest weatherManifest = new WeatherServiceManifest();
weatherManifest.setCurrentForecastSupported(true);
weatherManifest.setMaxMultidayForecastAmount(10);
weatherManifest.setMaxHourlyForecastAmount(24);
weatherManifest.setMaxMinutelyForecastAmount(60);
weatherManifest.setWeatherForLocationSupported(true);
manifest.setWeatherServiceManifest(weatherManifest);
```
!@

### 2. Publish Your Service
Once you have created your service manifest, publishing your app service is simple.

@![iOS]

##### Objective-C
```objc
SDLPublishAppService *publishServiceRequest = [[SDLPublishAppService alloc] initWithAppServiceManifest:<#Manifest Object#>];
[self.sdlManager sendRequest:publishServiceRequest withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if (error != nil || !response.success.boolValue) { return; }

    SDLPublishAppServiceResponse *publishServiceResponse = (SDLPublishAppServiceResponse *)response;
    SDLAppServiceRecord *serviceRecord = publishServiceResponse.appServiceRecord
    <#Use the response#>
}];
```

##### Swift
```swift
let publishServiceRequest = SDLPublishAppService(appServiceManifest: <#Manifest Object#>)
sdlManager.send(request: publishServiceRequest) { (req, res, err) in
    guard let response = res as? SDLPublishAppServiceResponse, response.success.boolValue == true, err == nil else { return }

    let serviceRecord = response.appServiceRecord
    <#Use the response#>
}
```
!@

@![android,javaSE,javaEE]
```java
PublishAppService publishServiceRequest = new PublishAppService();
publishServiceRequest.setAppServiceManifest(manifest);
publishServiceRequest.setOnRPCResponseListener(new OnRPCResponseListener() {
	@Override
	public void onResponse(int correlationId, RPCResponse response) {
		<#Use the response#>
	}

	@Override
	public void onError(int correlationId, Result resultCode, String info){
		<#Error Handling#>
	}
});
sdlManager.sendRPC(publishServiceRequest);
```
!@

Once you have your publish app service response, you will need to store the information provided in its `appServiceRecord` property. You will need the information later when you want to update your service's data.

#### Watching for App Record Updates
As noted in the introduction to this guide, one service for each type may become the "active" service. If your service is the active service, your @![iOS]`SDLAppServiceRecord`!@ @![android,javaEE,javaSE]`AppServiceRecord`!@ parameter `serviceActive` will be updated to note that you are now the active service.

After the initial app record is passed to you in the @![iOS]`SDLPublishAppServiceResponse`!@ @![android,javaSE,javaEE] `PublishAppServiceResponse`!@, you will need to be notified of changes in order to observe whether or not you have become the active service. To do so, you will have to observe the new @![iOS]`SDLSystemCapabilityTypeAppServices`!@ @![android,javaSE,javaEE]`SystemCapabilityType.APP_SERVICES`!@ using `GetSystemCapability` and @![iOS]`OnSystemCapability`!@ @![android,javaSE,javaEE] `OnSystemCapabilityUpdated`!@.

For more information, see the [Using App Services guide](Other SDL Features/Using App Services) and see the "Getting and Subscribing to Services" section.

### 3. Update Your Service's Data
After your service is published, it's time to update your service data. First, you must send an `onAppServiceData` RPC notification with your updated service data. RPC notifications are different than RPC requests in that they will not receive a response from the connected head unit @![iOS], and must use a different `SDLManager` method call to send!@.

!!! NOTE
You should only update your service's data when you are the active service; service consumers will only be able to see your data when you are the active service.
!!!

First, you will have to create an @![iOS]`SDLMediaServiceData`!@ @![android,javaSE,javaEE]`MediaServiceData`!@, @![iOS]`SDLNavigationServiceData`!@ @![android,javaSE,javaEE]`NavigationServiceData`!@ or @![iOS]`SDLWeatherServiceData`!@ @![android,javaSE,javaEE]
`WeatherServiceData`!@ object with your service's data. Then, add that service-specific data object to an @![iOS]`SDLAppServiceData`!@ @![android,javaSE,javaEE]`AppServiceData`!@ object. Finally, create an @![iOS]`SDLOnAppServiceData`!@ @![android,javaSE,javaEE]`OnAppServiceData`!@ notification, append your @![iOS]`SDLAppServiceData`!@ @![android,javaEE,javaSE]`AppServiceData`!@ object, and send it.

#### Media Service Data
@![iOS]

##### Objective-C
```objc
SDLImage *currentImage = [[SDLImage alloc] initWithName:@"some artwork name" isTemplate:NO];
SDLMediaServiceData *mediaData = [[SDLMediaServiceData alloc] initWithMediaType:SDLMediaTypeMusic mediaImage:currentImage mediaTitle:@"Some media title" mediaArtist:@"Some media artist" mediaAlbum:@"Some album" playlistName:@"Some playlist" isExplicit:YES trackPlaybackProgress:45 trackPlaybackDuration:90 queuePlaybackProgress:45 queuePlaybackDuration:150 queueCurrentTrackNumber:2 queueTotalTrackCount:3];
SDLAppServiceData *appData = [[SDLAppServiceData alloc] initWithMediaServiceData:mediaData serviceId:myServiceId];

SDLOnAppServiceData *onAppData = [[SDLOnAppServiceData alloc] initWithServiceData:appData];
[self.sdlManager sendRPC:onAppData];
```

##### Swift
```swift
let currentImage = SDLImage(name: "some artwork name", isTemplate: false)
let mediaData = SDLMediaServiceData(mediaType: .music, mediaImage: currentImage, mediaTitle: "Some media title", mediaArtist: "Some artist", mediaAlbum: "Some album", playlistName: "Some playlist", isExplicit: true, trackPlaybackProgress: 45, trackPlaybackDuration: 90, queuePlaybackProgress: 45, queuePlaybackDuration: 150, queueCurrentTrackNumber: 2, queueTotalTrackCount: 3)
let appMediaData = SDLAppServiceData(mediaServiceData: mediaData, serviceId: serviceId)

let onAppData = SDLOnAppServiceData(serviceData: appMediaData)
sdlManager.sendRPC(onAppData)
```
!@

@![android,javaSE,javaEE]
```java
MediaServiceData mediaData = new MediaServiceData();
mediaData.setMediaTitle("Some media title");
mediaData.setMediaArtist("Some media artist");
mediaData.setMediaAlbum("Some album");
mediaData.setMediaImage(new Image("Some image", ImageType.DYNAMIC));
mediaData.setPlaylistName("Some playlist");
mediaData.setIsExplicit(true);
mediaData.setTrackPlaybackProgress(45);
mediaData.setTrackPlaybackDuration(90);
mediaData.setQueuePlaybackProgress(45);
mediaData.setQueuePlaybackDuration(150);
mediaData.setQueueCurrentTrackNumber(2);
mediaData.setQueueTotalTrackCount(3);

AppServiceData appData = new AppServiceData();
appData.setServiceID(myServiceId);
appData.setServiceType(AppServiceType.MEDIA.toString());
appData.setMediaServiceData(mediaData);

OnAppServiceData onAppData = new OnAppServiceData();
onAppData.setServiceData(appData);

sdlManager.sendRPC(onAppData);
```
!@

#### Navigation Service Data
@![iOS]

##### Objective-C
```objc
UIImage *image = [UIImage imageNamed:imageName];
if (image == nil) { return; }

SDLArtwork *artwork = [SDLArtwork artworkWithImage:image name:imageName asImageFormat:SDLArtworkImageFormatJPG];
[self.sdlManager.fileManager uploadFile:artwork completionHandler:^(BOOL success, NSUInteger bytesAvailable, NSError * _Nullable error) {
    // Make sure the image is uploaded to the system before publishing your service
    SDLLocationCoordinate *coordinate = [[SDLLocationCoordinate alloc] initWithLatitudeDegrees:42 longitudeDegrees:43];
    SDLLocationDetails *location = [[SDLLocationDetails alloc] initWithCoordinate:coordinate];
    SDLNavigationInstruction *instruction = [[SDLNavigationInstruction alloc] initWithLocationDetails:location action:SDLNavigationActionTurn];
    instruction.image = [[SDLImage alloc] initWithName:imageName isTemplate:NO];

    SDLDateTime *timestamp = [[SDLDateTime alloc] initWithHour:2 minute:3 second:4 millisecond:847];
    SDLNavigationServiceData *navServiceData = [[SDLNavigationServiceData alloc] initWithTimestamp:timeStamp];
    navServiceData.instructions = @[instruction];

    SDLAppServiceData *appServiceData = [[SDLAppServiceData alloc] initWithNavigationServiceData:navServiceData serviceId:@"<#Your saved serviceID#>"];

    SDLOnAppServiceData *onAppServiceData = [[SDLOnAppServiceData alloc] initWithServiceData:appServiceData];
    [self.sdlManager sendRPC:onAppServiceData];
}];
```

##### Swift
```swift
guard let image = UIImage(named: imageName) else { return }
let artwork = SDLArtwork(image: image, name: imageName, persistent: false, as: .JPG)

sdlManager.fileManager.upload(file: artwork) { [weak self] (success, bytesAvailable, error) in
    guard success else { return }

    // Make sure the image is uploaded to the system before publishing your service
    let coordinate = SDLLocationCoordinate(latitudeDegrees: 42, longitudeDegrees: 43)
    let location = SDLLocationDetails(coordinate: coordinate)
    let instruction = SDLNavigationInstruction(locationDetails: location, action: .turn)
    instruction.image = SDLImage(name: imageName, isTemplate: false)

    let timestamp = SDLDateTime(hour: 2, minute: 3, second: 4, millisecond: 847)
    let navServiceData = SDLNavigationServiceData(timestamp: timestamp)
    navServiceData.instructions = [instruction]

    let appServiceData = SDLAppServiceData(navigationServiceData: navServiceData, serviceId: "<#Your saved serviceID#>")

    let onAppServiceData = SDLOnAppServiceData(serviceData: appServiceData)
    self?.sdlManager.sendRPC(onAppServiceData)
}
```
!@

@![android,javaSE,javaEE]
```java
final SdlArtwork navInstructionArt = new SdlArtwork("turn", FileType.GRAPHIC_PNG, R.drawable.turn, true);

// We have to send the image to the system before it's used in the app service.
sdlManager.getFileManager().uploadFile(navInstructionArt, new CompletionListener() {
    @Override
    public void onComplete(boolean success) {
        if (success){
            Coordinate coordinate = new Coordinate(42f,43f);

            LocationDetails locationDetails = new LocationDetails();
            locationDetails.setCoordinate(coordinate);

            NavigationInstruction navigationInstruction = new NavigationInstruction(locationDetails, NavigationAction.TURN);
            navigationInstruction.setImage(navInstructionArt.getImageRPC());

            DateTime dateTime = new DateTime();
            dateTime.setHour(2);
            dateTime.setMinute(3);
            dateTime.setSecond(4);

            NavigationServiceData navigationData = new NavigationServiceData(dateTime);
            navigationData.setInstructions(Collections.singletonList(navigationInstruction));

            AppServiceData appData = new AppServiceData();
            appData.setServiceID(myServiceId);
            appData.setServiceType(AppServiceType.NAVIGATION.toString());
            appData.setNavigationServiceData(navigationData);

            OnAppServiceData onAppData = new OnAppServiceData();
            onAppData.setServiceData(appData);

            sdlManager.sendRPC(onAppData);
        }
    }
});
```
!@

#### Weather Service Data
@![iOS]

##### Objective-C
```objc
UIImage *image = [UIImage imageNamed:imageName];
if (image == nil) { return; }

SDLArtwork *artwork = [SDLArtwork artworkWithImage:image name:imageName asImageFormat:SDLArtworkImageFormatJPG];

// We have to send the image to the system before it's used in the app service.
__weak typeof(self) weakself = self;
[self.sdlManager.fileManager uploadFile:artwork completionHandler:^(BOOL success, NSUInteger bytesAvailable, NSError * _Nullable error) {
    [weakSelf updateWeatherServiceWithImage:success];
}];

- (void)updateWeatherServiceWithImage:(BOOL)useImage {
    SDLWeatherData *weatherData = [[SDLWeatherData alloc] init];
    weatherData.weatherIconImageName = useImage ? imageName : nil;

    SDLWeatherServiceData *weatherServiceData = [[SDLWeatherServiceData alloc] initWithLocation:[[SDLLocationDetails alloc] initWithCoordinate:[[SDLLocationCoordinate alloc] initWithLatitudeDegrees:42.331427 longitudeDegrees:-83.0457538]]];

    SDLAppServiceData *appServiceData = [[SDLAppServiceData alloc] initWithWeatherServiceData:weatherServiceData serviceId:@"<#Your saved serviceID#>"];

    SDLOnAppServiceData *onAppServiceData = [[SDLOnAppServiceData alloc] initWithServiceData:appServiceData];
    [self.sdlManager sendRPC:onAppServiceData];
}
```

##### Swift
```swift
guard let image = UIImage(named: imageName) else { return }
let artwork = SDLArtwork(image: image, name: imageName, persistent: false, as: .JPG)

// We have to send the image to the system before it's used in the app service.
sdlManager.fileManager.upload(file: artwork) { [weak self] (success, bytesAvailable, error) in
    self?.updateWeatherService(shouldUseImage: success)
}

private func updateWeatherService(shouldUseImage: Bool) {
    let weatherData = SDLWeatherData()
    weatherData.weatherIconImageName = shouldUseImage ? imageName : nil

    let weatherServiceData = SDLWeatherServiceData(location: SDLLocationDetails(coordinate: SDLLocationCoordinate(latitudeDegrees: 42.3314, longitudeDegrees: 83.0458)), currentForecast: weatherData, minuteForecast: nil, hourlyForecast: nil, multidayForecast: nil, alerts: nil)

    let appServiceData = SDLAppServiceData(weatherServiceData: weatherServiceData, serviceId: "<#Your saved serviceID#>")

    let onAppServiceData = SDLOnAppServiceData(serviceData: appServiceData)
    sdlManager.sendRPC(onAppServiceData)
}
```
!@

@![android,javaSE,javaEE]
```java
final SdlArtwork weatherImage = new SdlArtwork("sun", FileType.GRAPHIC_PNG, R.drawable.sun, true);

// We have to send the image to the system before it's used in the app service.
sdlManager.getFileManager().uploadFile(weatherImage, new CompletionListener() { 
    @Override
    public void onComplete(boolean success) {
        if (success) {

            WeatherData weatherData = new WeatherData();
            weatherData.setWeatherIcon(weatherImage.getImageRPC());

            Coordinate coordinate = new Coordinate(42f, 43f);

            LocationDetails locationDetails = new LocationDetails();
            locationDetails.setCoordinate(coordinate);

            WeatherServiceData weatherServiceData = new WeatherServiceData(locationDetails);

            AppServiceData appData = new AppServiceData();
            appData.setServiceID(myServiceId);
            appData.setServiceType(AppServiceType.WEATHER.toString());
            appData.setWeatherServiceData(weatherServiceData);

            OnAppServiceData onAppData = new OnAppServiceData();
            onAppData.setServiceData(appData);

            sdlManager.sendRPC(onAppData);
        }
    }
});
```
!@

### 4. Handling App Service Subscribers
If you choose to make your app service available to other apps, you will have to handle requests to get your app service data when a consumer requests it directly.

Handling app service subscribers is a two step process. First, you must @![iOS]register for notifications from!@ @![android,javaSE,javaEE]setup listeners for!@ the subscriber. Then, when you get a request, you will either have to send a response to the subscriber with the app service data or if you have no data to send, send a response with a relevant failure result code.

#### Listening for Requests
First, you will need to @![iOS]register for `GetAppServiceDataRequest`s notifications.!@@![android,javaSE,javaEE]setup a listener for `GetAppServiceDataRequest`s!@. Then, when you get the request you need to respond with your app service data. This means that you will need to store your current service data after your most recent update using `OnAppServiceData` (see the section Updating Your Service Data).

@![iOS]
##### Objective-C
```objc
[self.sdlManager subscribeToRPC:SDLDidReceiveGetAppServiceDataRequest withBlock:^(__kindof SDLRPCMessage * _Nonnull message) {
    SDLGetAppServiceData *getAppServiceRequest = message;

    // Send a response
    SDLGetAppServiceDataResponse *response = [[SDLGetAppServiceDataResponse alloc] initWithAppServiceData:<#Your App Service Data#>];
    response.correlationID = getAppServiceRequest.correlationID;
    response.success = @YES;
    response.resultCode = SDLResultSuccess;
    response.info = @"<#Use to provide more information about an error#>";
    [self.sdlManager sendRPC:response];
}];
```

##### Swift
```swift
sdlManager.subscribe(to: SDLDidReceiveGetAppServiceDataRequest) { [weak self] (message) in
    guard let getAppServiceRequest = message as? SDLGetAppServiceData else { return }
    
    // Send a response
    let response = SDLGetAppServiceDataResponse(appServiceData: <#Your App Service Data#>)
    response.correlationID = getAppServiceRequest.correlationID
    response.success = true as NSNumber
    response.resultCode = .success
    response.info = "<#Use to provide more information about an error#>"
    self?.sdlManager.sendRPC(response)
}
```
!@

@![android,javaSE,javaEE]
```java
sdlManager.addOnRPCRequestListener(FunctionID.GET_APP_SERVICE_DATA, new OnRPCRequestListener() {
    @Override
    public void onRequest(RPCRequest request) {
        GetAppServiceData getAppServiceData = (GetAppServiceData) request;

         // Send a response
        GetAppServiceDataResponse response = new GetAppServiceDataResponse();
        response.setSuccess(true);
        response.setCorrelationID(getAppServiceData.getCorrelationID());
        response.setResultCode(Result.SUCCESS);
        response.setInfo("<#Use to provide more information about an error#>");
        response.setServiceData(<#Your App Service Data#>);
        sdlManager.sendRPC(response);
    }
});
```
!@

## Supporting Service RPCs and Actions

### 5. Service RPCs
Certain RPCs are related to certain services. The chart below shows the current relationships:

| MEDIA | NAVIGATION | WEATHER |
| ----- | ---------- | ------- |
| ButtonPress (OK) | SendLocation | |
| ButtonPress (SEEKLEFT) | GetWayPoints | |
| ButtonPress (SEEKRIGHT) | SubscribeWayPoints | |
| ButtonPress (TUNEUP) | OnWayPointChange | |
| ButtonPress (TUNEDOWN) | | |
| ButtonPress (SHUFFLE) | | |
| ButtonPress (REPEAT) | | |

When you are the active service for your service's type (e.g. media), and you have declared that you support these RPCs in your manifest (see section 1. Creating an App Service Manifest), then these RPCs will be automatically routed to your app. You will have to set up @![iOS]notifications!@ @![android,javaSE,javaEE]listeners!@ to be aware that they have arrived, and you will then need to respond to those requests.

@![iOS]
##### Objective-C
```objc
SDLAppServiceManifest *manifest = [[SDLAppServiceManifest alloc] init];
// Everything else for your manifest
NSNumber *buttonPressRPCID = [[SDLFunctionID sharedInstance] functionIdForName:SDLRPCFunctionNameButtonPress];
manifest.handledRPCs = @[buttonPressRPCID];

[self.sdlManager subscribeToRPC:SDLDidReceiveButtonPressRequest withObserver:self selector:@selector(buttonPressRequestReceived:)];

- (void)buttonPressRequestReceived:(SDLRPCRequestNotification *)request {
    SDLButtonPress *buttonPressRequest = (SDLButtonPress *)request.request;
    // Check the request for the button name and long / short press

    // Send a response
    SDLButtonPressResponse *response = [[SDLButtonPressResponse alloc] init];
    response.correlationID = buttonPressRequest.correlationID;
    response.success = @YES;
    response.resultCode = SDLVehicleDataResultCodeSuccess;
    response.info = @"<#Use to provide more information about an error#>";

    [self.sdlManager sendRPC:response];
}
```

##### Swift
```swift
let manifest = SDLAppServiceManifest()
// Everything else for your manifest
let buttonPressRPCID = SDLFunctionID.sharedInstance().functionId(forName: .buttonPress)
manifest.handledRPCs = [buttonPressRPCID]

sdlManager.subscribe(to: SDLDidReceiveButtonPressRequest, observer: self, selector: #selector(buttonPressRequestReceived(_:)))

@objc private func buttonPressRequestReceived(_ notification: SDLRPCRequestNotification) {
    guard let interactionRequest = notification.request as? SDLButtonPress else { return }

    // A result you want to send to the consumer app.
    let response = SDLButtonPressResponse()

    // These are very important, your response won't work properly without them.
    response.success = true
    response.resultCode = .success
    response.correlationID = interactionRequest.correlationID
    response.info = "<#Use to provide more information about an error#>"

    sdlManager.sendRPC(response)
}
```
!@

@![android,javaSE,javaEE]
```java
AppServiceManifest manifest = new AppServiceManifest(AppServiceType.MEDIA.toString());
...
manifest.setHandledRpcs(Collections.singletonList(FunctionID.BUTTON_PRESS.getId()));
```

```java
sdlManager.addOnRPCRequestListener(FunctionID.BUTTON_PRESS, new OnRPCRequestListener() {
    @Override
    public void onRequest(RPCRequest request) {
        ButtonPress buttonPress = (ButtonPress) request;

        ButtonPressResponse response = new ButtonPressResponse();
        response.setSuccess(true);
        response.setResultCode(Result.SUCCESS);
        response.setCorrelationID(buttonPress.getCorrelationID());
        response.setInfo("<#Use to provide more information about an error#>");
        sdlManager.sendRPC(response);
    }
});
```
!@

### 6. Service Actions
App actions are the ability for app consumers to use the SDL services system to send URIs to app providers in order to activate actions on the provider. Service actions are *schema-less*, i.e. there is no way to define the appropriate URIs through SDL. If you already provide actions through your app and want to expose them to SDL, or if you wish to start providing them, you will have to document your available actions elsewhere (such as your website).

@![iOS]
If you're wondering how to get started with actions and routing, this is a very common problem in iOS! Many apps support the [x-callback-URL](http://x-callback-url.com) format as a common inter-app communication method. There are also [many](https://github.com/devxoul/URLNavigator) [libraries](https://github.com/joeldev/JLRoutes) [available](https://github.com/skyline75489/SwiftRouter) for the purpose of supporting URL routing.
!@

In order to support actions through SDL services, you will need to observe and respond to the `PerformAppServiceInteraction` RPC request.

@![iOS]

##### Objective-C
```objc
// Subscribe to PerformAppServiceInteraction requests
[self.sdlManager subscribeToRPC:SDLDidReceivePerformAppServiceInteractionRequest withObserver:self selector:@selector(performAppServiceInteractionRequestReceived:)];

- (void)performAppServiceInteractionRequestReceived:(SDLRPCRequestNotification *)notification {
    SDLPerformAppServiceInteraction *interactionRequest = notification.request;

    // If you have multiple services, this will let you know which of your services is being addressed
    NSString *serviceID = interactionRequest.serviceID;

    // The app id of the service consumer app that sent you this message
    NSString *originAppId = interactionRequest.originApp;

    // The URL sent by the consumer. This must be something you understand, e.g. a URL scheme call. For example, if you were YouTube, it could be a URL to play a specific video. If you were a music app, it could be a URL to play a specific song, activate shuffle / repeat, etc.
    NSURLComponents *interactionURLComponents = [NSURLComponents componentsWithString:interactionRequest.serviceUri];

    // A result you want to send to the consumer app.
    NSString *result = @"Uh oh";
    SDLPerformAppServiceInteractionResponse *response = [[SDLPerformAppServiceInteractionResponse alloc] initWithServiceSpecificResult:result];

    // These are very important, your response won't work properly without them.
    response.success = @NO;
    response.resultCode = SDLResultGenericError;
    response.correlationID = interactionRequest.correlationID;

    [self.sdlManager sendRPC:response];
}
```

##### Swift
```swift
// Subscribe to PerformAppServiceInteraction requests
sdlManager.subscribe(to: SDLDidReceivePerformAppServiceInteractionRequest, observer: self, selector: #selector(performAppServiceInteractionRequestReceived(_:)))

@objc private func performAppServiceInteractionRequestReceived(_ notification: SDLRPCRequestNotification) {
    guard let interactionRequest = notification.request as? SDLPerformAppServiceInteraction else { return }

    // If you have multiple services, this will let you know which of your services is being addressed
    let serviceID = interactionRequest.serviceID

    // The app id of the service consumer app that sent you this message
    let originAppId = interactionRequest.originApp

    // The URL sent by the consumer. This must be something you understand, e.g. a URL scheme call. For example, if you were YouTube, it could be a URL to play a specific video. If you were a music app, it could be a URL to play a specific song, activate shuffle / repeat, etc.
    let interactionURLComponents = URLComponents(string: interactionRequest.serviceUri)

    // A result you want to send to the consumer app.
    let result = "Uh oh"
    let response = SDLPerformAppServiceInteractionResponse(serviceSpecificResult: result)

    // These are very important, your response won't work properly without them.
    response.success = true
    response.resultCode = .success
    response.correlationID = interactionRequest.correlationID

    sdlManager.sendRPC(response)
}
```
!@

@![android,javaSE,javaEE]
```java
// Perform App Services Interaction Request Listener
sdlManager.addOnRPCRequestListener(FunctionID.PERFORM_APP_SERVICES_INTERACTION, new OnRPCRequestListener() {
    @Override
    public void onRequest(RPCRequest request) {
        PerformAppServiceInteraction performAppServiceInteraction = (PerformAppServiceInteraction) request;

        // If you have multiple services, this will let you know which of your services is being addressed
        serviceID = performAppServiceInteraction.getServiceID();

        // The URI sent by the consumer. This must be something you understand
        String serviceURI = performAppServiceInteraction.getServiceUri();

        // A result you want to send to the consumer app.
        PerformAppServiceInteractionResponse response = new PerformAppServiceInteractionResponse();
        response.setServiceSpecificResult("Some Result");
        response.setCorrelationID(performAppServiceInteraction.getCorrelationID());
        response.setInfo("<#Use to provide more information about an error#>");
        response.setSuccess(true);
        response.setResultCode(Result.SUCCESS);
        sdlManager.sendRPC(response);
    }
});
```
!@

## Updating Your Published App Service
Once you have published your app service, you may decide to update its data. For example, if you have a free and paid tier with different amounts of data, you may need to upgrade or downgrade a user between these tiers and provide new data in your app service manifest. If desired, you can also delete your app service by unpublishing the service. 

### 7. Updating a Published App Service Manifest (RPC v6.0+)
@![iOS]
##### Objective-C
```objc
SDLAppServiceManifest *manifest = [[SDLAppServiceManifest alloc] initWithAppServiceType:SDLAppServiceTypeWeather];
manifest.weatherServiceManifest = <#Updated weather service manifest#>

SDLPublishAppService *publishServiceRequest = [[SDLPublishAppService alloc] initWithAppServiceManifest:manifest];
[self.sdlManager sendRequest:publishServiceRequest];
```

##### Swift
```swift
let manifest = SDLAppServiceManifest(appServiceType: .weather)
manifest.weatherServiceManifest = <#Updated weather service manifest#>

let publishServiceRequest = SDLPublishAppService(appServiceManifest: manifest)
sdlManager.send(publishServiceRequest)
```
!@

@![android,javaSE,javaEE]
```java
AppServiceManifest manifest = new AppServiceManifest(AppServiceType.WEATHER.toString());
manifest.setWeatherServiceManifest("<#Updated weather service manifest>");

PublishAppService publishServiceRequest = new PublishAppService(manifest);
sdlManager.sendRPC(publishServiceRequest);
```
!@

### 8. Unpublishing a Published App Service Manifest (RPC v6.0+)
@![iOS]
##### Objective-C
```objc
SDLUnpublishAppService *unpublishAppService = [[SDLUnpublishAppService alloc] initWithServiceID:@"<#The serviceID of the service to unpublish#>"];
[self.sdlManager sendRequest:unpublishAppService];
```

##### Swift
```swift
let unpublishAppService = SDLUnpublishAppService(serviceID: "<#The serviceID of the service to unpublish#>")
sdlManager.send(unpublishAppService)
```
!@

@![android,javaSE,javaEE]
```java
UnpublishAppService unpublishAppService = new UnpublishAppService("<#The serviceID of the service to unpublish>");
sdlManager.sendRPC(unpublishAppService);
```
!@

