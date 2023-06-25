# Java Logging Tutorial: Configuration Examples to Get Started - Sematext
Table of Contents

*   [Java Logging Frameworks](#java-logging-frameworks)
*   [SLF4J](#slf4j)
*   [Java.util.logging](#java-util-logging)
*   [Logback](#logback)
*   [Log4j](#log4j)
*   [Log4j 2](#log4j-2)
*   [Choosing the Right Logging Solution for Your Application](#choosing-the-right-logging-solution-for-your-application)
*   [Logging in Java: How to Log Using the Java Logging API](#logging-in-java-how-to-log-using-the-java-logging-api)
*   [Logger](#logger)
*   [Handler](#handler)
*   [Logging Levels](#logging-levels)
*   [Formatter](#formatter)
*   [The Abstraction Layers](#the-abstraction-layers)
*   [How Do You Enable Logging: java.util.logging Example](#how-do-you-enable-logging-java-util-logging-example)
*   [Configuring java.util.logging via Configuration File](#configuring-java-util-logging-via-configuration-file)
*   [Configuring java.util.logging Programmatically](#configuring-java-util-logging-programmatically)
*   [Efficient Java Logging with Log Management Solutions](#efficient-java-logging-with-log-management-solutions)
*   [Conclusion](#conclusion)

When it comes to troubleshooting Java application performance, [JVM metrics](https://sematext.com/blog/jvm-metrics/) are no longer enough. To fully understand the environment you also need Java logs and traces. Today, we’re going to focus on your Java application logs.

Logging in Java could be done just by easily writing data to a file, however, this is not the simplest or most convenient way of logging. There are frameworks for Java that provide the object, methods, and unified configuration methods that help setting up logging, store logs, and in some cases even ship them to a log centralization solution. On top of that, there are also abstraction layers that enable easy switching of the underlying framework without the need of changing the implementation. This can come in handy if you ever need to replace your logging library for whatever reason – for example performance, configuration, or even simplicity.

Sounds complicated? It doesn’t have to be.

In this blog post, we will focus on how to properly set up logging for your code to avoid all those mistakes that we already did. We will cover:

*   Logging abstraction layers for Java
*   Out of the box Java logging capabilities
*   Java logging libraries
*   Logging the important information
*   Log centralization solutions.

Let’s get started!

Java Logging Frameworks
-----------------------

### SLF4J

The [Simple Logging Facade](http://www.slf4j.org/) for Java or simply [SLF4J](http://www.slf4j.org/) is an abstraction layer for various logging frameworks allowing us, as users, to choose the logging framework during the deploy time, rather than during the development time. This enables quick and easy change of logging framework of choice when it is needed.

If you’re new to SLF4J, we have just the blog post for you. In our [SLF4J tutorial](https://sematext.com/blog/slf4j-tutorial/), we explain everything there is to know about the abstraction layer so that you can log easier and smarter.

### Java.util.logging

The java.util.logging package provides the classes and interfaces for Java core logging facilities. It comes bundled with the Java Development Kit and is available for every developer using Java since Java 1.4.

### Logback

[Logback](http://logback.qos.ch/) started as an intended successor to the first version of the Log4j project claiming faster implementation, better tests, native SLF4J integration, and [more](http://logback.qos.ch/reasonsToSwitch.html). It is built out of three main modules and integrates with Servlet containers such as Tomcat or Jetty to provide HTTP-access log functionality. If this is your framework of choice, check out our [Logback tutorial](https://sematext.com/blog/logback-tutorial/) where we go over how it works and how to configure it for logging Java applications.

### Log4j

[Log4j](http://logging.apache.org/log4j/1.2/) is one of the most widely known Java logging libraries and the predecessor for such projects as Logback or Log4j 2. Log4j reached its end of life on 5th of August 2015 and users are recommended to use Log4j 2. If you are still using this framework, check out our [Log4j tutorial](https://sematext.com/blog/log4j-tutorial/) where we go over its functionality.

### Log4j 2

[Log4j 2](https://logging.apache.org/log4j/2.x/index.html) is the newest in the list of all the mentioned Java logging frameworks. It incorporates lots of improvements over its predecessor and promises to provide Logback improvements while fixing issues with some of its architectural problems. If you are starting a new project and you are looking for a library of choice Log4j 2 should be the main focus. You can read more about how to use it for logging Java applications from our [Log4j 2 tutorial](https://sematext.com/blog/log4j2-tutorial/).

Choosing the Right Logging Solution for Your Application
--------------------------------------------------------

The best logging solution for your Java application is….well, it’s complicated. You have to think and look at your current environment and organization’s needs. If you already have a framework of choice that the majority of your applications use – go for that. It is very likely that you will already have an established format for your logs. That also means that you may already have an easy way to ship your logs to a [log centralization solution](https://sematext.com/blog/best-log-management-tools/) of your choice, and you just need to follow the existing pattern.

However, if your organization does not have any kind of common logging framework, go and see what kind of logging is used in the application you are using. Are you using [Elasticsearch](https://sematext.com/guides/elasticsearch/)? It uses [Log4j 2](https://logging.apache.org/log4j/2.x/). Do the majority of the third party applications also use [Log4j 2](https://logging.apache.org/log4j/2.x/)? If so, consider using it. Why? Because you will probably want to have your logs in one place and it will be easier for you to just work with a single configuration or pattern. However, there is a pitfall here. Don’t go that route if it means using an outdated technology when a newer and mature alternative already exists.

Finally, if you select a new logging framework I suggest using an abstraction layer and a logging framework. Such an approach gives you the flexibility to switch to a different logging framework when needed and, what’s most important, without needing to change the code. You will only have to update the dependencies and configuration, the rest will stay the same.

In most cases, the [SLF4J](http://www.slf4j.org/) with bindings to the logging framework of your choice will be a good idea. The [Log4j 2](https://logging.apache.org/log4j/2.x/) is the go-to framework for many projects out there both open and closed source ones.

Logging in Java: How to Log Using the Java Logging API
------------------------------------------------------

Java contains the Java logging API which allows you to configure what type of log messages are written. The API comes with a standard set of key elements that we should know about to be able to proceed and discuss more than just the basic logging. The java.util.logging package provides the Logger class to log application messages.

### Logger

The **logger** is the main entity that an application uses to make logging calls, to capture LogRecords to the appropriate handler. The **LogRecord** (or logging event) is the entity used to pass the logging requests between the framework that is used for logging and the handlers that are responsible for log shipping.

The Logger object is usually used for a single class or a single component to provide context-bound to a specific use case.

Adding logging to our Java application is usually about configuring the library of choice and including the Logger. That allows us to add logging into the parts of our application that we want to know about.

### Handler

The **handler** is used to export the **LogRecord** entity to a given destination. Those destinations can be the memory, console, files, and remote locations via sockets and various APIs. There are different standard handlers. Below are a few examples of such a handlers:

*   **ConsoleHandler**
*   **FileHandler**
*   **SyslogHandler**

The **ConsoleHandler** is designed to send your log messages to the standard output. The **FileHandler** writes the log messages to a dedicated file and the **SyslogHandler** sends the data to the [Syslog](https://sematext.com/blog/what-is-syslog-daemons-message-formats-and-protocols/) compatible daemon.

Turning any handler on and off is just a matter of including it in the configuration file for the logging library of your choice. You don’t have to do anything more.

You can also create your own handler by extending the **Handler** class. Here’s a very simple example:

```null
public class ExampleHandler extends Handler {
  @Override
  public void publish(LogRecord logRecord) {
    System.out.println(String.format("Log level: %s, message: %s",
        logRecord.getLevel().toString(), logRecord.getMessage()));
  }

  @Override
  public void flush() {
  }

  @Override
  public void close() throws SecurityException {
  }
}
```

### Logging Levels

**Java log levels** are a set of standard severities of the logging message. Tells how important the log record is. If you’re new to the topic, make sure to check out our guide on [log levels](https://sematext.com/blog/logging-levels/) where dive deeper into the subject.

For example, the following log levels are sorted from least to most important with some explanation on how I see them:

**TRACE** – Very fine-grained information only used in a rare case where you need the full visibility of what is happening. In most cases, the TRACE level will be very verbose, but you can also expect a lot of information about the application. Use for annotating the steps in an algorithm that are not relevant in everyday use.

```null
LOGGER.trace("This is a TRACE log level message");
```

**DEBUG** – Less granular than **TRACE** level, but still more granular than you should need in your normal, everyday use. The DEBUG level should be used for information that can be useful for troubleshooting and is not needed for looking at the everyday application state.

```null
LOGGER.debug("This is a DEBUG log level message");
```

**INFO** – The standard level of log information that indicates normal application action – for example “Created a user {} with id {}” is an example of a log message on an INFO level that gives you information about a certain process that finished with a success. In most cases, if you are not looking into how your application performs you could ignore most if not all of the INFO level logs.

```null
LOGGER.info("This is a INFO log level message");
```

**WARN** – Log level that usually indicates a state of the application that might be problematic or that it detected an unusual execution. Something may be wrong, but it doesn’t mean that the application failed. For example, a message was not parsed correctly, because it was not correct. The code execution is continuing, but we could log that with the WARN level to inform us and others that potential problems are happening.

```null
LOGGER.warn("This is a WARN log level message");
```

**ERROR** – Log level that indicates an issue with a system that prevents certain functionality from working. For example, if you provide login via social media as one way of logging into your system, the failure of such a module is an **ERROR** level log for sure.

```null
LOGGER.error("This is a ERROR log level message");
```

**FATAL** – The log level indicating that your application encountered an event that prevents it from working or a crucial part of it from working. A **FATAL** log level is, for example, an inability to connect to a database that your system relies on or to an external payment system that is needed to check out the basket in your e-commerce system. The **FATAL** error doesn’t have the representation in SLF4J.

### Formatter

The formatter supports the formatting of the **LogRecord** objects. By default, two formatters are available:

*   [SimpleFormatter](https://docs.oracle.com/en/java/javase/14/docs/api/java.logging/java/util/logging/SimpleFormatter.html) which prints the **LogRecord** object in a human-readable form using one or two lines
*   [XMLFormatter](https://docs.oracle.com/en/java/javase/14/docs/api/java.logging/java/util/logging/XMLFormatter.html) which writes the messages in the standard XML format.

You can also build your own custom formatter. Here’s an example:

```null
public class TestFormatter extends Formatter {

  @Override
  public String format(LogRecord logRecord) {
    return String.format("Level: %s, message: %s",
        logRecord.getLevel(), logRecord.getMessage());
  }
}
```

The Abstraction Layers
----------------------

Of course, your application can use the basic logging API provided out of the box by Java via the [java.util.logging](https://docs.oracle.com/en/java/javase/14/docs/api/java.logging/java/util/logging/package-summary.html) package. There is nothing wrong with that, but note that this will limit what you can do with your logs. Certain libraries provide easy to configure formatters, out-of-the-box industry standard destinations, and top-notch performance. If you wish to use one of those frameworks and you would also like to be able to switch the framework in the future, you should look at the abstraction layer on top of the logging APIs.

[SLF4J](http://www.slf4j.org/) – The Simple Logging Facade for Java is one such abstraction layer. It provides bindings for common logging frameworks such as [Log4j](https://logging.apache.org/log4j/2.x/), [Logback](http://logback.qos.ch/), and the out-of-the-box [java.util.logging](https://docs.oracle.com/en/java/javase/14/docs/api/java.logging/java/util/logging/package-summary.html) package. You can imagine the process of writing the log message in the following, simplified way:

![](_assets/java-logging-2.png.webp)

But how would that look from the code perspective? Well, that’s a very good question. Let’s start by looking at the out-of-the-box [java.util.logging](https://docs.oracle.com/en/java/javase/14/docs/api/java.logging/java/util/logging/package-summary.html) code. For example, if we would like to just start our application and print something to the log it would look as follows:

```null
package com.sematext.blog.logging;

import java.util.logging.Level;
import java.util.logging.Logger;

public class JavaUtilsLogging {
  private static Logger LOGGER = Logger.getLogger(JavaUtilsLogging.class.getName());

  public static void main(String[] args) {
    LOGGER.log(Level.INFO, "Hello world!");
  }
}
```

You can see that we initialize the static Logger class by using the class name. That way we can clearly identify where the log message comes from and we can reuse a single Logger for all the log messages that are generated by a given class.

The output of the execution of the above code will be as follows:

```null
Jun 26, 2020 2:58:40 PM com.sematext.blog.logging.JavaUtilsLogging main
INFO: Hello world!
```

Now let’s do the same using the SLF4J abstraction layer:

```null
package com.sematext.blog.logging;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class JavaSLF4JLogging {
  private static Logger LOGGER = LoggerFactory.getLogger(JavaSLF4JLogging.class);

  public static void main(String[] args) {
    LOGGER.info("Hello world!");
  }
}
```

This time the output is slightly different and looks like this:

```null
[main] INFO com.sematext.blog.logging.JavaSLF4JLogging - Hello world!
```

This time, things are a bit different. We use the LoggerFactory class to retrieve the logger for our class and we use a dedicated info method of the Logger object to write the log message using the INFO level.

Starting with [SLF4J](http://www.slf4j.org/) 2.0 the [fluent logging API](http://www.slf4j.org/manual.html#fluent) was introduced, but at the time of writing of this blog it is still in alpha and is considered experimental, so we will skip it for now as the API may change.

So to finish up with the abstraction layer – it gives us a few major advantages compared to using the [java.util.logging](https://docs.oracle.com/en/java/javase/14/docs/api/java.logging/java/util/logging/package-summary.html):

*   A common API for all your application logging
*   An easy way to use the desired logging framework
*   An easy way to exchange the logging framework and not having to go through the whole code when wanting to switch

Hopefully, that sheds some light on what the log levels are and how you can use them in your application.

How Do You Enable Logging: java.util.logging Example
----------------------------------------------------

When using the standard [java.util.logging](https://docs.oracle.com/en/java/javase/14/docs/api/java.logging/java/util/logging/package-summary.html) package we don’t need any kind of external dependencies. Everything that we need is already present in the JDK distribution so we can just jump on it and start including logging to our awesome application.

There are two ways we can include and configure logging using the [java.util.logging](https://docs.oracle.com/en/java/javase/14/docs/api/java.logging/java/util/logging/package-summary.html) package – by using a configuration file or programmatically. For the demo purposes let’s assume that we want to see our log messages in a single line starting with a date and time, severity of the log message and of course the log message itself.

### Configuring java.util.logging via Configuration File

To show you how to use the [java.util.logging](https://docs.oracle.com/en/java/javase/14/docs/api/java.logging/java/util/logging/package-summary.html) package we’ve created a simple Java project and shared it in our [Github](https://github.com/sematext/blog-java_logging/tree/master/configjul) repository. The code that generates the log and sets up our logging looks as follows:

```null
package com.sematext.blog.loggging;
import java.io.InputStream;
import java.util.logging.Level;
import java.util.logging.LogManager;
import java.util.logging.Logger;
public class UtilLoggingConfiguration {
  private static Logger LOGGER = Logger.getLogger(UtilLoggingConfiguration.class.getName());
  static {
    try {
      InputStream stream = UtilLoggingConfiguration.class.getClassLoader()
          .getResourceAsStream("logging.properties");
      LogManager.getLogManager().readConfiguration(stream);
    } catch (Exception ex) {
      ex.printStackTrace();
    }
  }
  public static void main(String[] args) throws Exception {
    LOGGER.log(Level.INFO, "An INFO level log!");
  }
}
```

We start with initializing the Logger for our UtilLoggingConfiguration class by using the name of the class. We also need to provide the name of our logging configuration. By default the default configuration lives in the JAVA\_HOME/jre/lib/logging.properties file and we don’t want to adjust the default configuration. Because of that, I created a new file called logging.properties and in the static block we just initialize it and provide it to the LogManager class by using its readConfiguration method. That is enough to initialize the logging configuration and results in the following output:

```null
[2020-06-29 15:34:49] [INFO ] An INFO level log!
```

Of course, you would only do the initialization once and not in each and every class. Keep that in mind.

Now that is different from what we’ve seen earlier in the blog post and the only thing that changed is the configuration that we included. Let’s now look at our logging.properties file contents:

```null
handlers=java.util.logging.ConsoleHandler java.util.logging.ConsoleHandler.formatter=java.util.logging.SimpleFormatter java.util.logging.SimpleFormatter.format=[%1$tF %1$tT] [%4$-7s] %5$s %n
```

You can see that we’ve set up the handler which is used to set the log records destination. In our case, this is the **java.util.logging.ConsoleHandler** that prints the logs to the console.

Next, we said what format we would like to use for our handler and we’ve said that the formatter of our choice is the **java.util.logging.SimpleFormatter**. This allows us to provide the format of the log record.

Finally by using the format property we set the format to **\[%1$tF %1$tT\] \[%4$-7s\] %5$s %n**. That is a very nice pattern, but it can be confusing if you see something like this for the first time. Let’s discuss it. The **SimpleFormatter** uses **String.format** method with the following list or arguments: (format, date, source, logger, level, message, thrown).

So the **\[%1$tF %1$tT\]** part tells the formatter to take the second argument and provide the date and time parts. Then the **\[%4$-7s\]** part reads the log level and uses 7 spaces for formatting that. So for the INFO level, it will add 3 additional spaces. The **%5$s** tells the formatter to take the message of the log record and print it as a string and finally, the %n is the new line printing. Now that should be clearer.

### Configuring java.util.logging Programmatically

Let’s now look at how to do similar formatting by configuring the formatter from the Java code level. We won’t be using any kind of properties file this time, but we will set up our Logger using Java.

The code that does that looks as follows:

```null
package com.sematext.blog.loggging;
import java.util.Date;
import java.util.logging.*;
public class UtilLoggingConfigurationProgramatically {
  private static Logger LOGGER = Logger.getLogger(UtilLoggingConfigurationProgramatically.class.getName());
  static {
    ConsoleHandler handler = new ConsoleHandler();
    handler.setFormatter(new SimpleFormatter() {
      private static final String format = "[%1$tF %1$tT] [%2$-7s] %3$s %n";
      @Override
      public String formatMessage(LogRecord record) {
        return String.format(format,
            new Date(record.getMillis()),
            record.getLevel().getLocalizedName(),
            record.getMessage()
        );
      }
    });
    LOGGER.setUseParentHandlers(false);
    LOGGER.addHandler(handler);
  }
  public static void main(String[] args) throws Exception {
    LOGGER.log(Level.INFO, "An INFO level log!");
  }
}
```

The difference between this method and the one using a configuration file is this static block. Instead of providing the location of the configuration file, we are creating a new instance of the ConsoleHandler and we override the formatMessage method that takes the LogRecord object as its argument. We provide the format, but we are not passing the same number of arguments to the String.format method, so we modified our format as well. We also said that we don’t want to use the parent handler which means that we would only like to use our own handler and finally we are adding our handler by calling the LOGGER.addHandler method. And that’s it – we are done. The output of the above code looks as follows:

```null
[2020-06-29 16:25:34] [INFO ] An INFO level log!
```

I wanted to show that setting up logging in Java can also be done programmatically, not only via configuration files. However, in most cases, you will end up with a file that configures the logging part of your application. It is just more convenient to use, simpler to adjust, modify, and work with.

Efficient Java Logging with Log Management Solutions
----------------------------------------------------

Now you know the basics about how to turn on [logging](https://sematext.com/guides/log-management/) in our Java application but with the complexity of the applications, the volume of the logs grows. You may get away with logging to a file and only using them when troubleshooting is needed, but working with huge amounts of data quickly becomes unmanageable and you should end up using a [log management solution](https://sematext.com/blog/best-log-management-tools/) for [log monitoring](https://sematext.com/log-monitoring/) and centralization. You can either go for an in-house solution based on the open-source software or use one of the products available on the market like [Sematext Logs](https://sematext.com/logsene/).

![](_assets/java-logging-3.png.webp)

A fully managed log centralization solution such as [Sematext Logs](https://sematext.com/logsene/) will give you the freedom of not needing to manage yet another, usually quite complex, part of your infrastructure. It will allow you to manage a plethora of sources for your logs. You can learn more about Sematext and how it stacks up against similar solutions from our reviews of the best [log management software](https://sematext.com/blog/best-log-management-tools/), [log analysis tools](https://sematext.com/blog/log-analysis-tools/), and [cloud logging services](https://sematext.com/blog/cloud-logging-services/) available out there.

You may want to include logs like JVM [garbage collection logs](https://sematext.com/blog/java-garbage-collection-logs/) in your managed log solution. After [turning them on](https://sematext.com/blog/java-garbage-collection/) for your applications and systems working on the JVM you will want to have them in a single place for correlation, [analysis](https://sematext.com/blog/log-analysis/), and to help you [tune the garbage collection](https://sematext.com/blog/java-garbage-collection-tuning/) in the JVM instances. Alert on logs, [aggregate](https://sematext.com/blog/log-aggregation/) the data, save and re-run the queries, hook up your favorite incident management software.

Read more tips about how to efficiently write Java logs from our blog post about [Java logging best practices](https://sematext.com/blog/java-logging-best-practices/).

Or you can check out this quick overview of Sematext Logs and see how you can easily centralize your log data.

Conclusion
----------

Logging is invaluable when troubleshooting Java applications. In fact, logging is invaluable in troubleshooting in general, no matter if that is a Java application or a hardware switch or firewall. Both the software and hardware give us a look into how they are working in the form of parametrized logs enriched with contextual information.

In this article, we started with the basics of logging in your Java applications. We’ve learned what are the options when it comes to Java logging, how to add logging to your application, how to configure it.

I hope this article gave you an idea of how to deal with Java application logs and why you should start working with them right away if you haven’t already. And don’t forget, the key to efficient application logging is using a good log management tool. Try [Sematext Logs](https://sematext.com/logsene/) to see how much easier things could be. There’s a 14-day free trial available for you to explore all its features. Good luck!