# Calling a Phone Number
The @![iOS]`SDLDialNumber`!@@![android,javaSE,javaEE]`DialNumber`!@ RPC allows you make a phone call via the user's phone. Regardless of platform (Android or iOS), you must be sure that a device is connected via Bluetooth (even if using USB) for this RPC to work. If the phone is not connected via Bluetooth, you will receive a result of `REJECTED` from Core.

!!! note
@![iOS]`SDLDialNumber`!@@![android,javaSE,javaEE]`DialNumber`!@ is an RPC that is usually restricted by OEMs. As a result, the OEM you are connecting to may limit app functionality if not approved for usage.
!!!

## Checking if Dial Number is Available
@![iOS]`SDLDialNumber`!@@![android,javaSE,javaEE]`DialNumber`!@ is a newer RPC, so there is a possibility that not all head units will support it. To find out if the RPC is supported by the head unit, check the system capability manager's @![iOS]`hmiCapabilities.phoneCall`!@ @![android,javaSE,javaEE]`hmiCapabilities.isPhoneCallAvailable()`!@ property after the manager has been started successfully.

@![iOS]
##### Objective-C
```objc
[self.sdlManager.systemCapabilityManager updateCapabilityType:SDLSystemCapabilityTypePhoneCall completionHandler:^(NSError * _Nullable error, SDLSystemCapabilityManager * _Nonnull systemCapabilityManager) {
    BOOL isDialNumberSupported = NO;
    if (error == nil) {
        isDialNumberSupported = systemCapabilityManager.phoneCapability.dialNumberEnabled.boolValue;
    }
    else {
        isDialNumberSupported = systemCapabilityManager.hmiCapabilities.phoneCall.boolValue;
    }

    <#If making phone calls is supported, send the `DialNumber` RPC#>
}];
```

##### Swift
```swift
sdlManager.systemCapabilityManager.updateCapabilityType(.phoneCall) { (error, systemCapabilityManager) in
    var isDialNumberSupported = false
    if error == nil {
        isDialNumberSupported = systemCapabilityManager.phoneCapability?.dialNumberEnabled?.boolValue ?? false;
    } else {
        isDialNumberSupported = systemCapabilityManager.hmiCapabilities?.phoneCall?.boolValue ?? false
    }

    <#If making phone calls is supported, send the `DialNumber` RPC#>
}
```
!@

@![android,javaSE,javaEE]
```java
sdlManager.getSystemCapabilityManager().getCapability(SystemCapabilityType.PHONE_CALL, new OnSystemCapabilityListener() {
	@Override
	public void onCapabilityRetrieved(Object capability) {
		PhoneCapability phoneCapability = (PhoneCapability) capability;
		Boolean isDialNumberSupported = ((PhoneCapability) capability).getDialNumberEnabled().booleanValue();
		// If making phone calls is supported, send the `DialNumber` RPC
	}

	@Override
	public void onError(String info) {
		// Perform a fallback check because the module does not support the phone capability
		isPhoneCallAvailable();
	}
}, false);

private void isPhoneCallAvailable() {
	HMICapabilities hmiCapabilities = (HMICapabilities)sdlManager.getSystemCapabilityManager().getCapability(SystemCapabilityType.HMI, new OnSystemCapabilityListener() {
		@Override
		public void onCapabilityRetrieved(Object capability) {
			HMICapabilities hmiCapabilities = (HMICapabilities) capability;
			Boolean isDialNumberSupported = hmiCapabilities.isPhoneCallAvailable();
			// If making phone calls is supported, send the `DialNumber` RPC
		}

		@Override
		public void onError(String info) {
			// DialNumber is not supported
		}
	}, false);
}
```
!@

## Sending a DialNumber Request
!!! note
`DialNumber` strips all characters except for `0`-`9`, `*`, `#`, `,`, `;`, and `+`.
!!!

@![iOS]
##### Objective-C
```objc
SDLDialNumber *dialNumber = [[SDLDialNumber alloc] initWithNumber: @"1238675309"];

[self.sdlManager sendRequest:dialNumber withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if (error != nil || ![response isKindOfClass:SDLDialNumberResponse.class]) {
        <#Encountered error sending DialNumber#>
        return;
    }

    SDLDialNumberResponse* dialNumber = (SDLDialNumberResponse *)response;
    SDLResult *resultCode = dialNumber.resultCode;
    if (!resultCode.success.boolValue) {
		if ([resultCode isEqualToEnum:SDLResultRejected]) {
	        <#DialNumber was rejected. Either the call was sent and cancelled or there is no device connected#>
	    } else if ([resultCode isEqualToEnum:SDLResultDisallowed]) {
	        <#Your app is not allowed to use DialNumber#>
	    } else { 	
	    	<#Some unknown error has occurred#>
	    }
	    return;
    }

	<#DialNumber successfully sent#>
}];
```

##### Swift
```swift
let dialNumber = SDLDialNumber(number: "1238675309")

sdlManager.send(request: dialNumber) { (request, response, error) in
    guard let response = response as? SDLDialNumberResponse, error == nil else {
        <#Encountered error sending DialNumber#>
        return
    }

    guard response?.success.boolValue == true else {
        switch response.resultCode {
        case .rejected:
            <#DialNumber was rejected. Either the call was sent and cancelled or there is no device connected#>
        case .disallowed:
            <#Your app is not allowed to use DialNumber#>
        default:
            <#Some unknown error has occurred#>
        }
        return
    }

    <#DialNumber successfully sent#>
}
```
!@

@![android,javaSE,javaEE]
```java
DialNumber dialNumber = new DialNumber();
dialNumber.setNumber("1238675309");
dialNumber.setOnRPCResponseListener(new OnRPCResponseListener() {
    @Override
    public void onResponse(int correlationId, RPCResponse response) {
        Result result = response.getResultCode();
        if(result.equals(Result.SUCCESS)){
            // `DialNumber` was successfully sent, and a phone call was initiated by the user.
        }else if(result.equals(Result.REJECTED)){
            // `DialNumber` was sent, and a phone call was cancelled by the user. Also, this could mean that there is no phone connected via Bluetooth.
        }else if(result.equals(Result.DISALLOWED)){
            // Your app does not have permission to use DialNumber.
        }
    }

    @Override
    public void onError(int correlationId, Result resultCode, String info){
        Log.e(TAG, "onError: "+ resultCode+ " | Info: "+ info );
    }
});

sdlManager.sendRPC(dialNumber);
```
!@

### DialNumber Result
`DialNumber` has 3 possible results that you should expect:

1. SUCCESS - DialNumber was successfully sent, and a phone call was initiated by the user.
2. REJECTED - DialNumber was sent, and a phone call was cancelled by the user. Also, this could mean that there is no phone connected via Bluetooth.
3. DISALLOWED - Your app does not have permission to use DialNumber.
