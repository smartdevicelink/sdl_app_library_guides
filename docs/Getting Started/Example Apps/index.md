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
To get the example app, download or clone the [sdl_java_suite](https://github.com/smartdevicelink/sdl_java_suite). The _Hello Sdl Android_ app is a package within the SDL Android library. Open the the `sdl_java_suite/android` project using "Open an existing Android Studio project" in [Android Studio](https://developer.android.com/studio/index.html). We will use Android Studio throughout this guide as it is the official IDE for Android development.
!@

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
To connect the example app to [Manticore](https://smartdevicelink.com/resources/manticore/) or another emulator, make sure you are using `tcpDebug` build flavor. You must update the IP address and port number in the _Hello Sdl Android_ project so it knows where your emulator is running. Please check the [Connecting to an Infotainment System](Getting Started/Connecting to an Infotainment System) guide for more detailed instructions on how to get the emulator's IP address and port number.

1. In the main Java folder of _Hello Sdl Android_, open up `SdlService.java`.
1. At the top of this file, locate the variable declaration for `DEV_MACHINE_IP_ADDRESS` and change it to your emulator's IP address. Set the `TCP_PORT` to your emulator's port number.

        private static final String DEV_MACHINE_IP_ADDRESS = "192.168.1.78"; // Update
        private static final int TCP_PORT = 12345; // Update

1. Make sure the emulator is running, then build and run the app on a real device or a simulated device. The SDL app should show up on the HMI.   

### Head Unit
To connect the example app to production or debug hardware via Bluetooth or USB, all you need to do to is select the `multi_sec_offDebug` build flavor and then run the app on a real Android device. You can find more information about the USB transport in the [Using AOA Protocol](Getting Started/Using AOA Protocol) guide. 

If using the Bluetooth transport, make sure to first pair your Android phone to the hardware before attempting to connect your SDL app.
!@

@![iOS,android]
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
1. You cannot connect to any of our open-source emulators using a USB cord or Bluetooth because Apple's [MFi Program](https://mfi.apple.com/MFiWeb/getFAQ.action#4-6) is confidential and can not be used in open source projects.

### iAP Production Transport
1. Make sure to use the default `SDLLifecycleConfiguration`.
1. Make sure the [protocol](https://smartdevicelink.com/en/guides/iOS/getting-started/sdk-configuration/) strings have been added to the app.
1. Make sure you have enabled background [capabilities](https://smartdevicelink.com/en/guides/iOS/getting-started/sdk-configuration/) for your app.
1. If the head unit (emulators do not support IAP) does not support Bluetooth, an iAP connection requires a USB cord.

### iAP Bluetooth Production Transport
1. Bluetooth transport support is automatic when you support the iAP production transport. It cannot be turned on or off separately.
1. Make sure the head unit supports Bluetooth transport for iPhones. Currently, only some head units support Bluetooth.
1. Make sure to use the default `SDLLifecycleConfiguration`.
1. Make sure Bluetooth is turned on - both on the head unit hardware and your iPhone.
1. Ensure your iPhone is properly paired with the head unit. 
!@

@![android]
### TCP Debug Transport
1. Make sure that you have changed the IP in `SdlService.java` to match the machine running SDL Core. Being on the same network is also important.
2. If you are sure that the IP is correct and it is still not showing up, make sure the Build Flavor that is running is `tcpDebug`.
3. If the two above don't work, make sure there is no firewall blocking the incoming port `12345` on the machine or VM running SDL Core. Also, make sure your firewall allows that outgoing port.
4. There are different network configurations needed for different virtualization software (virtualbox, vmware, etc). Make sure yours is set up correctly. Or use [Manticore](https://smartdevicelink.com/resources/manticore/).

### Bluetooth
1. Make sure the build flavor `multi_sec_offDebug` is selected.
2. Ensure your phone is properly paired with the TDK
3. Make sure Bluetooth is turned on - on both the TDK and your phone
4. Make sure apps are enabled on the TDK (in settings)
!@

@![javaSE]
First, make sure you download or clone the latest release from [GitHub](https://github.com/smartdevicelink/sdl_java_suite). It is a [project](https://github.com/smartdevicelink/sdl_java_suite/tree/master/hello_sdl_java) within the SDL Java Suite root directory. Then, open the _Hello Sdl_ project in [IntelliJ IDEA](https://www.jetbrains.com/idea/).
!@

@![javaEE]
Make sure that you follow the steps in [Installation](Getting Started/Installation) and [Integration Basics](Getting Started/Integration Basics) sections to create a new JavaEE SDL project before continuing this section.

!!! NOTE
The [Hello Sdl JavaEE](https://github.com/smartdevicelink/sdl_java_suite/tree/master/hello_sdl_java_ee) has some code commented out and cannot be compiled. The project just includes samples for `SdlService` and `Main` classes that can be copied to the new javaEE project that you create by following the steps in [Installation page](Getting Started/Installation).
!!!
!@

@![javaSE,javaEE]
## Connecting to an Infotainment System 
To connect the sample app to the infotainment system, please follow the instructions in the [Connecting to an Infotainment System](Getting Started/Connecting to an Infotainment System) guide
!@


