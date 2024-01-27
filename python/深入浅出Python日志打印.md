# 深入浅出Python日志打印
0.引言
----

在编程过程中，日志记录是一项非常重要的任务，无论是用于调试代码、记录系统运行状态，还是跟踪可能出现的问题，日志都能发挥重要作用。然而，许多开发者习惯使用简单的print语句来记录信息，这种方法虽然简单，但在处理复杂的日志记录任务时，会显得力不从心。

Python的`logging`模块是Python标准库的一部分，它提供了丰富的配置选项和灵活的接口，能够帮助我们更好地进行日志记录。无论是简单的将日志消息输出到控制台，还是将日志消息发送到远程服务器，logging模块都能轻松应对。本文主要介绍python中打印日志的方法，并分享一些实践技巧，给出使用logging模块的过程中遇到问题的解决办法，帮助你更有效地使用这个工具。

1\. 基本概念
--------

在给出具体的代码示例之前，先来了解logging模块的基本概念，理解这些基本概念是使用Python logging模块的关键。

### 1.1 日志级别（Log Levels）

Python的logging模块定义了五种日志级别，用于区分日志信息的重要性。从低到高依次为：DEBUG, INFO, WARNING, ERROR, CRITICAL。

*   DEBUG：用于调试目的，记录所有详细的信息。
*   INFO：用于记录正常运行时的事件，确认事情按预期进行。
*   WARNING：用于表示可能的问题，它不会阻止程序运行，但可能会在将来。
*   ERROR：用于记录更严重的问题，它阻止了程序执行某个功能。
*   CRITICAL：用于记录非常严重的问题，可能导致程序无法运行。

### 1.2 日志记录器（Loggers）

Logger是执行日志记录操作的主要接口。每个Logger实例都有一个名字，这些名字是层次化的，像文件系统路径一样，使用点分隔。这使得在应用程序的不同部分可以有细粒度的控制，包括日志消息的处理方式和日志级别。

### 1.3 日志处理器（Handlers）

Handler对象负责将LogRecord分发到指定的目的地。logging模块提供了多种Handler类，可以将日志消息发送到控制台、文件、网络服务器等。你也可以创建自己的Handler类来满足特殊需求。

### 1.4 日志过滤器（Filters）

Filter可以提供更细粒度的控制，决定哪些日志记录将被输出。它们可以被应用于Logger和Handler对象上，用于决定是否处理或输出特定的LogRecord。

### 1.5 日志格式化（Formatters）

Formatter对象用于配置日志信息的最终顺序、结构和内容。它们可以添加时间戳、消息级别、调用者信息等。通过自定义Formatter，你可以让日志信息满足各种格式要求。

2.基本用法
------

### 2.1 日志的初始化及分级打印

上一章节中提到，logger作为日志的记录器，是可以被初始化的，`logging.getLogger()`函数用于返回一个Logger对象，如果没有指定name参数，将返回root logger。这个函数只有一个参数：

*   name：Logger的名称，这个名称通常是模块名，比如`__name__`。如果没有指定name，将返回root logger。

如果我们为`getLogger()`函数提供了name参数，那么它将返回一个以这个name命名的Logger对象。如果这个Logger之前已经被创建过，那么将返回之前创建的Logger对象；如果这个Logger之前没有被创建过，那么将创建一个新的Logger对象并返回。在创建新的Logger对象时，它的日志级别默认为NOTSET，处理器列表为空。如果我们为这个Logger设置了处理器，那么它就可以处理日志消息；否则，它将会忽略所有的日志消息。如果我们没有为这个Logger设置日志级别，那么它将会继承其父Logger的日志级别。

最简单的方式，直接引入logging类之后就可以在需要打印日志的地方完成日志打印，这时使用的logger为root logger。

```python
import logging

logging.debug('This is a debug message')
logging.info('This is an info message')
logging.warning('This is a warning message')
logging.error('This is an error message')
logging.critical('This is a critical message')

```

以上输出结果为：

```vbnet
WARNING:root:This is a warning message
ERROR:root:This is an error message
CRITICAL:root:This is a critical message

```

注意，debug和info级别的消息没有被输出，这是因为默认的日志级别设置为WARNING，只有此级别及以上的消息才会被记录。直接引入root logger时，所有的属性用的都是默认配置，正常情况下，需要单独创建一个logger.

```python
import logging

logger = logging.getLogger('my_logger')

logger.setLevel(logging.DEBUG)

```

### 2.2 定义Handler

