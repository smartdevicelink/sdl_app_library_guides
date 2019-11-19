# Finding Supported Features
New features are always being added to SDL Core, however, you or your users may be connecting to older head units that do not support the newest features. If your SDL app attempts to send an unsupported RPC, your request will simply be ignored by SDL Core.

### How to Handle Unsupported Features
When you are implementing a feature, you should always assume that some of the head units that your users are connecting to will not support the feature or that your user may have disabled permission to use this feature on their head unit. The best way to deal with unsupported features is to handle unsuccessful responses to the requests you send to SDL Core. 

### Checking if a Feature is Supported
When you connect successfully to a head unit, you can get the most current [RPC Spec](https://github.com/smartdevicelink/rpc_spec/blob/master/MOBILE_API.xml) version that both the head unit and your mobile SDL SDK support. If the feature you want to support was added in a version less than or equal to the version returned by the head unit, then your head unit may support the feature.

@![iOS]
##### Objective-C
```objc
SDLMsgVersion *rpcSpecVersion = self.sdlManager.registerResponse.sdlMsgVersion;
```

##### Swift
```swift
let rpcSpecVersion = sdlManager.registerResponse.sdlMsgVersion
```
!@

@![android]
```java
SdlMsgVersion rpcSpecVersion = sdlManager.getRegisterAppInterfaceResponse().getSdlMsgVersion();
```
!@

