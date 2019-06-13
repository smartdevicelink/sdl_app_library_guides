# Setting Security Level for Multiplexing

When connecting to Core via Multiplex Bluetooth transport, your SDL app will use a Router Service housed within your app or another SDL enabled app.

To help ensure the validility of the Router Service, you can select the security level explicity when you create your Multiplex Bluetooth transport in your app's SdlService:

```java
int securityLevel = FLAG_MULTI_SECURITY_MED;

BaseTransport transport = MultiplexTransportConfig(context, appId, securityLevel);
```

If you create the transport without specifying the security level, it will be set to `FLAG_MULTI_SECURITY_MED` by default.

## Security Levels

Security Flag   | Meaning
------------|------------------------------------------------------------
`FLAG_MULTI_SECURITY_OFF`       | Multiplexing security turned off. All router services are trusted.
`FLAG_MULTI_SECURITY_LOW`  | Multiplexing security will be minimal. Only trusted router services will be used. Trusted router list will be obtained from server. List will be refreshed every 20 days or during next connection session if an SDL enabled app has been installed or uninstalled. 
`FLAG_MULTI_SECURITY_MED`     | Multiplexing security will be on at a normal level. Only trusted router services will be used. Trusted router list will be obtained from server. List will be refreshed every 7 days or during next connection session if an SDL enabled app has been installed or uninstalled.
`FLAG_MULTI_SECURITY_HIGH`	| Multiplexing security will be very strict. Only trusted router services installed from trusted app stores will be used. Trusted router list will be obtained from server. List will be refreshed every 7 days or during next connection session if an SDL enabled app has been installed or uninstalled.


## Applying to the Trusted Router Service Database
For an Android application to be added to the Trusted Router Service database, the application will need to be registered on the SDL Developer Portal and certified by the SDLC.  For more information on registration, please see [this guide](https://d83tozu1c8tt6.cloudfront.net/media/resources/SDL_Developer_Portal_Registration_Guide.pdf).  

Any Android application that is certified by the SDLC will be added to the Trusted Router Service database; there are no additional steps required as it is part of the certification process.  

Please consult the [Trusted Router Service FAQs](https://smartdevicelink.com/en/guides/android/frequently-asked-questions/trusted-router-service/) if you have any additional questions.