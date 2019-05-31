# Configuring SDL Logging
SDL iOS v5.0 includes a powerful new built-in logging framework designed to make debugging easier. It provides many of the features common to other 3rd party logging frameworks for iOS and can be used by your own app as well. We recommend that your app's integration with SDL provide logging using this framework rather than any other 3rd party framework your app may be using or `NSLog`. This will consolidate all SDL related logs in a common format and to common destinations.

SDL will configure its logging into a production-friendly configuration by default. If you wish to use a debug or a custom configuration, then you will have to specify this yourself. `SDLConfiguration` allows you to pass a `SDLLogConfiguration` with custom values. A few of these values will be covered in this section, the others are in their own sections below.

When setting up your `SDLConfiguration` you can pass a different log configuration:

##### Objective-C
```objc
SDLConfiguration* configuration = [SDLConfiguration configurationWithLifecycle:lifecycleConfiguration lockScreen:[SDLLockScreenConfiguration enabledConfiguration] logging:[SDLLogConfiguration debugConfiguration] fileManager:[SDLFileManagerConfiguration defaultConfiguration]];
```

##### Swift
```swift
let configuration = SDLConfiguration(lifecycle: lifecycleConfiguration, lockScreen: .enabled(), logging: .debug(), fileManager: .default())
```

### Format Type
Currently, SDL provides three output formats for logs (for example into the console or file log), these are "Simple", "Default", and "Detailed".

**Simple:**
```
09:52:07:324 🔹 (SDL)Protocol – I'm a log!
```

**Default:**
```
09:52:07:324 🔹 (SDL)Protocol:SDLV2ProtocolHeader:25 – I'm also a log!
```

**Detailed:**
```
09:52:07:324 🔹 DEBUG com.apple.main-thread:(SDL)Protocol:[SDLV2ProtocolHeader parse:]:74 – Me three!
```

### Log Synchronicity
The configuration provides two properties, `asynchronous` and `errorsAsynchronous`. By default `asynchronous` is true and `errorsAsynchronous` is false. This means that any logs that are not logged at the error log level will be logged asynchronously on a separate serial queue, while those on the error log level will be logged synchronously on the separate queue (but the thread that logged it will be blocked until that log completes).

### Log level
The `globalLogLevel` defines which logs will be logged to the target outputs. For example, if you set the log level to `debug`, all error, warning, and debug level logs will be logged, but verbose level logs will not be logged.

| SDLLogLevel | Visible Logs | 
| ------------- | ------------- |
| Off | none |
| Error | error |
| Warning | error and warning |
| Debug | error, warning and debug |
| Verbose | error, warning, debug and verbose |

