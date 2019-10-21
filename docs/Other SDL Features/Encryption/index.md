# Encryption
Some OEMs that you work with may want to encrypt messages being passed between your SDL app and the head unit. When you submit your app to the OEM for review, they will ask you to add a security library to your SDL app. It is also possible to encrypt messages even if the OEM does not require encryption. In this case you will have to work with the OEM in order to get a security library. 

## When Encryption is Needed
### OEM Required Encrypted RPCs
Some OEMs may want to encrypt all RPCs being transmitted between your SDL app and SDL Core. Other OEMS may want to encrypt specific RPCs, such as those that access vehicle data. The library will handle encrypting and decrypting RPCs that the OEM requires be encrypted. 

### OEM Required Encrypted Video and Audio 
Some OEMs might want to encrypt video and audio streaming. Information on how to setup encrypted video and audio streaming can be found in [Video Streaming for Navigation Apps > Introduction](Video Streaming for Navigation Apps/Introduction). The library will handle encrypting the video and audio data sent to the head unit.

### Optional Encryption
You may want to encrypt some or all of the RPCs you send to the head unit even if the OEM does not require that they be protected. In that case you will have to manually configure the payload protection of every RPC that you send. If the OEM does not require that you use a security library, you will have to work with the OEM to get a security library in order for this feature to work. Please note that if you require that an RPC be encrypted but there is no security manager configured for the head unit the app is connected with then the RPC will not be sent by the library. 

### Setting the Encryption Configuration
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
// TODO
```
!@

### Getting the Encryption Status
Since it can take a few moments to setup the encryption manager, you must wait until you know that setup has completed before sending encrypted RPCs. If your RPC is sent before setup has completed, your RPC will not be sent. You can implement the !@[iOS]`SDLServiceEncryptionDelegate`!@@![android,javaSE,javaEE]`ServiceEncryptionListener`!@, which is set in !@[iOS]`SDLEncryptionConfiguration`!@@![android,javaSE,javaEE]`Builder.setSdlSecurity`!@, to get updates to the encryption manager state.

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

### Setting Optional Encryption
If you want to encrypt a specific RPC, you must configure the payload protected status of the RPC before you send it to the head unit.

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