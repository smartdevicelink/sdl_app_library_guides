# Updating from 4.5 to 4.6 


This guide is to help developers get setup with the SDL Android library 4.6. It is assumed that the developer is already updated to 4.5 of the library. There are a few important changes that we need to make to the integration to keep things working well. The first is removing some of the BroadcastReceiver's intent filters in `AndroidManifest.xml` that are now unnecessary. Secondly, the gradle integration of our library should now use `implementation` instead of `compile`. Lastly, the `RPCRequestFactory` class has been deprecated and constructors with mandatory parameters have been added for each RPC class.

We will make changes to:

* AndroidManifest.xml
* build.gradle
* any usage of RPCRequestFactory

## AndroidManifest.xml Updates
Assuming the manifest was up to date with version 4.5, we can now remove some of the intent-filters (`ACL_DISCONNECTED`, `STATE_CHANGED`, `AUDIO_BECOMING_NOISY`) for your app's BroadcastReceiver. The BroadcastReceiver section of the manifest should look as follows:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.company.mySdlApplication">

    <application>

        ...

        <receiver
            android:name=".SdlReceiver"
            android:exported="true"
            android:enabled="true">

            <intent-filter>
                <action android:name="android.bluetooth.device.action.ACL_CONNECTED" />
                <action android:name="sdl.router.startservice" />
            </intent-filter>

        </receiver>

    </application>

...

</manifest>
```

## Gradle Update
The previous way of including the library via `compile` should now use `implementation`. The dependencies section of your app's `build.gradle` file should now appear as:

```xml
dependencies {
    implementation 'com.smartdevicelink:sdl_android:4.+'
}
```

## Deprecation of RPCRequestFactory
The RPCRequestFactory has been deprecated in 4.6. To build RPC requests, developers should use the constructors in the desired RPC request class. For example, instead of using `RPCRequestFactory.buildAddCommand(...)` to build an `AddCommand` request, try the following:

```java
AddCommand addCommand = new AddCommand(100);
addCommand.setMenuParams(new MenuParams("Skip"));
proxy.sendRPCRequest(addCommand);
```