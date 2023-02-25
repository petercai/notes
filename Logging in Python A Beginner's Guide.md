# Logging in Python: A Beginner's Guide
Logging is an important aspect of any application. It is the process of providing information about the happenings in your application. If done well, It can be useful for helping a developer pinpoint reasons for app crashes, errors, possible bugs, and areas of poor performance faster.

In this article, we will learn how we can add logging into our python application. To follow this article, basic knowledge of python would be useful.

[Permalink](#heading-logging-components "Permalink")Logging components
----------------------------------------------------------------------

At a high level, logging in python has three main components. These are:

1.  **Loggers**: These are python objects that allow us to define the log messages.
    
2.  **Handlers**: These are objects that can be attached to a logger. They decide how to process a log.
    
3.  **Formatters**: These are objects that can be attached to a handler. They help to format the way logs are displayed.
    

[Permalink](#heading-loggers "Permalink")Loggers
------------------------------------------------

Python makes it easy to quickly get started with logging. Let us create an `app.py` file to see how it works.

```
import logging

LOGGER = logging.getLogger('foo.bar')

LOGGER.debug('Nothing to see here, Just trying to Debug')
LOGGER.info('Hey, here is some information you might find useful')
LOGGER.warning('something may be wrong here!!! warning')
LOGGER.error('An error definetely just happened. You might want to take a look!')
LOGGER.critical('Oops.. Critical server error.')

```

If we execute the file with `python3 app.py` on our terminal, we will get the below in our terminal.

```
something may be wrong here!!! warning
An error definetely just happened. You might want to take a look!
Oops.. Critical server error.

```

We use the logging module provided by python to create a `Logger` object and log messages onto our terminal.

But wait🤨, we only see three of the five logs we wrote, what's happening? Well, this is because of how logs are propagated. Python provides us with a default **root** logger which is the topmost logger. By default, loggers in python follow a hierarchical order, where logs are propagated upwards to their parents till the log reaches the root logger.

A logger named `foo.bar.baz` would be a child of a logger created at `foo.bar` and `foo`, and hence, will send logs upwards to them.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675104348411/e90eccea-ce1b-461b-ae37-4cec71d8e8e8.png?auto=compress,format&format=webp)

Now, the root logger has what we call an effective level of "_warning"_. An effective level is what a logger uses to decide if it will consider processing a log message. If a logger has an effective level of "_warning_", it will only consider processing logs of priority "warning" and above, a log message of priority "debug" or "info" will be ignored. This is exactly why we only saw three of our five logs being printed above.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675131347525/810809c9-60b9-4fd4-9dae-49d9a3691064.png?auto=compress,format&format=webp)

By default, we have the following priority levels provided.

| 

`CRITICAL`

 | 

50

 |
| 

`ERROR`

 | 

40

 |
| 

`WARNING`

 | 

30

 |
| 

`INFO`

 | 

20

 |
| 

`DEBUG`

 | 

10

 |
| 

`NOTSET`

 | 

0

 |

All loggers, except the root logger, are created with an effective level `NOTSET`. An effective level of `NOTSET` means the logger will delegate the decision of logging this message to its parent logger.

If we would like to log messages with a priority lower than that of our root logger's effective level. We need to explicitly create a handler for our logger.

