# What is the Android Router Service?
The Android OS has limitations around the availability of certain transports (Bluetooth RFCOMM channels, single app AOA/USB permissions). Therefore, SmartDeviceLink introduced a service that operates as a router, using a single transport pipe and extending it to many different bound apps. The router service is part of the required integration to become SDL enabled and can be hosted by any of the SDL enabled apps on a phone. Some OEMs might choose to have their own companion app that always hosts a router service for their specific hardware.

# What is a Trusted Router Service?
Since information is being shared through the Android router service it is important that the app hosting the router service can be trusted. This is done through a certification process and a back-end server that maintains a database of apps that can act as a Trusted Router Service. The SDLC will verify the integration of SDL apps to ensure there is no malicious activity. If the app is certified, it will be added to the Trusted Router Service database and be able to act as a Trusted Router Service. 

# How do I add my app to the SDL Trusted Router Service database?
For an Android application to be added to the Trusted Router Service database, the application will need to be registered on the SDL Developer Portal and certified by the SDLC.  For more information on registration, please see [this guide](https://d83tozu1c8tt6.cloudfront.net/media/resources/SDL_Developer_Portal_Registration_Guide.pdf).  Any Android application that is certified by the SDLC will be added to the Trusted Router Service database; there are no additional steps required as it is part of the certification process.

# How do I know if an app is hosting a Trusted Router Service?
Each app will retrieve and cache a list of Trusted Router Services from the back-end server. Based on that app's security levels, they will perform checks against the currently running router service, and if trusted it will bind to the Trusted Router Service. If not, the app will attempt to use its own local transport.
