# Example Apps
SDL provides two example apps: one written in Objective-C and one in Swift. Both implement the same features.

The example apps are located in the [sdl_ios](https://github.com/smartdevicelink/sdl_ios) repository. To try them, you can download the repository and run the example app targets, or you may use `pod try SmartDeviceLink` with [CocoaPods](https://cocoapods.org) installed on your Mac.

!!! NOTE
If you download or clone the SDL repository in order to run the example apps, you must first obtain the BSON submodule. You can do so by running `git submodule init` and `git submodule update` in your terminal when in the main directory of the cloned repository.
!!!

The example apps implement soft buttons, template text and images, a main menu and submenu, vehicle data, popup menus, voice commands, and capturing in-car audio.

## Connecting to Hardware
To connect the example app to [Manticore](https://smartdevicelink.com/resources/manticore/) or another emulator, make sure you are on the `TCP Debug` tab and type in the IP address and port, then press "Connect". The button will turn green when you are connected.

To connect the example app to production or debug hardware, make sure you are on the `iAP` tab and press "Connect". The button will turn green when you are connected.