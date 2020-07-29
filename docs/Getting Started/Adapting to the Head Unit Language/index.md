# Adapting to the Head Unit Language
Since a head unit can support multiple languages, you may want to add support for more than one language to your SDL app. The SDL library allows you to check which language is currently used by the head unit. If desired, the app's name and the app's text-to-speech (TTS) name can be customized to reflect the head unit's current language. If your app name is not part of the current lexicon, you should tell the VR system how a native speaker will pronounce your app name by setting the TTS name using [phonemes](https://en.wikipedia.org/wiki/Phoneme) from either the Microsoft SAPI phoneme set or from the LHPLUS phoneme set.

## Setting the Default Language
The initial configuration of the @![iOS]`SDLManager`!@@![android,javaSE,javaEE,javascript]`SdlManager`!@ requires a default language when setting the @![iOS]`SDLLifecycleConfiguration`!@@![android,javaSE,javaEE]`Builder`!@@![javascript]`SDL.manager.LifecycleConfig`!@. If not set, the SDL library uses American English (*EN_US*) as the default language. The connection will fail if the head unit does not support the `language` set in the @![iOS]`SDLLifecycleConfiguration`!@@![android,javaSE,javaEE]`Builder`!@@![javascript]`SDL.manager.LifecycleConfig`!@. The `RegisterAppInterface` response RPC will return `INVALID_DATA` as the reason for rejecting the request.

### What if My App Does Not Support the Head Unit Language?
If your app does not support the current head unit language, you should decide on a default language to use in your app. All text should be created using this default language. Unfortunately, your VR commands will probably not work as the VR system will not recognize your users' pronunciation.


### Checking the Current Head Unit Language
After starting the `SDLManager` you can check the @![iOS]`registerResponse`!@@![android,javaSE,javaEE,javascript]`sdlManager.getRegisterAppInterfaceResponse()`!@ property for the head unit's `language` and `hmiDisplayLanguage`. The `language` property gives you the current VR system language; `hmiDisplayLanguage` the current display text language.

@![iOS]
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
!@
@![android,javaSE,javaEE]
```java
Language headUnitLanguage = sdlManager.getRegisterAppInterfaceResponse().getLanguage();
Language headUnitHMILanguage = sdlManager.getRegisterAppInterfaceResponse().getHmiDisplayLanguage();
```
!@
@![javascript]
```javascript
const headUnitLanguage = sdlManager.getRegisterAppInterfaceResponse().getLanguage();
const headUnitHMILanguage = sdlManager.getRegisterAppInterfaceResponse().getHmiDisplayLanguage();
```
!@

## Updating the SDL App Name
@![javascript]
The SDL JavaScript Suite currently does not support updating the app name. This will be addressed in a future release.
!@
@![iOS,android,javaSE,javaEE]
To customize the app name for the head unit's current language, implement the following steps:

1. Set the default `language` in the !@@![iOS]`SDLLifecycleConfiguration`!@@![android,javaSE,javaEE]`Builder`.!@
@![iOS]
2. Add all languages your app supports to `languagesSupported` in the `SDLLifecycleConfiguration`.
3. Implement the `SDLManagerDelegate`'s `managerShouldUpdateLifecycleToLanguage:hmiLanguage:` method. If the module's current HMI language or voice recognition (VR) language is different from the app's default language, the method will be called with the module's current HMI and/or VR language. Please note that the delegate method will only be called if your app supports the head unit's current language. Return a `SDLLifecycleConfigurationUpdate` object with the new `appName` and/or `ttsName`.
!@
@![android,javaSE,javaEE]
2. Implement the `sdlManagerListener`'s `managerShouldUpdateLifecycle(Language language, Language hmiLanguage)` method. If the module's current HMI language or voice recognition (VR) language is different from the app's default language, the listener will be called with the module's current HMI and/or VR language. Return a `LifecycleConfigurationUpdate` with the new `appName` and/or `ttsName`.
!@

@![iOS]
##### Objective-C
```objc
// The `hmiLanguage` is the text language of the head unit, the `language` is the VR language of the head unit. These will usually be the same, but not always. You may want to update your `appName` (text) and `ttsName` (VR) separately.
- (nullable SDLLifecycleConfigurationUpdate *)managerShouldUpdateLifecycleToLanguage:(SDLLanguage)language hmiLanguage:(SDLLanguage)hmiLanguage {
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

    update.ttsName = [SDLTTSChunk textChunksFromString:update.appName];
    return configurationUpdate;
}
```

##### Swift
```swift
// The `hmiLanguage` is the text language of the head unit, the `language` is the VR language of the head unit. These will usually be the same, but not always. You may want to update your `appName` (text) and `ttsName` (VR) separately.
func managerShouldUpdateLifecycle(toLanguage language: SDLLanguage, hmiLanguage: SDLLanguage) -> SDLLifecycleConfigurationUpdate? {
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

    update.ttsName = [SDLTTSChunk(text: update.appName!, type: .text)]
    return configurationUpdate
}
```
!@

@![android,javaSE,javaEE]
```java
@Override
public LifecycleConfigurationUpdate managerShouldUpdateLifecycle(Language language, Language hmiLanguage) {
    boolean isNeedUpdate = false;
    String appName = APP_NAME;
    String ttsName = APP_NAME;
    switch (language) {
        case ES_MX:
            isNeedUpdate = true;
            ttsName = APP_NAME_ES;
            break;
        case FR_CA:
            isNeedUpdate = true;
            ttsName = APP_NAME_FR;
            break;
        default:
            break;
    }
    switch (hmiLanguage) {
        case ES_MX:
            isNeedUpdate = true;
            appName = APP_NAME_ES;
            break;
        case FR_CA:
            isNeedUpdate = true;
            appName = APP_NAME_FR;
            break;
        default:
            break;
    }
    if (isNeedUpdate) {
        return new LifecycleConfigurationUpdate(appName, null, TTSChunkFactory.createSimpleTTSChunks(ttsName), null);
    } else {
        return null;
    }
}
```
!@
