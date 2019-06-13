## Hello SDL Android

### Introduction

In this guide we take you through the steps to get our sample project, Hello Sdl Android, running and connected to Sdl Core as well as showing up on the generic HMI.

First, make sure you download or clone the latest release from [GitHub](https://github.com/smartdevicelink/sdl_android). It is a package within the SDL Android library.

Open the project in [Android Studio](https://developer.android.com/studio/index.html). We will exclusively use Android Studio as it is the current supported platform for Android development.

### Getting Started

If you are not using a Ford TDK for development, we will assume that you have [SDL Core](https://github.com/smartdevicelink/sdl_core) (We recommend Ubuntu 16.04) and an [HMI](https://github.com/smartdevicelink/generic_hmi) set up prior to this point. Most people getting started with this tutorial will not have a Ford TDK, so sample outputs will be using Sdl Core and our Generic HMI.

If you don't want to set up a virtual machine for testing, we offer [Manticore](https://smartdevicelink.com/resources/manticore/), which is a free service that allows you to test your apps via TCP/IP in the cloud.

!!! NOTE
Sdl Core and an HMI or Manticore are needed to run Hello Sdl Android and to ensure that it connects
!!!

#### Build Flavors

Hello Sdl Android has been built with different **build flavors**.

To access the Build Variant menu to choose your flavor, click on the menu ```Build``` then ```Select Build Variant```. A small window will appear on the bottom left of your IDE window that allows you to choose a flavor.

There are many flavors to choose from and for now we will only be concerned with the debug versions.

Versions Include:

* `multi` - Multiplexing (Bluetooth, USB, TCP (as secondary transport))
* `multi_high_bandwidth` - Multiplexing for apps that require a high bandwidth transport
* `tcp` - Transmission Control Protocol - used only for debugging purposes

We will mainly be dealing with `multi` (if using a TDK) or `tcp` (if connecting to SDL Core via a virtual machine or your localhost, or to Manticore)

### Transports

#### Configure for TCP

If you aren't using a TDK or head unit, you can connect to SDL core via a virtual machine or to your localhost. To do this we will use the flavor ```tcpDebug```.

For TCP to work, you will have to know the IP address of your machine that is running Sdl Core. If you don't know what it is, running ```ifconfig``` in a linux terminal will usually let you see it for the interface you are connected with to your network. We have to modify the IP address in Hello Sdl Android to let it know where your instance of Sdl Core is running.

In the main Java folder of Hello Sdl Android, open up ```SdlService.java```

In the top of this file, locate the variable declaration for ```DEV_MACHINE_IP_ADDRESS```. Change it to your Sdl Core's IP. Leave the ```TCP_PORT``` set to ```12345```.

```java
	// TCP/IP transport config
	private static final int TCP_PORT = 12345; // if using manticore, change to assigned port
	private static final String DEV_MACHINE_IP_ADDRESS = "192.168.1.78"; // change to your IP
```

!!! NOTE
if you do not change the target IP address, the application will not connect to Sdl Core or show up on the HMI
!!!

#### Configure for Bluetooth

Right out of the box, all you need to do to run bluetooth is to select the ```multi_sec_offDebug``` (Multiplexing) build flavor.

#### Configure for USB (AOA)

To connect to an SDL Core instance or TDK via USB transport, select the ```multi_sec_offDebug ``` (Multiplexing) build flavor. There is more information for USB transport under [Getting Started - Using AOA Protocol](Getting Started/Using AOA Protocol).

### Building the Project

For TCP, you may use the built-in Android emulator or an Android phone on the same network as Sdl Core. For Bluetooth, you will need an Android phone that is paired to a TDK or head unit via Bluetooth.

!!! MUST
Make sure Sdl Core and the HMI are running prior to running Hello Sdl Android
!!!

Run the project in Android Studio, targeting the device you want Hello Sdl Android installed on.

Hello Sdl Android should compile and launch on your device of choosing:

![Main Screen](assets/mainscreen.png)

Following this, you should see an application appear on the TDK or HMI. In the case of the Generic HMI (using TCP), you will see the following:

![HMI Apps](assets/hmi1.png)

Click on the Hello Sdl icon in the HMI.

![HMI Apps](assets/hmi2.png)

This is the main screen of the Hello Sdl App. If you get to this point, the project is working.

On the device you are running the app on, a lock screen should now appear once the app is opened on the HMI if distracted driver notifications are set to `DD_On`:

![Lockscreen](assets/lockscreen.png)

!!! NOTE
Lock Screens are an important part of Sdl enabled applications. The goal is to keep the driver's eyes forward and off of the device
!!!

At this point Hello Sdl Android has been compiled and is running properly! Continue reading through our guides to learn about all of the
RPCs (Remote Procedure Calls) that can be made with the library.

### Troubleshooting

Sometimes things don't always go as planned, and so this section exists. If your app compiles and does NOT show up on the HMI, there are a few things to check out.

#### TCP

1. Make sure that you have changed the IP in ```SdlService.java``` to match the machine running Sdl Core. Being on the same network is also important.
2. If you are sure that the IP is correct and it is still not showing up, make sure the Build Flavor that is running is ```tcpDebug```.
3. If the two above dont work, make sure there is no firewall blocking the incoming port ```12345``` on the machine or VM running SDL Core. In the same breath, make sure your firewall allows that outgoing port.
4. There are different network configurations needed for different virtualization software (virtualbox, vmware, etc). Make sure yours is set up correctly. Or use [Manticore](https://smartdevicelink.com/resources/manticore/).

#### Bluetooth

1. Make sure the build flavor ```multi_sec_offDebug ``` is selected.
2. Ensure your phone is properly paired with the TDK
3. Make sure Bluetooth is turned on - on Both the TDK and your phone
4. Make sure apps are enabled on the TDK (in settings)
