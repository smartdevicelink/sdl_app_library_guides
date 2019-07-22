# Adding the Lock Screen
The lock screen is a vital part of SmartDeviceLink as the lock screen prevents the user from using your application while the vehicle is in motion. SDL takes care of the lock screen for you. It still allows you to use your own @![iOS]view controller!@ @![android]activity!@ if you prefer your own look, but still want the recommended logic that SDL provides for free.


@![iOS]
If you would not like to use any of the following code, you may use the `SDLLockScreenConfiguration` class function `disabledConfiguration`, and manage the entire lifecycle of the lock screen yourself. However, it is strongly recommended that you use the provided lock screen manager, even if you use your own view controller.

To see where the `SDLLockScreenConfiguration` is used, refer to the [Integration Basics](Getting Started/Integration Basics) guide.
!@

@![android]
## Configure the Lock Screen Activity

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
@![iOS]Using the default lock screen is simple. Using the lock screen this way will automatically load an automaker's logo, if available, to show alongside your logo. If it is not, the default lock screen will show your logo alone.!@

@![iOS]
![Generic Lock Screen](assets/GenericLockScreen.png)

To do this, instantiate a new `SDLLockScreenConfiguration`:

##### Objective-C
```objc
SDLLockScreenConfiguration *lockScreenConfiguration = [SDLLockScreenConfiguration enabledConfiguration];
```

##### Swift
```swift
let lockScreenConfiguration = SDLLockScreenConfiguration.enabled()
```
!@

@![android]
If you have implemented the `SdlManager` and have defined the `SDLLockScreenActivity` in your manifest but have not defined any lock screen configuration, you are already have a working default configuration.
![Generic Lock Screen](assets/GenericLockScreen_Android.png)
!@


## Customizing the Default Lock Screen
It is possible to customize the background color and app icon in the default provided lockscreen. If you choose not to set your own app icon the library will use the SDL logo.

@![iOS]
![Custom Lock Screen](assets/CustomLockScreen.png)
!@

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

### Custom App Icon
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

### Showing the OEM Logo
@![iOS]
The default lock screen handles retrieving and setting the OEM logo from head units that support this feature. Currently this feature can not be disabled on the default lock screen.
!@

@![android]
This sets whether or not to show the connected device's logo on the default lock screen. The logo will come from the connected hardware if set by the manufacturer. When using a custom view, the custom layout will have to handle the logic to display the device logo. The default setting is false, but some OEM partners may require it.
In your `LockScreenConfig` object, you can set the boolean of whether or not you want the device logo shown, if available:
```java
lockScreenConfig.showDeviceLogo(true);
```
!@

## Create a Custom Lock Screen
If you would like to use your own lock screen instead of the one provided by the library, but still use the logic we provide, you can use a new initializer within @![iOS]`SDLLockScreenConfiguration`!@ @![android]`LockScreenConfig`!@:

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

### Customizing the Lock Screen State
### Disabling the Lock Screen
Please note that a lock screen will be required by most OEMs. You can disable the lock screen manager, but you will then be required to implement your own logic for showing and hiding the lock screen. This is not recommended as the @![iOS]`SDLLockScreenConfiguration` !@ @![android]`LockScreenConfig`!@ adheres to most OEM lock screen requirements. However, if you must create a lock screen manager from scratch, the library's lock screen manager can be disabled via the @![iOS]`SDLLockScreenConfiguration`!@ @![android]`LockScreenConfig`!@ as follows:

@![iOS]
##### Objective-C
```objc
SDLLockScreenConfiguration *lockScreenConfiguration = [SDLLockScreenConfiguration enabledConfiguration];
lockScreenConfiguration.enableAutomaticLockScreen = NO;
```

##### Swift
```swift
let lockScreenConfiguration = SDLLockScreenConfiguration.enabledConfiguration()
lockScreenConfiguration.enableAutomaticLockScreen = false
```
!@

@![android]
```java
lockScreenConfig.setEnabled(false);
```
!@

### Making the Lock Screen Always On
The lock screen manager is configured to dismiss the lock screen when it is safe to do so. To always have the lock screen visible when the device is connected to the head unit, simply update the lock screen configuration. 

@![iOS]
##### Objective-C
```objc
SDLLockScreenConfiguration *lockScreenConfiguration = [SDLLockScreenConfiguration enabledConfiguration];
lockScreenConfiguration.showInOptionalState = YES;
```

##### Swift
```swift
let lockScreenConfiguration = SDLLockScreenConfiguration.enabledConfiguration()
lockScreenConfiguration.showInOptionalState = true
```
!@

@![android]
```java
// TODO
```
!@

