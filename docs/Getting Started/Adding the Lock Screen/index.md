# Adding the Lock Screen
The lock screen is a vital part of SmartDeviceLink, as the lock screen prevents the user from using your application while the vehicle is in motion. SDL takes care of the lock screen for you. It still allows you to use your own view controller if you prefer your own look, but still want the recommended logic that SDL provides for free.

A benefit to using the provided Lock Screen is that we also handle retrieving a lock screen icon for versions of Core that support it, so that you do not have to be concerned with what car manufacturer you are connected to. If you subclass the provided lock screen, you can do the same in your own lock screen view controller.

@![iOS]
If you would not like to use any of the following code, you may use the `SDLLockScreenConfiguration` class function `disabledConfiguration`, and manage the entire lifecycle of the lock screen yourself. However, it is strongly recommended that you use the provided lock screen manager, even if you use your own view controller.

To see where the `SDLLockScreenConfiguration` is used, refer to the [Integration Basics](Getting Started/Integration Basics) guide.
!@

@![android]
There is a manager called the `LockScreenManager` that is accessed through the `SdlManager` that handles much of the logic for you. If you have implemented the `SdlManager` and have defined the `SDLLockScreenActivity` in your manifest but have not defined any lock screen configuration, you are already have a working default configuration. This guide will go over specific configurations you are able to implement using the `LockScreenManager` functionality.

You must declare the `SDLLockScreenActivity` in your manifest. To do so, simply add the following to your app's `AndroidManifest.xml` if you have not already done so:

```xml
<activity android:name="com.smartdevicelink.managers.lockscreen.SDLLockScreenActivity"
                  android:launchMode="singleTop"/>
```

!!! MUST
This manifest entry must be added for the lock screen feature to work.
!!!
!@

## Using the Provided Lock Screen
Using the default lock screen is simple. Using the lock screen this way will automatically load an automaker's logo, if available, to show alongside your logo. If it is not, the default lock screen will show your logo alone.

![Generic Lock Screen](/assets/GenericLockScreen.png)

@![iOS]
To do this, instantiate a new `SDLLockScreenConfiguration`:
!@

@![android]
There is a setter in the `SdlManager.Builder` that allows you to set a `LockScreenConfig` by calling `builder.setLockScreenConfig(lockScreenConfig)`. The following options are available to be configured with the`LockScreenConfig`.

In order to to use these features, create a `LockScreenConfig` object and set it using `SdlManager.Builder` before you build `SdlManager`.
!@

##### Objective-C
```objc
SDLLockScreenConfiguration *lockScreenConfiguration = [SDLLockScreenConfiguration enabledConfiguration];
```

##### Swift
```swift
let lockScreenConfiguration = SDLLockScreenConfiguration.enabled()
```

@![android]
`TODO Add code example`
!@

## Customizing the Provided Lock Screen
If you would like to use the provided lock screen but would like to add your own appearance to it, we provide that as well. @![iOS]`SDLLockScreenConfiguration`!@ @![android]`LockScreenConfig`!@ allows you to customize the background color as well as your app's icon. If the app icon is not included, we will use the SDL logo.

![Custom Lock Screen](/assets/CustomLockScreen.png)

### Custom Background Color
@![iOS]
##### Objective-C
```objc
UIColor *backgroundColor = <# Desired Background Color #>
SDLLockScreenConfiguration *lockScreenConfiguration = [SDLLockScreenConfiguration enabledConfigurationWithAppIcon:<# Retreive App Icon #> backgroundColor:backgroundColor];
```

##### Swift
```swift
let backgroundColor: UIColor = <# Desired Background Color #>
let lockScreenConfiguration = SDLLockScreenConfiguration.enabledConfiguration(withAppIcon: <# Retrieve App Icon #>, backgroundColor: backgroundColor)
```
!@

@![android]
```java
lockScreenConfig.setBackgroundColor(resourceColor); // For example, R.color.black
```
!@

#### Custom App Icon
@![iOS]
##### Objective-C
```objc
let appIcon: UIImage = <# Retrieve App Icon #>
SDLLockScreenConfiguration *lockScreenConfiguration = [SDLLockScreenConfiguration enabledConfigurationWithAppIcon:appIcon backgroundColor:<# Desired Background Color #>];
```

##### Swift
```swift
let appIcon: UIImage = <# Retrieve App Icon #>
let lockScreenConfiguration = SDLLockScreenConfiguration.enabledConfiguration(withAppIcon: appIcon, backgroundColor: <# Desired Background Color #>)
```
!@

@![android]
```java
lockScreenConfig.setAppIcon(appIconInt); // For example, R.drawable.lockscreen icon
```
!@

## Using Your Own Lock Screen
If you would like to use your own lock screen instead of the provided SDL one, but still use the logic we provide, you can use a new initializer within @![iOS]`SDLLockScreenConfiguration`!@ @![android]`LockScreenConfig`!@:

@![iOS]
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
!@

@![android]
```java
lockScreenConfig.setCustomView(customViewInt);
```
!@

#### Disabling the Lock Screen Manager

Please note that a lock screen will likely be required by OEMs. You can disable the Lock Screen Manager, but you will then be required to create your own implementation. This is not recommended as the @![iOS]`SDLLockScreenConfiguration` !@ @![android]`LockScreenConfig`!@ should enable all possible settings while still adhering to most OEM requirements. However, if it  is unavoidable to create one from scratch the Lock Screen Manager can be disabled via the @![iOS]`SDLLockScreenConfiguration`!@ @![android]`LockScreenConfig`!@ as follows.

@![android]
```java
lockScreenConfig.setEnabled(false);
```
!@

@![iOS]
##### Objective-C
```objc
SDLLockScreenConfiguration *lockScreenConfiguration = [SDLLockScreenConfiguration enabledConfiguration];
lockScreenConfiguration.enableAutomaticLockScreen = NO;
```

```swift
let lockScreenConfiguration = SDLLockScreenConfiguration.enabledConfiguration()
lockScreenConfiguration.enableAutomaticLockScreen = false
!@
