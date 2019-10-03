# Multiple Process Items

The SmartDeviceLink Android library uses multiple process and there are some items that should be understood about why that is necessary and precautions to take while handling that situation. 

## Why does the router service run in its own process?

The router service is designed to live outside the normal lifecycle of the app integrating the SDL framework. The different process allows a level of security to cut off access to the hosting application's data. It also allows the router service to not interfere with the hosting application's runtime; this means if the router service unexpectedly stops or crashes, it will not take down the entire app. This relationship also works in the opposite direction, which is important to maintain a good user experience when apps are connected through a router service.

## Content providers and multiple process issues

Android content providers have a unique lifecycle that does not work in the expected flow. Content providers are actually started before the `Application` class and following Activities, Services, etc. Some libraries use this to know when their code/module can initialize and always be ready for the entire lifecycle of the application. This is found with many Google libraries (Firebase, Jetpack, etc).

The issue is that, by default, content providers are only attached to the main process. This process is the same as the application package unless otherwise specified. This means, when the main process starts the content provider will be started, but if a different process is started the content provider will not be started. So if the app has its first start from a component that is designed to run in a different process, the content provider won't be ready by the time those components start up; this includes the application instance for that process. 

### Why is this a problem?

The issue occurs when there is code in a developer's custom `Application` class that assumes the content provider has already been initialized, but an instance of that child `Application` class is created for a process outside of the main process.

For example:

```
public class MyApplication extends Application {


    @Override
    public void onCreate() {
        super.onCreate();
        
         ModuleUsingContentProviderForInit.doSomething();

    }
}

```

If an instance is created outside the main process, this application will crash with a runtime exception. 

### Workaround

Depending on the module that uses a content provider for initialization, it could be possible to start/initialize it from the `onCreate` method of the extended `Application` class. It should be noted that the module would then need to be set up for a multiple process environment, which is not always the case.

If the module can't be initialized in this way, then the `Application` child class will need to keep a flag that prevents code from executing that would cause the error. 

For SDL the solution can be as follows:

```
public class MyApplication extends Application {

    private static final String ROUTER_SERVICE_PROCESS = "com.smartdevicelink.router";
    
    boolean isSdlProcessFlag = false;
    
    @Override
    public void onCreate() {
        super.onCreate();
        
        isSdlProcessFlag = isSdlProcess()
        if (isSdlProcessFlag) {
          //This application instance is running in the SDL process
        	return;
        }
        
        ModuleUsingContentProviderForInit.doSomething();

    }
    
    /**
    *
    * @return if this process is the SDL router service process
    */
    private boolean isSdlProcess(){
        int myPid = android.os.Process.myPid();
        ActivityManager am = (ActivityManager)this.getSystemService(ACTIVITY_SERVICE);
        if (am == null || am.getRunningAppProcesses() == null) {
            return false;
        }
        
        for (ActivityManager.RunningAppProcessInfo processInfo : am.getRunningAppProcesses()) {
            if (processInfo != null && processInfo.pid == myPid) {
                return ROUTER_SERVICE_PROCESS.equals(processInfo.processName);
            }
        }
        return false;
    }
}

```

!!! NOTE
If other callback methods in your `Application` class are used, they must also check this flag to prevent unintended behavior.
!!!

The use of this flag can help prevent errors when extending the `Application` class that assume it always has the main process started first. This solution could be modified to change the flag to monitor if this is the main process or not very easily. 


### Custom Application classes instance for each process

While the documentation on this is a little scarce, the Android OS creates a new instance of the supplied `Application` class for each process that is started in your app. This means your custom `Application` class needs to be ready to run on different processes. The previous example is a good sample that can prevent code from executing in your custom class that is only intended to run on the main process. 