创建完logger之后，需要定义Handler，不同的Handler可以将日志内容输出到不同地方，logger的`addHandler`方法可以添加Handler，一个logger可以添加多个Handler。

```ini

file_handler = logging.FileHandler('my_logger.log')


console_handler = logging.StreamHandler()




logger.addHandler(file_handler)
logger.addHandler(console_handler)

```

以下是Python logging模块中提供的一些常见Handler类的主要功能和构造参数：

| Handler类 | 主要功能 | 构造参数 |
| --- | --- | --- |
| StreamHandler | 将日志信息输出到任何可以写入的流（如：sys.stdout, sys.stderr或任何文件-like对象）。 | `strm=None` |
| FileHandler | 将日志信息写入到磁盘文件中。 | `filename, mode='a', encoding=None, delay=False` |
| RotatingFileHandler | 将日志信息写入到磁盘文件，并支持日志文件按大小切割。 | `filename, mode='a', maxBytes=0, backupCount=0, encoding=None, delay=False` |
| TimedRotatingFileHandler | 将日志信息写入到磁盘文件，并支持按照某个时间间隔切割日志文件。 | `filename, when='h', interval=1, backupCount=0, encoding=None, delay=False, utc=False, atTime=None` |
| SocketHandler | 将日志信息通过TCP/IP协议发送到网络。 | `host, port` |
| DatagramHandler | 将日志信息通过UDP协议发送到网络。 | `host, port` |
| SMTPHandler | 将日志信息通过电子邮件发送。 | `mailhost, fromaddr, toaddrs, subject, credentials=None, secure=None, timeout=1.0` |
| SysLogHandler | 将日志信息发送到一个Unix syslog守护进程。 | `address=('localhost', SYSLOG_UDP_PORT), facility=LOG_USER, socktype=socket.SOCK_DGRAM` |
| NTEventLogHandler | 将日志信息发送到一个Windows NT/2000/XP的事件日志。 | `appname, dllname=None, logtype='Application'` |
| MemoryHandler | 将日志信息缓存到内存中，到达某个水平后或者特定的情况下再将它们写入到目标位置。 | `capacity, flushLevel=ERROR, target=None, flushOnClose=True` |

以上就是logging模块中提供的一些Handler类，每种Handler都有其特定的应用场景，除了上述的Handler，还可以通过继承现有的Handler，做一些个性化的定制，来满足一些特殊的需求，如笔者通过继承`RotatingFileHandler`，实现了`RotatingFileZipHandler`，当日志的大小超出大小的阈值时，将其压缩归档，以节省存储空间。

### 2.3 定义Formatter

对于不同的Handler，可以定义不同的日志格式。Handler中的`setFormatter`方法可以定义的日志格式。

```python

formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
file_handler.setFormatter(formatter)
console_handler.setFormatter(formatter)

```

在Python的logging模块中，通过设置Formatter可以自定义日志信息的输出格式。Formatter使用字符串格式化操作，可以在字符串中插入一些特殊的字段，这些字段会被替换为LogRecord的相应属性。

以下是一些常见的字段：

*   %(name)s：Logger的名字。
*   %(levelno)s：数字形式的日志级别。
*   %(levelname)s：文本形式的日志级别。
*   %(pathname)s：调用日志记录函数的源码文件的全路径。
*   %(filename)s：pathname的文件名部分。
*   %(module)s：filename的名称部分，不包括后缀。
*   %(lineno)d：调用日志记录函数的源码所在的行号。
*   %(funcName)s：调用日志记录函数的函数名。
*   %(message)s：日志记录的文本内容，通过msg % args计算得到。

此外，还可以使用以下字段表示日期和时间：

*   %(asctime)s：产生日志的时间，可通过asctime属性自定义时间格式。
*   %(created)f：LogRecord被创建的时间，表示为从UNIX纪元开始的秒数。
*   %(relativeCreated)d：LogRecord被创建的时间，表示为相对于logging模块加载的秒数。
*   %(msecs)d：LogRecord被创建的时间的毫秒部分。
*   %(thread)d：线程ID。可能没有在所有平台上可用。
*   %(threadName)s：线程名。

当执行`logger.info('This is a log message.')`，以上定义的logger的效果为：

> 2023-11-15 23:57:27,956 - my_logger - INFO - This is a log message.

这将会生成包含时间、Logger名称、日志级别和日志消息的日志记录，这条记录可以在控制台和`my_logger.log`中看到。

### 2.4 定义Filter

过滤器的工作原理是提供一个filter()方法，这个方法接受一个LogRecord对象作为参数，如果这个方法返回True，这条日志记录会被处理，如果返回False，这条日志记录会被忽略。

