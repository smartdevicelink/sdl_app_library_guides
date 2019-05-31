# Using App Services
App services is a powerful feature enabling both a new kind of vehicle-to-app communication and app-to-app communication via SDL.

App services are used to publish navigation, weather and media data (such as temperature, navigation waypoints, or the current playlist name). This data can then be used by both the vehicle head unit and, if the publisher of the app service desires, other SDL apps. Creating an app service is covered [in another guide](Other SDL Features/Creating an App Service).

Vehicle head units may use these services in various ways. One app service for each type will be the "active" service to the module. For media, for example, this will be the media app that the user is currently using or listening to. For navigation, it would be a navigation app that the user is using to navigate. For weather, it may be the last used weather app, or a user-selected default. The system may then use that service's data to perform various actions (such as navigating to an address with the active service or to display the temperature as provided from the active weather service).

An SDL app can also subscribe to a published app service. Once subscribed, the app will be sent the new data when the app service publisher updates its data. This guide will cover subscribing to a service. Subscribed apps can also send certain RPCs and generic URI-based actions (see the section Supporting App Actions, below) to your service.

Currently, there is no high-level API support for using an app service, so you will have to use raw RPCs for all app service related APIs.

## Getting and Subscribing to Services
Once your app has connected to the head unit, you will first want to be notified of all available services and updates to the metadata of all services on the head unit. Second, you will narrow down your app to subscribe to an individual app service and subscribe to its data. Third, you may want to interact with that service through RPCs, or fourth, through service actions.

### 1. Getting and Subscribing to Available Services
To get information on all services published on the system, as well as on changes to published services, you will use the `GetSystemCapability` request / response as well as the `OnSystemCapabilityUpdated` notification.

##### Objective-C
```objc
- (void)systemCapabilityDidUpdate:(SDLRPCNotificationNotification *)notification {
    SDLOnSystemCapabilityUpdated *updateNotification = notification.notification;
    SDLLogD(@"On System Capability updated: %@", updateNotification);

    <#Code#>
}

- (void)setupAppServicesCapability {
    [NSNotificationCenter.defaultCenter addObserver:self selector:@selector(systemCapabilityDidUpdate:) name:SDLDidReceiveSystemCapabilityUpdatedNotification object:nil];

    SDLGetSystemCapability *getAppServices = [[SDLGetSystemCapability alloc] initWithType:SDLSystemCapabilityTypeAppServices subscribe:YES];
    [self.sdlManager sendRequest:getAppServices withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
        if (!response || !response.success.boolValue) {
            SDLLogE(@"Error sending get system capability: Req %@, Res %@, err %@", request, response, error);
            return;
        }

        SDLAppServicesCapabilities *serviceRecord = response.systemCapability.appServicesCapabilities
        SDLLogD(@"Get system capability app service response: %@, serviceRecord %@", response, serviceRecord);
        
        <#Use the service record#>
    }];
}
```

##### Swift
```swift
@objc private func systemCapabilityDidUpdate(_ notification: SDLRPCNotificationNotification) {
    guard let capabilityNotification = notification.notification as? SDLOnSystemCapabilityUpdated else { return }

    SDLLog.d("OnSystemCapabilityUpdated: \(capabilityNotification)")
    <#Code#>
}

private func setupAppServicesCapability() {
    Notification.default.addObserver(self, selector: #selector(systemCapabilityDidUpdate(_:)), name: .SDLDidReceiveSystemCapabilityUpdated, object: nil)

    let getAppServices = SDLGetSystemCapability(type: .appServices, subscribe: true)
    sdlManager.send(request: getAppServices) { (req, res, err) in
        guard let response = res as? SDLGetSystemCapabilityResponse, let serviceRecord = response.systemCapability.appServicesCapabilities, response.success.boolValue == true, err == nil else { return }

        <#Use the service record#>
    }
}
```

#### Checking the App Service Capability
Once you've retrieved the initial list of app service capabilities (in the `GetSystemCapability` response), or an updated list of app service capabilities (from the `OnSystemCapabilityUpdated` notification), you may want to inspect the data to find what you are looking for. Below is example code with comments explaining what each part of the app service capability is used for.

