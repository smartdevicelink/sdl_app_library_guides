# Creating an OEM Cloud App Store (RPC v5.1+)
SDL allows OEMs to offer an app store that lets users browse and install remote cloud apps. If the cloud app requires users to login with their credentials, the app store can use an authentication token to automatically login users after their first session.

!!! note
An OEM app store can be a mobile app or a cloud app.
!!!

## User Authentication
App stores can handle user authentication for the installed cloud apps. For example, users can log in after installing a cloud app using the app store. After that, the app store will save an authentication token for the cloud app in the local policy table. Then, the cloud app can retrieve the authentication token from the local policy table and use it to authenticate a user with the application. If desired, an optional parameter, `CloudAppVehicleID`, can be used to identify the head unit.

## Setting and Getting Cloud App Properties
An OEM's app store can manage the properties of a specific cloud app by setting and getting its `CloudAppProperties`. This table summarizes the properties that are included in `CloudAppProperties`:

| Parameter Name  |  Description |
| ------------- | ------------- |
| appID | appID for the cloud app |
| nicknames | List of possible names for the cloud app. The cloud app will not be allowed to connect if its name is not contained in this list |
| enabled | If true, cloud app will be displayed on HMI |
| authToken | Used to authenticate the user, if the app requires user authentication |
| cloudTransportType | Specifies the connection type Core should use. Currently Core supports WS and WSS , but an OEM can implement their own transport adapter to handle different values |
| hybridAppPreference | Specifies the user preference to use the cloud app version, mobile app version, or whichever connects first when both are available |
| endpoint | Remote endpoint for websocket connections |

!!! note
Only trusted app stores are allowed to set or get `CloudAppProperties` for other cloud apps.
!!!

### Setting Cloud App Properties
App stores can set properties for a cloud app by sending a `SetCloudAppProperties` request to Core to store them in the local policy table. For example, in this piece of code, the app store can set the `authToken` to associate a user with a cloud app after the user logs in to the app by using the app store:

@![iOS]
##### Objective-C
```objc
SDLCloudAppProperties *properties = [[SDLCloudAppProperties alloc] initWithAppID:<#app id#>];
properties.authToken = <#auth token#>;
SDLSetCloudAppProperties *setCloud = [[SDLSetCloudAppProperties alloc] initWithProperties:properties];
[self.sdlManager sendRequest:setCloud withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if (!response.success.boolValue) {
        SDLLogE(@"Error sending set cloud properties: Req %@, Res %@, err %@", request, response, error);
        return;
    }

    SDLSetCloudAppPropertiesResponse *setCloudResponse = (SDLSetCloudAppPropertiesResponse *)response;

    <#Use the response#>
}];
```

##### Swift
```swift
let properties = SDLCloudAppProperties(appID: <#app id#>)
properties.authToken = <#auth token#>
let setCloud = SDLSetCloudAppProperties(properties: properties)
sdlManager.send(request: setCloud) { (req, res, err) in
    guard let response = res as? SDLSetCloudAppPropertiesResponse, response.success.boolValue == true, err == nil else {
        return
    }

    <#Use the response#>
}
```
!@

@![android,javaSE,javaEE]
```java
CloudAppProperties cloudAppProperties = new CloudAppProperties("<appId>");
cloudAppProperties.setAuthToken("<auth token>");
SetCloudAppProperties setCloudAppProperties = new SetCloudAppProperties(cloudAppProperties);
setCloudAppProperties.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse response) {
        if (response.getSuccess()) {
            Log.i("SdlService", "Request was successful.");
        } else {
            Log.i("SdlService", "Request was rejected.");
        }
    }
});
sdlManager.sendRPC(setCloudAppProperties);
```
!@

@![javascript]
```js
const cloudAppProperties = new SDL.rpc.structs.CloudAppProperties()
    .setAppID("<appId>")
    .setAuthToken("<auth token>");

const setCloudAppProperties = new SDL.rpc.messages.SetCloudAppProperties()
    .setProperties(cloudAppProperties);

// sdl_javascript_suite v1.1+
const response = await sdlManager.sendRpcResolve(setCloudAppProperties);
if (response.getSuccess()) {
    console.log("Request was successful.");
} else {
    console.log("Request was rejected.");
}
// thrown exceptions should be caught by a parent function via .catch()

// Pre sdl_javascript_suite v1.1
const response = await sdlManager.sendRpc(setCloudAppProperties).catch(error => error);
if (response.getSuccess()) {
    console.log("Request was successful.");
} else {
    console.log("Request was rejected.");
}
```
!@