以下是一个简单的过滤器示例，这个过滤器只允许INFO级别的日志记录通过：

```python
import logging

class InfoFilter(logging.Filter):
    def filter(self, record):
        return record.levelno == logging.INFO

logger = logging.getLogger(__name__)
handler = logging.StreamHandler()
handler.addFilter(InfoFilter())
logger.addHandler(handler)

logger.setLevel(logging.DEBUG)

logger.debug('This is a debug message')
logger.info('This is an info message')
logger.warning('This is a warning message')

```

运行这段代码，你会看到只有"INFO:**main**:This is an info message"被输出。其他的日志记录都被InfoFilter过滤掉了。在实际应用中，我们可以定义更复杂的过滤器，比如基于日志记录的其他属性（如logger的名字、日志消息的内容等）来决定是否处理这条日志记录。

3.高级用法
------

### 3.1 使用多个处理器

使用多个处理器，可以为不同的日志级别设置不同的输出目标和格式。

```python
import logging


logger = logging.getLogger('my_logger')
logger.setLevel(logging.DEBUG)


fh = logging.FileHandler('my_logger.log')
fh.setLevel(logging.WARNING)


ch = logging.StreamHandler()
ch.setLevel(logging.DEBUG)


formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
fh.setFormatter(formatter)
ch.setFormatter(formatter)


logger.addHandler(fh)
logger.addHandler(ch)


logger.debug('This is a debug message')

```

在这段代码中，我们创建了一个Logger，并为它添加了两个Handler：一个FileHandler用于将所有级别及以上的日志信息写入到文件，一个StreamHandler用于将WARNING级别以上的日志信息输出到控制台。通过不同的Handler，可以实现不同方式的分级记录，可以让控制台只记录关键日志，而日志文件记录详细日志，这样便于在运行中发现问题，进而到日志文件中去进一步定位问题。

### 3.2 配置日志记录器的层次结构

配置日志记录器的层次结构，可以在不同的模块和组件中进行更细粒度的日志控制。

```python
import logging


logger = logging.getLogger('my_logger')
logger.setLevel(logging.DEBUG)


module_logger = logging.getLogger('my_logger.my_module')
module_logger.setLevel(logging.WARNING)


fh = logging.FileHandler('my_logger.log')
fh.setLevel(logging.DEBUG)
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
fh.setFormatter(formatter)


logger.addHandler(fh)


module_logger.debug('This is a debug message')
module_logger.warning('This is a warning message')

```

在这段代码中，我们创建了一个名为'my\_module'的子Logger，它继承自'my\_logger'这个父Logger。我们可以为每个子Logger设置不同的日志级别，以便在不同的模块和组件中进行更细粒度的日志控制。

4.日志的配置
-------

除了用代码的方式实现初始化，还可以通过配置来让Logger的初始化一步到位，logger提供了三种配置方式。

### 4.1 使用logging.basicConfig()函数进行简单配置

`logging.basicConfig(**kwargs)`函数提供了一种简单的方式来配置默认的`logging`模块。你可以通过关键字参数来指定日志的等级、日志格式、输出目标等。

以下是一段示例代码：

```python
import logging

logging.basicConfig(level=logging.INFO, 
                    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
                    filename='app.log')

logging.info('This is a log info')
logging.warning('This is a log warning')

```

在这个例子中，我们使用`logging.basicConfig()`函数来配置`logging`模块。`level`参数指定了日志的等级，`format`参数指定了日志的格式，`filename`参数指定了日志的输出文件。

### 4.2 使用logging.config.dictConfig()函数加载字典配置

`logging.config.dictConfig(config)`函数可以接受一个字典作为参数，用于配置`logging`模块。这个字典可以包含handlers、formatters、filters和loggers等配置信息。

以下是一段示例代码：

```python
import logging.config

config = {
    'version': 1,
    'handlers': {
        'console': {
            'class': 'logging.StreamHandler',
            'level': 'INFO',
            'formatter': 'simple',
            'stream': 'ext://sys.stdout',
        }
    },
    'root': {
        'level': 'INFO',
        'handlers': ['console']
    }
}

logging.config.dictConfig(config)
logging.info('This is a log info')

```

在这个例子中，我们定义了一个字典`config`，然后使用`logging.config.dictConfig()`函数来配置`logging`模块。字典中的`handlers`键用于定义处理器，`root`键用于定义root logger。

### 4.3 使用logging.config.fileConfig()函数加载配置文件

