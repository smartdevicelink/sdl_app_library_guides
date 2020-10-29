# Integration Basics
### How SDL Works
SmartDeviceLink works by sending remote procedure calls (RPCs) back and forth between a smartphone application and the SDL Core. These RPCs allow you to build the user interface, detect button presses, play audio, and get vehicle data, among other things. You will use the SDL library to build your app on the SDL Core.

The type of app you can make will depend on the build library you select. If you are using the Node.js build, your app can run as a WebSocket server or as a TCP client. If you are using the vanilla JavaScript build (a minified JS file not tied to any specific build system or server structure), your app can run as a WebSocket client. This guide will cover topics that apply to both Node.js and vanilla JS library builds.

## Basic Configuration
In order to correctly connect to an SDL enabled head unit, developers need to create an `AppConfig` configuration object to pass into an instance of the `SdlManager` class and then start it. This configuration object requires a `LifecycleConfig` object which contains the majority of the settings you need to set in order for the app to function. You will want to use the following methods to get started: `setAppId`, `setAppName`, `setLanguageDesired`, `setAppTypes`, and `setTransportConfig`. These configuration objects support method chaining, which allow you to create the following example configuration object:

```js
const lifecycleConfig = new SDL.manager.LifecycleConfig()
    .setAppId('hello-js')
    .setAppName('Hello JS')
    .setLanguageDesired(SDL.rpc.enums.Language.EN_US)
    .setAppTypes([
        SDL.rpc.enums.AppHMIType.DEFAULT,
    ]);
```

!!! NOTE
For WebEngine apps, most of this configuration will happen in the `manifest.js` file and does not need to be duplicated here.
!!!

## Transport Configuration
The transport configuration will depend on the environment you're using. For example, you can make a TCP connection with Node.js, but not with vanilla JS. See below for example transport configurations.

##### Node.js TCP Transport
```js
lifecycleConfig.setTransportConfig(new SDL.transport.TcpClientConfig(HOST, PORT));
```

For a WebSocket server connection, the developer is expected to set up the server component, and pass in incoming client connections to the `WebSocketServerConfig`. The `ws` node module provides the necessary object to the config.
##### Node.js WebSocket Server
```js
const WS = require('ws');
const PORT = 3000;

// create a WebSocket Server
const appWebSocketServer = new WS.Server({
    port: PORT,
});
console.log(`WebSocket Server listening on port ${PORT}`);

appWebSocketServer.on('connection', (connection) => {
    // app setup goes here
    lifecycleConfig.setTransportConfig(
        new SDL.transport.WebSocketServerConfig(
            connection,
            CONNECTION_LOST_TIMEOUT // connection timeout in milliseconds (default is 60 seconds)
        )
    );
});

```
##### Vanilla JS WebSocket Client
```js
lifecycleConfig.setTransportConfig(new SDL.transport.WebSocketClientConfig(HOST, PORT));
```

## Additional Configuration Options
There are several additional basic configuration options to set up your app, like the app name and icon.

### App Icon
An app icon can be set in the `LifecycleConfig` to automatically upload and set the icon image. Note that although the implementation of retreiving files are different between the JS browser and Node.js environments, the developer can use the same API in both cases, and the SDL library will cover the implementation details for the developer depending on which build they are using.

```js
const filePath = './app_icon.png';
const file = new SDL.manager.file.filetypes.SdlFile()
    .setName('AppIcon')
    .setFilePath(filePath)
    .setType(SDL.rpc.enums.FileType.GRAPHIC_PNG)
    .setPersistent(true);

lifecycleConfig.setAppIcon(file);
```

In this case, the code snippet expects there to be an `app_icon.png` file present in the same directory for the app icon.

### Listening for RPC notifications and events
You can listen for specific events using the `LifecycleConfig`'s `setRpcNotificationListeners`. The following example shows how to listen for HMI Status notifications. Additional listeners can be added for specific RPCs by using their corresponding `FunctionID` in place of the `OnHMIStatus` in the following example.

```js
lifecycleConfig.setRpcNotificationListeners({
    [SDL.rpc.enums.FunctionID.OnHMIStatus]: (onHmiStatus) => {
        // HMI Level updates
        const hmiLevel = onHmiStatus.getHmiLevel();
        console.log("Current HMI Level: ", hmiLevel);
    }
});
```

It is recommended to use this method over the `SdlManager.addRpcListener` method for the `OnHMIStatus` RPC, or any RPC Notifications that your app cannot afford to miss during the initial connection.

## Setting Up the SDL Manager
After creating the `LifecycleConfig`, it can be set into the `AppConfig` and then passed into the `SdlManager`. The following snippet will set up the `SdlManager` and start it up. A listener is attached to the manager listener to let you know when there is a connection and the managers are ready.

```js
const appConfig = new SDL.manager.AppConfig()
    .setLifecycleConfig(lifecycleConfig);

const managerListener = new SDL.manager.SdlManagerListener()
    .setOnStart((sdlManager) => {
        // managers are ready
    })
    .setOnError((sdlManager, info) => {
        console.error('Error from SdlManagerListener: ', info);
    });

const sdlManager = new SDL.manager.SdlManager(appConfig, managerListener)
    .start();
```

## Configuring the WebEngine App HTML File
For WebEngine apps, there are slight modifications for integrating the library, such as importing the manifest file and passing it into the `LifecycleConfig.loadManifest` method. Additionally, since the data for the connection information is part of the URL query, the `WebSocketClientConfig` class requires no arguments, and the library will read the URL query values instead. The resulting `index.html` may look something like this as a result:

```html
<html>
    <head>
        <script src='./SDL.min.js'></script>
    </head>
    <body>
        <script type='module'>
            import sdl_manifest from './manifest.js';

            const lifecycleConfig = new SDL.manager.LifecycleConfig()
                .loadManifest(sdl_manifest)
                .setLanguageDesired(SDL.rpc.enums.Language.EN_US);

            lifecycleConfig.setTransportConfig(new SDL.transport.WebSocketClientConfig());

            lifecycleConfig.setRpcNotificationListeners({
                [SDL.rpc.enums.FunctionID.OnHMIStatus]: (onHmiStatus) => {
                    // HMI Level updates
                    const hmiLevel = onHmiStatus.getHmiLevel();
                    console.log("Current HMI Level: ", hmiLevel);
                }
            });

            const appConfig = new SDL.manager.AppConfig()
                .setLifecycleConfig(lifecycleConfig);

            const managerListener = new SDL.manager.SdlManagerListener()
                .setOnStart((sdlManager) => {
                    // managers are ready
                })
                .setOnError((sdlManager, info) => {
                    console.error('Error from SdlManagerListener: ', info);
                });

            const sdlManager = new SDL.manager.SdlManager(appConfig, managerListener)
                .start();
        </script>
    </body>
</html>
```