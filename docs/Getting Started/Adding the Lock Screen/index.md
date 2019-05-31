# Adding the Lock Screen
The lock screen is a vital part of SmartDeviceLink, as the lock screen prevents the user from using your application while the vehicle is in motion. SDL takes care of the lock screen for you. It still allows you to use your own view controller if you prefer your own look, but still want the recommended logic that SDL provides for free.

A benefit to using the provided Lock Screen is that we also handle retrieving a lock screen icon for versions of Core that support it, so that you do not have to be concerned with what car manufacturer you are connected to. If you subclass the provided lock screen, you can do the same in your own lock screen view controller.

If you would not like to use any of the following code, you may use the `SDLLockScreenConfiguration` class function `disabledConfiguration`, and manage the entire lifecycle of the lock screen yourself. However, it is strongly recommended that you use the provided lock screen manager, even if you use your own view controller.

To see where the `SDLLockScreenConfiguration` is used, refer to the [Integration Basics](Getting Started/Integration Basics) guide.

## Using the Provided Lock Screen
Using the default lock screen is simple. Using the lock screen this way will automatically load an automaker's logo, if available, to show alongside your logo. If it is not, the default lock screen will show your logo alone.

![Generic Lock Screen](/assets/GenericLockScreen.png)

To do this, instantiate a new `SDLLockScreenConfiguration`:

##### Objective-C
```objc
SDLLockScreenConfiguration *lockScreenConfiguration = [SDLLockScreenConfiguration enabledConfiguration];
```

##### Swift
```swift
let lockScreenConfiguration = SDLLockScreenConfiguration.enabled()
```

## Customizing the Provided Lock Screen
If you would like to use the provided lock screen but would like to add your own appearance to it, we provide that as well. `SDLLockScreenConfiguration` allows you to customize the background color as well as your app's icon. If the app icon is not included, we will use the SDL logo.

![Custom Lock Screen](/assets/CustomLockScreen.png)

##### Objective-C
```objc
UIImage *appIcon = <# Retreive App Icon #>
UIColor *backgroundColor = <# Desired Background Color #>
SDLLockScreenConfiguration *lockScreenConfiguration = [SDLLockScreenConfiguration enabledConfigurationWithAppIcon:appIcon backgroundColor:backgroundColor];
```

##### Swift
```swift
let appIcon: UIImage = <# Retrieve App Icon #>
let backgroundColor: UIColor = <# Desired Background Color #>
let lockScreenConfiguration = SDLLockScreenConfiguration.enabledConfiguration(withAppIcon: appIcon, backgroundColor: backgroundColor)
```

## Using Your Own Lock Screen
If you would like to use your own lock screen instead of the provided SDL one, but still use the logic we provide, you can use a new initializer within `SDLLockScreenConfiguration`:

##### Objective-C
```objc
UIViewController *lockScreenViewController = <# Initialize Your View Controller #>;
SDLLockScreenConfiguration *lockScreenConfiguration = [SDLLockScreenConfiguration enabledConfigurationWithViewController:lockScreenViewController];
```

##### Swift
```swift
let lockScreenViewController = <# Initialize Your View Controller #>
let lockScreenConfiguration = SDLLockScreenConfiguration.enabledConfiguration(with: lockScreenViewController)
```

### Using the Vehicle's Icon
If you want to build your own lock screen view controller, it is recommended that you subclass `SDLLockScreenViewController` and use the public `appIcon`, `vehicleIcon`, and `backgroundColor` properties.