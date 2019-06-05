# Integration Basics
### How SDL Works
SmartDeviceLink works by sending remote procedure calls (RPCs) back and forth between a smartphone application and the SDL Core. These RPCs allow you to build the user interface, detect button presses, play audio, and get vehicle data, among other things. You will use the SDL library to build your app on the SDL Core.

`// TODO: Align with iOS guide more and move SDK configuration content to the SDK configuration guide`

## Getting Started

In this guide, we exclusively use IntelliJ. We are going to set-up a bare-bones application so you get started using SDL.

!!! IMPORTANT
The SDL Java library supports Java 7 and above.
!!!
 

## SmartDeviceLink Service

A SmartDeviceLink Java Service should be created to manage the lifecycle of the SDL session. The `SdlService` should build and start an instance of the `SdlManager` which will automatically connect with a headunit when available. This `SdlManager` will handle sending and receiving messages to and from SDL after connected.

Create a new class and name it appropriately, for this guide we are going to call it `SdlService`. 
 
```java
public class SdlService {
    //...
}
```
 

### Implementing SDL Manager

In order to correctly connect to an SDL enabled head unit developers need to implement methods for the proper creation and disposing of an `SdlManager` in our `SdlService`.

!!! NOTE
An instance of SdlManager cannot be reused after it is closed and properly disposed of. Instead, a new instance must be created. Only one instance of SdlManager should be in use at any given time.
!!!


```java
public class SdlService {

    //The manager handles communication between the application and SDL
    private SdlManager sdlManager = null;

    //...

    private void buildSdlManager(BaseTransportConfig transport) {

        if (sdlManager == null) {

            // The app type to be used
            Vector<AppHMIType> appType = new Vector<>();
            appType.add(AppHMIType.MEDIA);

            // The manager listener helps you know when certain events that pertain to the SDL Manager happen
            SdlManagerListener listener = new SdlManagerListener() {

                @Override
                public void onStart() {
                    // After this callback is triggered the SdlManager can be used to interact with the connected SDL session (updating the display, sending RPCs, etc)
                }

                @Override
                public void onDestroy() {
                }

                @Override
                public void onError(String info, Exception e) {
                }
            };

            // Create App Icon, this is set in the SdlManager builder
            SdlArtwork appIcon = new SdlArtwork(ICON_FILENAME, FileType.GRAPHIC_PNG, ICON_PATH, true);

            // The manager builder sets options for your session
            SdlManager.Builder builder = new SdlManager.Builder(APP_ID, APP_NAME, listener);
            builder.setAppTypes(appType);
            builder.setTransportType(transport);
            builder.setAppIcon(appIcon);
            sdlManager = builder.build();
            sdlManager.start();
        }

    }
}
```

!!! IMPORTANT
The `sdlManager` must be shutdown properly if this class is shutting down in the respective method using the method `sdlManager.dispose()`.
!!!

@![javaEE]
### Adding EJB and Websockets
Create a new package where all the JavaEE-specific code will go. 

The SDL Java library comes with a `CustomTransport` class which takes the role of sending messages between incoming sdl_core connections and your SDL application. You need to pass that class to the `SdlManager` builder to make the SDL Java library aware that you want to use your JavaEE websocket server as the transport.

Create a Java class in the new package which will be the `SDLSessionBean` class. This class utilizes the `CustomTransport` class and EJB JavaEE API which will make it the entry point of your app when a connection is made. It will open up a websocket server at `/` and create stateful beans, where the bean represents the logic of your cloud app. Every new connection to this endpoint creates a new bean containing your app logic, allowing for load balancing across all the instances of your app that were automatically created. 

```java
import com.smartdevicelink.transport.CustomTransport;
import javax.ejb.Stateful;
import javax.websocket.*;
import javax.websocket.server.ServerEndpoint;
import java.io.IOException;
import java.nio.ByteBuffer;

@ServerEndpoint("/")
@Stateful(name = "SDLSessionEJB")
public class SDLSessionBean {

    CustomTransport websocket;

    public class WebSocketEE extends CustomTransport {
        Session session;
        public WebSocketEE(String address, Session session) {
            super(address);
            this.session = session;
        }
        public void onWrite(byte[] bytes, int i, int i1) {
            try {
                session.getBasicRemote().sendBinary(ByteBuffer.wrap(bytes));
            }
            catch (IOException e) {

            }
        }
    }
    
    @OnOpen
    public void onOpen (Session session) {
        websocket = new WebSocketEE("http://localhost", session) {};
        //TODO: pass your CustomTransport instance to your SDL app here
    }

    @OnMessage
    public void onMessage (ByteBuffer message, Session session) {
        websocket.onByteBufferReceived(message); //received message from core
    }
}
```

