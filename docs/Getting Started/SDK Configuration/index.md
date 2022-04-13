# SDK Configuration

@![iOS, android, javaSE, javaEE, javascript]
## 1. Get an App Id
An app id is required for production level apps. The app id gives your app special permissions to access vehicle data. If your app does not need to access vehicle data, a dummy app id (i.e. creating a fake id like "1234") is sufficient during the development stage. However, you must get an app id before releasing the app to the public.

To obtain an app id, sign up at [smartdevicelink.com](https://www.smartdevicelink.com).
!@

@![iOS]
## 2. Enable Background Capabilities
Your application must be able to maintain a connection to the SDL Core even when it is in the background. This capability must be explicitly enabled for your application (available for iOS 5+). To enable the feature, select your application's build target, go to *Capabilities*, *Background Modes*, and select *External accessory communication mode*.

## 3. Add SDL Protocol Strings
Your application must support a set of SDL protocol strings in order to be connected to SDL enabled head units. Go to your application's **.plist** file and add the following code under the top level dictionary.

!!! NOTE
This is only required for USB and Bluetooth enabled head units. It is not necessary during development using SDL Core.
!!!

```xml
<key>UISupportedExternalAccessoryProtocols</key>
<array>
<string>com.smartdevicelink.prot29</string>
<string>com.smartdevicelink.prot28</string>
<string>com.smartdevicelink.prot27</string>
<string>com.smartdevicelink.prot26</string>
<string>com.smartdevicelink.prot25</string>
<string>com.smartdevicelink.prot24</string>
<string>com.smartdevicelink.prot23</string>
<string>com.smartdevicelink.prot22</string>
<string>com.smartdevicelink.prot21</string>
<string>com.smartdevicelink.prot20</string>
<string>com.smartdevicelink.prot19</string>
<string>com.smartdevicelink.prot18</string>
<string>com.smartdevicelink.prot17</string>
<string>com.smartdevicelink.prot16</string>
<string>com.smartdevicelink.prot15</string>
<string>com.smartdevicelink.prot14</string>
<string>com.smartdevicelink.prot13</string>
<string>com.smartdevicelink.prot12</string>
<string>com.smartdevicelink.prot11</string>
<string>com.smartdevicelink.prot10</string>
<string>com.smartdevicelink.prot9</string>
<string>com.smartdevicelink.prot8</string>
<string>com.smartdevicelink.prot7</string>
<string>com.smartdevicelink.prot6</string>
<string>com.smartdevicelink.prot5</string>
<string>com.smartdevicelink.prot4</string>
<string>com.smartdevicelink.prot3</string>
<string>com.smartdevicelink.prot2</string>
<string>com.smartdevicelink.prot1</string>
<string>com.smartdevicelink.prot0</string>
<string>com.smartdevicelink.multisession</string>
<string>com.ford.sync.prot0</string>
</array>
```
!@

@![android]
## 2. Add Required System Permissions
Some permissions are required to be granted to the SDL app in order for it to work properly. In the AndroidManifest file, we need to ensure we have the following system permissions:

* [Internet](https://developer.android.com/reference/android/Manifest.permission.html#INTERNET) - Used by the mobile library to communicate with a SDL Server
* [Bluetooth](https://developer.android.com/reference/android/Manifest.permission.html#BLUETOOTH) - Primary transport for SDL communication between the device and the vehicle's head-unit
* [Access Network State](https://developer.android.com/reference/android/Manifest.permission.html#ACCESS_NETWORK_STATE) - Required to check if WiFi is enabled on the device

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.company.mySdlApplication">

    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.BLUETOOTH"/>
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.FOREGROUND_SERVICE" />

</manifest>
```

!!! NOTE
If the app is targeting Android S (API Level 31) or higher, the Android Manifest file also needs to include the following permission to allow the app to be notified of Bluetooth Connections:

This permission is a runtime permission and will require the user to grant the permission.

```xml
<uses-permission android:name="android.permission.BLUETOOTH_CONNECT"
    tools:targetApi="31"/>
```
!!!

## 3. Add Required SDL Queries

If targeting Android R (API Level 30) or higher, it is required to add the SDL specific entries into the app's `queries` tag in the `AndroidManifest.xml`. If the tag already exists, just the intents need to be added. If the tag does not yet exist in the manifest, they can be added after the permissions are declared but before the `application` tag is opened.

```xml
<queries>
    <intent>
         <action android:name="com.smartdevicelink.router.service" />
    </intent>
    <intent>
        <action android:name="sdl.router.startservice" />
    </intent>
</queries>
```

The SDL Android library uses these queries to determine which app should host the router service, what apps to notify when there's an SDL connection, etc. As will be seen in the next sections, these intents are used in the intent filters for the `SdlRouterService` and the `SdlBroadcastReceiver`.
!@