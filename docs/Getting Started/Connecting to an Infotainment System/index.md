# Connecting to an Infotainment System
To connect to an emulator, such as Manticore or a local Ubuntu SDL Core-based emulator, make sure to implement a TCP (`debug`) connection. The emulator and app should be on the same network (i.e. remember to set the correct IP address and port number in the `SDLLifecycleConfiguration`). The IP will most likely be the IP address of the operating system running the emulator. The port will most likely be `12345`.

!!! IMPORTANT
Known issues due to using a TCP connection:

* When app is in the background mode, the app will be unable to communicate with SDL Core. This will work on IAP connections.
* Audio will not play on the emulator. Only IAP connections are currently able to play audio because this happens over the standard Bluetooth / USB system audio channel.
* You cannot connect to an emulator using a USB connection due to Apple limitations with IAP connections.
!!!

## Connecting with a Vehicle Head Unit or a Development Kit (TDK)
### Production
To connect your iOS device directly to a vehicle head unit or TDK, make sure to implement an iAP (`default`) connection in the `SDLLifecycleConfiguration`. Then connect the iOS device to the head unit or TDK using a USB cord or Bluetooth if the head unit supports it.

### Debugging
If you are testing with a vehicle head unit or TDK and wish to see debug logs in Xcode while the app is running, you must either use another app called the [relay app](https://github.com/smartdevicelink/relay_app_ios) to help you connect to the device, or you may use [Xcode 9 / iOS 11 wireless debugging](https://developer.apple.com/videos/play/wwdc2017/404/). When using the relay app, make sure to implement a TCP connection, if using iOS 11 wireless debugging, implement a IAP connection. Please see the [guide](Developer Tools/Relay App) for the relay app to learn how to set up the connection between the device, the relay app and your app.

!!! NOTE
The same issues apply when connecting the relay app with a TDK or head unit as do when connecting to SDL Core. Please see the issues above, under the *Connect with an Emulator* heading.
!!!