!!! NOTE
Although the `default` log level is defined in the SDLLogLevel enum, it should not be used as a global log level. See the [API documentation](https://smartdevicelink.com/en/docs/iOS/master/Enums/SDLLogLevel/) for more detail.
!!!

### Targets
Targets are the output locations where the log will appear. By default, in both default and debug configurations, only the Apple System Logger target (iOS 9 and below) or OSLog (iOS 10+) will be enabled.

#### Apple System Log Target
The Apple System Logger target, `SDLLogTargetAppleSystemLogger`, is the default log target for both default and debug configurations on devices running iOS 9 or older. This will log to the Xcode console and the device console.

#### OS Log Target
The OSLog target, `SDLLogTargetOSLog`, is the default log target in both default and debug configurations for devices running iOS 10 or newer. For more information on this logging system see [Apple's documentation](https://developer.apple.com/reference/os/logging). SDL's OSLog target will take advantage of subsystems and levels to allow you powerful runtime filtering capabilities through MacOS Sierra's Console app with a connected device.

#### File Target
The File target, `SDLLogTargetFile`, allows you to log messages to a rolling set of files (default 3) which will be stored on the device, specifically in the `Documents/smartdevicelink/log/` folder. The file names will be timestamped with the start time.

To access the file, you can either access it from runtime on the device (for example, to attach it to an email that the user sends), or if you have access to the device, you can access them via iTunes as well with small modifications to your app:

1. Add the key `UIFileSharingEnabled` to your `info.plist`. Set the value to `YES`.
2. Connect the device to a computer that has iTunes installed.
3. Open iTunes, click on the icon for the device, then click on "File Sharing" > "(Your App Name)"
4. You should see a folder called "smartdevicelink". Select the file and click save. When you open the folder on your computer, you will see the log files for each session (default maxes out at 3).

You should not keep this info.plist key when you submit your app to Apple.

#### Custom Log Targets
The protocol all log targets conform to, `SDLLogTarget`, is public. If you wish to make a custom log target in order to, for example, log to a server, it should be fairly easy to do so. If it can be used by other developers and is not specific to your app, then submit it back to the SmartDeviceLink iOS library project! If you want to add targets *in addition* to the default target that will output to the console:

##### Objective-C
```objc
logConfig.targets = [logConfig.targets setByAddingObjectsFromArray:@[[SDLLogTargetFile logger]]];
```

##### Swift
```swift
let _ = logConfig.targets.insert(SDLLogTargetFile())
```

### Modules
A module is a set of files packaged together. Create modules using the `SDLLogFileModule` class and add it to the configuration. Modules are used when outputting a log message. The log message may specify a module instead of a specific file name for clarity's sake. The SDL library will automatically add the modules corresponding to its own files after you submit your configuration. For your specific use case, you may wish to provide a module corresponding to your whole app's integration and simply name it with your app's name, or, you could split it up further if desired. To add modules to the configuration:

##### Objective-C
```objc
logConfig.modules = [logConfig.modules setByAddingObjectsFromArray:@[[SDLLogFileModule moduleWithName:@"Test" files:[NSSet setWithArray:@[@"File1", @"File2"]]]]];
```

##### Swift
```swift
logConfig.modules.insert(SDLLogFileModule(name: "Test", files: ["File1, File2"]))
```

### Filters
Filters are a compile-time concept of filtering in or out specific log messages based on a variety of possible factors. Call `SDLLogFilter` to easily set up one of the default filters or to create your own using a custom `SDLLogFilterBlock`. You can filter to only allow certain files or modules to log, only allow logs with a certain string contained in the message, or use regular expressions.

##### Objective-C
```objc
SDLLogFilter *filter = [SDLLogFilter filterByDisallowingString:@"Test" caseSensitive:NO];
```

##### Swift
```swift
let filter = SDLLogFilter(disallowingString: "Test", caseSensitive: false)
```

## Logging with the SDL Logger
In addition to viewing the library logs, you also have the ability to log with the SDL logger. All messages logged through the SDL logger, including your own, will use your `SDLLogConfiguration` settings.

### Objective-C Projects
First, import the the `SDLLogMacros` header.

```
#import "SDLLogMacros.h"
```

Then, simply use the convenient log macros to create a custom SDL log in your project. 

```objc
SDLLogV(@"This is a verbose log");
SDLLogD(@"This is a debug log");
SDLLogW(@"This is a warning log");
SDLLogE(@"This is an error log");
```

### Swift Projects
To add custom SDL logs to your Swift project you must first install a submodule called **SmartDeviceLink/Swift**.

#### CocoaPods
If the SDL iOS library was installed using [CocoaPods](https://cocoapods.org), simply add the submodule to the **Podfile** and then install by running `pod install` in the root directory of the project.

```swift
target '<#Your Project Name#>' do
    pod 'SmartDeviceLink', '~> <#SDL Version#>'
    pod 'SmartDeviceLink/Swift', '~> <#SDL Version#>'
end
```

#### Logging in Swift
After the submodule has been installed, you can use the `SDLLog` functions in your project.

```swift
SDLLog.v("This is a verbose log")
SDLLog.d("This is a debug log")
SDLLog.w("This is a warning log")
SDLLog.e("This is an error log")
```
