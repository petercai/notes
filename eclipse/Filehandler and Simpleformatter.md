# Filehandler and Simpleformatter
1\. Logging Handlers in Java
----------------------------

The Java Logger directs the information to be captured to Handlers. The Logger has the capability of information filtering based on the Logging Level set to it. The same way, Handler also capable of filtering the messages. We call this as 2nd level of Log Filtering. One can attach the Logger with multiple Handlers. There are different flavors of Handlers support available in Java. They are:

1.  Console Handler
2.  File Handler
3.  Socket Handler
4.  Memory Handler
5.  Stream Handler

The _**“Console Handler”**_ produces the Log output to Console Window by directing the Log records to System.Err. When the Handler is not set with Log Level, it defaults to INFO. The same way, the default formatter of Console Handler is SimpleFormatter.

The _**“File Handler”**_ produces the Log output to a flat file in the file system. It has the capability of generating the “Rotating File Set” when a log file grows to a certain extent. Unlike the Console Handler the default Logging Level is “ALL” and the default formatter is “XML Formatter”.

When we want to publish the log record to a dedicated machine, the _**“Socket Handler”**_ is the solution for it. Application designer choose this handler when they want to capture huge volume of logs. These log entries are directed to a dedicated machine so that logs are maintained there.

In the above Handlers, Console and File are the most used ones. In this example, we will use “FileHandler” to capture the logging output in a Rotating Set of files.

2\. Logging Formatters
----------------------

We can attach the Formatter to a Handler. There should be only one Formatter for a Handler and java won’t allow more than one Formatter for a Handler. Be that as it may, the Logger allows multiple Handlers and thereby we can attach multiple Formatter to a Logger.

We use Formatter to arrange the Logging output in such a way so that it is easily readable. Java supports two kinds of Formatter. One is _**"SimpleFormatter"**_ and other one _**"XMLFormatter"**_. SimpleFormatter is useful for representing the output in an Ascii Standard Text Files whereas the XMLFormatter arranges the log output in the XML File. In this example, we will look at the SimpleFormatter and how it formats the output in the Text File.