`logging.config.fileConfig(fname, defaults=None, disable_existing_loggers=True)`函数可以加载一个INI格式的配置文件，用于配置`logging`模块。

以下是一个配置文件的例子：

```ini
[loggers]
keys=root,sampleLogger

[handlers]
keys=consoleHandler

[formatters]
keys=sampleFormatter

[logger_root]
level=DEBUG
handlers=consoleHandler

[logger_sampleLogger]
level=DEBUG
handlers=consoleHandler
qualname=sampleLogger
propagate=0

[handler_consoleHandler]
class=StreamHandler
level=DEBUG
formatter=sampleFormatter
args=(sys.stdout,)

[formatter_sampleFormatter]
format=%(asctime)s - %(name)s - %(levelname)s - %(message)s
datefmt=

```

然后我们可以使用`logging.config.fileConfig()`函数来加载这个配置文件：

```python
import logging.config

logging.config.fileConfig('logging.ini')
logger = logging.getLogger('sampleLogger')

logger.debug('This is a log debug')

```

在这个例子中，我们使用`logging.config.fileConfig()`函数加载了一个名为`logging.ini`的配置文件，然后获取了一个名为`sampleLogger`的logger，并使用这个logger打印了一条debug日志。在实际使用中，建议采用`logging.config.fileConfig()`来配置日志文件，一方面减少了定义日志的代码量，另一方面也让日志的定义参数都集中在一个文件中，提高了可读性的同时，也便于后期维护。

5.最佳实践
------

### 5.1 如何正确的在模块和包中使用日志记录器

在每个模块或包中，都应该创建一个名为“`__name__`”的logger对象。这将创建一个以模块完全限定名称（全路径）为名的logger，当你查看调试输出时，将允许你清楚地看到哪个模块生成了记录项。创建logger的示例代码如下：

```python
import logging

logger = logging.getLogger(__name__)

def function_with_logging():
    logger.info('This is an info message from {} module.'.format(__name__))

```

这种方式可以让你在其他模块中轻松控制日志级别，且不会影响别的模块。根logger可以通过名字为空字串来获取。

### 5.2 如何避免日志记录性能问题

大量的日志记录会带来性能问题。对于加了多个Filter的情况，在你记录一条消息之前，通常需要检查它的级别是否会被logger处理。示例代码如下：

```python
if logger.isEnabledFor(logging.INFO):
    logger.info("This is an info message")

```

这将避免一些不必要的计算和内存分配，即使日志消息最终不会被处理。

### 5.3 如何处理多线程和多进程环境下的日志记录

`logging`模块是线程安全的，所以你可以在多线程环境中安全地使用它。为了避免记录交错问题，你可以使用`threading`模块提供的`Lock`。示例代码如下：

```python
import logging
import threading

logger = logging.getLogger(__name__)
lock = threading.Lock()

def function_with_threadsafe_logging():
    with lock:
        logger.info('This is a thread-safe info message from {} module.'.format(__name__))

```

在多进程环境下，一种常见的方法是每个进程记录到一个独立的文件，或使用`concurrent_log_handler`包的`ConcurrentRotatingFileHandler`。

总的来说，应用`logging`模块的最佳实践依赖于你的具体需求和环境。无论工程规模大小，编写清晰、整洁的日志都是专业开发者的一个标志。

### 5.4 如何用装饰器优雅地打印日志

关于logging模块，可以结合装饰器使用，

以下是一个使用装饰器封装日志的例子：

```python
import time
import logging

def get_logger(name):
    logger = logging.getLogger(name)
    ch = logging.StreamHandler()
    ch.setLevel(logging.DEBUG)
    ch.setFormatter(logging.Formatter('%(levelname)s | %(name)s | %(message)s'))
    logger.addHandler(ch)
    return logger

def exec_log(func):
    def wrapper(*args, **kwargs):
        logger = get_logger(func.__name__)
        logger.setLevel(logging.DEBUG)
        logger.info('开始执行[%s]:%s' % (func.__name__, args[0]))
        start_time = time.time()
        result = func(*args, **kwargs)
        end_time = time.time()
        logger.info('执行结束,总耗时：%fs' % (end_time - start_time))
        logger.info('执行结果:%s' % result)
        return result
    return wrapper

@exec_log
def app(name):
    print(name)
    return name

if __name__ == '__main__':
    app('test1')
    app('test2')
    app('test3')

```

