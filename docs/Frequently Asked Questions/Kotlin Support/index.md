# Does the SDL Java Suite library work with Kotlin?

The library has not been fully tested when being referenced from a Kotlin environment. Everything should work as expected, but if you find errors please report them to the github.

## Are there any currently known issues with Kotlin?
Even though Java is compatible with Kotlin, Kotlin has more strict rules for access modifiers than Java. For that reason, you may see this warning when using `SdlManager`'s `Builder` class:
```
Type BaseSdlManager.Builder! is inaccessible in this context due to: public open class Builder defined in com.smartdevicelink.managers.BaseSdlManager
```

While the warning is present, the functionality should continue to work in Kotlin. However, as a workaround, developers can create a Java class `SdlManagerFactory` that can be accessed from Kotlin code with a static method to create an `SdlManager` instance and handle all the builder code there. This will prevent the warning from the Kotlin side.

```java
public class SdlManagerFactory {

    public static SdlManager createSdlManager(Context context, String appID, String appName, SdlManagerListener listener, Vector<AppHMIType> appTypes, SdlArtwork appIcon) {
        SdlManager.Builder builder = new SdlManager.Builder(context, appID, appName, listener);
        builder.setAppTypes(appTypes);
        builder.setTransportType(new MultiplexTransportConfig(context, appID));
        builder.setAppIcon(appIcon);
        return builder.build();
    }
}
```

Then from the Kotlin side:

```kotlin 
val sdlManager = SdlManagerFactory.createSdlManager(this, APP_ID, APP_NAME, listener, appTypes, appIcon);
```

