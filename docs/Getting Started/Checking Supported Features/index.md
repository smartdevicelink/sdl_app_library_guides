# Checking Supported Features
New features are always being added to SDL, however, you or your users may be connecting to modules that do not support the newest features. If your SDL app attempts to use an unsupported feature your request will be ignored by the module.

When you are implementing a feature you should always assume that some modules your users are connecting to will not support the feature or that your user may have disabled your permission to use this feature on their head unit. The best way to deal with unsupported features is to check if the feature is available before attempting to use it and to handle error responses.

### Checking the System Capability Manager
The easiest way to check if a feature is supported is to query the library's System Capability Manager. For more details on how get this information please see the [Adaptive Interface Capabilities](Displaying a User Interface/Adaptive Interface Capabilities) guide.

#### Handling Error Responses
When you are trying to use a feature, you can watch for an error response to the request you sent to the module. If the response contains an error, you may be able to check the `result` enum to determine if the feature is disabled. If the response that comes back is of the type `GenericResponse`, the module doesn't understand your request.

@![iOS]
##### Objective-C
```objc
[self.sdlManager sendRequest:<#Your Request#> withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if (response == nil || !response.success) {
        <#The request was not successful, check the error and response.resultCode for more information#>
        return;
    }

    <#The request was successful#>
}];
```

##### Swift
```swift
sdlManager.send(request: <#Your Request#>) { (request, response, error) in
    guard let response = response, !response.success.boolValue else {
        <#The request was not successful, check the error and response.resultCode for more information#>
        return
    }

    <#The request was successful#>
}
```
!@

@![android,javaSE,javaEE]
```java
<#Your Request#>.setOnRPCResponseListener(new OnRPCResponseListener() {
	@Override
	public void onResponse(int correlationId, RPCResponse response) {
		if (!response.getSuccess()) {
			<#The request was not successful, check the onError handler for more information#>
            return;
		}

        <#The request was successful#>
	}

	@Override
	public void onError(int correlationId, Result resultCode, String info) {
		<#The request was not successful, check the resultCode and info for more information#>
	}
});
sdlManager.sendRPC(<#Your Request#>);
```
!@

### Checking if a Feature is Supported
When you connect successfully to a head unit, SDL will automatically negotiate the maximum SDL RPC version supported by both the module and your SDL SDK. If the feature you want to support was added in a version less than or equal to the version returned by the head unit, then your head unit may support the feature. Remember that the module may still disable the feature, or the user may still have disabled permissions for the feature in some cases. It's best to check if the feature is supported through the System Capability Manager first, but you may also check the negotiated version to know if the head unit was built before the feature was designed.

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
