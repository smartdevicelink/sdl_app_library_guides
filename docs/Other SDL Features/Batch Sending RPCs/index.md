# Batch Sending RPCs
There are two ways to send multiple requests to the head unit: concurrently and sequentially. Which method you should use depends on the type of RPCs being sent. Concurrently sent requests might finish in a random order and should only be used when none of the requests in the group depend on the response of another, such as when subscribing to several hard buttons. Sequentially sent requests only send the next request in the group when a response has been received for the previously sent RPC. Requests should be sent sequentially when you need to know the result of a previous request before sending the next, like when sending the several different requests needed to create a menu.

@![iOS]
Both methods have optional progress and completion handlers. Use the `progressHandler` to check the status of each sent RPC; it will tell you if there was an error sending the request and what percentage of the group has completed sending. The optional `completionHandler` is called when all RPCs in the group have been sent. Use it to check if all of the requests have been sent successfully or not.!@

@![android,javaSE,javaEE]
Both methods have optional listeners that are specific to them, the `OnMultipleRequestListener`. This listener will provide more information than the normal `OnRPCResponseListener`.!@

## Sending Concurrent Requests
When you send multiple RPCs concurrently, it will not wait for the response of the previous RPC before sending the next one. Therefore, there is no guarantee that responses will be returned in order, and you will not be able to use information sent in a previous RPC for a later RPC.

@![iOS]
##### Objective-C
```objc
SDLSubscribeButton *subscribeButtonLeft = [[SDLSubscribeButton alloc] initWithName:SDLButtonNameSeekLeft];
SDLSubscribeButton *subscribeButtonRight = [[SDLSubscribeButton alloc] initWithName:SDLButtonNameSeekRight];
[self.sdlManager sendRequests:@[subscribeButtonLeft, subscribeButtonRight] progressHandler:^(__kindof SDLRPCRequest * _Nonnull request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error, float percentComplete) {
    NSLog(@"Command %@ sent %@, percent complete %f%%", request.name, response.resultCode == SDLResultSuccess ? @"successfully" : @"unsuccessfully", percentComplete * 100);
} completionHandler:^(BOOL success) {
    NSLog(@"All requests sent %@", success ? @"successfully" : @"unsuccessfully");
}];
```

##### Swift
```swift
let subscribeButtonLeft = SDLSubscribeButton(name: SDLButtonNameSeekLeft)
let subscribeButtonRight = SDLSubscribeButton(name: SDLButtonNameSeekRight)
sdlManager.send([subscribeButtonLeft, subscribeButtonRight], progressHandler: { (request, response, error, percentComplete) in
    print("Command \(request.name) sent \(response?.resultCode == .success ? "successfully" : "unsuccessfully"), percent complete \(percentComplete * 100)")
}) { success in
    print("All requests sent \(success ? "successfully" : "unsuccessfully")")
}
```
!@

@![android,javaSE,javaEE]
```java
List<RPCRequest> rpcs = new ArrayList<>();

// rpc 1
SubscribeButton subscribeButtonRequestLeft = new SubscribeButton();
subscribeButtonRequestLeft.setButtonName(ButtonName.SEEKLEFT);
rpcs.add(subscribeButtonRequestLeft);

// rpc 2
SubscribeButton subscribeButtonRequestRight = new SubscribeButton();
subscribeButtonRequestRight.setButtonName(ButtonName.SEEKRIGHT);
rpcs.add(subscribeButtonRequestRight);


sdlManager.sendRPCs(rpcs, new OnMultipleRequestListener() {
    @Override
    public void onUpdate(int remainingRequests) {

    }

    @Override
    public void onFinished() {

    }

    @Override
    public void onResponse(int correlationId, RPCResponse response) {

    }

    @Override
    public void onError(int correlationId, RPCResponse response) {

    }
    });

```
!@

## Sending Sequential Requests
Requests sent sequentially are sent in a set order. The next request is only sent when a response has been received for the previously sent request.

The code example below shows how to create a perform interaction choice set. When creating a perform interaction choice set, the @![iOS]`SDLPerformInteraction`!@ @![android,javaSE,javaEE]`PerformInteraction`!@ RPC can only be sent after the @![iOS]`SDLCreateInteractionChoiceSet`!@ @![android,javaSE,javaEE]`CreateInteractionChoiceSet`!@ RPC has been registered by Core, which is why the requests must be sent sequentially.

@![iOS]
##### Objective-C
```objc
SDLChoice *choice = [[SDLChoice alloc] initWithId:<#Choice Id#> menuName:@"<#Menu Name#>" vrCommands:@[@"<#VR Command#>"]];
SDLCreateInteractionChoiceSet *createInteractionChoiceSet = [[SDLCreateInteractionChoiceSet alloc] initWithId:<#Choice Set Id#> choiceSet:@[choice]];
SDLPerformInteraction *performInteraction = [[SDLPerformInteraction alloc] initWithInteractionChoiceSetId:<#Choice Set Id#>];

[self.sdlManager sendSequentialRequests:@[createInteractionChoiceSet, performInteraction] progressHandler:^BOOL(__kindof SDLRPCRequest * _Nonnull request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error, float percentComplete) {
    NSLog(@"Command %@ sent %@, percent complete %f%%", request.name, response.resultCode == SDLResultSuccess ? @"successfully" : @"unsuccessfully", percentComplete * 100);
} completionHandler:^(BOOL success) {
    NSLog(@"All requests sent %@", success ? @"successfully" : @"unsuccessfully");
}];
```

##### Swift
```swift
let choice = SDLChoice(id: <#Choice Id#>, menuName: "<#Menu Name#>", vrCommands: ["<#VR Command#>"])
let createInteractionChoiceSet = SDLCreateInteractionChoiceSet(id: <#Choice Set Id#>, choiceSet: [choice])
let performInteraction = SDLPerformInteraction(interactionChoiceSetId: <#Choice Set Id#>)

sdlManager.sendSequential(requests: [createInteractionChoiceSet, performInteraction], progressHandler: { (request, response, error, percentageCompleted) -> Bool in
    print("Command \(request.name) sent \(response?.resultCode == .success ? "successfully" : "unsuccessfully"), percent complete \(percentComplete * 100)")
}) { success in
    print("All requests sent \(success ? "successfully" : "unsuccessfully")")
}
```
!@

@![android,javaSE,javaEE]
`// TODO maybe update the example to match iOS example. The texts states using createInteractionChoiceSet and performInteraction but shows an example of button subscribing. The choice menu manager is nearly done for Android so it would be nice if the code examples matched for better alignment.`

```java
List<RPCRequest> rpcs = new ArrayList<>();

// rpc 1
SubscribeButton subscribeButtonRequestLeft = new SubscribeButton();
subscribeButtonRequestLeft.setButtonName(ButtonName.SEEKLEFT);
rpcs.add(subscribeButtonRequestLeft);

// rpc 2
SubscribeButton subscribeButtonRequestRight = new SubscribeButton();
subscribeButtonRequestRight.setButtonName(ButtonName.SEEKRIGHT);
rpcs.add(subscribeButtonRequestRight);

sdlManager.sendSequentialRPCs(rpcs, new OnMultipleRequestListener() {
    @Override
    public void onUpdate(int remainingRequests) {

    }

    @Override
    public void onFinished() {

    }

    @Override
    public void onResponse(int correlationId, RPCResponse response) {

    }

    @Override
    public void onError(int correlationId, Result resultCode, String info) {

    }
    });
```
!@
