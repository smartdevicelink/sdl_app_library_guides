# Installation
In order to build your app on a SmartDeviceLink (SDL) Core, the SDL software development kit (SDK) must be installed in your app. The following steps will guide you through adding the SDL SDK to your workspace and configuring the environment.

!!! NOTE
The SDL SDK is currently supported on @![iOS]iOS 8.0!@@![android]Android 2.2 (Froyo)!@@![javaSE, javaEE]Java 7 (1.7)!@@![javascript]Node 9.11.2!@ and above.
!!!

## Install SDL SDK
@![iOS]
There are four different ways to install the SDL SDK in your project: Accio, CocoaPods, Carthage, or manually.

### CocoaPods Installation

1\. Xcode should be closed for the following steps.

2\. Open the terminal app on your Mac.

3\. Make sure you have the latest version of [CocoaPods](https://cocoapods.org) installed. For more information on installing CocoaPods on your system please consult: [https://cocoapods.org](https://cocoapods.org).

```
sudo gem install cocoapods
```

4\. Navigate to the root directory of your app. Make sure your current folder contains the **.xcodeproj** file.

5\. Create a new **Podfile**.

```
pod init
```

6\. In the **Podfile**, add the following text. This tells CocoaPods to install SDL SDK for iOS. SDL Versions are available on [Github](https://github.com/smartdevicelink/sdl_ios/releases). We suggest always using the latest release.

```ruby
target ‘<#Your Project Name#>’ do
    pod ‘SmartDeviceLink’, ‘~> <#SDL Version#>’
end
```
    
7\. Install SDL SDK for iOS: 

```
pod install
```

8\. There will be a newly created **.xcworkspace** file in the directory in addition to the **.xcodeproj** file. Always use the **.xcworkspace** file from now on.

9\. Open the **.xcworkspace** file. To open from the  terminal, type:  

```
open <#Your Project Name#>.xcworkspace
```

### Accio Installation
You can install this library using [Accio](https://github.com/JamitLabs/Accio), which is based on SwiftPM syntax. Please follow the steps on the Accio README linked above to initialize Accio into your application. Once installed and initialized into your Xcode project, the root directory should contain a Package.swift file.

1\. Open the Package.swift file.

2\. Add the following line to the dependencies array of your package file. We suggest always using the latest release of the SDL library. 

```swift
.package(url: "https://github.com/smartdevicelink/sdl_ios.git", .upToNextMajor(from: "<#SDL Version#>")),
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
You cannot submit your app to the app store with the framework as is. You MUST strip the simulator part of the framework first.
!!!

You can check the architectures of your built framework like so:

```bash
lipo -info SmartDeviceLink.framework/SmartDeviceLink
```

Use a script like this to strip the simulator part of the framework.

```bash
lipo -remove i386 -remove x86_64 -o SmartDeviceLink.framework/SmartDeviceLink SmartDeviceLink.framework/SmartDeviceLink
```
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

To compile release 4.12.0, use the following line:

```
dependencies {
    implementation 'com.smartdevicelink:sdl_android:4.12.0'
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

To compile release 4.12.0, use the following line:

```
dependencies {
    implementation 'com.smartdevicelink:sdl_java_se:4.12.0'
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

@![android, javaSE, javaEE]
To Find more information on this pleace read our [README](https://github.com/smartdevicelink/sdl_java_suite).
!@

@![javascript]
You can find the most recent release of the SDL JavaScript Suite [here](https://github.com/smartdevicelink/sdl_javascript_suite/releases). The project comes with prebuilt bundles of the library in the form of `SDL.min.js` files. There is a vanilla JavaScript distribution of the library as well as one for Node.js. They are located in the `lib/js/dist` and `lib/node/dist` directories respectively. 

## Vanilla JavaScript Setup
This build allows you to create apps that run on the browser. In order to have the built JS file be imported into your HTML, you'll need to run a simple web server that can serve that JS file. We will be using [Node.js](https://nodejs.org/en/) and [npm](https://docs.npmjs.com/about-npm/) for this task, but you can use any software that lets you serve HTML and JS to the browser.

In a new directory, save your `SDL.min.js` file there, and then run the following: `npm init`. Press Enter repeatedly through the prompts to set up some default npm configuration. Then, install the `express` package by running `npm install express --save`. Express is a popular Node.js package that allows easy setup of a server. Then, create a `index.js` file in the same directory, and save the following to it:

```js
const express = require('express');
const app = express();
app.use(express.static(__dirname));
const PORT = process.env.PORT || 3000;
const server = app.listen(PORT, async function () {
    console.log('Server running on port', PORT);
});
```

This code will start up a server on port 3000 on localhost, and serve any files in the directory it is running in. Now that the server code is complete, create a new file named `index.html` that will contain the app logic and save it in the same directory. Make the contents of the HTML file the following:

```html
<html>
    <head>
        <script src='./SDL.min.js'></script>
    </head>
    <body>
        <script>
            console.log(SDL);
        </script>
    </body>
</html>
```

Finally, run the server by running this in a Terminal window in the same directory: `node index.js`. The console should print `Server running on port 3000`. Go to your browser and enter `localhost:3000` in the address bar. If you open up your browser's console on the blank page that shows up you should see the SDL library version be printed and the imported SDL object. If so, then you have successfully set up SDL in your browser! If not, then make sure your SDL build name is correct, and that the HTML file, build file, and JS server file are all in the same directory. If you still have  questions, ask them in the javascript-suite-help channel in the SDL Slack. You can sign up for our Slack [here](https://slack.smartdevicelink.com/).

## Node.js Setup
This build allows you to create apps that run through a server on your computer. You will need to have installed [Node.js](https://nodejs.org/en/) and [npm](https://docs.npmjs.com/about-npm/) are before beginning work on this app.

In a new directory, save your `SDL.min.js` file, then create a new file in the same directory named `index.js`. Make the contents of that file the following:

```js
const SDL = require('./SDL.min.js');
console.log(SDL);
```

Finally, run the file by entering the following in a Terminal window in the same directory: `node index.js`. You should see the SDL library version and the imported SDL object be printed to the console. If so, then you have successfully set up the SDL library!

## WebEngine App Setup
WebEngine apps use the vanilla JavaScript build, and are set up in a similar fashion to those JS apps where it will also run in the browser. Set up the `index.js` and `index.html` file like in the **Vanilla JavaScript Setup** section. The majority of the configuration for the app will now be separated into a `manifest.js` file and then imported into the `index.html` file. Create a `manifest.js` file like below and save it in the same directory as the other two files. The entrypoint's value should have the same name as your app's HTML file. 

```js
export default {
    "entrypoint": "./index.html",
    "appIcon": "./app_icon.png",
    "appId": "hello-webengine",
    "appName": "Hello WebEngine",
    "category": "DEFAULT",
    "additionalCategories": [],
    "locales": {
        "de_DE": {
            "appName": "Hallo JS",
            "appIcon": "./app_icon.png"
        }
    },
    "appVersion": "1.0.0",
    "minRpcVersion": "6.0",
    "minProtocolVersion": "5.0"
};
```

Note that the manifest file is using the import/export module syntax; in the HTML file it should be imported in as a module:

```js
import sdl_manifest from './manifest.js';
```

In the transport configuration the parameters for `WebSocketClientConfig` will be empty. For WebEngine apps those connection details are expected as query parameters in the URL. See below for an example of what the URL is expected to be once the server is running. `sdl-host` is the location of the SDL Core WebSocket server. `sdl-port` is the port of that WebSocket server. `sdl-transport-role` refers to SDL Core's role, which is as a server (as opposed to a client).

```
http://localhost:3000/?sdl-host=HOST&sdl-transport-role=ws-server&sdl-port=PORT
```

!@
