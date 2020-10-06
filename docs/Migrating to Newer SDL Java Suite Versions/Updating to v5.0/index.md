# Upgrading to 5.0

## Overview

This guide is to help developers get setup with the SDL Java library version 5.0. It is assumed that the developer is already updated to at least version 4.11 or 4.12 of the library.

The full release notes are published [here](https://github.com/smartdevicelink/sdl_java_suite/releases).

## New minimum SDK 
SDL now has a new minimun required SDK version of 16. You can change the minimum SDK version in the apps build.gradle file by changing minSdkVersion to 16. An example:

```Gradle
    defaultConfig {
        applicationId "com.sdl.mobileweather"
        minSdkVersion 16
        targetSdkVersion 26
        versionCode 27
        versionName "1.7.15"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
```

## AndroidX
SDL now uses AndroidX. To migrate your app to use AndroidX, In Android Stuido or IntelliJ, click on Refactor, then Migrate to AndroidX.

 ![Android Stuido](assets/migrate_androidX.png )

!!! NOTE
To migrate to AndroidX you must set the `compileSdkVersion` to 28 in the apps build.gradle file
!!!

## New imports due to package changes

## SdlManagerListener managerShouldUpdateLifecycle changes
In 4.12 a new managerShouldUpdateLifecycle method was added and the old managerShouldUpdateLifecycle method was deprecated. In 5.0 the deprecated method was removed.

Before:

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
                public LifecycleConfigurationUpdate managerShouldUpdateLifecycle(Language language) {
                    return null;
                }
            };
```

Now:

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
            };
```

## sending RPC's listener updates (onError removal)

## Use Multiplex instead of legacy BT & USB

## ScreenManager Template Managment

## Chainable RPC setters

## Use the new DebugTool methods(especially in javaSE, there is no log.x() methods anymore)

## TTSChunkFactory removal
`TTSChunkFactory.java` was removed. To create a voice command you should now use `TTSChunk` An example of creating and sending a voice command:

Before:
```Java
Speak msg = new Speak(TTSChunkFactory.createSimpleTTSChunks("Voice Message to speak"));
sdlManager.sendRPC(msg);
```
Now:
```Java
Speak msg = new Speak(Collections.singletonList(new TTSChunk("Voice Message to speak", SpeechCapabilities.TEXT)));
sdlManager.sendRPC(msg);
```






