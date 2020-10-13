# Updating from v6.7 to v7.0
The iOS library has made a number of breaking changes in SDL v7.0. This means that your project is unlikely to compile without changes.

### iOS Minimum Version Changes
SDL iOS 7.0 now requires that your app's minimum supported version be iOS 10.0 or greater – previously it was iOS 8.0. If your app's minimum version is already iOS 10.0 or greater, then there's nothing you need to do! However, if you target a lower iOS version as your minimum version, you will need to stay on SDL iOS v6.7 until you can move up your minimum version. SDL iOS 7.0 has removed functionality shims necessary to allow it to function properly on iOS versions below 10.0.

### Changes to RPC Initializers
All deprecated methods have been removed. Most of the removed methods are RPC initializers. If you are affected by this change, please look at the new available initializers and choose one of those to use.

For example:
```objc
// This was deprecated and now removed
SDLAlert *alert = [[SDLAlert alloc] initWithAlertText1:@"Text1" alertText2:@"Text2" alertText3:@"Text3"];

// Replacement
SDLAlert *alert = [[SDLAlert alloc] init];
alert.alertText1 = @"Text1";
alert.alertText2 = @"Text2";
alert.alertText3 = @"Text3";
```

We will be looking to improve RPC initializers in the future.

### Other API Removals
#### Configurations
Previously deprecated `SDLConfiguration` and other configuration APIs have been removed.

In `SDLLifecycleConfiguration`, the `SDLLifecycleConfiguration defaultConfigurationWithAppName:appId:` method and it's `debugConfiguration` counterpart have been officially removed. You must now use `defaultConfigurationWithAppName:fullAppId:` and it's `debugConfiguration` counterpart. Note that if you set a legacy app id into the `fullAppId` field, everything will continue to work as it did on the previous API.

Several `SDLStreamingMediaConfiguration` APIs have also been removed. Any API that took a security manager is now gone. In their stead, add your security manager onto an `SDLEncryptionConfiguration` instance and pass it to the `SDLConfiguration`.

#### Delegate API Removals
Delegate API removals require special attention because if you still implement the deprecated method, that method will no longer work and there will no longer be a warning.

`SDLKeyboardDelegate updateAutocompleteWithInput:completionHandler:` has been removed and is superseded by `updateAutocompleteWithInput:autoCompleteResultsHandler:`. The new method allows you to return a list of results. On older head units, only the top result will be used.

`SDLManagerDelegate managerShouldUpdateLifecycleToLanguage:` has been removed and is superseded by `managerShouldUpdateLifecycleToLanguage:hmiLangugage:`. The new method will alert you if either the VR language or the text language changes.

#### Permission Manager
The `SDLPermissionManager` has had several API removals. The primary change is to use the `SDLRPCFunctionName` enum in place of the `SDLPermissionRPCName` `NSString` typedef. This provides additional type safety when checking permissions for an RPC. Also note that `subscribeToRPCPermissions:groupType:withHandler:` has slightly different behavior than `addObserverForRPCs:groupType:withHandler:` when the `groupType` is `SDLPermissionGroupTypeAllAllowed` for the initial callback. It will now only callback if all items are allowed, whereas before it would callback no matter the initial status of the group.

### Other Deprecations
We did also deprecate a few APIs in this release.

1. `SDLServiceUpdateReason` enums were not correctly formed. We deprecated the previous APIs and introduced new ones that are correctly formed.
2. Existing `SDLCharacterSet` sets were not standards-compliant and are deprecated. New character sets have been added and will be used in future head units to describe text fields.
3. The `SDLLockScreenStatus` notification now has a new type of payload. It has changed from an `SDLOnLockScreenStatus` RPC to a `SDLLockScreenStatusInfo` object. This is only important if you have built your own lock screen management system instead of using the one provided through the SDL iOS library.

### New Features
#### Changing Template Layout
The primary new feature is `SDLManager.screenManager changeLayout:withCompletionHandler:`. This wraps the `SDLSetDisplayLayout` and `SDLShow` (on RPC v6.0+) ability to change the template layout and color scheme. `SDLSetDisplayLayout` will be deprecated in a future release, and this is now the preferred API to manage layouts.