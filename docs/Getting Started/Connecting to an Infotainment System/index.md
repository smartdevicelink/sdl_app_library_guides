# Connecting to an Infotainment System
@![iOS, android]
In order to view your SDL app, you must connect your device to a head unit that supports SDL Core. If you do not have access to a head unit, we recommend using a web-based emulator called [Manticore](https://smartdevicelink.com/sign-in/?next=/resources/manticore/) for testing how your SDL app reacts to real-world vehicle events, on-screen interactions and voice recognition. 

You will have to manually configure a different type of connection based on whether you are connecting to a head unit or an emulator. When connecting to a head unit, you must configure @![iOS]an `iAP`!@@![android]a `Multiplex`!@ connection. Likewise, when connecting to an emulator, a `TCP` connection must to be configured. 

## Connecting to an Emulator
To connect to an emulator such as [Manticore](https://smartdevicelink.com/sign-in/?next=/resources/manticore/) or a local Ubuntu [SDL Core](https://github.com/smartdevicelink/sdl_core)-based emulator you must implement a TCP connection when configuring your SDL app. 


### Getting the IP Address and Port
#### Ubuntu SDL Core
To connect to a virtual machine running the Ubuntu [SDL Core](https://github.com/smartdevicelink/sdl_core)-based emulator, you will use the IP address of the Ubuntu OS and `12345` for the port. You may have to enable port forwarding on your virtual machine you want to connect using a real device instead of a simulated device. 

#### Manticore
Once you launch an instance of Manticore, you will be given an IP address and port number that you can use to configure your TCP connection. 

@![iOS]
##### Objective-C
```objc
SDLLifecycleConfiguration *lifecycleConfiguration = [SDLLifecycleConfiguration debugConfigurationWithAppName:@"<#App Name#>" fullAppId:@"<#App Id#>" ipAddress:@"<#IP Address#>" port:<#Port#>];
```
##### Swift
```swift
let lifecycleConfiguration = SDLLifecycleConfiguration(appName: "<#App Name#>", fullAppId: "<#App Id#>", ipAddress: "<#IP Address#>", port: <#Port#>)
```
!@

@![android]
```java
// Set the SdlManager.Builder transport
builder.setTransportType(new TCPTransportConfig(<IP ADDRESS>, <PORT>, false));
```
!@

#### Troubleshooting 
If you are having issues with connecting to an emulator, please see our [troubleshooting tips](Getting Started/Example Apps) in the Example Apps section. 

## Connecting to a Head Unit
### Production
To connect your iOS device directly to a production vehicle head unit or Test Development Kit (TDK), make sure to implement an @![iOS]an `iAP`!@@![android]a `Multiplex`!@ connection. Then connect the device to the head unit or TDK using a USB cord or Bluetooth if the head unit supports it.

@![iOS]
##### Objective-C
```objc
SDLLifecycleConfiguration* lifecycleConfiguration = [SDLLifecycleConfiguration defaultConfigurationWithAppName:@"<#App Name#>" fullAppId:@"<#App Id#>"];
```

##### Swift
```swift
let lifecycleConfiguration = SDLLifecycleConfiguration(appName:"<#App Name#>", fullAppId: "<#App Id#>")
```
!@

@![android]
```java
// Set the SdlManager.Builder transport
builder.setTransportType(new MultiplexTransportConfig(context, <APP ID>));
```
!@

@![iOS]
### Viewing Realtime Logs
If you are testing with a vehicle head unit or TDK and wish to see realtime debug logs in the Xcode console, you should use [wireless debugging](https://developer.apple.com/videos/play/wwdc2017/404/).
!@ 

## Running the SDL App
Build and run the project in @![iOS]Xcode!@@![android]Android Studio!@, targeting the device or simulator that you want to test your app with. Your app should compile and launch on your device of choosing. Soon after, you should see your SDL app icon appear on the TDK or emulator:

![HMI Apps](assets/hmi1.png)

Click on the SDL icon in the HMI.

![HMI Apps](assets/hmi2.png)

This is the main screen of your SDL app. If you get to this point, the project is working.
!@

@![javaSE,javaEE]
## Getting Started
We assume that you have [SDL Core](https://github.com/smartdevicelink/sdl_core) (We recommend Ubuntu 18.04) and an [HMI](https://github.com/smartdevicelink/generic_hmi) set up prior to this point. Most people getting started with this tutorial will be using SDL Core and our Generic HMI. If you don't want to set up a virtual machine for testing, we offer [Manticore](https://smartdevicelink.com/resources/manticore/), which is a free service that allows you to test your apps in the cloud.

!!! NOTE
SDL Core and an HMI or Manticore are needed to run the cloud app to ensure that it connects.
!!!

## Configuring Core
To let SDL Core connect to your app, first you will have to know the IP address of the machine that is running the cloud app. If you don't know what it is, running `ifconfig` in the terminal will usually let you see it for the interface you are connected with to your network.

After getting the IP address, you will have to set App ID, App Websocket Endpoint, and App Nicknames in SDL Core to let it know where your instance of cloud app is running.

!!! NOTE
The App Websocket Endpoint contains the IP Address and port as the following: `ws://<ip address>:<port>/`.
!!!

### Manticore
If you are using Manticore, the app information can be easily set in the settings tab:

![Main Screen](assets/manticore1.png)

!!! NOTE
Manticore needs to access you machine's IP address to be able to start a websocket connection with your app. If you are hosting the app on your local machine, you may need to do extra setup to make your machine publicly accessible. The other solution is to setup Core and HMI on your machine instead of using Manticore so Core can access your local IP address.
!!!

### SDL Core and Generic HMI
If you are using SDL Core and Generic HMI, you will have to add a policy table entry as the following one to the existing entries:

```JSON
 "8678309": {
     "keep_context": false,
     "steal_focus": false,
     "priority": "NONE",
     "default_hmi": "NONE",
     "groups": ["Base-4"],
     "RequestType": [],
     "RequestSubType": [],
     "hybrid_app_preference": "CLOUD",
     "endpoint": "ws://<ip address>:<port>",
     "enabled": true,
     "auth_token": "",
     "cloud_transport_type": "WS",
     "nicknames": ["<app name>"]
 }
```

For more information about policy tables please visit the [Policy Table](https://smartdevicelink.com/en/guides/sdl-server/api-reference-documentation/policy-table/overview) guide.

!!! NOTE
Don't forget to replace `ws://<ip address>:<port>` with your own IP address and app port. Also `<app name>` should be replaced with the actual app name.
!!!

Following this, you should see an application appear on HMI as in the following screenshot:

![HMI Apps](assets/hmi1.png)

!!! NOTE
Even though you the app appears on the HMI, you still cannot lunch the app at this point. You will have to run the app from the IntelliJ IDEA first as described next.
!!!

## Running the App
After setting the app information in SDL Core, you can run the project in IntelliJ IDEA. The cloud should compile and launch on your your machine. After that, you can click on the app icon in the HMI.

![HMI Apps](assets/hmi2.png)

This is the main screen of the  app. If you get to this point, the project is working.
!@