##### Objective-C
```objc
// From GetSystemCapabilityResponse
SDLGetSystemCapabilityResponse *getResponse = <#From whereever you got it#>;
SDLAppServicesCapabilities *capabilities = getResponse.systemCapability.appServicesCapabilities;

// This array contains all currently available app services on the system
NSArray<SDLAppServiceCapability *> *appServices = capabilities.appServices;
SDLAppServiceCapability *aCapability = appServices.first;

// This will be nil since it's the first update
SDLServiceUpdateReason *capabilityReason = aCapability.updateReason;

// The app service record will give you access to a service's generated id, which can be used to address the service directly (see below), it's manifest, used to see what data it supports, whether or not the service is published (it always will be here), and whether or not the service is the active service for its service type (only one service can be active for each type)
SDLAppServiceRecord *serviceRecord = aCapability.updatedAppServiceRecord;

// From OnSystemCapabilityUpdated
SDLOnSystemCapabilityUpdated *serviceNotification = <#From whereever you got it#>;
SDLAppServicesCapabilities *capabilities = serviceNotification.systemCapability.appServicesCapabilities;

// This array contains all recently updated services
NSArray<SDLAppServiceCapability *> *appServices = capabilities.appServices;
SDLAppServiceCapability *aCapability = appServices.first;

// This won't be nil. It will tell you why a service is in the list of updates
SDLServiceUpdateReason *capabilityReason = aCapability.updateReason;

// The app service record will give you access to a service's generated id, which can be used to address the service directly (see below), it's manifest, used to see what data it supports, whether or not the service is published (if it's not, it was just removed and should not be addressed), and whether or not the service is the active service for its service type (only one service can be active for each type)
SDLAppServiceRecord *serviceRecord = aCapability.updatedAppServiceRecord;
```

##### Swift
```swift
// From GetSystemCapabilityResponse
let getResponse: SDLGetSystemCapabilityResponse = <#From whereever you got it#>;
let capabilities = getResponse.systemCapability.appServicesCapabilities;

// This array contains all currently available app services on the system
let appServices: [SDLAppServiceCapability] = capabilities.appServices
let aCapability = appServices.first;

// This will be nil since it's the first update
let capabilityReason = aCapability.updateReason;

// The app service record will give you access to a service's generated id, which can be used to address the service directly (see below), it's manifest, used to see what data it supports, whether or not the service is published (it always will be here), and whether or not the service is the active service for its service type (only one service can be active for each type)
let serviceRecord = aCapability.updatedAppServiceRecord;

// From OnSystemCapabilityUpdated
let serviceNotification: SDLOnSystemCapabilityUpdated = <#From whereever you got it#>;
let capabilities = serviceNotification.systemCapability.appServicesCapabilities;

// This array contains all recently updated services
let appServices: [SDLAppServiceCapability] = capabilities.appServices;
let aCapability = appServices.first;

// This won't be nil. It will tell you why a service is in the list of updates
let capabilityReason = aCapability.updateReason;

// The app service record will give you access to a service's generated id, which can be used to address the service directly (see below), it's manifest, used to see what data it supports, whether or not the service is published (if it's not, it was just removed and should not be addressed), and whether or not the service is the active service for its service type (only one service can be active for each type)
let serviceRecord = aCapability.updatedAppServiceRecord;
```

#### Using the System Capability Manager
In the current release, while the system capability manager supports getting the app services capability, it does not support automatically updating itself when the app services change. It must currently be polled through the `updateCapabilityType:completionHandler:` method. Therefore, until the `SystemCapabilityManager` is updated with support for auto-updating, we currently recommend using the raw `SDLGetSystemCapability` RPC and subscribing.

### 2. Getting and Subscribing to a Service Type's Data
Once you have information about all of the services available, you may want to view or subscribe to a service type's data. To do so, you will use the `GetAppServiceData` RPC.

Note that you will currently only be able to get data for the *active* service of the service type. You can attempt to make another service the active service by using the `PerformAppServiceInteraction` RPC, discussed below in "Sending an Action to a Service Provider."