[Permalink](#heading-handlers "Permalink")Handlers
--------------------------------------------------

Handlers assist in the decision-making of whether to process a log message based on the log's priority and its own specified priority level. To create one, edit your _[app.py](http://app.py/)_ file.

```
import logging

LOGGER = logging.getLogger('foo.bar')

stream_handler = logging.StreamHandler()
stream_handler.setLevel(logging.DEBUG)

LOGGER.addHandler(stream_handler)
LOGGER.setLevel(logging.DEBUG)

LOGGER.debug('Nothing to see here, Just trying to debug')
LOGGER.info('Hey, here is some information you might find useful')
LOGGER.warning('something may be wrong here!!! warning')
LOGGER.error('An error definitely just happened. You might want to take a look!')
LOGGER.critical('Oops.. Critical server error.')

```

We added a handler named `StreamHandler` to our logger, this handler will output the logs it receives to the terminal (by default, _stderr_). We set its level to `DEBUG` to allow it to process messages of priority "DEBUG" and higher. We also set the effective level of the logger object to "DEBUG", since, if you recall, all loggers (except root) have a default effective level of "NOTSET" making them unable to act on a log by themselves, and will propagate the log upwards.

Now if we execute our [app.py](http://app.py/) script

```
Nothing to see here, Just trying to debug
Hey, here is some information you might find useful
something may be wrong here!!! warning
An error definitely just happened. You might want to take a look!
Oops.. Critical server error.

```

Great! 😎 We see all our logs now. You can tweak the priority levels of both the logger and a handler to get your desired output.

### [Permalink](#heading-using-a-filehandler "Permalink")Using a FileHandler

We can also have multiple handlers on a single logger. Another common handler is the `FileHandler`. Let's see it in action, update your [app.py](http://app.py/)

```
import logging

LOGGER = logging.getLogger('foo.bar')

stream_handler = logging.StreamHandler()
stream_handler.setLevel(logging.DEBUG)

LOGGER.addHandler(stream_handler)
LOGGER.setLevel(logging.DEBUG)

file_handler = logging.FileHandler('app.log')
file_handler.setLevel(logging.DEBUG)
LOGGER.addHandler(file_handler)


LOGGER.debug('Nothing to see here, Just trying to debug')
LOGGER.info('Hey, here is some information you might find useful')
LOGGER.warning('something may be wrong here!!! warning')
LOGGER.error('An error definetely just happened. You might want to take a look!')
LOGGER.critical('Oops.. Critical server error.')

```

Execute the script and you should see a new file named `app.log` appear in your working directory. Inspecting it, you should see the logs written into it.

It is common practice to add multiple handlers with different priority levels to a handler. You might want all logs printed into the terminal using a `StreamHandler` but only want **error** and **critical** logs written to your log files. This can easily be achieved by setting the StreamHandler level to DEBUG and the FIleHandler's level to "ERROR".

### [Permalink](#heading-integrating-the-rotatingfilehandler "Permalink")Integrating the RotatingFileHandler

For large services with lots of logs being emitted, log files tend to get very large very quickly and this can be a problem as it gets harder to manage and can end up taking up a lot of the space on the machine. To solve this, we use a `RotatingFileHandler`. After our log file reaches a specified size, it will stop logging into the file and rename it usually appending an increasing integer value at the end of the file name, then creating an empty log file of the original name and resuming logging into that. So you will tend to see log files like `app.log, app.log.1, app.log.2, app.log.3`

Update the [app.py](http://app.py/) file

```
import logging
from logging.handlers import RotatingFileHandler 

LOGGER = logging.getLogger('foo.bar')

stream_handler = logging.StreamHandler()
stream_handler.setLevel(logging.DEBUG)

LOGGER.addHandler(stream_handler)
LOGGER.setLevel(logging.DEBUG)

file_handler = RotatingFileHandler('app.log', maxBytes=100, backupCount=5) 
file_handler.setLevel(logging.DEBUG)
LOGGER.addHandler(file_handler)

LOGGER.debug('Nothing to see here, Just trying to debug')
LOGGER.info('Hey, here is some information you might find useful')
LOGGER.warning('something may be wrong here!!! warning')
LOGGER.error('An error definetely just happened. You might want to take a look!')
LOGGER.critical('Oops.. Critical server error.')

```

We imported the `RotatingFileHandler` class and then instantiated it with the file name and two arguments

1.  **maxBytes**: specifies the size the file must reach before we carry out a log rotation. We use a low value here to easily see the rotation in action, usually, it would be much higher, edit it to suit your needs.
    
2.  **backupCount**: specifies the number of logs to keep before beginning to delete the oldest logs.
    

Execute the _[app.py](http://app.py/)_ script, you should see about two new log files in your working directory.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1675114598988/3c956f54-6e4e-4b97-ac9b-a28e97082408.png?auto=compress,format&format=webp)

When app.log got filled, it was renamed as app.log.1, then a new app.log was made, when that _new_ app.log got filled, it became app.log.1 and the previous app.log.1 became app.log.2.

[Permalink](#heading-formatters "Permalink")Formatters
------------------------------------------------------

Formatters are objects that can be attached to a handler. They define the way logs should be displayed. Let us take an example

```
import logging
from logging.handlers import RotatingFileHandler

LOGGER = logging.getLogger('foo.bar')

stream_handler = logging.StreamHandler()
stream_handler.setLevel(logging.DEBUG)

LOGGER.addHandler(stream_handler)
LOGGER.setLevel(logging.DEBUG)

file_handler = RotatingFileHandler('app.log', maxBytes=100, backupCount=5)
file_handler.setLevel(logging.DEBUG)
LOGGER.addHandler(file_handler)


formatter = logging.Formatter(fmt='%(asctime)s: %(name)s - %(levelname)s - %(message)s', datefmt='%Y-%m-%d %H:%M:%S')
file_handler.setFormatter(formatter)
stream_handler.setFormatter(formatter)

LOGGER.debug('Nothing to see here, Just trying to debug')
LOGGER.info('Hey, here is some information you might find useful')
LOGGER.warning('something may be wrong here!!! warning')
LOGGER.error('An error definetely just happened. You might want to take a look!')
LOGGER.critical('Oops.. Critical server error.')

```

We created a formatter and specified how to display our logs. We then attached it to the file and stream handler. Let us execute the script to see the result.

```
2023-01-31 00:44:08: foo.bar - DEBUG - Nothing to see here, Just trying to Debug
2023-01-31 00:44:08: foo.bar - INFO - Hey, here is some information you might find useful
2023-01-31 00:44:08: foo.bar - WARNING - something may be wrong here!!! warning
2023-01-31 00:44:08: foo.bar - ERROR - An error definetely just happened. You might want to take a look!
2023-01-31 00:44:08: foo.bar - CRITICAL - Oops.. Critical server error

```

As you can see, new information has been added to our logs and it resembles the structure we defined for the formatter with the `fmt` argument.

[Permalink](#heading-tidy-up-logging-with-configurations "Permalink")Tidy up logging with configurations
--------------------------------------------------------------------------------------------------------

Python provides us with several ways of configuring all or most of our loggers in one place. By providing a dictionary configuration we can specify which loggers should exist, the handlers to be attached, as well as formatters. Let us see how this is done.

### [Permalink](#heading-logging-with-dictconfig "Permalink")Logging with DictConfig

In a new file called `logging_config.py` paste the following code

```
LOGGING_CONFIG = { 
    'version': 1,
    'disable_existing_loggers': True,
    'formatters': { 
        'basic': {
            'format': '%(asctime)s: %(name)s - %(levelname)s - %(message)s',
            'datefmt' : '%Y-%m-%d %H:%M:%S'
        },
    },
    'handlers': { 
        'stream_handler': { 
            'level': 'DEBUG',
            'formatter': 'basic',
            'class': 'logging.StreamHandler',
        },
        'rotating_file_handler': { 
            'level': 'ERROR',
            'formatter': 'basic',
            'class': 'logging.handlers.RotatingFileHandler',
            'backupCount': 5,
            'maxBytes': 100,
            'filename': 'app.log'
        },
    },
    'loggers': { 
        '': {
            'handlers': ['stream_handler'],
            'level': 'WARNING',
            'propagate': False
        },
        'app': { 
            'handlers': ['stream_handler', 'rotating_file_handler'],
            'level': 'DEBUG',
            'propagate': False
        },
    } 
}

```

The config file is very similar to what we have been doing with logging helper methods like `setFormatter` and `addHandler`. Dissecting this config dictionary:

*   We have one formatter here defined, named as "**basic".** We specify our log format with the `format` key and our date format with the `datefmt` key.
    
*   We define two handlers, stream\_handler and rotating\_file_handler, similar to what we have done before. We set the priority level of "DEBUG" for the stream handler and "ERROR" for the rotating file handler.
    
*   Finally, we define our loggers. The empty string key is synonymous with saying 'root'. We give our root logger the _stream_handler_ handler we defined earlier. Then define a new logger named **app** and give it both handlers.
    
*   We set propagate key to false since we explicitly defined handlers for our loggers and we don't want duplicated logging from loggers higher up on the hierarchy.
    

Now in our [app.py](http://app.py/) file

```
import logging
import logging.config
from logger_config import LOGGING_CONFIG

logging.config.dictConfig(LOGGING_CONFIG)

LOGGER = logging.getLogger('app')

LOGGER.debug('Nothing to see here, Just trying to debug')
LOGGER.info('Hey, here is some information you might find useful')
LOGGER.warning('something may be wrong here!!! warning')
LOGGER.error('An error definetely just happened. You might want to take a look!')
LOGGER.critical('Oops.. Critical server error.')

```

As you can see, our app file looks a lot cleaner. Now, run the script again.

You should notice new logs in both the terminal and the log files. If you look closely, you should realize all logs are displayed on the console, but only "ERROR" and "CRITICAL" logs are shown in our log file. This is because of how we defined our `LOGGING_CONFIG`, "stream\_handler" has its effective level set to "DEBUG" and "rotating\_file_handler" has its effective level as "ERROR".

...And that's it✨.

[Permalink](#heading-conclusion "Permalink")Conclusion
------------------------------------------------------

In this article, we learned the importance of logging in python, the components of logging in python, and how to create and use our own loggers. We also saw how to specify new formats for our logs, and how to send logs to different outputs like console/terminal and files.

* * *

