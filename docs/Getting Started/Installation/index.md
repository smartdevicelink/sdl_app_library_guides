# Installation
In order to build your app on a SmartDeviceLink (SDL) Core, the SDL software development kit (SDK) must be installed in your app. The following steps will guide you through adding the SDL SDK to your workspace and configuring the environment.

!!! NOTE
The SDL SDK is currently supported on @![iOS]iOS 8.0!@@![android]Android 2.2 (Froyo)!@@![javaSE, javaEE]Java 7 (1.7)!@ and above.
!!!

## Install SDL SDK
@![iOS]
There are four different ways to install the SDL SDK in your project: Accio, CocoaPods, Carthage, or manually.

### CocoaPods Installation

1. Xcode should be closed for the following steps.
1. Open the terminal app on your Mac.
1. Make sure you have the latest version of [CocoaPods](https://cocoapods.org) installed. For more information on installing CocoaPods on your system please consult: [https://cocoapods.org](https://cocoapods.org).

        sudo gem install cocoapods

1. Navigate to the root directory of your app. Make sure your current folder contains the **.xcodeproj** file
1. Create a new **Podfile**.

        pod init

1. In the **Podfile**, add the following text. This tells CocoaPods to install SDL SDK for iOS. SDL Versions are available on [Github](https://github.com/smartdevicelink/sdl_ios/releases). We suggest always using the latest release.

        target ‘<#Your Project Name#>’ do
                pod ‘SmartDeviceLink’, ‘~> <#SDL Version#>’
        end
    
1. Install SDL SDK for iOS: 

        pod install

1. There will be a newly created **.xcworkspace** file in the directory in addition to the **.xcodeproj** file. Always use the **.xcworkspace** file from now on.
1. Open the **.xcworkspace** file. To open from the  terminal, type:  

        open <#Your Project Name#>.xcworkspace

### Accio Installation
You can install this library using [Accio](https://github.com/JamitLabs/Accio), which is based on SwiftPM syntax. Please follow the steps on the Accio README linked above to initialize Accio into your application. Once installed and initialized into your Xcode project, the root directory should contain a Package.swift file.

1\. Open the Package.swift file.

2\. Add the following line to the dependencies array of your package file. We suggest always using the latest release of the SDL library. 

```swift
.package(url: "https://github.com/smartdevicelink/sdl_ios.git", .upToNextMajor(from: "6.4.0")),
```

!!! NOTE
Please see [package manifest format](https://github.com/apple/swift-package-manager/blob/master/Documentation/PackageDescription.md) to specify dependencies to a specific branch / version of SDL.
!!!

3\. Add `"SmartDeviceLink"` or `"SmartDeviceLinkSwift"` to the dependencies array in your target. Use `"SmartDeviceLink"`for Objective-C applications and `"SmartDeviceLinkSwift"` for Swift applications.
            
4\.  Install the SDK by running `accio install` in the root folder of your project in Terminal.

### Carthage Installation
SDL iOS supports Carthage! Install using Carthage by following [this guide](https://github.com/Carthage/Carthage#adding-frameworks-to-an-application).

### Manual Installation
Tagged to our releases is a dynamic framework file that can be drag-and-dropped into the application. 

!!! NOTE
You cannot submit your app to the app store with the framework as is. You MUST strip the simulator part of the framework first. Use a script such as Carthage's to accomplish this.
!!!

!@

@![android]
Each [SDL Android](https://github.com/smartdevicelink/sdl_java_suite) library release is published to JCenter. By adding a few lines in their app's gradle script, developers can compile with the latest SDL Android release.

To gain access to the JCenter repository, make sure your app's `build.gradle` file includes the following:

```
repositories {
    jcenter()
}
```

### Gradle Build

To compile with the a release of SDL Android, include the following line in your app's `build.gradle` file,

```
dependencies {
    implementation 'com.smartdevicelink:sdl_android:{version}'
}
```

and replace `{version}` with the desired release version in format of `x.x.x`. The list of releases can be found [here](https://github.com/smartdevicelink/sdl_java_suite/releases). 

### Examples

To compile release 4.10.1, use the following line:

```
dependencies {
    implementation 'com.smartdevicelink:sdl_android:4.10.1'
}
```

To compile the latest minor release of major version 4, use:

```
dependencies {
    implementation 'com.smartdevicelink:sdl_android:4.+'
}
```
!@

@![javaSE]
Each [SDL JavaSE](https://github.com/smartdevicelink/sdl_java_suite) library release is published to JCenter. By adding a few lines in their app's gradle script, developers can compile with the latest SDL JavaSE release.

To gain access to the JCenter repository, make sure your app's `build.gradle` file includes the following:

```
repositories {
    google()
    jcenter()
}
```

### Gradle Build

To compile with the a release of SDL JavaSE, include the following line in your app's `build.gradle` file,

```
dependencies {
    implementation 'com.smartdevicelink:sdl_java_se:{version}'
}
```

and replace `{version}` with the desired release version in format of `x.x.x`. The list of releases can be found [here](https://github.com/smartdevicelink/sdl_java_suite/releases). 

### Examples

To compile release 4.10.1, use the following line:

```
dependencies {
    implementation 'com.smartdevicelink:sdl_java_se:4.10.1'
}
```

To compile the latest minor release of major version 4, use:

```
dependencies {
    implementation 'com.smartdevicelink:sdl_java_se:4.+'
}
```
!@

@![javaEE]
Each SDL JavaEE library release is published to [Github](https://github.com/smartdevicelink/sdl_java_suite). By building and importing the library JAR file to the project, developers can compile with the latest SDL JavaEE release. In this guide we exclusively use IntelliJ to compile and build the project.

### Building The JavaEE Library JAR    
To build the library JAR from the source code, first clone the [SDL Java Suite](https://github.com/smartdevicelink/sdl_java_suite) repository then, simply call:

```
gradle build
```

from within the JavaEE directory and a JAR should be generated in the build/libs folder.

### Creating a New SDL Project
* [Download Glassfish 5.0.0 Full Platform](https://javaee.github.io/glassfish/download)
* Start a new IntelliJ Project (Menu -> New -> Project)
* Select Java Enterprise as project type.
* Ensure Java EE 8 is the version used.
* Set Application Server to the directory of Glassfish 5.0.0 download.
* Check the following: (they should all be using the libraries from Glassfish)
    * Glassfish 5.0.0 - EJB
    * Glassfish 5.0.0 - WebSocket
    * Glassfish 5.0.0 - Web Application
* Give the project a name.
* Once the project is created, add the SDL Java JAR library
  (Right-click project -> Open Module Settings -> Libraries -> +)
* Go to artifacts. A problem may appear concerning the exploded war not having the library. 
  Add SDL Java to the war exploded artifact (IntelliJ has an auto fix for it).
  Apply and OK.
* In the artifacts ->  choose ejb exploded -> in the "output layout" tab, click on the + -> Extracted directory ->  add the SDL Java JAR library
* In the artifacts ->  choose war exploded -> in the "output layout" tab, click on the + -> Extracted directory ->  add the SDL Java JAR library
* You can now start creating your SDL enabled application. If you already have source code to start with, you can copy it into the new project along with any jars and assets.

!!! NOTE
Glassfish 5.0.0 only works on JDK 8 and lower.
!!!
!@
