# Multiple Transports (Protocol v5.1+)
The multiple transports feature allows apps to carry their SDL session over multiple transports. The first transport that the app connects with is referred to as the primary transport and a transport connected at a later point is the secondary transport. For example, apps can register over Bluetooth or USB as a primary transport, then connect over WiFi when necessary (ex. to allow video/audio streaming) as a secondary transport. This feature is supported on connections with protocol version 5.1+, which is supported on SDL @![android]Android 4.7+!@@![iOS]iOS 6.1+!@ and SDL Core 5.0+.

## Primary Transports
On head units that support multiple transports, the primary transport will be used for RPC communication while the secondary transport will be used for high bandwidth services such as streaming video data for navigation applications. If no high-bandwidth secondary transport is present, the primary transport will be used for all needed services that the transport supports.

@![iOS]
The only primary transport available for iOS in production applications is iAP. 
!@

@![android]
### Supporting specific primary transports
Whether your app supports both Bluetooth and/or USB connections is determined by what you set as acceptable primary transports. By default, both USB and Bluetooth are supported and should be kept unless there is a specific reason otherwise. If you list multiple primary transports and one disconnects, if another included transport is available the app will automatically attempt to connect and register to it.

```java
List<TransportType> multiplexPrimaryTransports = Arrays.asList(TransportType.USB, TransportType.BLUETOOTH);
MultiplexTransportConfig mtc = new MultiplexTransportConfig(this, APP_ID, MultiplexTransportConfig.FLAG_MULTI_SECURITY_OFF);
mtc.setPrimaryTransports(multiplexPrimaryTransports);
```

If you only want to use Bluetooth or USB, simply pass in a list with the one you want.

!!! Note
For the best compatibility we suggest supporting both primary transports.
!!!

### Requiring High Bandwidth

Certain app types will require a high bandwidth transport to be available, which could be either primary or secondary transports. If this is the case, an app will only be registered if a high bandwidth transport is either connected or available to connect.

If this is the case for your app you can set the `setRequiresHighBandwidth` flag to `true`:

```java
MultiplexTransportConfig mtc = new MultiplexTransportConfig(this, APP_ID, MultiplexTransportConfig.FLAG_MULTI_SECURITY_OFF);

mtc.setRequiresHighBandwidth(true);
```

### High bandwidth app with low bandwidth support

While some app's main integration  requires high bandwidth, it is possible to support a low bandwidth integration for better visibility. As an example, a navigation app might require high bandwidth transport to stream their map view but could provide a low bandwidth integration that displays turn-by-turn directions. Another simple low bandwidth integration could simply be displaying a message that instructs the user to connect USB or WiFi to enable the app. In this case the app should set the requires high bandwidth flag to false, as it is by default.


```java
MultiplexTransportConfig mtc = new MultiplexTransportConfig(this, APP_ID, MultiplexTransportConfig.FLAG_MULTI_SECURITY_OFF);

mtc.setRequiresHighBandwidth(false);
```
!@

## Secondary Transports

Secondary transports must be enabled by the module to which the app is connecting. @![android]In addition to Bluetooth and USB (which are primary transports), TCP over WiFi is a supported secondary transport.!@@![iOS]TCP over WiFi can be configured as a supported secondary transport.!@

@![android]
Setting secondary transports that your app supports is similar to setting the primary transports:

```java
List<TransportType> multiplexPrimaryTransports = Arrays.asList(TransportType.USB, TransportType.BLUETOOTH);
List<TransportType> multiplexSecondaryTransports = Arrays.asList(TransportType.TCP, TransportType.USB, TransportType.BLUETOOTH);
MultiplexTransportConfig mtc = new MultiplexTransportConfig(this, APP_ID, MultiplexTransportConfig.FLAG_MULTI_SECURITY_OFF);
mtc.setPrimaryTransports(multiplexPrimaryTransports);
mtc.setSecondaryTransports(multiplexSecondaryTransports);
```
By default, all three transports are set as supported secondary transports.!@

@![iOS]
By default, TCP is a configured secondary transport, but this can be disabled.

##### Objective-C
```objc
SDLLifecycleConfiguration *lifecycleConfig = [SDLLifecycleConfiguration defaultConfigurationWithAppName:<#AppName#> fullAppId:<#AppID#>];
lifecycleConfig.allowedSecondaryTransports = SDLSecondaryTransportsNone;
```
##### Swift
```swift
let lifecycleConfig = SDLLifecycleConfiguration(appName: <#AppName#>, fullAppId: <#AppID#>)
lifecycleConfiguration.allowedSecondaryTransports = []
```
!@

