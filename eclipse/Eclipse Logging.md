# Eclipse Logging
> Using Eclipse logging. This tutorial gives an overview how to do logging in an Eclipse application.

[](#eclipse-logging)[1\. Eclipse Logging](#eclipse-logging)
-----------------------------------------------------------

### [](#platform-logging)[1.1. Platform logging](#platform-logging)

The `Platform` class provides simple logging API.

```
Platform.getLog(getClass()).error();
Platform.getLog(getClass()).warn();
Platform.getLog(getClass()).info();
```

### [](#logger-service)[1.2. Logger service](#logger-service)

The `org.eclipse.e4.core.services` plug-in contains the `Logger` class. An instance of this class can get injected via dependency injection. The default implementation is provided by the `WorkbenchLogger` class.

The `Logger` implementation wraps the `IStatus` into an simpler interface and provides several methods to log info, warning or error message. The following code shows example calls.

```
logger.info("Info: Closing application");
logger.error("Error: Closing application");
logger.warn("Warn: Closing application");
```

Customers can replace the `WorkbenchLogger` implementation in the Context with there own implementation. This way customers could log to the Eclipse system as well as to other external log systems.

The log level can be setup in the _config.ini_ file of your application via the _eclipse.log.level_ parameter. The value can be set to _INFO_, _WARNING_ and _ERROR_, _INFO_ will for example show all log message.

By default Eclipse logs all messages.

In case dependency injection is not available in the class where logging is requested, the logger can be obtained by using the service API:

```
Logger logger = PlatformUI.getWorkbench().getService(org.eclipse.e4.core.services.log.Logger.class);
logger.info("Using the eclipse logger");
```

[](#slf4j-in-eclipse-applications)[2\. SLF4J in eclipse applications](#slf4j-in-eclipse-applications)
-----------------------------------------------------------------------------------------------------

Many Java developers are more familiar with the [SLF4J logging facade](http://www.slf4j.org/), which is more powerful than Eclipse’s default logger.

### [](#obtaining-the-slf4j-library-from-orbit)[2.1. Obtaining the SLF4J library from Orbit](#obtaining-the-slf4j-library-from-orbit)

The [Eclipse Orbit Project](http://www.eclipse.org/orbit/) provides many different open source libraries, which can be obtained as OSGi bundles from Obrit’s p2 update sites. This makes it easy to use third party open source libraries for target definitions and a Maven Tycho build.

![](_assets/slf4j-orbit-updatesite.png)

One of the most popular slf4j implementations is the [Logback logger](http://logback.qos.ch/), which is also available on Orbit’s p2 update site.

![](_assets/logback-orbit-updatesite.png)

|  | 

The Orbit p2 update sites URLs change quite regularly. Therefore it is recommended to look in the [Orbit downloads section](https://download.eclipse.org/tools/orbit/downloads/) to find the right p2 update site URL.

 |

### [](#using-slf4j)[2.2. Using SLF4J](#using-slf4j)

The SLF4J dependencies just have to be added to the desired bundle.

![](_assets/slf4j-package-import.png)

After doing this the SLF4J logger API can be used in the classes of the bundle.

```
private static final Logger LOG = LoggerFactory.getLogger(ClassWithInfosToBeLogged.class);
```

|  | It saves a lot of effort, when a Template for the creation of this `LOG` instance is created. |

### [](#configuring-logback)[2.3. Configuring Logback](#configuring-logback)

In an OSGi environment it is not that easy for Logback to automatically determine where the _logback.xml_ logger configuration is stored.

For Logback the `ch.qos.logback.classic.joran.JoranConfigurator` is used to parse the _logback.xml_ configuration file.

By obtaining the `ch.qos.logback.classic.LoggerContext` and using the `JoranConfigurator` the location of the _logback.xml_ file can be set programmatically.

```
package com.vogella.logging.config;

import java.io.IOException;
import java.net.URL;

import org.eclipse.core.runtime.FileLocator;
import org.eclipse.core.runtime.Path;
import org.eclipse.core.runtime.Platform;
import org.eclipse.osgi.service.datalocation.Location;
import org.osgi.framework.Bundle;
import org.osgi.framework.BundleActivator;
import org.osgi.framework.BundleContext;
import org.slf4j.LoggerFactory;

import ch.qos.logback.classic.LoggerContext;
import ch.qos.logback.classic.joran.JoranConfigurator;
import ch.qos.logback.core.joran.spi.JoranException;

public class Activator implements BundleActivator {

    @Override
    public void start(BundleContext bundleContext) throws Exception {
        configureLogbackInBundle(bundleContext.getBundle());
    }

    @Override
    public void stop(BundleContext bundleContext) throws Exception {
    }

    private void configureLogbackInBundle(Bundle bundle) throws JoranException, IOException {
        LoggerContext context = (LoggerContext) LoggerFactory.getILoggerFactory();
        JoranConfigurator jc = new JoranConfigurator();
        jc.setContext(context);
        context.reset();

        // overriding the log directory property programmatically
        String logDirProperty = // ... get alternative log directory location
        context.putProperty("LOG_DIR", logDirProperty);

        // this assumes that the logback.xml file is in the root of the bundle.
        URL logbackConfigFileUrl = FileLocator.find(bundle, new Path("logback.xml"),null);
        jc.doConfigure(logbackConfigFileUrl.openStream());
    }

}
```

In some cases it is pretty handy to have this _logback.xml_ logger configuration in the _configuration_ folder of the RCP application rather than bundled in a JAR file. So _logback.xml_ logger configuration can easily be changed in a production environment if necessary. The local version of the logback.xml is still kept so that the developer does not have to copy the _logback.xml_ file manually to the `Platform.getInstallLocation()` location.

```
private void configureLogbackExternal(Bundle bundle) throws JoranException, IOException {
    LoggerContext context = (LoggerContext) LoggerFactory.getILoggerFactory();
    JoranConfigurator jc = new JoranConfigurator();
    jc.setContext(context);
    context.reset();

    // overriding the log directory property programmatically
    String logDirProperty = // ... get alternative log directory location
    context.putProperty("LOG_DIR", logDirProperty);

    // get the configuration location where the logback.xml is located
    Location configurationLocation = Platform.getInstallLocation();
    File logbackFile = new File(configurationLocation.getURL().getPath(), "logback.xml");
    if(logbackFile.exists()) {
        jc.doConfigure(logbackFile);
    } else {
        URL logbackConfigFileUrl = FileLocator.find(bundle, new Path("logback.xml"), null);
        jc.doConfigure(logbackConfigFileUrl.openStream());
    }
}
```

|  | 

How to provide external static files like a _logback.xml_ file for a product build with Maven Tycho can be found here: [Eclipse Tycho Tutorial](https://www.vogella.com/tutorials/EclipseTycho/article.html#exercise-tycho-build-for-products-with-root-files)

 |

The _LOG\_DIR_ property, which is overridden in the code above, can be obtained like this:

```
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <!-- With a context scope the ${LOG_DIR} variable can be obtained from the LoggerContext -->
    <!-- By using :- in the value attribute a default can be specified -->
    <property scope="context" name="LOG_DIRECTORY" value="${LOG_DIR}:-\home\default-log-dir" />

    <!-- Make use of the LOG_DIRECTORY property for the general xml configuration of Logback -->

</configuration>
```

[](#exercise-slf4j-and-logback-in-eclipse-applications)[3\. Exercise - SLF4J and Logback in Eclipse applications](#exercise-slf4j-and-logback-in-eclipse-applications)
----------------------------------------------------------------------------------------------------------------------------------------------------------------------

### [](#obtaining-the-slf4j-and-logback-library-from-orbit)[3.2. Obtaining the SLF4J and Logback library from Orbit](#obtaining-the-slf4j-and-logback-library-from-orbit)

Install SLF4J and Logback into your IDE and reuse the IDE’s target platform or create a target definition file and add all necessary features, like eclipse SDK and logger libs, to it.

See how to obtain the SLF4J libraries from Orbit and Logback dependencies.

### [](#create-a-plugin-using-slf4j)[3.3. Create a plugin using SLF4J](#create-a-plugin-using-slf4j)

Create a _com.vogella.logging.rcp_ plug-in project, which uses the E4 Application template.

Add the SLF4J dependencies:

![](_assets/slf4j-package-import.png)

Use a `Logger` in the generated `AboutHandler` class and log some debug messages or throw some `Exceptions`, which are logged.

```
package com.vogella.logging.rcp.handlers;

import org.eclipse.e4.core.di.annotations.Execute;
import org.eclipse.jface.dialogs.MessageDialog;
import org.eclipse.swt.widgets.Shell;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class AboutHandler {

    private static final Logger LOG = LoggerFactory.getLogger(AboutHandler.class);

    @Execute
    public void execute(Shell shell) {
        LOG.debug("Executing about handler");
        MessageDialog.openInformation(shell, "About", "Eclipse 4 RCP Application");
    }
}
```

|  | It saves a lot of effort, when a Template for the creation of this `LOG` instance is created. |

### [](#creating-a-plug-in-that-configures-logback)[3.4. Creating a plug-in that configures Logback](#creating-a-plug-in-that-configures-logback)

Create a _com.vogella.logging.config_ plug-in project with an Activator.

The root folder of this plug-in should contain the following _logback.xml_ file:

```
<?xml version="1.0" encoding="UTF-8"?>
<configuration>

    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <layout class="ch.qos.logback.classic.PatternLayout">
            <Pattern>
                %d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n
            </Pattern>
        </layout>
    </appender>

    <logger name="com.vogella.logger" level="debug"
        additivity="false">
        <appender-ref ref="STDOUT" />
    </logger>

    <root level="debug">
        <appender-ref ref="STDOUT" />
    </root>

</configuration>
```

The following dependencies are required to configure Logback.

![](_assets/logback-dependencies.png)

|  | 

Also using _Imported Packages_ for the Logback dependencies would be preferable, but unfortunately Logback has so many packages that they won’t fit on the screenshot.

 |

The `Activator` is supposed to initialize the Logback configuration.

```
package com.vogella.logging.config;

import java.io.IOException;
import java.net.URL;

import org.eclipse.core.runtime.FileLocator;
import org.eclipse.core.runtime.Path;
import org.eclipse.core.runtime.Platform;
import org.eclipse.osgi.service.datalocation.Location;
import org.osgi.framework.Bundle;
import org.osgi.framework.BundleActivator;
import org.osgi.framework.BundleContext;
import org.slf4j.LoggerFactory;

import ch.qos.logback.classic.LoggerContext;
import ch.qos.logback.classic.joran.JoranConfigurator;
import ch.qos.logback.core.joran.spi.JoranException;

public class Activator implements BundleActivator {

    @Override
    public void start(BundleContext bundleContext) throws Exception {
        configureLogbackInBundle(bundleContext.getBundle());
    }

    @Override
    public void stop(BundleContext bundleContext) throws Exception {
    }

    private void configureLogbackInBundle(Bundle bundle) throws JoranException, IOException {
        LoggerContext context = (LoggerContext) LoggerFactory.getILoggerFactory();
        JoranConfigurator jc = new JoranConfigurator();
        jc.setContext(context);
        context.reset();

        // this assumes that the logback.xml file is in the root of the bundle.
        URL logbackConfigFileUrl = FileLocator.find(bundle, new Path("logback.xml"),null);
        jc.doConfigure(logbackConfigFileUrl.openStream());
    }

}
```

### [](#startup-the-logging-config-bundle)[3.5. Startup the logging config bundle](#startup-the-logging-config-bundle)

The configuration of the SLF4J implementation (Logback) should be separated from the other bundles. So that no dependency to the _com.vogella.logging.config_ bundle is necessary and other bundles just need to import the org.slf4j package, like it is done _com.vogella.logging.rcp_.

Due to that the _com.vogella.logging.config_ bundle will not be started automatically. Therefore it must be added to the _Start Levels_ in the product configuration.

![](_assets/logging-start-levels-config.png)

### [](#validate)[3.6. Validate](#validate)

Start the sample RCP application and see whether the logging looks like expected according to your _logback.xml_ configuration.