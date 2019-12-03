# Checking Supported Features
New features are always being added to SDL, however, you or your users may be connecting to modules that do not support the newest features. If your SDL app attempts to use an unsupported feature, your request will be ignored.

### How to Handle Unsupported Features
When you are implementing a feature, you should always assume that some modules your users are connecting to will not support the feature or that your user may have disabled your permission to use this feature on their head unit. The best way to deal with unsupported features is to check if the feature is available before attempting to use it and to handle error responses.

### Checking if a Feature is Supported
When you connect successfully to a head unit, SDL will automatically negotiate the maximum SDL RPC version supported by both the module and your SDL SDK. If the feature you want to support was added in a version less than or equal to the version returned by the head unit, then your head unit may support the feature. Remember that the module may still disable the feature, or the user may still have disabled permissions for the feature in some cases.

Throughout these guides you may see headers that contain text like "RPC 6.0+". That means that if the negotiated version is 6.0 or greater, then SDL supports the feature but the above caveats may still apply.

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
