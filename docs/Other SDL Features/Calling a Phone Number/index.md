# Calling a Phone Number
Dialing a Phone Number allows you to send a phone number to dial on the user's phone. Regardless of platform (Android or iOS), you must be sure that a device is connected via Bluetooth (even if using USB) for this RPC to work. If not connected to Bluetooth, you will receive a REJECTED @![iOS]`resultCode`!@ @![android,javaSE,javaEE] `Result`!@.

!!! note
DialNumber is an RPC that is usually restricted by OEMs. As a result, the OEM you are connecting to may limit app functionality if not approved for usage.
!!!

## Detecting if DialNumber is Available

`DialNumber` is a newer RPC, so there is a possibility that not all head units will support it. To see if `DialNumber` is supported, you may look at
@![iOS]`SDLManager`'s `systemCapabilityManager.hmiCapabilities.phoneCall` property after the ready handler is called !@ @![android, javaSE, javaEE] the `HMICapabilities` that can be retrieved using `SystemCapabilityManager`.!@.

@![iOS]
##### Objective-C
```objc
BOOL isPhoneCallSupported = NO;

[self.sdlManager startWithReadyHandler:^(BOOL success, NSError * _Nullable error) {
    if (!success) {
        NSLog(@"SDL encountered an error starting up: %@", error);
        return;
    }

    SDLHMICapabilities *hmiCapabilities = self.sdlManager.systemCapabilityManager.hmiCapabilities;
    if (hmiCapabilities != nil) {
        isPhoneCallSupported = hmiCapabilities.phoneCall.boolValue;
    }
}];
```

##### Swift
```swift
var isPhoneCallSupported = false

sdlManager.start { (success, error) in
    if !success {
        print("SDL errored starting up: \(error.debugDescription)")
        return
    }

    if let hmiCapabilities = self.sdlManager.systemCapabilityManager.hmiCapabilities, let phoneCallsSupported = hmiCapabilities.phoneCall?.boolValue {
        isPhoneCallSupported = phoneCallsSupported
    }
}
```
!@

@![android, javaSE, javaEE]
```java
HMICapabilities hmiCapabilities = (HMICapabilities) sdlManager.getSystemCapabilityManager().getCapability(SystemCapabilityType.HMI);
if(hmiCapabilities.isPhoneCallAvailable()){
    // DialNumber supported
}else{
    // DialNumber is not supported
}
```
!@

## Sending a DialNumber Request
!!! note
For DialNumber, all characters are stripped except for `0`-`9`, `*`, `#`, `,`, `;`, and `+`
!!!

@![iOS]
##### Objective-C
```objc
SDLDialNumber *dialNumber = [[SDLDialNumber alloc] init];
dialNumber.number = @"1238675309";

[self.sdlManager sendRequest:dialNumber withResponseHandler:^(__kindof SDLRPCRequest * _Nullable request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error) {
    if (error != nil || ![response isKindOfClass:SDLDialNumberResponse.class]) {
        NSLog(@"Encountered Error sending DialNumber: %@", error);
        return;
    }

    SDLDialNumberResponse* dialNumber = (SDLDialNumberResponse *)response;
    SDLResult *resultCode = dialNumber.resultCode;
    if (![resultCode isEqualToEnum:SDLResultSuccess]) {
		if ([resultCode isEqualToEnum:SDLResultRejected]) {
	        NSLog(@"DialNumber was rejected. Either the call was sent and cancelled or there is no device connected");
	    } else if ([resultCode isEqualToEnum:SDLResultDisallowed]) {
	        NSLog(@"Your app is not allowed to use DialNumber");
	    } else { 	
	    	NSLog(@"Some unknown error has occurred!");
	    }
	    return;
    }

	// Successfully sent!
}];
```

##### Swift
```swift
let dialNumber = SDLDialNumber()
dialNumber.number = "1238675309"

sdlManager.send(request: dialNumber) { (request, response, error) in
    guard let response = response as? SDLDialNumberResponse else { return }
    
    if let error = error {
        print("Encountered Error sending DialNumber: \(error)")
        return
    }
    
    if response.resultCode != .success {
        if response.resultCode == .rejected {
            print("DialNumber was rejected. Either the call was sent and cancelled or there is no device connected")
        } else if response.resultCode == .disallowed {
            print("Your app is not allowed to use DialNumber")
        } else {
            print("Some unknown error has occurred!")
        }
        return
    }
    
    // Successfully sent!
}
```
!@

@![android, javaSE, javaEE]
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
});
    
sdlManager.sendRPC(dialNumber);
```
!@

### DialNumber Result
`DialNumber` has 3 possible results that you should expect:

1. SUCCESS - DialNumber was successfully sent, and a phone call was initiated by the user.
2. REJECTED - DialNumber was sent, and a phone call was cancelled by the user. Also, this could mean that there is no phone connected via Bluetooth.
3. DISALLOWED - Your app does not have permission to use DialNumber.
