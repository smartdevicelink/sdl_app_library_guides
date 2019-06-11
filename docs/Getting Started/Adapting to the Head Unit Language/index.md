# Adapting to the Head Unit Language
Since a head unit can support multiple languages, you may want to add support for more than one language to your SDL app. The SDL library allows you to check which language is currently be used by the head unit. If desired, the app's name and the app's text-to-speech (TTS) name can be customized to reflect the head unit's current language. If your app name is not part of the current lexicon, you should tell the VR system how a native speaker will pronounce your app name by setting the TTS name using [phonemes](https://en.wikipedia.org/wiki/Phoneme) from either the Microsoft SAPI phoneme set or from the LHPLUS phoneme set.

## Setting the Default Language
The initial configuration of the @![iOS]`SDLManager`!@@![android,javaSE,javaEE]`SdlManager`!@ requires a default language when setting the @![iOS]`SDLLifecycleConfiguration`!@@![android,javaSE,javaEE]`Builder`!@. If not set, the SDL library uses American English (*EN_US*) as the default language. The connection will fail if the head unit does not support the `language` set in the @![iOS]`SDLLifecycleConfiguration`!@@![android,javaSE,javaEE]`Builder`!@. The `RegisterAppInterface` response RPC will return `INVALID_DATA` as the reason for rejecting the request.

### What if My App Does Not Support the Head Unit Language?
If your app does not support the current head unit language, you should decide on a default language to use in your app. All text should be created using this default language. Unfortunately, your VR commands will probably not work as the VR system will not recognize your users' pronunciation.

@![iOS]
### Checking the Current Head Unit Language
After starting the `SDLManager` you can check the `registerResponse` property for the head unit's `language` and `hmiDisplayLanguage`. The `language` property gives you the current VR system language; `hmiDisplayLanguage` the current display text language.

##### Objective-C
```objc
SDLLanguage headUnitLanguage = self.sdlManager.registerResponse.language;
SDLLanguage headUnitHMIDisplayLanguage = self.sdlManager.registerResponse.hmiDisplayLanguage;
```

##### Swift
```swift
let headUnitLanguage = sdlManager.registerResponse?.language
let headUnitHMIDisplayLanguage = sdlManager.registerResponse?.hmiDisplayLanguage
```

## Updating the SDL App Name
To customize the app name for the head unit's current language, implement the following steps:

1. Set the default `language` in the `SDLLifecycleConfiguration`.
2. Add all languages your app supports to `languagesSupported` in the `SDLLifecycleConfiguration`.
3. Implement the `SDLManagerDelegate`'s `managerShouldUpdateLifecycleToLanguage:` method. If the head unit's language is different from the default language and is a supported language, the method will be called with the head unit's current language. Return a `SDLLifecycleConfigurationUpdate` object with the new `appName` and/or `ttsName`.

##### Objective-C
```objc
- (nullable SDLLifecycleConfigurationUpdate *)managerShouldUpdateLifecycleToLanguage:(SDLLanguage)language {
    SDLLifecycleConfigurationUpdate *configurationUpdate = [[SDLLifecycleConfigurationUpdate alloc] init];

    if ([language isEqualToEnum:SDLLanguageEnUs]) {
        update.appName = <#App Name in English#>;
    } else if ([language isEqualToEnum:SDLLanguageEsMx]) {
        update.appName = <#App Name in Spanish#>;
    } else if ([language isEqualToEnum:SDLLanguageFrCa]) {
        update.appName = <#App Name in French#>;
    } else {
        return nil;
    }

    return configurationUpdate;
}
```

##### Swift
```swift
func managerShouldUpdateLifecycle(toLanguage language: SDLLanguage) -> SDLLifecycleConfigurationUpdate? {
    let configurationUpdate = SDLLifecycleConfigurationUpdate()

    switch language {
    case .enUs:
        configurationUpdate.appName = <#App Name in English#>
    case .esMx:
        configurationUpdate.appName = <#App Name in Spanish#>
    case .frCa:
        configurationUpdate.appName = <#App Name in French#>
    default:
        return nil
    }

    return configurationUpdate
}
```
!@

@![android,javaSE,javaEE]

# Handling a Language Change

When a user changes the language on a head unit, an `OnLanguageChange` notification will be sent from Core. Then your app will disconnect. In order for your app to automatically reconnect to the head unit, there are a few changes to make in the following files:

* Local SDL Broadcast Receiver
* Local SDL Service

## SDL Broadcast Receiver

When the SDL Service's connection to core is closed, we want to tell our local SDL Broadcast Receiver to restart the SDL Service. To do this, first add a public String in your app's local SDL Broadcast Receiver class that can be included as an extra in a broadcast intent.

`public static final String RECONNECT_LANG_CHANGE = "RECONNECT_LANG_CHANGE";`

Then, override the `onReceive()` method of the local SDL Broadcast Receiver to call `onSdlEnabled()` when receiving that action:

```java
@Override
public void onReceive(Context context, Intent intent) {
	super.onReceive(context, intent); // Required if overriding this method

	if (intent != null) {
		String action = intent.getAction();
		if (action != null){
			if(action.equalsIgnoreCase(TransportConstants.START_ROUTER_SERVICE_ACTION)) {
				if (intent.getBooleanExtra(RECONNECT_LANG_CHANGE, false)) {
					onSdlEnabled(context, intent);
				}
			}
		}
	}
}
```

!!! MUST
Be sure to call `super.onReceive(context, intent);` at the start of the method!
!!!

!!! NOTE
This guide also assumes your local SDL Broadcast Receiver implements the `onSdlEnabled()` method as follows:
!!!

```java
@Override
public void onSdlEnabled(Context context, Intent intent) {
	intent.setClass(context, SdlService.class);
	context.startService(intent);
}
```

## SDL Service

We want to tell our local SDL Broadcast Receiver to restart the service when an `OnLanguageChange` notification is received from Core . To do so, add a notification listener as follows:

```java
sdlManager.addOnRPCNotificationListener(FunctionID.ON_LANGUAGE_CHANGE, new OnRPCNotificationListener() {
    @Override
    public void onNotified(RPCNotification notification) {
        SdlService.this.stopSelf();
        Intent intent = new Intent(TransportConstants.START_ROUTER_SERVICE_ACTION);
        intent.putExtra(SdlReceiver.RECONNECT_LANG_CHANGE, true);
        AndroidTools.sendExplicitBroadcast(context, intent, null);
    }
});
```
!@