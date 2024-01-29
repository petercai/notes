# Installing the Business Central on WildFly 
There are two options to install **jBPM Business Central on WildFly**. You can either download the **jBPM bundle distribution** ([https://www.jbpm.org/download/download.html](https://www.jbpm.org/download/download.html)) which includes WildFly, the Kie Server and the Business Central or you can install the single components on the top of an existing WildFly distribution. This tutorial cover the latter option.

Download required server and components
---------------------------------------

To get started, you need a **WildFly Application Server** where the KieServer and Business Console can be deployed. Download it from [http://wildfly.org/downloads/](http://wildfly.org/downloads/)

Please note that you need to choose the version that matches with the Business Central WAR version. So, for the Business Central 7.51.0 you will need to download WildFly 19

![](https://go.ezodn.com/utilcave_com/ezoicbwa.png)

Then, from [http://www.drools.org/download/download.html](http://www.drools.org/download/download.html) download the following components:

**1) Business Central Workbench**

**2) KIE Execution Server**

![](https://go.ezodn.com/utilcave_com/ezoicbwa.png)

![](https://www.mastertheboss.com/images/stories/drools/business-central-download.png)

Install the Business Central and Kie Components on WildFly
----------------------------------------------------------

Next, copy the **Business Central, Kie Server, Kie Server Controller** and **Kie Server Router Proxy** to the deployments folder of WildFly, renaming them as follows:

-rw-rw-r--. 1 francesco francesco 216030437 Apr 1  10:24 business-central.war

-rw-rw-r--. 1 francesco francesco 26483852 Mar 9  11:59 kie-server-controller.war

-rw-rw-r--. 1 francesco francesco 4572533 Mar 9  11:57 kie-server-router-proxy-7.51.0.Final.jar

-rw-rw-r--. 1 francesco francesco 92753370 Mar 9  12:00 kie-server.war

-rw-rw-r--. 1 francesco francesco 216030437 Apr 1 10:24 business-central.war -rw-rw-r--. 1 francesco francesco 26483852 Mar 9 11:59 kie-server-controller.war -rw-rw-r--. 1 francesco francesco 4572533 Mar 9 11:57 kie-server-router-proxy-7.51.0.Final.jar -rw-rw-r--. 1 francesco francesco 92753370 Mar 9 12:00 kie-server.war

```
-rw-rw-r--. 1 francesco francesco 216030437 Apr  1 10:24 business-central.war
-rw-rw-r--. 1 francesco francesco  26483852 Mar  9 11:59 kie-server-controller.war
-rw-rw-r--. 1 francesco francesco   4572533 Mar  9 11:57 kie-server-router-proxy-7.51.0.Final.jar
-rw-rw-r--. 1 francesco francesco  92753370 Mar  9 12:00 kie-server.war

```

Add Users for Accessing the Console and the Kie Server
------------------------------------------------------

Use the add-user.sh script to add some users which will be used to access the Business Central:

![](https://go.ezodn.com/utilcave_com/ezoicbwa.png)

$ ./add-user.sh -a -u kieserver -p kieserver1! -g kie-server

$ ./add-user.sh -a -u krisv -p krisv -g admin,kie-server,rest-all

$ ./add-user.sh -a -u kieserver -p kieserver1! -g kie-server $ ./add-user.sh -a -u krisv -p krisv -g admin,kie-server,rest-all

```
$ ./add-user.sh -a -u kieserver -p kieserver1! -g kie-server
$ ./add-user.sh -a -u krisv -p krisv -g admin,kie-server,rest-all
```

Configure System Properties in WildFly
--------------------------------------

Within your **standalone-full.xml** file of WildFly, add the following System Properties:

<property  name="org.kie.server.id"  value="sample-server"/>

<property  name="org.kie.server.controller"  value="http://localhost:8080/business-central/rest/controller"/>

<property  name="org.kie.server.location"  value="http://localhost:8080/kie-server/services/rest/server"/>

<property  name="org.kie.server.user"  value="krisv"/>

<property  name="org.kie.server.pwd"  value="krisv"/>

<property  name="org.kie.server.controller.user"  value="krisv"/>

<property  name="org.kie.server.controller.pwd"  value="krisv"/>

<system-properties> <property name="org.kie.server.id" value="sample-server"/> <property name="org.kie.server.controller" value="http://localhost:8080/business-central/rest/controller"/> <property name="org.kie.server.location" value="http://localhost:8080/kie-server/services/rest/server"/> <property name="org.kie.server.user" value="krisv"/> <property name="org.kie.server.pwd" value="krisv"/> <property name="org.kie.server.controller.user" value="krisv"/> <property name="org.kie.server.controller.pwd" value="krisv"/> </system-properties>

```
    <system-properties>
        <property name="org.kie.server.id" value="sample-server"/>
        <property name="org.kie.server.controller" value="http://localhost:8080/business-central/rest/controller"/>
        <property name="org.kie.server.location" value="http://localhost:8080/kie-server/services/rest/server"/>
        <property name="org.kie.server.user" value="krisv"/> 
        <property name="org.kie.server.pwd" value="krisv"/>
        <property name="org.kie.server.controller.user" value="krisv"/>
        <property name="org.kie.server.controller.pwd" value="krisv"/>
    </system-properties>

```

For the sake of simplicity we are using clear text passwords. In production you should use Elytron Credential Stores to protect sensitive texts (See this article: [Using Elytron Credential Stores in WildFly](https://www.mastertheboss.com/jbossas/jboss-security/using-credential-stores-to-store-your-passwords-in-wildfly-11/).

Finally, you should increase default Memory settings for WildFly to allow running the Business Central and Kie Server. Change the standalone.conf file accordingly. For example:

![](https://go.ezodn.com/utilcave_com/ezoicbwa.png)

JAVA_OPTS="-Xms64m -Xmx2512m -XX:MetaspaceSize=96M -XX:MaxMetaspaceSize=512m -Djava.net.preferIPv4Stack=true"

JAVA_OPTS="-Xms64m -Xmx2512m -XX:MetaspaceSize=96M -XX:MaxMetaspaceSize=512m -Djava.net.preferIPv4Stack=true"

```
 JAVA_OPTS="-Xms64m -Xmx2512m -XX:MetaspaceSize=96M -XX:MaxMetaspaceSize=512m -Djava.net.preferIPv4Stack=true"

```

Start WildFly
-------------

Now, we will start WildFly and verify that the Business Central is available.

$ ./standalone.sh -c standalone-full.xml

$ ./standalone.sh -c standalone-full.xml

```
$ ./standalone.sh -c standalone-full.xml

```

The Business Central is available at [http://localhost:8080/business-console](http://localhost:8080/business-console) . Login with the user we have created: **krisv/krisv**

You should access the main page of the Business Central.

![](https://www.mastertheboss.com/images/stories/drools/business_central1.png)
 Choose **Deploy** to Administer your Kie Servers.

From there, you should be able to see the “**sample-server**” registered as Remote Server

![](https://www.mastertheboss.com/images/stories/drools/configurebc-wildfly.png)

Congratulations, now you can start adding Deployment Units on your **Kie-Server**!