![](https://images.saymedia-content.com/.image/c_limit%2Ccs_srgb%2Cq_auto:eco%2Cw_700/MTc0NDcyNzk4MDcxMTcwNDA4/java-examples-logging-with-filehandler-and-simpleformatter.png)

Default Logging of Java

Author

Look at the above illustration. Here, we do not have any explicit Formatter and Handler. The Application sends the Log request to Logger and the Logger produces the output.

3\. Logging Components Together
-------------------------------

Now we know the Components involved in Logging. Let us put this together and we will explore further. Have a look at the below illustration:

![](https://images.saymedia-content.com/.image/c_limit%2Ccs_srgb%2Cq_auto:eco%2Cw_700/MTc0NjQ3MDYzOTUzNDgzNTM1/java-examples-logging-with-filehandler-and-simpleformatter.png)

Logging Component Together - A Design Model

Author

This is one of several possibilities of deployment model of a Logging system. Moreover, in the above model we can see One Application and One Logger. When an Application want to write a Log Records, it sends that request to the Logger component.

As we already know, an application can attach a Logger to multiple Handlers and in this depiction, we can see that the Logger is attached with three different types of Handlers called Console Handler, FileHandler and SocketHandler. On the other hand, the Handler can be attached to only one Formatter.

A Handler can be attached to a SimpleFormatter or a XMLFormatter. In the above depiction, we can say that except Socket Handler, other Handlers are using the SimpleFormatter. The formatters take care of formatting the incoming Log message and generate the Final Log Output. Next, it hands over the Final Output to the Handler. The Handler produces the formatted Log Record to the receiver. In the depiction, the receiver of the Log Records are Socket Client, File and Console Window.

4\. The Code Example
--------------------

### 4.1 Package Inclusion

First, let us include the required packages for this example. The IOException Class is included from the java.io package to handle exceptions that may raise during the file handling. In this example, we will write our Log output to a disk file. We included IOException in order to handle any error on file operations. Next, we included all the classes from the Logging package and the code is below:

```
//Snippet 01: Package inclusion
import java.io.IOException;
import java.util.logging.*;
```

### 4.2 Create Logger and Set Log Level

We create the _**"LogManager"**_ instance from the static call to getLogManager() method. Then, we get the **_Logger_** from it by making use of getLogger() method call. After this, we set Logging Level as ALL and this state that the Logger performs none of Log Message filtering. Below is the code:

```
//Snippet 02: Get the Log Manager Instance
LogManager lgMan = LogManager.getLogManager();

//Snippet 03: Get Logger from Log Manager
String LoggerName = Logger.GLOBAL_LOGGER_NAME;
Logger Logr = lgMan.getLogger(LoggerName);

//Snippet 04: Set the Log Level @ Logger
Logr.setLevel(Level.ALL);

```

### 4.3 Create FileHandler

The FileHandler Class helps in writing the Log content to a text file. In our example, we create the FileHanlder to write the log output to a text file in C:\\Temp path. Now look at the code below:

```
//Snippet 05: Create Handler and Set Formatter
FileHandler fh = new FileHandler("C:\\Temp\\TheLog_%g.log", 
100, 10);


```

The FileName is appended with %g and it specifies that the FileHanlder should create “Rotating Set of Files” when Log entries exceeds certain quota. The space limit is specified while creating the FileHandler. In the above example, we set this limit as 100 bytes which is passed to the constructor as second parameter.

Now when the file size crosses the 100 bytes, the FileHandler will create one more file by increasing the number in the place holder of %g. The last parameter specifies that maximum limit for the Rotating Set of Files which is 10 in our case. It means maximum 10 files will be used for Logging. In our case, when the 10th log is full with 100 bytes, the FileHandler will overwrite the very first log file (Old content). Because of this behavior, we call the log files are Rotating Set of Files. Look at the Depiction below:

![](https://images.saymedia-content.com/.image/c_limit%2Ccs_srgb%2Cq_auto:eco%2Cw_700/MTc0NjQ3MDYzOTUzMzUyNDYz/java-examples-logging-with-filehandler-and-simpleformatter.png)

FileHandler with Rotating Set of Files

Author

In left side of depiction, we see that File Handler created two files TheLog\_1 and TheLog\_2. Moreover, it is still writing the content in TheLog\_0. To put it differently, we can say the oldest Log content is in TheLog\_2 and latest content is in TheLog_1. Sooner or later, the Log writing ends with the stage as shown in the center circle in the depiction. Here comes the number of File Limit.

In our example, we set maximum File Limit as 10 and when the 10 Log File crosses the 100 bytes limit; the FileHandler deletes the content in the old File. As a result, the oldest content in the File TheLog_9 gets deleted and new Log contents are written to it. This is shown in the third circle. Here, the FileHandler writes the Log content into 10 files by reusing it (Rotating it). It is always a good practice to make use of the time stamp in the Log entry when the Log files are analyzed

### 4.4 Attach Formatter to Handler

In our example, First, we are creating _**“SimpleFormatter”**_ which suits for text based formatting. Next, the Formatter object is linked to the FileHandler which was initiated recently. The method _**"setFormatter()"**_ takes Formatter as object and the Formatter can be Simple Formatter or XML Formatter. Notably, one can attach only one Formatter for a FileHandler. For example, in our example we attached the FileHandler to SimpleFormatter and now, it is not possible to attach it to XML Handler

We set Logging Level as FINEST at the handler level using _**"setLevel"**_ method. Now, we have two Logging Levels set with our Logging System example. The first one is at Logger and it is Level.ALL and the other one is here at FileHandler which is set to FINE. As a result, even though the Logger allows all Logging messages, the Sub-System which is the FileHandler here filters the FINER and FINEST Logging messages. The code is below:

```
fh.setFormatter(new SimpleFormatter());
fh.setLevel(Level.FINE);

```

### 4.5 Attach FileHandler with Logger

Now, our FileHandler is ready, and it is attached to the Formatter as well. We will attach this handler to the logger object which we created earlier. Below is the code:

```
//Snippet 06: Add the File Handler to Logger
Logr.addHandler(fh);
```

### 4.6 Log Different Types of Messages

Now our is Logger is ready with Handler and the Formatter and we will write some sample Log Messages through our Logging System. Below is the code which attempts logging the message through our Logging example:

```
//Snippet 05: Test Log Entries with Different
//Logging level
//5.1: Log a Fatal Error
Logr.log(Level.SEVERE, "Fatal Error 17: Message" );

//5.2: Log Some Warning Messages
Logr.log(Level.WARNING, "Warning 1: Warning Message");
Logr.log(Level.WARNING, "Warning 2: Warning Message");

//5.3: Log Some Informational Messages
Logr.log(Level.INFO , "Info 1: The Message");
Logr.log(Level.INFO , "Info 2: The Message");
Logr.log(Level.INFO , "Info 3: The Message");
Logr.log(Level.INFO , "Info 4: The Message");
Logr.log(Level.INFO , "Info 5: The Message");
Logr.log(Level.INFO , "Info 6: The Message");

//5.4: Log Some Informational Messages
Logr.log(Level.FINE  , "Fine 1: The Message");
Logr.log(Level.FINE , "Fine 2: The Message");
Logr.log(Level.FINE , "Fine 3: The Message");

```

5\. Running the Example
-----------------------

In our example, the FileHandler makes use of SimpleFormatter. We must specify the format of the Log message output to the SimpleFormatter so that it will do its duty before producing the Log Records. In java -D switch is used to specify the formatting. Now look at the Table below which describes the place holder and its meaning as defined by the SimpleFormatter:

| Place-Holder | Meaning |
| --- | --- |
| 

1

 | 

Date and Time of Log Entry

 |
| 

2

 | 

Class and Method Name in which log method is called

 |
| 

3

 | 

Name of the Logger

 |
| 

4

 | 

Log Level of the Message (Ex: WARNING)

 |
| 

5

 | 

Actual Log Message Content

 |
| 

6

 | 

Exception Stack Trace Information

 |

Now Look at the output and also note how we specify the SimpleFormatter.Format as part of -D java option:

![](https://images.saymedia-content.com/.image/c_limit%2Ccs_srgb%2Cq_auto:eco%2Cw_700/MTc0NjQ3MDYzOTUzMjg2OTI3/java-examples-logging-with-filehandler-and-simpleformatter.png)

Specifying the Format for SimpleFormatter and Formatted output in Console Window

Authtor

Even though we don’t create any handler window for our logger it still picks-up the formatting. The reason is that every java application has default ConsoleHandler if it not created explicitly. Moreover, the default Formatter for the default ConsoleHandler is SimpleFormatter. To know more about these defaults look at the logging.properties in the JRE location (..\\JRE\\Lib). Now look at the output generated in the Rotating Set of Log Files:

![](https://images.saymedia-content.com/.image/c_limit%2Ccs_srgb%2Cq_auto:eco%2Cw_700/MTc0NjQ3MDYzOTUzNDE3OTk5/java-examples-logging-with-filehandler-and-simpleformatter.png)

Rotating Set of Log Files

Author

The complete example is below:

```
//Snippet 01: Package inclusion
import java.io.IOException;
import java.util.logging.*;

public class Main
{
    public static void main(String[] args)
    {
        //Snippet 02: Get the Log Manager Instance
        LogManager lgMan = LogManager.getLogManager();

        //Snippet 03: Get Logger from Log Manager
        String LoggerName = Logger.GLOBAL_LOGGER_NAME;
        Logger Logr = lgMan.getLogger(LoggerName);

        //Snippet 04: Set the Log Level @ Logger
        Logr.setLevel(Level.ALL);

        try
        {
            //Snippet 05: Create Handler and Set Formatter
            FileHandler fh = new 
                    FileHandler("C:\\Temp\\TheLog_%g.log", 
                    100, 10);
            fh.setFormatter(new SimpleFormatter());
            fh.setLevel(Level.FINE);

            //Snippet 06: Add the File Handler to Logger
            Logr.addHandler(fh);
        }
        catch(IOException Ex)
        {
            System.out.println(Ex.getMessage());
        }

        //Snippet 05: Test Log Entries with Different
        //Logging level
        //5.1: Log a Fatal Error
        Logr.log(Level.SEVERE, 
                "Fatal Error 17: Message" );

        //5.2: Log Some Warning Messages
        Logr.log(Level.WARNING, 
                "Warning 1: Warning Message");
        Logr.log(Level.WARNING, 
                "Warning 2: Warning Message");

        //5.3: Log Some Informational Messages
        Logr.log(Level.INFO , 
                "Info 1: The Message");
        Logr.log(Level.INFO , 
                "Info 2: The Message");
        Logr.log(Level.INFO , 
                "Info 3: The Message");
        Logr.log(Level.INFO , 
                "Info 4: The Message");
        Logr.log(Level.INFO , 
                "Info 5: The Message");
        Logr.log(Level.INFO , 
                "Info 6: The Message");

        //5.4: Log Some Informational Messages
        Logr.log(Level.FINE  , 
                "Fine 1: The Message");
        Logr.log(Level.FINE , 
                "Fine 2: The Message");
        Logr.log(Level.FINE , 
                "Fine 3: The Message");
    }
}

```

**© 2018 sirama**

**noora** on October 13, 2019:

Thank you very much. nice explanation .understood easily