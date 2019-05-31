# Batch Sending RPCs
There are two ways to send multiple requests to the head unit: concurrently and sequentially. Which method you should use depends on the type of RPCs being sent. Concurrently sent requests might finish in a random order and should only be used when none of the requests in the group depend on the response of another, such as when uploading a group of artworks. Sequentially sent requests only send the next request in the group when a response has been received for the previously sent RPC. Requests should be sent sequentially when you need to know the result of a previous request before sending the next, like when sending the several different requests needed to create a menu.

Both methods have optional progress and completion handlers. Use the `progressHandler` to check the status of each sent RPC; it will tell you if there was an error sending the request and what percentage of the group has completed sending. The optional `completionHandler` is called when all RPCs in the group have been sent. Use it to check if all of the requests have been sent successfully or not.

## Sending Concurrent Requests
When you send multiple RPCs concurrently there is no guarantee of the order in which the RPCs will be sent or in which order Core will return responses.

##### Objective-C
```objc
SDLArtwork *artwork1 = [SDLArtwork artworkWithImage:[UIImage imageNamed:@"<#Image name#>"] asImageFormat:<#SDLArtworkImageFormat#>];
SDLArtwork *artwork2 = [SDLArtwork artworkWithImage:[UIImage imageNamed:@"<#Image name#>"] asImageFormat:<#SDLArtworkImageFormat#>];
[self.sdlManager sendRequests:@[artwork1, artwork2] progressHandler:^(__kindof SDLRPCRequest * _Nonnull request, __kindof SDLRPCResponse * _Nullable response, NSError * _Nullable error, float percentComplete) {
    NSLog(@"Command %@ sent %@, percent complete %f%%", request.name, response.resultCode == SDLResultSuccess ? @"successfully" : @"unsuccessfully", percentComplete * 100);
} completionHandler:^(BOOL success) {
    NSLog(@"All requests sent %@", success ? @"successfully" : @"unsuccessfully");
}];
```

##### Swift
```swift
let artwork1 = SDLArtwork(image: <#UIImage#>, persistent: <#Bool#>, as: <#SDLArtworkImageFormat#>)
let artwork2 = SDLArtwork(image: <#UIImage#>, persistent: <#Bool#>, as: <#SDLArtworkImageFormat#>)
sdlManager.send([artwork1, artwork2], progressHandler: { (request, response, error, percentComplete) in
    print("Command \(request.name) sent \(response?.resultCode == .success ? "successfully" : "unsuccessfully"), percent complete \(percentComplete * 100)")
}) { success in
    print("All requests sent \(success ? "successfully" : "unsuccessfully")")
}
```

## Sending Sequential Requests
Requests sent sequentially are sent in a set order. The next request is only sent when a response has been received for the previously sent request.

The code example below shows how to create a perform interaction choice set. When creating a perform interaction choice set, the `SDLPerformInteraction` RPC can only be sent after the `SDLCreateInteractionChoiceSet` RPC has been registered by Core, which is why the requests must be sent sequentially.

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