##### Objective-C
```objc
// Get service data once
SDLGetAppServiceData *getServiceData = [[SDLGetAppServiceData alloc] initWithServiceType:SDLAppServiceTypeMedia];

// Subscribe to service data in perpetuity via `OnAppServiceData` notifications.
SDLGetAppServiceData *subscribeServiceData = [[SDLGetAppServiceData alloc] initAndSubscribeToServiceType:SDLAppServiceTypeMedia];

// Unsubscribe to service data previously subscribed
SDLGetAppServiceData *unsubscribeServiceData = [[SDLGetAppServiceData alloc] initWithServiceType:SDLAppServiceTypeMedia];
unsubscribeServiceData.subscribe = NO;

// Get the service's data
[self.sdlManager sendRequest:getServiceData withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if (!response || !response.success.boolValue) {
        SDLLogE(@"Error sending get system capability: Req %@, Res %@, err %@", request, response, error);
        return;
    }

    SDLGetAppServiceDataResponse *serviceResponse = (SDLGetAppServiceDataResponse *)response;
    SDLMediaServiceData *mediaData = serviceResponse.serviceData.mediaServiceData;
}];
```

##### Swift
```swift
// Get service data once
let getServiceData = SDLGetAppServiceData(serviceType: .media)

// Subscribe to service data in perpetuity via `OnAppServiceData` notifications.
let subscribeServiceData = SDLGetAppServiceData(andSubscribeToAppServiceType: .media)

// Unsubscribe to service data previously subscribed
let unsubscribeServiceData = SDLGetAppServiceData(serviceType: .media)
unsubscribeServiceData.subscribe = false

// Get the service's data
let getServiceData = SDLGetAppServiceData(serviceType: .media)
sdlManager.send(request: getServiceData) { (req, res, err) in
    guard let response = res as? SDLGetAppServiceDataResponse, response.success.boolValue == true, err == nil, let mediaData = response.serviceData.mediaData else { return }

    <#Use the mediaData#>
}
```

## Interacting with a Service Provider
Once you have a service's data, you may want to interact with a service provider by sending RPCs or actions.

### 3. Sending RPCs to a Service Provider
Only certain RPCs are available to be passed to the service provider based on their service type. See the [Creating an App Service](Other SDL Features/Creating an App Service) guide (under the "Supporting Service RPCs and Actions" section) for a chart detailing which RPCs work with which service types. The RPC can only be sent to the active service of a specific service type, not to any inactive service.

Sending an RPC works exactly the same as if you were sending the RPC to the head unit system. The head unit will simply route your RPC to the appropriate app automatically.

!!! NOTE
Your app may need special permissions to use the RPCs that route to app service providers.
!!!

##### Objective-C
```objc
SDLButtonPress *buttonPress = [[SDLButtonPress alloc] initWithButtonName:SDLButtonNameOk moduleType:SDLModuleTypeAudio];

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
let buttonPress = SDLButtonPress(buttonName: .ok, moduleType: .audio)
sdlManager.send(request: getServiceData) { (req, res, err) in
    guard let response = res as? SDLButtonPressResponse, response.success.boolValue == true, err == nil else { return }

    <#Use the response#>
}
```

### 4. Sending an Action to a Service Provider
Actions are generic URI-based strings sent to any app service (active or not). You can also use actions to request to the system that they make the service the active service for that service type. Service actions are *schema-less*, i.e. there is no way to define the appropriate URIs through SDL. The service provider must document their list of available actions elsewhere (such as their website).

##### Objective-C
```objc
SDLPerformAppServiceInteraction *performAction = [[SDLPerformAppServiceInteraction alloc] initWithServiceURI:@"sdlexample://x-callback-url/showText?x-source=MyApp&text=My%20Custom%20String" serviceID: <#Previously Retrived ServiceID#> originApp: <#Your App Id#> requestServiceActive: NO];
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
let performAction = SDLPerformAppServiceInteraction(serviceUri: "sdlexample://x-callback-url/showText?x-source=MyApp&text=My%20Custom%20String", serviceID: <#Previously Retrived ServiceID#>, originApp: <#Your App Id#>, requestServiceActive: false)
sdlManager.send(request: performAction) { (req, res, err) in
    guard let response = res as? SDLPerformAppServiceInteractionResponse else { return }

    <#Check the error and response#>
}
```