Unfortunately, [there's no way to get a client's IP address using the standard API](https://stackoverflow.com/a/23025059), so localhost is passed to the CustomTransport for now as the transport address (this is only used locally in the library so it is not necessary). 

The `SDLSessionBean` class’s @OnOpen method is where you will start your app, and should call your entry of your application and invoke whatever is needed to start it. You need to pass the instantiated `CustomTransport` object to your application so that the connection can be passed into the `SdlManager`.

The SdlManager will need you to create a `CustomTransportConfig`, pass in the `CustomTransport` instance from the `SDLSessionBean` instance, then set the `SdlManager` Builder’s transport type to that config. This will set your transport type into `CUSTOM` mode and will use your `CustomTransport` instance to handle the read and write operations.

```java
// Set transport config. builder is a SdlManager.Builder
CustomTransportConfig transport = new CustomTransportConfig(websocket);
builder.setTransportType(transport);
```

!!! IMPORTANT
The `SDLSessionBean` should be inside a Java package other than the default package in order for it to work properly.
!!!

##### Add a New Artifact:

* Right-click project -> Open Module Settings -> Artifacts -> + ->
  Web Application: Archive -> for your war: exploded artifact which should already exist
* Create Manifest. Apply + OK.
* Run Build -> Build Artifacts to get a .war file in the /out folder.
!@

@![javaSE, javaEE]
### Determining SDL Support
You have the ability to determine a minimum SDL protocol and a minimum SDL RPC version that your app supports. We recommend not setting these values until your app is ready for production. The OEMs you support will help you configure the correct `minimumProtocolVersion` and `minimumRPCVersion` during the application review process.

If a head unit is blocked by protocol version, your app icon will never appear on the head unit's screen. If you configure your app to block by RPC version, it will appear and then quickly disappear. So while blocking with `minimumProtocolVersion` is preferable, `minimumRPCVersion` allows you more granular control over which RPCs will be present.


```java
builder.setMinimumProtocolVersion(new Version("3.0.0"));
builder.setMinimumRPCVersion(new Version("4.0.0"));

```

### Listening for RPC notifications and events

We can listen for specific events using `SdlManager`'s builder `setRPCNotificationListeners`. The following example shows how to listen for HMI Status notifications. Additional listeners can be added for specific RPCs by using their corresponding `FunctionID` in place of the `ON_HMI_STATUS` in the following example and casting the `RPCNotification` object to the correct type. 

##### Example of a listener for HMI Status:

```java
Map<FunctionID, OnRPCNotificationListener> onRPCNotificationListenerMap = new HashMap<>();
onRPCNotificationListenerMap.put(FunctionID.ON_HMI_STATUS, new OnRPCNotificationListener() {
    @Override
    public void onNotified(RPCNotification notification) {
        OnHMIStatus onHMIStatus = (OnHMIStatus) notification;
        if (onHMIStatus.getHmiLevel() == HMILevel.HMI_FULL && onHMIStatus.getFirstRun()){
            // first time in HMI Full
        }
    }
});
builder.setRPCNotificationListeners(onRPCNotificationListenerMap);
```
!@

@![javaSE]
### Main Class

Now that the basic connection infrastructure is in place, we should add methods to start the `SdlService` when our application starts. In `main(String[] args)` in your main class, you will create and start an instance of the `SdlService` class.

You will also need to fill in what port the app should listen on for an incoming web socket connection.

```java
public class Main {

	Thread thread;
	SdlService sdlService;
	
    public static void main(String[] args) {
        Main main = new Main();
        main.startSdlService();
    
    }
    
private void startSdlService() {

        thread = new Thread(new Runnable() {

            @Override
            public void run() {
                sdlService  = new SdlService(new WebSocketServerConfig(PORT, -1));
                sdlService.start();


            }
        });
        thread.start();
    }
}
```
!@