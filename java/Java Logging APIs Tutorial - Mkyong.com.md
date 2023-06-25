# Java Logging APIs Tutorial - Mkyong.com
[![](_assets/fslogo-green.svg)
](https://freestar.com/?utm_campaign=branding&utm_medium=banner&utm_source=mkyong.com&utm_content=mkyong_top_leaderboard)

![](_assets/622c70d2908e68ecc070ca6754245bb2.jpg)

This tutorial shows logging using the Java built-in logging APIs in the `java.util.logging` package.

Table of contents

*   [1\. Java Logging APIs Hello World](#java-logging-apis-hello-world)
*   [2\. Logging Levels](#logging-levels)
*   [3\. logging.properties](#loggingproperties)
*   [4\. java.util.logging.config.file](#javautilloggingconfigfile)
*   [5\. Why choose java.util.logging](#why-choose-javautillogging)
*   [6\. Download Source Code](#download-source-code)
*   [7\. References](#references)

_P.S The `java.util.logging` is bundled with the Java since JDK 1.4._

[](#java-logging-apis-hello-world)1\. Java Logging APIs Hello World  

----------------------------------------------------------------------

1.1 A Java Logging APIs or `java.util.logging` hello world example, logs some messages and display them in the console.

HelloWorld.java

```null

package com.mkyong;

import java.util.logging.Level;
import java.util.logging.Logger;

public class HelloWorld {

    private static Logger logger = Logger.getLogger(HelloWorld.class.getName());

    public static void main(String[] args) {

        
        

        
        logger.info("This is level info logging");

        logger.log(Level.WARNING, "This is level warning logging");

        logger.log(Level.SEVERE, "This is level severe logging");

        System.out.println("Hello Java Logging APIs.");
    }

}

```

Output

Terminal

```null

Jun 05, 2021 1:35:08 PM com.mkyong.HelloWorld main
INFO: This is level info logging
Jun 05, 2021 1:35:08 PM com.mkyong.HelloWorld main
WARNING: This is level warning logging
Jun 05, 2021 1:35:08 PM com.mkyong.HelloWorld main
SEVERE: This is level severe logging
Hello Java Logging APIs.

```

[](#logging-levels)2\. Logging Levels  

----------------------------------------

2.1 There are 7 logging levels in `java.util.logging` package :

*   SEVERE (highest value)
*   WARNING
*   INFO (default)
*   CONFIG
*   FINE
*   FINER
*   FINEST (lowest value)

2.2 If we set the logging level to `SEVERE`, it will log messages equal to or above the `SEVERE` level. And since the `SEVERE` is the highest logging level, it logs messages which is `SEVERE` level only.

HelloWorld2.java

```null

package com.mkyong;

import java.util.logging.Level;
import java.util.logging.Logger;

public class HelloWorld2 {

    private static Logger logger = Logger.getLogger(HelloWorld.class.getName());

    public static void main(String[] args) {

        
        logger.setLevel(Level.SEVERE);

        
        logger.info("This is level info logging");

        logger.log(Level.WARNING, "This is level warning logging");

        logger.log(Level.SEVERE, "This is level severe logging");

        System.out.println("Hello Java Logging APIs.");
    }

}

```

Output

Terminal

```null

Jun 05, 2021 1:41:07 PM com.mkyong.HelloWorld main
SEVERE: This is level severe logging
Hello Java Logging APIs.

```

2.3 If we set the logging level to `WARNING`, it will log messages equal to or above the `WARNING` level, which are `Level.SEVERE` and `Level.WARNING`.

```null

  
  logger.setLevel(Level.WARNING);

```

Output

Terminal

```null

Jun 05, 2021 5:08:11 PM com.mkyong.HelloWorld main
WARNING: This is level warning logging
Jun 05, 2021 5:08:11 PM com.mkyong.HelloWorld main
SEVERE: This is level severe logging
Hello Java Logging APIs.

```

[](#loggingproperties)3\. logging.properties  

-----------------------------------------------

The Java logging APIs (`java.util.logging`) default loads [logging.properties](https://mkyong.com/logging/logging-properties-example/) in the `$JAVA_HOME/jre/lib/` (Java 8 and before); for Java 9 and above, the `logging.properties` file moved to `$JAVA_HOME/conf`.

3.1 Below is a `logging.properties` sample, which logs messages to both console and a file.

logging.properties

```null


handlers= java.util.logging.FileHandler, java.util.logging.ConsoleHandler

.level= SEVERE


java.util.logging.FileHandler.pattern = %h/java%u.log
java.util.logging.FileHandler.limit = 50000
java.util.logging.FileHandler.count = 1
java.util.logging.FileHandler.formatter = java.util.logging.XMLFormatter


java.util.logging.ConsoleHandler.level = INFO
java.util.logging.ConsoleHandler.formatter = java.util.logging.SimpleFormatter

java.util.logging.SimpleFormatter.format=[%1$tc] %4$s: %5$s %n


com.mkyong.level = SEVERE

```

3.2 We can put the `logging.properties` in the `resources` folder (Maven standard directory), and use the system property `java.util.logging.config.file` to define the location of `logging.properties`.

![](_assets/jul-logging-properties.png)

LoadLogPropertiesFile.java

```null

package com.mkyong;

import java.util.logging.Level;
import java.util.logging.Logger;

public class LoadLogPropertiesFile {

  static {
      
      String path = LoadLogPropertiesFile.class.getClassLoader()
                        .getResource("logging.properties").getFile();
      System.setProperty("java.util.logging.config.file", path);
  }

  private static Logger logger = Logger.getLogger(LoadLogPropertiesFile.class.getName());

  public static void main(String[] args) {

      logger.fine("This is level fine logging");

      
      logger.info("This is level info logging");

      logger.log(Level.SEVERE, "This is level severe logging");

      logger.log(Level.WARNING, "This is level warning logging");

      System.out.println("Hello Java Logging APIs.");
  }

}

```

Output in console using `SimpleFormatter`.

Terminal

```null

[Sat Jun 05 14:04:18 MYT 2021] SEVERE: This is level severe logging
Hello Java Logging APIs.

```

Output in a file using `XMLFormatter`.

$user\_home/java0.log

```null

<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<!DOCTYPE log SYSTEM "logger.dtd">
<log>
<record>
<date>2021-06-05T06:04:18.276214600Z</date>
<millis>1622873058276</millis>
<nanos>214600</nanos>
<sequence>0</sequence>
<logger>com.mkyong.LoadLogPropertiesFile</logger>
<level>SEVERE</level>
<class>com.mkyong.LoadLogPropertiesFile</class>
<method>main</method>
<thread>1</thread>
<message>This is level severe logging</message>
</record>
</log>

```

[](#javautilloggingconfigfile)4\. java.util.logging.config.file  

------------------------------------------------------------------

The system property `java.util.logging.config.file` defined the location of `logging.properties`.

In command line, we can use system property `java.util.logging.config.file` to define the location of a `logging.properties` file at runtime.

Terminal

```null

$ java -jar -Djava.util.logging.config.file=/path/logging.properties server.jar

```

[](#why-choose-javautillogging)5\. Why choose java.util.logging  

------------------------------------------------------------------

This Java logging APIs or `java.util.logging` is part of the Java JDK, suitable for the simple project which required only basic logging features like logs to console or a file.

For more advanced logging features like log file rotation, sending email for error logs, send logs to a database, etc. There are many third-party logging frameworks like the logging facade [SLF4j](http://www.slf4j.org/), or logging framework like [Logback](http://logback.qos.ch/) and [Apache Log4j 2](https://logging.apache.org/log4j/2.x/).

[](#download-source-code)6\. Download Source Code  

----------------------------------------------------

[](#references)7\. References  

--------------------------------

*   [JavaDoc java.util.logging](https://docs.oracle.com/en/java/javase/11/docs/api/java.logging/java/util/logging/package-summary.html)
*   [Java Logging Overview](https://docs.oracle.com/en/java/javase/11/core/java-logging-overview.html)
*   [Stackoverflow- Why are the Level.FINE logging messages not showing?](https://stackoverflow.com/questions/6315699/why-are-the-level-fine-logging-messages-not-showing)
*   [Ultimate Guide to Logging](https://www.loggly.com/ultimate-guide/java-logging-basics/)
*   [Java – Read a file from resources folder](https://mkyong.com/java/java-read-a-file-from-resources-folder/)
*   [logging.properties example](https://mkyong.com/logging/logging-properties-example/)
*   [How to load logging.properties for java.util.logging](https://mkyong.com/logging/how-to-load-logging-properties-for-java-util-logging/)
*   [tinylog Tutorial](https://mkyong.com/logging/tinylog-tutorial/)

![](_assets/622c70d2908e68ecc070ca6754245bb2.jpg)

#### Comments