# Updating to 5.1

## Overview

This guide is to help developers get setup with the SDL Java library version 5.1. It is assumed that the developer is already updated to at least version 5.0 of the library.

The full release notes are published [here](https://github.com/smartdevicelink/sdl_java_suite/releases).

## Maven Central
Starting with SDL Java library version 5.1 the release will be published to Maven Central instead of JCenter.

To gain access to the Maven Central repository, make sure your app's `build.gradle` file includes the following:

```
repositories {
    mavenCentral()
}
```

## SdlManagerListener changes
In 5.1 a new onSystemInfoReceived method was added to the SdlManagerListener. More detail can be found [here](Getting Started/Integration Basics - Java)

!!! MUST
`SdlManagerListener` method: `onSystemInfoReceived` auto generates in Android Studio to returns false. This will cause your app to not connect. You must change it to true or implement logic to check system info to see if you wish for your app to connect to that system.
!!!

```java
SdlManagerListener listener = new SdlManagerListener() {
    @Override
    public void onStart() {
    }

    @Override
    public void onDestroy() {
    }

    @Override
    public void onError(String info, Exception e) {
    }

    @Override
    public LifecycleConfigurationUpdate managerShouldUpdateLifecycle(Language language, Language hmiLanguage) {
        return null;
    }

    @Override
    public boolean onSystemInfoReceived(SystemInfo systemInfo) {
        //Check the SystemInfo object to ensure that the connection to the device should continue
        return true;
    }
};
```

## Alert View
In 5.1 rather than sending an Alert RPC we now recommend sending an AlertView through the ScreenManagers presentAlert method. More detail can be found [here](Displaying a User Interface/Alerts and Subtle Alerts)

Before:

```java
 private void showAlert(String text) {
        Alert alert = new Alert();
        alert.setAlertText1(text);
        alert.setDuration(5000);
        sdlManager.sendRPC(alert);
}
```

Now:

```java
 private void showAlert(String text) {
        AlertView.Builder builder = new AlertView.Builder();
        builder.setText(text);
        builder.setTimeout(5);
        AlertView alertView = builder.build();
        sdlManager.getScreenManager().presentAlert(alertView, new AlertCompletionListener() {
            @Override
            public void onComplete(boolean success, Integer tryAgainTime) {
                Log.i(TAG, "Alert presented: "+ success);
            }
        });
}
```

@![android]
## SDLRemoteDisplay
In 5.1 a new `onViewResized` method was added to the `SDLRemoteDisplay` class that you will need to implement in your presentation class. More detail can be found [here](Video Streaming for Navigation Apps/Video Streaming - Java).

Before:

```java
public static class MyDisplay extends SdlRemoteDisplay{
    public MyDisplay(Context context, Display display) {
        super(context, display);
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        //...
    }
}   
```

Now:

```java
public static class MyDisplay extends SdlRemoteDisplay{
    public MyDisplay(Context context, Display display) {
        super(context, display);
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        //...
    }

    @Override
    public void onViewResized(int width, int height) {
        DebugTool.logInfo(TAG, "Remote view new width and height ("+ width + ", " + height + ")");
        //Update presentation based on new resolution
    }
}   
```
!@