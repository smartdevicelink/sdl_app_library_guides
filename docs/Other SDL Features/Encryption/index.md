# Encryption
Some OEMs may want to encrypt messages passed between your SDL app and the head unit. If this is the case, when you submit your app to the OEM for review, they will ask you to add a security library to your SDL app. It is also possible to encrypt messages even if the OEM does not require encryption. In this case, you will have to work with the OEM to get a security library. This section will show you how to add the security library to your SDL app and configure optional encryption.

## When Encryption is Needed
### OEM Required Encrypted RPCs
OEMs may want to encrypt all or some of the RPCs being transmitted between your SDL app and SDL Core. The library will handle encrypting and decrypting RPCs that are required to be encrypted. 

@![iOS,android]
### OEM Required Encrypted Video and Audio 
OEMs may want to encrypt video and audio streaming. Information on how to set up encrypted video and audio streaming can be found in [Video Streaming for Navigation Apps > Introduction](Video Streaming for Navigation Apps/Introduction). The library will handle encrypting the video and audio data sent to the head unit.
!@

### Optional Encryption
You may want to encrypt some or all of the RPCs you send to the head unit even if the OEM does not require that they be protected. In that case you will have to manually configure the payload protection status of every RPC that you send. Please note that if you require that an RPC be encrypted but there is no security manager configured for the connected head unit, then the RPC will not be sent by the library. 

!!! IMPORTANT
For optional encryption to work, you must work with each OEM to obtain their proprietary security library.
!!!

## Creating the Encryption Configuration
Each OEM that supports SDL will have their own proprietary security library. You must add all required security libraries in the encryption configuration when you are configuring the SDL app. 

@![iOS]
##### Objective-C
```objc
SDLEncryptionConfiguration *encryptionConfig = [[SDLEncryptionConfiguration alloc] initWithSecurityManagers:@[OEMSecurityManager.self] delegate:self];
SDLConfiguration *config = [[SDLConfiguration alloc] initWithLifecycle:lifecycleConfig lockScreen:[SDLLockScreenConfiguration enabledConfiguration] logging:[SDLLogConfiguration defaultConfiguration] fileManager:[SDLFileManagerConfiguration defaultConfiguration] encryption:encryptionConfig];
```

##### Swift
```swift
let encryptionConfig = SDLEncryptionConfiguration(securityManagers: [OEMSecurityManager.self], delegate: self)
let config = SDLConfiguration(lifecycle: lifecycleConfig, lockScreen: .enabled(), logging: .default(), fileManager: .default(), encryption: encryptionConfig)
```
!@

@![android,javaSE,javaEE]
```java
List<Class<? extends SdlSecurityBase>> secList = new ArrayList<>();
secList.add(OEMSdlSecurity.class);
builder.setSdlSecurity(secList, <# Optional serviceEncryptionListener>);
```
!@

@![javascript]
Encryption will be supported in a future release.
!@

## Getting the Encryption Status
Since it can take a few moments to set up the encryption manager, you must wait until you know that setup has completed before sending encrypted RPCs. If your RPC is sent before setup has completed, your RPC will not be sent. You can implement the @![iOS]`SDLServiceEncryptionDelegate`!@@![android,javaSE,javaEE,javascript]`ServiceEncryptionListener`!@, which is set in @![iOS]`SDLEncryptionConfiguration`!@@![android,javaSE,javaEE,javascript]`Builder.setSdlSecurity`!@, to get updates to the encryption manager state.

@![iOS]
##### Objective-C
```objc
- (void)serviceEncryptionUpdatedOnService:(SDLServiceType)type encrypted:(BOOL)encrypted error:(nullable NSError *)error {
    if (encrypted) {
        <#Encryption manager can encrypt#>
    }
}
```

##### Swift
```swift
func serviceEncryptionUpdated(serviceType type: SDLServiceType, isEncrypted encrypted: Bool, error: Error?) {
    if encrypted {
        <#Encryption manager can encrypt#>
    }
}
```
!@

@![android,javaSE,javaEE]
```java
ServiceEncryptionListener serviceEncryptionListener = new ServiceEncryptionListener() {
	@Override
	public void onEncryptionServiceUpdated(@NonNull SessionType serviceType, boolean isServiceEncrypted, @Nullable String error) {
		if (isServiceEncrypted) {
			// Encryption manager can encrypt
		}
	}
};
```
!@

@![javascript]
Encryption will be supported in a future release.
!@

## Setting Optional Encryption
If you want to encrypt a specific RPC, you must configure the payload protected status of the RPC before you send it to the head unit. In order to send RPCs with optional encryption you must call `startRPCEncryption` on the `sdlManager` to make sure the encryption manager gets started correctly. The best place to put `startRPCEncryption` is in the successful callback of @![iOS]`startWithReadyHandler`.!@@![android,javaSE,javaEE,javascript]the `SdlManagerListener`'s `onStart` method.!@

@![iOS]
##### Objective-C
```objc
[self.sdlManager startRPCEncryption];
```

##### Swift
```swift
sdlManager.startRPCEncryption()
```
!@

@![android,javaSE,javaEE]
```java
sdlManager.startRPCEncryption();
```
!@

@![javascript]
Encryption will be supported in a future release.
!@

Then, once you know the encryption manager has started successfully via encryption manager state updates to your @![iOS]`SDLServiceEncryptionDelegate`!@@![android,javaSE,javaEE,javascript]`ServiceEncryptionListener`!@ object, you can start to send encrypted RPCs by setting @![iOS]`payloadProtected`!@@![android,javaSE,javaEE,javascript]`setPayloadProtected`!@ to `true`.

@![iOS]
##### Objective-C
```objc
SDLGetVehicleData *getVehicleData = [[SDLGetVehicleData alloc] init];
getVehicleData.gps = @YES;
getVehicleData.payloadProtected = @YES;

[self.sdlManager sendRequest:getVehicleData];
```

##### Swift
```swift
let getVehicleData = SDLGetVehicleData()
getVehicleData.gps = true as NSNumber
getVehicleData.isPayloadProtected = true

sdlManager.send(getVehicleData)
```
!@

@![android,javaSE,javaEE]
```java
GetVehicleData getVehicleData = new GetVehicleData();
getVehicleData.setGps(true);
getVehicleData.setPayloadProtected(true);

sdlManager.sendRPC(getVehicleData);
```
!@

@![javascript]
Encryption will be supported in a future release.
!@
