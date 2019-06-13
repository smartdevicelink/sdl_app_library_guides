## Deploying to AWS

### Deploying your JavaEE App on EC2

When you want to run the project outside the IDE, take the war artifact and deploy it using a Payara server. Payara is built on top of Glassfish and is more well-maintained, and it also solves an issue with Glassfish on redeploying a JavaEE Websocket app where no connections can happen the second time. 

Once you get an EC2 machine, log on it and install some libraries:

```
sudo yum install java-1.8.0-openjdk-headless.x86_64 java-1.8.0-openjdk-devel.x86_64 -y
wget -O payara.zip  https://search.maven.org/remotecontent?filepath=fish/payara/distributions/payara/5.184/payara-5.184.zip
jar xvf payara.zip
cd payara5/glassfish/bin/
sudo chmod 755 asadmin
```

There are two ways to start Payara. The first way will start the server without extra configuration, but it is not in a production context. asadmin’s start-domain command is what runs the Payara server. Then, you need to deploy your war application. Modify the command below to point to where your war file is. When it deploys, it will be running on the root path "/" over port 8080. So, your websocket connection from SDL Core should point to the IP of the machine this is running on, port 8080, and nothing more.

```
./asadmin start-domain
./asadmin deploy --contextroot / ~/my-javaee-app.war
```

The second way is in a production context, but if you’re using a small EC2 machine then it will likely fail due to out of memory errors. This is because the server will consume 2 GB of memory by default. To change this, modify the configuration file located here:

```
payara5/glassfish/domains/production/config/domain.xml
```

Find the part of the file where it says:

```
        <jvm-options>-Xmx2g</jvm-options>
        <jvm-options>-Xms2g</jvm-options>
```

“Xmx2g” means to start the server with 2 GB. Change the number to 1 to make it 1 GB, or change “g” to “m” to make it run in MB. Change both lines.


Starting in production mode:

```
./asadmin start-domain production
./asadmin deploy --contextroot / ~/bocks-ee_war.war
```

Run SDL Core and an HMI. If you're serving the HMI over nginx then nginx should be exposed on a different port than 8080, because Payara runs on 8080 by default. Be sure to modify the default policy table's app_policies object so that SDL Core is aware of where your app is:

```json
"8675829": {
    "keep_context": false,
    "steal_focus": false,
    "priority": "NONE",
    "default_hmi": "NONE",
    "groups": ["Base-4"],
    "RequestType": [],
    "RequestSubType": [],
    "hybrid_app_preference": "CLOUD",
    "endpoint": "ws://$LOCAL_IP:8080/",
    "enabled": true,
    "auth_token": "no auth token",
    "cloud_transport_type": "WS",
    "nicknames": ["App1"]
},
```

### Limitations and Issues

[Follow the guidelines located here.](https://www.oracle.com/technetwork/java/restrictions-142267.html)

These are restrictions for what your logic should do in your EJB. Since this guide puts the whole logic of your app in an EJB, you should follow the restrictions specified above. You can still utilize other aspects of JavaEE to get around some of the limitations, but that will not be covered here.

Notable limitations include not starting or managing threads in your app, not reading from or writing to a file directly, not creating or deleting files or directories, not starting websocket connections yourself and not loading native libraries.

Memory usage increases with both redeployments and with many users connecting and disconnecting over time. The Payara server needs to be shut down to reset memory usage. [This is the only post I could find online which had a similar issue.](https://javaee.groups.io/g/glassfish/topic/6181966#89)

When Payara or Glassfish is unable to handle the load, not only does your JavaEE app stop, but the server also stops.

### Useful Information and Commands

Unzipping a jar file: `unzip my_jar.jar`
    
Packaging all items in your directory to a jar file: `jar cf my_jar.jar *`

When your app gets deployed on Payara on the default domain, it shows up in this directory:

```
payara5/glassfish/domains/domain1/applications
```

However, if the app happens to require loading assets from the same directory it originally resided in, that location changes once it is deployed, and is now located here:

```
payara5/glassfish/domains/domain1/config
```

In the PRODUCTION domain, the locations change to this:

```
payara5/glassfish/domains/production/applications
payara5/glassfish/domains/production/config
```

You can start Payara or Glassfish in different contexts, so be aware of how you started the server because it will change where you should move and modify files.

This is a load test ran using a sample app, connections closing after 5 seconds, and using a t3.small (2 GB memory)

| Connections      | Memory % |
| ----------- | ----------- |
| 0     | 24.8      |
| 300   | 25.2      |
| 700   | 25.4      |
| 1000   | 25.6     |
| 1500   | 26.1     |
| 2000   | 26.2     |
| 3000   | 26.2     |
| 4000   | 26.7     |
| 6000   | 26.8     |
| 8000   | 26.3     |
| 10000   | 27.0    |
| 15000   | 30.0    |
| 20000   | 33.5    |
| 25000   | 37.1    |
| 30000   | 39.8    |
| 40000   | crashed |


The test app could not handle more than 20 new connections per second.

Limit of simultaneous connections:          3562 
After applying the max connections config:  7179

[This page shows how to increase your max connection count for an EC2 machine](https://blog.jayway.com/2015/04/13/600k-concurrent-websocket-connections-on-aws-using-node-js/)
[This page shows the settings to tune for a more effective Payara server](https://blog.payara.fish/fine-tuning-payara-server-in-production)

