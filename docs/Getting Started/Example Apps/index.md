# Example Apps
@![iOS]
SDL provides two example apps: one written in Objective-C and one in Swift. Both implement the same features.

The example apps are located in the [sdl_ios](https://github.com/smartdevicelink/sdl_ios) repository. To try them, you can download the repository and run the example app targets, or you may use `pod try SmartDeviceLink` with [CocoaPods](https://cocoapods.org) installed on your Mac.

!!! NOTE
If you download or clone the SDL repository in order to run the example apps, you must first obtain the BSON submodule. You can do so by running `git submodule init` and `git submodule update` in your terminal when in the main directory of the cloned repository.
!!!

The example apps implement soft buttons, template text and images, a main menu and submenu, vehicle data, popup menus, voice commands, and capturing in-car audio.

## Connecting to Hardware
To connect the example app to [Manticore](https://smartdevicelink.com/resources/manticore/) or another emulator, make sure you are on the `TCP Debug` tab and type in the IP address and port, then press "Connect". The button will turn green when you are connected.

To connect the example app to production or debug hardware, make sure you are on the `iAP` tab and press "Connect". The button will turn green when you are connected.

## Troubleshooting
### TCP Debug Transport
1. Make sure the correct IP address and port number is set in the `SDLLifecycleConfiguration`.
1. Make sure the device and the SDL emulator are on the same network.
1. If you are running an SDL Core emulator on a virtual machine, and you are using port forwarding to connect your device to the virtual machine, the IP address should be the IP address of your machine hosting the VM, not the IP address of the VM. The port number will be `12345`.
1. Make sure there is no firewall blocking the incoming port `12345` on the machine or VM running the SDL Core emulator. Also make sure your firewall allows that outgoing port.
1. Your SDL app will not work when the device app is in the background, because the OS will terminate background tasks after a short amount of time. This is not an issue with production IAP connections because Apple's External Accessory framework allows your app unlimited background time.
1. If you have a media SDL app, audio will not play on the emulator. Only production IAP connections are currently able to play audio because this happens over the standard Bluetooth / USB system audio channel.
1. You cannot connect to any of our open-source emulators using a USB cord or Bluetooth because Apple's [MFi Program](https://mfi.apple.com/MFiWeb/getFAQ.action#4-6) is confidential and can not be used in open source projects.

### iAP Production Transport
1. Make sure to use the default `SDLLifecycleConfiguration`.
1. Make sure the [protocol](https://smartdevicelink.com/en/guides/iOS/getting-started/sdk-configuration/) strings have been added to the app.
1. Make sure you have enabled background [capabilities](https://smartdevicelink.com/en/guides/iOS/getting-started/sdk-configuration/) for your app.
1. If the head unit (emulators do not support IAP) does not support Bluetooth, an iAP connection requires a USB cord.

#### iAP Bluetooth Production Transport
1. Bluetooth transport support is automatic when you support the iAP production transport. It cannot be turned on or off separately.
1. Make sure the head unit supports Bluetooth transport for iPhones. Currently, only some head units support Bluetooth.
1. Make sure to use the default `SDLLifecycleConfiguration`.
1. Make sure Bluetooth is turned on - both on the head unit hardware and your iPhone.
1. Ensure your iPhone is properly paired with the head unit. 
!@

@![android, javaSE, javaEE]
This guide takes you through the steps needed to get the sample project, Hello Sdl, connected a module.
!@

@![android]
First, download or clone the latest release from [GitHub](https://github.com/smartdevicelink/sdl_java_suite). The Hello Sd Android app is a package within the SDL Android library.

Open the the `sdl_java_suite/android` project using "Open an existing Android Studio project" in [Android Studio](https://developer.android.com/studio/index.html). We will use Android Studio throughout this guide as it is the official IDE for Android development.

## Getting Started
If you are not using a production head unit for development, we recommend using [SDL Core](https://github.com/smartdevicelink/sdl_core) and [Generic HMI](https://github.com/smartdevicelink/generic_hmi) for testing. 

If you don't want to set up a virtual machine for testing, we recommend using the [Manticore web-based emulator](https://smartdevicelink.com/resources/manticore/) for testing how your SDL app reacts to real-world vehicle events, on-screen interactions and voice recognition.

### Build Flavors
Hello Sdl Android has been built with different **build flavors**.

To access the **Build Variant** menu to choose your flavor, click on the menu **Build** then **Select Build Variant`**. A small window will appear on the bottom left of your IDE that allows you to choose a flavor.

There are many flavors to choose from but for now we will only be concerned with the debug versions. Versions available include:

* `multi` - Multiplexing - Bluetooth, USB, TCP (as secondary transport)
* `multi_high_bandwidth` - Multiplexing for apps that require a high bandwidth transport
* `tcp` - Transmission Control Protocol - Used only for debugging purposes

You will mainly be dealing with `multi` (if using a TDK) or `tcp` (if connecting to SDL Core or Manticore)

## Transports

### Configure for TCP
If you aren't using a TDK or head unit, you can connect to SDL Core via a virtual machine or to your localhost. To do this we will use the flavor `tcpDebug`.

For TCP to work, you need to know the IP address of the machine running SDL Core. If needed, you can get the IP address by running `ifconfig` in the terminal of the machine. Then, you must update the IP address in Hello Sdl Android to let it know where your instance of SDL Core is running.

1. In the main Java folder of Hello Sdl Android, open up `SdlService.java`.
1. At the top of this file, locate the variable declaration for `DEV_MACHINE_IP_ADDRESS`. Change it to your SDL Core's IP. Set the `TCP_PORT` to `12345`.

    ```java
    private static final int TCP_PORT = 12345; // if using Manticore, change to assigned port
    private static final String DEV_MACHINE_IP_ADDRESS = "192.168.1.78"; // change to your IP
    ```

### Configure for Bluetooth
Right out of the box, all you need to do to run Bluetooth is to select the `multi_sec_offDebug` (Multiplexing) build flavor.

### Configure for USB (AOA)
To connect to a SDL Core instance or TDK via USB transport, select the `multi_sec_offDebug ` (Multiplexing) build flavor. You can find more information about the USB transport in the [Using AOA Protocol](Getting Started/Using AOA Protocol) section.

## Building the Project
For TCP, you may use the built-in Android emulator or an Android phone on the same network as SDL Core. For Bluetooth, you will need an Android phone that is paired to a TDK or head unit via Bluetooth.

!!! MUST
Make sure SDL Core and the HMI are running prior to running Hello Sdl Android
!!!

To find out more about how to connect the app to the infotainment system, please check the [Connecting to an Infotainment System](Getting Started/Connecting to an Infotainment System) guide.

### Troubleshooting
If your app compiles and but does not show up on the HMI, there are a few things you should check:

#### TCP
1. Make sure that you have changed the IP in `SdlService.java` to match the machine running SDL Core. Being on the same network is also important.
2. If you are sure that the IP is correct and it is still not showing up, make sure the Build Flavor that is running is `tcpDebug`.
3. If the two above don't work, make sure there is no firewall blocking the incoming port `12345` on the machine or VM running SDL Core. Also, make sure your firewall allows that outgoing port.
4. There are different network configurations needed for different virtualization software (virtualbox, vmware, etc). Make sure yours is set up correctly. Or use [Manticore](https://smartdevicelink.com/resources/manticore/).

#### Bluetooth
1. Make sure the build flavor `multi_sec_offDebug` is selected.
2. Ensure your phone is properly paired with the TDK
3. Make sure Bluetooth is turned on - on both the TDK and your phone
4. Make sure apps are enabled on the TDK (in settings)
!@

@![javaSE]
First, make sure you download or clone the latest release from [GitHub](https://github.com/smartdevicelink/sdl_java_suite). It is a [project](https://github.com/smartdevicelink/sdl_java_suite/tree/master/hello_sdl_java) within the SDL Java Suite root directory. Then, open the Hello Sdl Android project in [IntelliJ IDEA](https://www.jetbrains.com/idea/) and wait for it to finish loading.
!@

@![javaEE]
Make sure that you follow the steps in [Installation](Getting Started/Installation) and [Integration Basics](Getting Started/Integration Basics) sections to create a new JavaEE SDL project before continuing this section.

!!! NOTE
The [Hello Sdl JavaEE](https://github.com/smartdevicelink/sdl_java_suite/tree/master/hello_sdl_java_ee) has some code commented out and cannot be compiled. The project just includes samples for `SdlService` and `Main` classes that can be copied to the new javaEE project that you create by following the steps in [Installation page](Getting Started/Installation).
!!!
!@

@![javaSE,javaEE]
## Connecting to Head Unit 
To connect the sample app to the infotainment system, please follow the instructions in the [Connecting to an Infotainment System guide](Getting Started/Connecting to an Infotainment System)
!@


