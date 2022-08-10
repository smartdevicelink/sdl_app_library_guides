# Example Apps
@![iOS]
SDL provides two example apps: one written in Objective-C and one in Swift. Both implement the same features.

The example apps are located in the [sdl_ios](https://github.com/smartdevicelink/sdl_ios) repository. To try them, you can download the repository and run the example app targets, or you can use `pod try SmartDeviceLink` with [CocoaPods](https://cocoapods.org) installed on your Mac.

!!! NOTE
If you download or clone the SDL repository in order to run the example apps, you must first obtain the BSON submodule. You can do so by running `git submodule init` and `git submodule update` in your terminal when in the main directory of the cloned repository.
!!!

The example apps implement soft buttons, template text and images, a main menu and submenu, vehicle data, popup menus, voice commands, and capturing in-car audio.
!@

@![android,javaSE,javaEE]
This guide takes you through the steps needed to get the sample project, _Hello Sdl_, connected a module.
!@

@![android]
To get the example app, download or clone the [sdl_java_suite](https://github.com/smartdevicelink/sdl_java_suite). The _Hello Sdl Android_ app is a package within the SDL Android library. Open the `sdl_java_suite/android` project using "Open an existing Android Studio project" in [Android Studio](https://developer.android.com/studio/index.html). We will use Android Studio throughout this guide as it is the official IDE for Android development.
!@

@![javascript]
The [JavaScript Suite repository on GitHub](https://github.com/smartdevicelink/sdl_javascript_suite/tree/master/examples) provides example apps for both the browser and for NodeJS. This includes a WebEngine app, a WebSocket client app, a WebSocket server app, and a TCP client app. The examples in the folders already come with their own SDL library build files. Check each example app's `readme.md` file for more information on how to run the respective app.
!@

## Additional Examples
For more examples go to the [Smart Device Link Examples](https://github.com/SmartDeviceLink-Examples) GitHub organization. Download or clone any of these projects.

@![iOS]
For iOS, the examples available include the [example weather app](https://github.com/SmartDeviceLink-Examples/example_weather_app_ios), and the [example navigation app](https://github.com/SmartDeviceLink-Examples/example_navigation_app_ios). The [example weather app](https://github.com/SmartDeviceLink-Examples/example_weather_app_ios) uses the OpenWeather API to implement a basic connected weather app with SDL UI. This example showcases changing screen template items for certain weather forecasts, displaying hourly and daily weather in popup menus, and showing weather alerts with SDL Alerts. Another example, the [example navigation app](https://github.com/SmartDeviceLink-Examples/example_navigation_app_ios) utilizes the MapBox API to create a basic navigation app. The example navigation app can be used as a reference for developers who want to create their own navigation app.
!@

@![android]
For android, the examples available include the [example weather app](https://github.com/SmartDeviceLink-Examples/example_weather_app_android), [example navigation app](https://github.com/SmartDeviceLink-Examples/example_navigation_app_android), and the [SDL Capabilities app](https://github.com/SmartDeviceLink-Examples/SDL-Capabilities-Android). The [example weather app](https://github.com/SmartDeviceLink-Examples/example_weather_app_android) uses the OpenWeather API to implement a basic connected weather app with SDL UI. This example showcases changing screen template items for certain weather forecasts, displaying hourly and daily weather in popup menus, and showing weather alerts with SDL Alerts. In addition, the [example navigation app](https://github.com/SmartDeviceLink-Examples/example_navigation_app_android) utlizes the MapBox API to create a basic navigation app. The example navigation app can be used as a reference for developers who want to create their own navigation app. In addition, the [SDL Capabilities app](https://github.com/SmartDeviceLink-Examples/SDL-Capabilities-Android) showcases display capabilities of SDL.
!@

!!! NOTE
Some examples require obtaining API tokens from third parties for data and services. For all of thse examples follow the setup instructions as outlined in their **README.md**.
!!!

@![iOS]
## Connecting to an Infotainment System
### Emulator
You can use a simulated or a real device to connect the example app to an emulator. To connect the example app to [Manticore](https://smartdevicelink.com/resources/manticore/) or another emulator, make sure you are on the `TCP Debug` tab of the example app. Then type in the IP address and port number and press the "Connect" button. The button will turn green when you are connected. Please check the [Connecting to an Infotainment System](Getting Started/Connecting to an Infotainment System) guide for more detailed instructions on how to get the emulator's IP address and port number.

### Head Unit
You need a real device to connect the example app to production or debug hardware. After building the running the app, make sure you are on the `iAP` tab of the example app and press "Connect". The button will turn green when you are connected.

If using the Bluetooth (BT) transport, make sure to first pair your phone to the hardware before attempting to connect your SDL app. If using the USB transport, you will need to connect your phone to the hardware using a USB cord. 

If the hardware supports both BT and USB transports, only one transport will be supported at once. If your phone is connected via BT and you then connect the phone to the head unit via a USB cord, the library will close the BT session and open a new session over USB. Likewise, when the USB cord is disconnected, the library will close the USB session and open session over BT.
!@

@![android]
### Build Flavors
_Hello Sdl Android_ has been built with different build flavors that allow you to quickly connect the app to an emulator or hardware. You can choose your flavor in the **Build Variant** menu. To open the menu, select **Build > Select Build Variant**. A small window will appear on the bottom left of your IDE that allows you to choose a flavor.

There are many flavors to choose from but for now we will only be concerned with the debug build variants:

* `multi` - Multiplexing - Bluetooth, USB, TCP (as secondary transport)
* `multi_high_bandwidth` - Multiplexing for apps that require a high bandwidth transport
* `tcp` - Transmission Control Protocol - Only used for debugging purposes

You will mainly be dealing with `multi` build variants if connecting to TDK, or `tcp` if connecting to Manticore or another emulator.

## Connecting to an Infotainment System
### Emulator
You can use a simulated or a real device to connect the example app to an emulator. To connect the example app to [Manticore](https://smartdevicelink.com/resources/manticore/) or another emulator, make sure you are using `tcpDebug` build flavor. You must update the IP address and port number in the _Hello Sdl Android_ project so it knows where your emulator is running. Please check the [Connecting to an Infotainment System](Getting Started/Connecting to an Infotainment System) guide for more detailed instructions on how to get the emulator's IP address and port number.

1\. In the main Java folder of _Hello Sdl Android_, open up `SdlService.java`.
2\. At the top of this file, locate the variable declaration for `DEV_MACHINE_IP_ADDRESS` and change it to your emulator's IP address. Set the `TCP_PORT` to your emulator's port number.

```java
private static final String DEV_MACHINE_IP_ADDRESS = "192.168.1.78"; // Update
private static final int TCP_PORT = 12345; // Update
```

3\. Make sure the emulator is running, then build and run the app on a real device or a simulated device. The SDL app should show up on the HMI.   

### Head Unit
You need a real device to connect the example app to production or debug hardware. To connect the example app via Bluetooth or USB, all you need to do to is select the `multi_sec_offDebug` build flavor and then run the app on an Android device. You can find more information about the USB transport in the [Using AOA Protocol](Getting Started/Using AOA Protocol) guide. 

If using the Bluetooth transport, make sure to first pair your Android phone to the hardware before attempting to connect your SDL app.
!@

@![iOS,android,javascript]
## Troubleshooting
If your app compiles and but does not show up on the HMI, there are a few things you should check:
!@

@![iOS]
### TCP Debug Transport
1. Make sure the correct IP address and port number is set in the `SDLLifecycleConfiguration`.
1. Make sure the device and the SDL Core emulator are on the same network.
1. If you are running an SDL Core emulator on a virtual machine, and you are using port forwarding to connect your device to the virtual machine, the IP address should be the IP address of your machine hosting the VM, not the IP address of the VM. The port number will be `12345`.
1. Make sure there is no firewall blocking the incoming port `12345` on the machine or VM running the SDL Core emulator. Also make sure your firewall allows that outgoing port.
1. Your SDL app will not work when the device app is in the background, because the OS will terminate background tasks after a short amount of time. This is not an issue with production IAP connections because Apple's External Accessory framework allows your app unlimited background time.
1. If you have a media SDL app, audio will not play on the emulator. Only production IAP connections are currently able to play audio because this happens over the standard Bluetooth / USB system audio channel.
1. You cannot connect to any of our open-source emulators using a USB cord or Bluetooth because Apple's [MFi Program](https://mfi.apple.com/en/faqs.html#qc1) is confidential and can not be used in open source projects.

### iAP Production Transport
1. Make sure to use the default `SDLLifecycleConfiguration`.
1. Make sure the [protocol](https://smartdevicelink.com/en/guides/iOS/getting-started/sdk-configuration/) strings have been added to the app.
1. Make sure you have enabled background [capabilities](https://smartdevicelink.com/en/guides/iOS/getting-started/sdk-configuration/) for your app.
1. If the head unit (emulators do not support IAP) does not support Bluetooth, an iAP connection requires a USB cord.

#### iAP Bluetooth Production Transport
1. Bluetooth transport support is automatic when you support the iAP production transport. It cannot be turned on or off separately.
1. Make sure the head unit supports Bluetooth transport for iPhones. Currently, only some head units support Bluetooth.
1. Make sure Bluetooth is turned on - both on the head unit hardware and your iPhone.
1. Ensure your iPhone is properly paired with the head unit. 
!@

@![android]
### TCP Debug Transport
1. Make sure that you have changed the IP in `SdlService.java` to match the machine running SDL Core. Being on the same network is also important.
2. If you are sure that the IP is correct and it is still not showing up, make sure the Build Flavor that is running is `tcpDebug`.
3. If the two above don't work, make sure there is no firewall blocking the incoming port `12345` on the machine or VM running SDL Core. Also, make sure your firewall allows that outgoing port.
4. There are different network configurations needed for different virtualization software (VirtualBox, VMware, etc). Make sure yours is set up correctly. Or use [Manticore](https://smartdevicelink.com/resources/manticore/).

### Bluetooth
1. Make sure the build flavor `multi_sec_offDebug` is selected.
2. Ensure your phone is properly paired with the TDK
3. Make sure Bluetooth is turned on - on both the TDK and your phone
4. Make sure apps are enabled on the TDK (in settings)
!@

@![javascript]
### TCP Debug Transport
1. Make sure that your `HOST` and `PORT` environment variables are set to match the machine running SDL Core.
1. Make sure there is no firewall blocking the incoming port `12345` on the machine or VM running SDL Core. Also, make sure your firewall allows that outgoing port.
1. There are different network configurations needed for different virtualization software (VirtualBox, VMware, etc). Make sure yours is set up correctly. Or use [Manticore](https://smartdevicelink.com/resources/manticore/).

### Websocket Transport
1. Make sure that the policy table of SDL Core has the correct app IDs and nicknames as well as `enabled=true`.
1. Make sure that the cloud endpoint and cloud transport type provided to SDL Core are correct and reachable by SDL core. There are different network configurations needed for different virtualization software (VirtualBox, VMware, etc). Make sure yours is set up correctly.

### WebEngine Transport
1. Make sure that the `manifest.js` has provided all [necessary fields](https://smartdevicelink.com/en/guides/core/developer-documentation/web-engine-app-support/#webengine-apps) and that the information is correct.
1. If you're unable to install your app from a cloud app store, make sure that the app has been compressed to a file archive that can be retrieved via the download URL you provided.
!@

@![javaSE]
First, make sure you download or clone the latest release from [GitHub](https://github.com/smartdevicelink/sdl_java_suite). It is a [project](https://github.com/smartdevicelink/sdl_java_suite/tree/master/javaSE/hello_sdl_java) within the SDL Java Suite root directory. Then, open the _Hello Sdl_ project in [IntelliJ IDEA](https://www.jetbrains.com/idea/).
!@

@![javaEE]
Make sure that you follow the steps in [Installation](Getting Started/Installation) and [Integration Basics](Getting Started/Integration Basics - Java) sections to create a new JavaEE SDL project before continuing this section.

!!! NOTE
The [Hello Sdl JavaEE](https://github.com/smartdevicelink/sdl_java_suite/tree/master/javaEE/hello_sdl_java_ee) has some code commented out and cannot be compiled. The project just includes samples for `SdlService` and `Main` classes that can be copied to the new JavaEE project that you create by following the steps in [Installation page](Getting Started/Installation).
!!!
!@

@![javaSE,javaEE]
## Connecting to an Infotainment System 
To connect the sample app to the infotainment system, please follow the instructions in the [Connecting to an Infotainment System](Getting Started/Connecting to an Infotainment System) guide.
!@

@![javascript]
## Connecting to an Infotainment System
### Connecting as a WebSocket Client
For vanilla JavaScript SDL apps, connecting as a WebSocket client requires a version of SDL Core that can accept incoming WebSocket connections (at least v6.1.0). If you are using Manticore to test your app, note that it currently does not support WebSocket connections.

A workaround to this limitation is to use a proxy program for your app to connect with which modifies the incoming WebSocket connection into a TCP connection. The proxy program then connects to SDL Core on the app's behalf and passes through your transport data. A Java program that does exactly this is available in the [repository's JavaScript example folder](https://github.com/smartdevicelink/sdl_javascript_suite/blob/master/examples/js/hello-sdl) called `proxy.jar`. Check the example app's `readme.md` for how to run it. Please check the [Connecting to an Infotainment System](Getting Started/Connecting to an Infotainment System) guide for more detailed instructions on how to get the emulator's IP address and port number.

This workaround for older versions of Core is also necessary for WebEngine apps.

### Connecting as a WebSocket Server
SDL Core acts as the WebSocket client in this case. The information about your app and how Core should connect to it must go into the policy table. Check the [Connecting to an Infotainment System](Getting Started/Connecting to an Infotainment System) guide's **Configuring the Connection** section for how to set up your policy table to point to your app

The following snippet is a truncated version of what is needed to set up the WebSocket server to accept and pass connections to the SDL library. This example uses the `ws` npm module for WebSocket connections. Refer to [the integration basics guide](Getting Started/Integration Basics - JS) for the full integration setup.

```js
const SDL = require('./SDL.min.js');
const WS = require('ws');
const PORT = 3000;

// create a WebSocket Server
const appWebSocketServer = new WS.Server({
    port: PORT,
});
console.log(`WebSocket Server listening on port ${PORT}`);

// Event listener for incoming WebSocket connections
appWebSocketServer.on('connection', (connection) => {
    ...
    /* truncated snippet to show only the transport configuration setup */
    /* each new connection corresponds to a new instance of your app */
    lifecycleConfig.setTransportConfig(
        new SDL.transport.WebSocketServerConfig(
            connection
        )
    );
    ...
});
```
!@
