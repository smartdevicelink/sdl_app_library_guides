@![iOS]
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
!@

@![android, javaSE, javaEE] `// TODO, Android and iOS guides on this section are very different should they be aligned better?`
# Adding the Lock Screen

In order for your SDL application to be certified with most OEMs you will be required to implement a lock screen on the mobile device. The lock screen will disable user interactions with the application on the mobile device while they are using the head-unit to control application functionality. OEMs may choose to send their logo for your app's lock screen to use; the `LockScreenManager` takes care of this automatically using the default layout.

!!! NOTE
This guide assumes that you have an SDL Service implemented as defined in the [Getting Started](Getting Started With Android/Installation) guide.
!!!

There is a manager called the `LockScreenManager` that is accessed through the `SdlManager` that handles much of the logic for you. If you have implemented the `SdlManager` and have defined the `SDLLockScreenActivity` in your manifest but have not defined any lock screen configuration, you are already have a working default configuration. This guide will go over specific configurations you are able to implement using the `LockScreenManager` functionality.

## Lock Screen Activity

You must declare the `SDLLockScreenActivity` in your manifest. To do so, simply add the following to your app's `AndroidManifest.xml` if you have not already done so:

```xml
<activity android:name="com.smartdevicelink.managers.lockscreen.SDLLockScreenActivity"
                  android:launchMode="singleTop"/>
```

!!! MUST
This manifest entry must be added for the lock screen feature to work.
!!!

## Configurations

The default configurations should work for most app developers and is simple to get up and and running. However, it is easy to perform deeper configurations to the lock screen for your app. Below are the options that are available to customize your lock screen which builds on top of the logic already implemented in the `LockScreenManager`.

There is a setter in the `SdlManager.Builder` that allows you to set a `LockScreenConfig` by calling `builder.setLockScreenConfig(lockScreenConfig)`. The following options are available to be configured with the`LockScreenConfig`.

In order to to use these features, create a `LockScreenConfig` object and set it using `SdlManager.Builder` before you build `SdlManager`.

#### Custom Background Color

In your `LockScreenConfig` object, you can set the background color to a color resource that you have defined in your `Colors.xml` file:

```java
lockScreenConfig.setBackgroundColor(resourceColor); // For example, R.color.black
```

#### Custom App Icon

In your `LockScreenConfig` object, you can set the resource location of the drawable icon you would like displayed:

```java
lockScreenConfig.setAppIcon(appIconInt); // For example, R.drawable.lockscreen_icon
```

#### Showing The Device Logo

This sets whether or not to show the connected device's logo on the default lock screen. The logo will come from the connected hardware if set by the manufacturer. When using a Custom View, the custom layout will have to handle the logic to display the device logo or not. The default setting is false, but some OEM partners may require it.

In your `LockScreenConfig` object, you can set the boolean of whether or not you want the device logo shown, if available:

```java
lockScreenConfig.showDeviceLogo(true);
```

#### Setting A Custom Lock Screen View

If you'd rather provide your own layout, it is easy to set. In your `LockScreenConfig` object, you can set the reference to the custom layout to be used for the lock screen. If this is set, the other customizations described above will be ignored:

```java
lockScreenConfig.setCustomView(customViewInt);
```

#### Disabling the Lock Screen Manager:

Please note that a lock screen will likely be required by OEMs. You can disable the `LockScreenManager`, but you will then be required to create your own implementation. This is not recommended as the `LockScreenConfig ` should enable all possible settings while still adhering to most OEM requirements. However, if it  is unavoidable to create one from scratch the `LockScreenManager` can be disabled via the `LockScreenConfig` as follows.

```java
lockScreenConfig.setEnabled(false);
```

!!! NOTE
When the enabled flag is set to `false` all other config options will be ignored.
!!!
!@
