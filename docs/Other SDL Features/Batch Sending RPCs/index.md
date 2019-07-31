# Batch Sending RPCs
There are two ways to send multiple requests to the head unit: concurrently and sequentially. Which method you should use depends on the type of RPCs being sent. Concurrently sent requests might finish in a random order and should only be used when none of the requests in the group depend on the response of another, such as when subscribing to several hard buttons. Sequentially sent requests only send the next request in the group when a response has been received for the previously sent RPC. Requests should be sent sequentially when you need to know the result of a previous request before sending the next, like when sending the several different requests needed to create a menu.

@![iOS]
Both methods have optional progress and completion handlers. Use the `progressHandler` to check the status of each sent RPC; it will tell you if there was an error sending the request and what percentage of the group has completed sending. The optional `completionHandler` is called when all RPCs in the group have been sent. Use it to check if all of the requests have been sent successfully or not.!@

@![android,javaSE,javaEE]
Both methods have optional listener that is specific to them, the `OnMultipleRequestListener`. This listener will provide more information than the normal `OnRPCResponseListener`.!@

## Sending Concurrent Requests
When you send multiple RPCs concurrently, it will not wait for the response of the previous RPC before sending the next one. Therefore, there is no guarantee that responses will be returned in order, and you will not be able to use information sent in a previous RPC for a later RPC.

@![iOS]
##### Objective-C
```objc
SDLSubscribeButton *subscribeButtonLeft = [[SDLSubscribeButton alloc] initWithName:SDLButtonNameSeekLeft];
SDLSubscribeButton *subscribeButtonRight = [[SDLSubscribeButton alloc] initWithName:SDLButtonNameSeekRight];
[self.sdlManager sendRequests:@[subscribeButtonLeft, subscribeButtonRight] progressHandler:^(__kindof SDLRPCRequest * _Nonnull request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error, float percentComplete) {
    <#Called as each request completes#>
} completionHandler:^(BOOL success) {
    <#Called when all requests complete#>
}];
```

##### Swift
```swift
let subscribeButtonLeft = SDLSubscribeButton(name: SDLButtonNameSeekLeft)
let subscribeButtonRight = SDLSubscribeButton(name: SDLButtonNameSeekRight)
sdlManager.send([subscribeButtonLeft, subscribeButtonRight], progressHandler: { (request, response, error, percentComplete) in
    <#Called as each request completes#>
}) { success in
    <#Called when all requests complete#>
}
```
!@

@![android,javaSE,javaEE]
```java
SubscribeButton subscribeButtonLeft = new SubscribeButton(ButtonName.SEEKLEFT);
SubscribeButton subscribeButtonRight = new SubscribeButton(ButtonName.SEEKRIGHT);
sdlManager.sendRPCs(Arrays.asList(subscribeButtonLeft, subscribeButtonLeft), new OnMultipleRequestListener() {
    @Override
    public void onUpdate(int remainingRequests) {

    }

    @Override
    public void onFinished() {

    }

    @Override
    public void onError(int correlationId, Result resultCode, String info) {

    }

    @Override
    public void onResponse(int correlationId, RPCResponse response) {

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
    <#Called as each request completes#>
    return YES;
} completionHandler:^(BOOL success) {
    <#Called when all requests complete#>
}];
```

##### Swift
```swift
let choice = SDLChoice(id: <#Choice Id#>, menuName: "<#Menu Name#>", vrCommands: ["<#VR Command#>"])
let createInteractionChoiceSet = SDLCreateInteractionChoiceSet(id: <#Choice Set Id#>, choiceSet: [choice])
let performInteraction = SDLPerformInteraction(interactionChoiceSetId: <#Choice Set Id#>)

sdlManager.sendSequential(requests: [createInteractionChoiceSet, performInteraction], progressHandler: { (request, response, error, percentageCompleted) -> Bool in
    <#Called as each request completes#>
    return true
}) { success in
    <#Called when all requests complete#>
}
```
!@

@![android,javaSE,javaEE]
```java
int choiceId = 111, choiceSetId = 222;
Choice choice = new Choice(choiceId, "Choice title");
CreateInteractionChoiceSet createInteractionChoiceSet = new CreateInteractionChoiceSet(choiceSetId, Collections.singletonList(choice));
PerformInteraction performInteraction = new PerformInteraction("Initial Text", InteractionMode.MANUAL_ONLY, Collections.singletonList(choiceSetId));
sdlManager.sendSequentialRPCs(Arrays.asList(createInteractionChoiceSet, performInteraction), new OnMultipleRequestListener() {
    @Override
    public void onUpdate(int i) {

    }

    @Override
    public void onFinished() {

    }

    @Override
    public void onError(int i, Result result, String s) {

    }

    @Override
    public void onResponse(int i, RPCResponse rpcResponse) {

    }
});
```
!@