在上述的例子中，采用装饰器的方法封装日志，带来的好处也是显而易见的。通过装饰器，我们可以在一个地方定义日志记录的逻辑，然后在需要记录日志的地方重复使用这个逻辑。这样就避免了在每个需要记录日志的地方重复编写日志记录的代码。同时可以将日志记录的代码从主要的业务逻辑代码中分离出来，让业务逻辑代码更加简洁，可读性更强。

但是上面的代码在执行的时候会存在一个日志重复打印的问题。 以上代码的输出如下：

```ruby
INFO | app | 开始执行[app]:test1
INFO | app | 执行结束,总耗时：0.000000s
INFO | app | 执行结果:test1
INFO | app | 开始执行[app]:test2
INFO | app | 开始执行[app]:test2
INFO | app | 执行结束,总耗时：0.000000s
INFO | app | 执行结束,总耗时：0.000000s
INFO | app | 执行结果:test2
INFO | app | 执行结果:test2
INFO | app | 开始执行[app]:test3
INFO | app | 开始执行[app]:test3
INFO | app | 开始执行[app]:test3
INFO | app | 执行结束,总耗时：0.000000s
INFO | app | 执行结束,总耗时：0.000000s
INFO | app | 执行结束,总耗时：0.000000s
INFO | app | 执行结果:test3
INFO | app | 执行结果:test3
INFO | app | 执行结果:test3

```

为解决此问题，让我们进入下一节讨论的内容。

### 5.5 如何避免日志重复打印

当我们在多层依赖中引入logger模块时，除了上一节中提到的装饰器，还有同时在子类和父类中定义Logger都有可能会遇到日志重复打印的问题。为解决此问题，首先需要分析日志重复的原因，在参考文章（[blog.csdn.net/qq_31455841…](https://link.juejin.cn/?target=https%3A%2F%2Fblog.csdn.net%2Fqq_31455841%2Farticle%2Fdetails%2F127889511 "https://blog.csdn.net/qq_31455841/article/details/127889511") ）中，详细探讨了日志重复打印的原因。

文中给出了日志重复打印的两种情况：

*   未定义 logger

默认使用了 RootLogger，一个 python 程序内全局唯一的，所有Logger对象的祖先，每次实例化返回的都是 RootLogger 对象

*   通过`logger = logging.getLogger(name)`定义了logger

该方法采用了单例模式，也就是说每次实例化返回的是同一个logger对象，在获取logger对象之后，每次都会调用 `logger.addHandler(handler)`方法添加日志处理器，导致 handlers 列表添加了相同的 handler（注意：日志的打印由handler控制）以此类推，调用几次就会有几个handler，然后前面打印的log就会影响后面定义的log。

以5.4中的代码为例，可以采用以下几种方法解决日志重复打印的问题：

*   定义不同的logger名以区分不同的logger

比如上一节中在logging.getLogger(name)中传入不同的name以示区分。但对于同样一个业务逻辑，定义太多的logger也会带来额外的性能开销，因此可以进一步从Handler中入手。

*   根据情况判断是否`addHandler`

```scss
def get_logger(name):
    logger = logging.getLogger(name)
    if not logger.handlers:
        ch = logging.StreamHandler()
        ch.setLevel(logging.DEBUG)
        ch.setFormatter(logging.Formatter('%(levelname)s | %(name)s | %(message)s'))
        logger.addHandler(ch)
    return logger

```

*   使用完成之后执行`removeHandler

```scss
def exec_log(func):
    def wrapper(*args, **kwargs):
        logger = get_logger(func.__name__)
        logger.setLevel(logging.DEBUG)
        logger.info('开始执行[%s]:%s' % (func.__name__, args[0]))
        start_time = time.time()
        result = func(*args, **kwargs)
        end_time = time.time()
        logger.info('执行结束,总耗时：%fs' % (end_time - start_time))
        logger.info('执行结果:%s' % result)
        # 在日志打印完成之后删除logger
        for handle in logger.handlers:
            logger.removeHandler(handle)
        return result
    return wrapper

```

6 全文小结
------

我们希望通过这篇文章，读者可以全面了解Python logging模块，掌握如何使用这个模块进行有效的日志记录。无论是在开发过程中调试问题，还是在生产环境中监控应用状态，日志记录都是一个非常重要的工具。通过合理地使用logging模块，我们可以更好地了解和控制我们的应用，提高开发和运维的效率。

关于logging模块更详尽的说明参见：[docs.python.org/3/howto/log…](https://link.juejin.cn/?target=https%3A%2F%2Fdocs.python.org%2F3%2Fhowto%2Flogging.html "https://docs.python.org/3/howto/logging.html")