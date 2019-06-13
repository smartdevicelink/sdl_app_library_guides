# Proguard Guidelines

SmartDeviceLink and its dependent libraries are open source and not intended to be obfuscated. When using Proguard in an app that integrates SmartDeviceLink, it is necessary to follow these guidelines.

## Required Proguard Rules
Apps that are code shrinking a release build with Proguard typically have a section resembling this snippet in their `build.gradle`:

```java
android {
    buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'),
                    'proguard-rules.pro'
        }
    }
    ...
}
```

Developers using Proguard in this manner should be sure to include the following lines in their `proguard-rules.pro` file:

```java
-keep class com.smartdevicelink.** { *; }
-keep class com.livio.** { *; }
# Video streaming apps must add the following line
-keep class ** extends com.smartdevicelink.streaming.video.SdlRemoteDisplay { *; }
```

!!! note
Failure to include these Proguard rules may result in a failed build or cause issues during runtime.
!!!