### Getting Cloud App Properties
To retrieve cloud properties for a specific cloud app from local policy table, app stores can send `GetCloudAppProperties` and specify the `appId` for that cloud app as in this example:

@![iOS]
##### Objective-C
```objc
SDLGetCloudAppProperties *getCloud = [[SDLGetCloudAppProperties alloc] initWithAppID:<#app id#>];
[self.sdlManager sendRequest:getCloud withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if (!response || !response.success.boolValue) {
        SDLLogE(@"Error sending set cloud properties: Req %@, Res %@, err %@", request, response, error);
        return;
    }

    SDLGetCloudAppPropertiesResponse *setCloudResponse = (SDLGetCloudAppPropertiesResponse *)response;
    <#Use the response#>
}];
```

##### Swift
```swift
let getCloud = SDLGetCloudAppProperties(appID: <#app id#>)
sdlManager.send(request: getCloud) { (req, res, err) in
    guard let response = res as? SDLGetCloudAppPropertiesResponse, response.success.boolValue == true, err == nil else {
        return
    }

    <#Use the response#>
}
```
!@

@![android,javaSE,javaEE]
```java
GetCloudAppProperties getCloudAppProperties = new GetCloudAppProperties("<appId>");
getCloudAppProperties.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse response) {
        if (response.getSuccess()) {
            Log.i("SdlService", "Request was successful.");
            GetCloudAppPropertiesResponse getCloudAppPropertiesResponse = (GetCloudAppPropertiesResponse) response;
            CloudAppProperties cloudAppProperties = getCloudAppPropertiesResponse.getCloudAppProperties();
            // Use cloudAppProperties
        } else {
            Log.i("SdlService", "Request was rejected.");
        }
    }
});
sdlManager.sendRPC(getCloudAppProperties);
```
!@

@![javascript]
```js
// sdl_javascript_suite v1.1+
const getCloudAppProperties = new SDL.rpc.message.GetCloudAppProperties()
    .setAppID("<appId>");

const response = await sdlManager.sendRpcResolve(getCloudAppProperties);
if (response.getSuccess()) {
    console.log("Request was successful.");
} else {
    console.log("Request was rejected.");
}
// thrown exceptions should be caught by a parent function via .catch()

// Pre sdl_javascript_suite v1.1
const getCloudAppProperties = new SDL.rpc.message.GetCloudAppProperties()
    .setAppID("<appId>");

const response = await sdlManager.sendRpc(getCloudAppProperties).catch(error => error);
if (response.getSuccess()) {
    console.log("Request was successful.");
} else {
    console.log("Request was rejected.");
}
```
!@

#### Getting the Cloud App Icon
Cloud app developers don't need to add any code to download the app icon. The cloud app icon will be automatically downloaded from the url provided by the policy table and sent to Core to be later displayed on the HMI.

## Getting the Authentication Token
When users install cloud apps from an OEM's app store, they may be asked to login to that cloud app using the app store. After logging in, app store can save the `authToken` in the local policy table to be used later by the cloud app for user authentication.
A cloud app can retrieve its `authToken` from local policy table after starting the RPC service. The `authToken` can be used later by the app to authenticate the user:

@![iOS]
##### Objective-C
```objc
NSString *authToken = self.sdlManager.authToken;
```

##### Swift
```swift
let authToken = sdlManager.authToken
```
!@

@![android,javaSE,javaEE]
```java
String authToken = sdlManager.getAuthToken();
```
!@

@![javascript]
```js
const authToken = sdlManager.getAuthToken();
```
!@

## Getting CloudAppVehicleID (Optional)
The `CloudAppVehicleID` is an optional parameter used by cloud apps to identify a head unit. The content of `CloudAppVehicleID` is up to the OEM's implementation. Possible values could be the VIN or a hashed VIN.
The `CloudAppVehicleID` value can be retrieved as part of the `GetVehicleData` RPC.  To find out more about how to retrieve `CloudAppVehicleID`, check out the [Retrieving Vehicle Data](Other SDL Features/Retrieving Vehicle Data) section.
