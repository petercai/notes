# java.util.logging Example - Examples Java Code Geeks - 2022
In this article, we will discuss about the logging functionality in Java. Logging in simple words refers to the recording of an application activity. Logging is used to store exceptions, information, and warnings as messages that occur during the execution of a program. Logging helps a programmer in the debugging process of a program.

Java provides logging facility in the `java.util.logging` package. The package consists of a set of classes and interfaces which are used in logging. The System uses a [`Logger`](http://docs.oracle.com/javase/7/docs/api/java/util/logging/Logger.html "Logger") object to log messages.

The `Logger` object is allocated with a [`LogRecord`](http://docs.oracle.com/javase/7/docs/api/java/util/logging/LogRecord.html "LogRecord") object which stores the message to be logged. This `LogRecord` object is forwarded to all the handlers assigned to the `Logger` object. Both loggers and handlers can optionally use a [`Filter`](http://docs.oracle.com/javase/7/docs/api/java/util/logging/Filter.html "Filter") that is associated with them, to filter the log messages. Then, the handlers published the logged messages to the external system.

[![](_assets/logging.jpg.webp)
](https://examples.javacodegeeks.com/wp-content/uploads/2014/06/logging.jpg)

Logging Framework

Let’s start with some of the important classes of the package.  

1\. Logger and Level
--------------------

A `Logger` class is used to create a logger object which is used to log messages. A logger object is provided with a name and has a set of methods which are used to log messages at the different levels. Although you can provide any name to the logger, it is recommended to provide a name based on the package and the class name in which the logger is created.

There are seven logging levels provided by the [`Level`](http://docs.oracle.com/javase/7/docs/api/java/util/logging/Level.html "Level") class.

• SEVERE (highest level)  
• WARNING  
• INFO  
• CONFIG  
• FINE  
• FINER  
• FINEST (lowest level)

All these levels are present in the `Level` class, as a static final field. You can use any of these levels, according to the level of the message you log. Additionally, it also provides a level OFF that can be used to turn off the logging and a level ALL to turn on the logging for all the levels.

Let’s see an example about how to create and use a logger.

_LoggerExample.java_

```

```

*   If we run the above code, we will have the following results:

```

```

In the above example, we have created a logger object using the `getLogger` static method. Then we have logged messages at different levels. We also have thrown an [`ArrayIndexOutOfBoundsException`](http://docs.oracle.com/javase/7/docs/api/java/lang/ArrayIndexOutOfBoundsException.html "ArrayIndexOutOfBoundsException") to illustrate the use of the `Logger`.

Let’s have a look at some of the methods used in this example.

[`Logger.getLogger(String name)`](http://docs.oracle.com/javase/7/docs/api/java/util/logging/Logger.html#getLogger(java.lang.String) "Logger.getLogger"): This method is used to create or find a logger by the name passed as a parameter.

[`void info(String msg)`](http://docs.oracle.com/javase/7/docs/api/java/util/logging/Logger.html#info(java.lang.String) "info()"): This instance method is used to log an INFO message, if the logger is currently enabled for the INFO message else the logged message is gets ignored.

[`void warning(String msg)`](http://docs.oracle.com/javase/7/docs/api/java/util/logging/Logger.html#warning(java.lang.String) "warning()"): This instance method is used to log a WARNING message, if the logger is currently enabled for the WARNING message else the logged message is gets ignored.

[`void config(String msg)`](http://docs.oracle.com/javase/7/docs/api/java/util/logging/Logger.html#config(java.lang.String) "config()"): This instance method is used to log a CONFIG message, if the logger is currently enabled for the CONFIG message else the logged message is gets ignored.

[`void log(Level level, String msg, Object param1)`](http://docs.oracle.com/javase/7/docs/api/java/util/logging/Logger.html#log(java.util.logging.Level,%20java.lang.String,%20java.lang.Object) "log()"): This method is used to log a message with a given level, and with an [`Object`](http://docs.oracle.com/javase/7/docs/api/java/lang/Object.html "Object") as a parameter. You can use this method when you want to store an object in the log as done in the above example where we have logged an exception object at SEVERE level.

Please note that the INFO level is the default level set in the `Logger`. Any message logged with the level lower than the INFO gets ignored. As you can see, the message logged at the WARNING level gets ignored and it did not get published in the console.

2\. Handler
-----------

A [`Handler`](http://docs.oracle.com/javase/7/docs/api/java/util/logging/Handler.html "Handler") is one of the components of the logging framework. It is responsible for printing the log message at a target destination. The destination can be a console or a file. The `Handler` is used to take a log message in the form of a `LogRecord` object and export it to the target destination.

A `Logger` can be associated with one or more handlers that eventually forward the logged message to all the handlers. A `Handler` is an abstract class in the `java.util.logging` package which is a base class for all types of handlers in Java. There are 4 types of built-in handlers in Java.

[`ConsoleHandler`](http://docs.oracle.com/javase/7/docs/api/java/util/logging/ConsoleHandler.html "ConsoleHandler"): A `ConsoleHandler` records all the log messages to [`System.err`](http://docs.oracle.com/javase/7/docs/api/java/lang/System.html#err "System.err"). By default, a `Logger` is associated with this handler.

[`FileHandler`](http://docs.oracle.com/javase/7/docs/api/java/util/logging/FileHandler.html "FileHandler"): A `FileHandler` is used to record all the log messages to a specific file or to a rotating set of files.

[`StreamHandler`](http://docs.oracle.com/javase/7/docs/api/java/util/logging/StreamHandler.html "StreamHandler"): A `StreamHandler` publishes all the log messages to an [`OutputStream`](http://docs.oracle.com/javase/7/docs/api/java/io/OutputStream.html "OutputStream").

[`SocketHandler`](http://docs.oracle.com/javase/7/docs/api/java/util/logging/SocketHandler.html "SocketHandler"): The `SocketHandler` publish the `LogRecords` to a network stream connection.

[`MemoryHandler`](http://docs.oracle.com/javase/7/docs/api/java/util/logging/MemoryHandler.html "MemoryHandler"): It is used to keep the `LogRecords` into a memory buffer. If the buffer gets full, the new `LogRecords` starts overwriting the old `LogRecords`.

_HandlerExample.java_

```

```

*   If we run the above code, we will have the following results:

```

```

This example also generates a log file javacodegeeks.log at the root directory of this project.

[![](_assets/handler_example-300x113.jpg.webp)
](https://examples.javacodegeeks.com/wp-content/uploads/2014/06/handler_example.jpg)

Log generated by a handler

The file contains the following log.

```

```

In this example, we have logged messages to both the `FileHandler` and the `ConsoleHandler`. Let’s discuss the above example.

[`ConsoleHandler()`](http://docs.oracle.com/javase/7/docs/api/java/util/logging/ConsoleHandler.html#ConsoleHandler() "ConsoleHandler()"): A constructor that creates a `ConsoleHandler` for `System.err`.

[`FileHandler(String pattern)`](http://docs.oracle.com/javase/7/docs/api/java/util/logging/FileHandler.html#FileHandler(java.lang.String) "FileHandler()"): A constructor that creates a `FileHandler` to log messages in the given file name.

[`void addHandler(Handler handler)`](http://docs.oracle.com/javase/7/docs/api/java/util/logging/Logger.html#addHandler(java.util.logging.Handler) "addHandler()"): It’s an instance method from the `Logger` class which is used to assign a handler to the logger object. You can assign multiple handlers to a single logger object. Like in this example, we have assigned both `ConsoleHandler` and `FileHandler` to a single logger object.

[`void setLevel(Level newLevel)`](http://docs.oracle.com/javase/7/docs/api/java/util/logging/Logger.html#setLevel(java.util.logging.Level) "setLevel()"): This method is from the `Logger` and the `Handler` class. It sets the log level specifying which message levels will be logged by this logger. Message levels lower than the level set will be ignored.

[`void removeHandler(Handler handler)`](http://docs.oracle.com/javase/7/docs/api/java/util/logging/Logger.html#removeHandler(java.util.logging.Handler) "removeHandler()"): It is used to remove the associated handler from the logger object. Once the handler is removed, it will not be able to publish any further logs. In this example, we removed the `ConsoleHandler` and all messages following it did not get published in the console.

[`void finer(String msg)`](http://docs.oracle.com/javase/7/docs/api/java/util/logging/Logger.html#finer(java.lang.String) "finer()"): This instance method is used to log a FINER message, if the logger is currently enabled for the FINER message else the logged message gets ignored.

The log messages published by the `FileHandler` in the above example are in the XML format. It’s the default format of the `FileHandler`. We can change the format of the handler using a [`Formatter`](http://docs.oracle.com/javase/7/docs/api/java/util/logging/Formatter.html "Formatter"). In the next section, we will discuss about the `Formatter` class and its uses.

3\. Formatter
-------------

A `Formatter` is used to format a `LogRecord`. Each handler is associated with a formatter. Java provides the `Formatter` as a parent class of two in-built formatter’s i.e. [`SimpleFormatter`](http://docs.oracle.com/javase/7/docs/api/java/util/logging/SimpleFormatter.html "SimpleFormatter") and [`XMLFormatter`](http://docs.oracle.com/javase/7/docs/api/java/util/logging/XMLFormatter.html "XMLFormatter"). Let’s see some examples:

_FormatterExample.java_

```

```

*   If we run the above code, we will have the following results:

```

```

This example also generates a log file javacodegeeks.formatter.log in the root directory of this project.

[![](_assets/formatter_example.jpg.webp)
](https://examples.javacodegeeks.com/wp-content/uploads/2014/06/formatter_example.jpg)

Log generated by Formatter Example

In the above example, we have used the `SimpleFormatter` in our example, which prints the `LogRecord` in a simple human readable format. Please note that before setting the `SimpleFormatter` to the handler, we have logged a message which is published in an XML format, this is because the `XMLFormatter` is the default formatter for the `FileHandler`. Also note that the `LogRecord` is also published in the console because the `ConsoleHandler` is by default associated with the `Logger`.

[`SimpleFormatter()`](http://docs.oracle.com/javase/7/docs/api/java/util/logging/SimpleFormatter.html#SimpleFormatter() "SimpleFormatter()"): This constructor is used to create a `SimpleFormatter` object.

[`void setFormatter(Formatter newFormatter)`](http://docs.oracle.com/javase/7/docs/api/java/util/logging/Handler.html#setFormatter(java.util.logging.Formatter) "setFormatter()"): This method is from the `Handler` class and used to set the formatter to the handler.

4\. Filter
----------

A `Filter` is an interface in `java.util.logging` package. It is used to control the messages to be logged by the handler. Every `Logger` and `Handler` optionally can have a `Filter`. The `Filter` has a `isLoggable` method which returns a `boolean`. Before publishing the message the `Logger` or the `Handler` calls this method, if the method returns true the `LogRecord` gets publish else it gets ignored.

_FilterExample.java_

```

```

*   If we run the above code, we will have the following results:

```

```

[`void setFilter(Filter newFilter)`](http://docs.oracle.com/javase/7/docs/api/java/util/logging/Logger.html#setFilter(java.util.logging.Filter) "setFilter()"): This method sets a `Filter` which controls the output on this `Logger`.

[`boolean isLoggable(LogRecord record)`](http://docs.oracle.com/javase/7/docs/api/java/util/logging/Filter.html#isLoggable(java.util.logging.LogRecord) "isLoggable()"): This method is from the `Filter` interface which checks if a given `LogRecord` object should be published or not.

5\. Configuration
-----------------

You can provide configuration properties to a `Logger` using a configuration file. This helps you to remove the configuration from the code and provides an easy way to re-configure whenever it is required without changing the code again and again. This flexibility is provided by the [`LogManager`](http://docs.oracle.com/javase/7/docs/api/java/util/logging/LogManager.html "LogManager") class.

_ConfigurationExample.java_

```

```

This example reads a property file which contains the following properties:

[![](_assets/configuration_example-300x128.jpg.webp)
](https://examples.javacodegeeks.com/wp-content/uploads/2014/06/configuration_example.jpg)

Property file

```

```

*   If we run the above code, we will have the following results:

```

```

Let’s us discuss about the code and the configuration properties.

**handlers**: used to set the default handlers for all the loggers.  
**.level**: sets the default `Level` for all the loggers to ALL.  
**java.util.logging.ConsoleHandler.level**: sets the default `Level` for all the `ConsoleHandler` to ALL.  
**java.util.logging.ConsoleHandler.formatter**: sets the default formatter for the `ConsoleHandler` to a `SimpleFormatter`.  
**confLogger.level**: sets the default level of the `Logger` named confLogger to ALL.

Please note that you can override these properties in the code.

[`LogManager.getLogManager()`](http://docs.oracle.com/javase/7/docs/api/java/util/logging/LogManager.html#getLogManager() "LogManager.getLogManager()"): This is a static factory method used to get a `LogManager` object. There is a single global `LogManager` object that is used to maintain a set of shared state about the `Loggers` and the log services.

[`void readConfiguration(InputStream ins)`](http://docs.oracle.com/javase/7/docs/api/java/util/logging/LogManager.html#readConfiguration(java.io.InputStream) "readConfiguration()"): This method is used to reinitialize the logging properties and re-read the logging configuration from the given stream, which should be in [`java.util.Properties`](http://docs.oracle.com/javase/7/docs/api/java/util/Properties.html "Properties") format.

6\. Download the source code
----------------------------

You can download the source code of this example from here: [LoggingExample.zip](https://examples.javacodegeeks.com/wp-content/uploads/2014/06/LoggingExample.zip)