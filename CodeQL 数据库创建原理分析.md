# CodeQL 数据库创建原理分析
**作者：六炅  
本文为作者投稿，Seebug Paper 期待你的分享，凡经采用即有礼品相送！ 投稿邮箱：paper@seebug.org**

Preface
-------

`CodeQL`是一款不错的代码分析扫描工具，于我而言对漏洞挖掘有很大的帮助。使用它也有一定时间了，之前一直接触的是开源项目，所以借助`CodeQL`进行数据库创建和分析还是相对简单的，不会有过多的限制。最近在进行`Java`反序列化利用链挖掘时，接触了[gadgetinspector](https://github.com/JackOfMostTrades/gadgetinspector.git)，它通过分析字节码来获取`AST`语法树并根据预定条件生成可能的调用链。于是我想如果借助`CodeQL`这类来分析应该会更方便些，可是在没有源码的情况对于编译型语言，无法从正常途径创建数据库。虽然网上已有部分此类的小工具，但还是希望通过了解`CodeQL`的部分原理来找寻无源码创建数据库的方式并加深对静态代码分析的理解。

以下内容以`Java`语言作为分析对象，分析的结论并不保证与实际完全相符。文章所用的测试项目可在[github](https://github.com/trganda/app)获取，包含`.idea/`你可以用`IDEA`打开，只需修改文件路径即可自己调试分析。

Pre Request
-----------

*   CodeQL CLI 2.9.1
*   Jdk 16
*   Windows OS

Building Database
-----------------

在`CodeQL`的历史文档中（加入`Github`之前），有大致描述其创建数据库的过程，如下图，见\[1\]

![](https://images.seebug.org/content/images/2022/06/c72eaefc-e52f-4691-a638-8780f2b23022.png-w331s)

它的工作流程大致是，在`javac`编译目标代码时，通过`Extractor`与其进行交互。`Extractor`会根据每一个`java`文件的内容生成一个`trap`文件，后续再根据`trap`文件生成实际的数据库。同时它会将处理的每一个`java`文件拷贝一份保存在数据中，便于后续展示查询结果时能看到代码的上下文。

而针对不同的语言都有各自独立的`Extractor`，文档中解释这样做的好处，毕竟不同语言特性不同。

`CodeQL`可以通过以下命令创建一个数据库，这里以一个最简单的`maven`项目为例，该项目仅包含一个输出`Hello World`的`java`文件

```
codeql database create -l java -c "mvn clean compile" C:\Users\trganda\Documents\databases\app
```

创建好的数据库的目录结构如下

```
|-- codeql-database.yml
|-- db-java # 数据库关系文件
|-- log # 各类日志
|   |-- agent.9008554372696040130.log
|   |-- agent.11337701303103251140.log
|   |-- build-tracer.log
|   |-- database-create-20220509.114127.634.log
|   |-- database-index-files-20220509.114151.283.log
|   |-- ext 用于javac的配置文件
|   |   |-- javac.args
|   |   |-- javac.env
|   |   |-- javac.orig
|   |   `-- javac.properties
|   |-- javac-errors.log
|   |-- javac-extractor-1683275.log
|   `-- javac-output-11812.log
`-- src.zip # 源码文件
```

### Analyze Build Process

由官方文档[using-indirect-build-tracing](https://codeql.github.com/docs/codeql-cli/creating-codeql-databases/#using-indirect-build-tracing)和`database-create-20220509.114127.634.log`可以看到数据库的创建过程其实是被分为了多个子步骤的。各步骤执行的命令如下

```
codeql database init --language=java --source-root=C:\Users\trganda\Documents\app --allow-missing-source-root=false --allow-already-existing -- C:\Users\trganda\Documents\databases\app

codeql database trace-command --working-dir=C:\Users\trganda\Documents\app --index-traceless-dbs --no-db-cluster -- C:\Users\trganda\Documents\databases\app mvn clean compile

codeql database finalize --mode=normal --no-db-cluster -- C:\Users\trganda\Documents\databases\app
    |-- codeql database trace-command --working-dir=C:\Users\trganda\Documents\app --no-tracing -- C:\Users\trganda\Documents\databases\app C:\Program Files\codeql\java\tools\pre-finalize.cmd
    |-- codeql dataset import --dbscheme=C:\Program Files\codeql\java\semmlecode.dbscheme -- C:\Users\trganda\Documents\databases\app\db-java C:\Users\trganda\Documents\databases\app\trap\java
    |-- codeql database cleanup --mode=normal -- C:\Users\trganda\Documents\databases\app
    `-- codeql dataset cleanup --mode=normal -- C:\Users\trganda\Documents\databases\app\db-java
```

与`Extractor`有关的为第二条命令，下面来看看它具体做了什么。

`codeql`命令对应的执行文件位于其安装目录下`codeql.cmd`，内容如下

```
@echo off
rem Wrapper provided for users who explicitly configured VS Code to point to codeql.cmd
"%~dp0\codeql.exe" %*
exit /b %errorlevel%
```

在`win`平台，它借助`exe`文件来处理要执行的命令，这不是我们想看到的。好在还有另一个`shell`脚本文件`codeql`，为`linux`平台提供服务。可以通过它来了解`codeql.exe`的内部逻辑 ![](https://images.seebug.org/content/images/2022/06/ac256114-ae65-45f3-8812-4160467fd352.png-w331s)
 ![](https://images.seebug.org/content/images/2022/06/ebf3502b-eceb-4c7e-aaaa-07b27dfe9616.png-w331s)

它的大概意思是，设置环境变量`CODEQL_PLATFORM`，`CODEQL_JAVA_HOME`和`CODEQL_DIST`后，执行`codeql.jar`。再回过头细看`database-create-20220509.114127.634.log`里面会记录使用成功加载`java`的`extracotr`（**Successfully loaded extractor Java**），位于`java\tools`目录下

```
|-- COPYRIGHT
|-- LICENSE
|-- codeql-extractor.yml
|-- semmlecode.dbscheme
|-- semmlecode.dbscheme.stats
`-- tools
    |-- autobuild-fat.jar
    |-- autobuild.cmd
    |-- autobuild.sh
    |-- codeql-java-agent.jar
    |-- compiler-tracing.spec
    |-- linux
    |   `-- ...
    |-- pre-finalize.cmd
    |-- pre-finalize.sh
    |-- semmle-extractor-java.jar
    `-- tracing-config.lua
```

这里可以看到一些`jar`包和脚本，以及配置文件`codeql-extractor.yml`。`codeql-java-agent.jar`为`agent`，在整个编译期开始前注入`jvm`中并用于执行`extractor`操作。而其它的部分内容，通过日志的信息，可以猜测其含义，这里暂不细纠。

既然是`jar`包，那么就能比较容易的去分析它。这里将`codeql.jar`和`java\tools`目录下的`autobuild-fat.jar`，`codeql-java-agent.jar`和`semmle-extractor-java.jar`拖入`IDEA`和`jd-gui`。

在`IDEA`的`Run/Debug Configurations`中新增`2`个`Jar Application`，配置分别如下

`codeql database init`

```
Path to JAR: C:\Program Files\codeql\tools\codeql.jar
VM options: --add-modules jdk.unsupported
Program arguments: database init --language=java --source-root=<your working path> --allow-missing-source-root=false --allow-already-existing -- <your database path>
Working directory: <your working path>
Enviroment variables: CODEQL_DIST=C:\Program Files\codeql;CODEQL_JAVA_HOME=C:\Program Files\codeql\tools\win64\java;CODEQL_PLATFORM=win64
```

`codeql database trace-command`

```
Path to JAR: C:\Program Files\codeql\tools\codeql.jar
VM options: --add-modules jdk.unsupported
Program arguments: database trace-command --working-dir=<your working path> --index-traceless-dbs --no-db-cluster -- <your database path> mvn clean compile
Working directory: <your working path>
Enviroment variables: CODEQL_DIST=C:\Program Files\codeql;CODEQL_JAVA_HOME=C:\Program Files\codeql\tools\win64\java;CODEQL_PLATFORM=win64
```

这里调试的目标是`codeql database trace-command`，在调试前先执行一次`codeql database init`完成数据库初始化。并在`com.semmle.cli2.CodeQL#main`打下断点再调试`codeql database trace-command`，与`database`相关的命令处理逻辑位于`com.semmle.cli2.database`，从类的名字可以很好找到与`trace-command`相关的类为`com.semmle.cli2.database.TraceCommandCommand`。大致查看这个类的代码，执行逻辑在`com.semmle.cli2.database.TraceCommandCommand#executeSubcommand`

```
protected void executeSubcommand() {
    this.actionVersion = new CodeQLActionVersion() {
        protected boolean isVeryOldAction() {
            return TraceCommandCommand.this.command.size() == 3 && ((String)TraceCommandCommand.this.command.get(1)).endsWith(File.separator + "working" + File.separator + "tracer-env.js") && ((String)TraceCommandCommand.this.command.get(2)).endsWith(File.separator + "working" + File.separator + "env.tmp");
        }
    };
    super.executeSubcommand();
}
```

在此处也打下一个断点，然后开启调试，顺利的话会执行到`super.executeSubcommand();`也就是`DatabaseProcessCommandCommon#executeSubcommand`这个方法，它的内容比较长，直接看尾部的一部分代码，

```
protected void executeSubcommand() {
    ...
    Iterator var32 = commandlines.iterator();

    while(var32.hasNext()) {
        List<String> cmdArgs = (List)var32.next();
        this.printProgress("Running command in {}: {}", new Object[]{workingDir, cmdArgs});
        Builder8 p = new Builder8(cmdArgs, LogbackUtils.streamFor(this.logger(), "build-stdout", true), LogbackUtils.streamFor(this.logger(), "build-stderr", true), Env.systemEnv().getenv(), workingDir.toFile());
        this.env.addToProcess(p);
        List<String> cmdProcessor = new ArrayList();
        CommandLine.addCommandProcessor(cmdProcessor, this.env.expander);
        p.prependArgs(cmdProcessor);
        tracerSetup.enableTracing(p);
        StreamAppender streamOutAppender = new StreamAppender(Streams.out());

        int result;
        try {
            LogbackUtils.addAppender(streamOutAppender);
            result = p.execute();
        } finally {
            LogbackUtils.removeAppender(streamOutAppender);
        }

        if (result != 0) {
            cmdProcessor.addAll(cmdArgs);
            throw new UserError("Exit status " + result + " from command: " + cmdProcessor);
        }
    }
    ...
}
```

它根据传入的命令`mvn clean compile`构造了一个`Buildr8`，它封装了`ProcessBuilder`，在构造完成后会调用`p.execute()`执行命令，完整执行的命令为

```
"C:\Program Files\codeql\tools\win64\tracer.exe" "C:\Program Files\codeql\tools\win64\runner.exe" cmd.exe /C type NUL && mvn clean compile
```

相关的环境变量（由`codeql`增加的）如下

```
CODEQL_PLATFORM=win64;
CODEQL_PLATFORM_DLL_EXTENSION=.dll;
CODEQL_EXTRACTOR_JAVA_LOG_DIR=C:\Users\trganda\Documents\databases\app2\log;
CODEQL_JAVA_HOME=C:\Program Files\codeql\tools\win64\java;
CODEQL_EXTRACTOR_JAVA_SCRATCH_DIR=C:\Users\trganda\Documents\databases\app2\working;
ODASA_TRACER_CONFIGURATION=C:\Users\trganda\Documents\databases\app2\working\tracing\compiler-tracing1707598060791117786.spec;
SEMMLE_JAVA_TOOL_OPTIONS='-javaagent:C:\Program Files\codeql\java\tools/codeql-java-agent.jar=ignore-project,java' '-Xbootclasspath/a:C:\Program Files\codeql\java\tools/codeql-java-agent.jar';
CODEQL_EXTRACTOR_JAVA_WIP_DATABASE=C:\Users\trganda\Documents\databases\app2;
CODEQL_EXTRACTOR_JAVA_ROOT=C:\Program Files\codeql\java;
CODEQL_EXTRACTOR_JAVA_TRAP_DIR=C:\Users\trganda\Documents\databases\app2\trap\java;
CODEQL_TRACER_LOG=C:\Users\trganda\Documents\databases\app2\log\build-tracer.log;
CODEQL_EXTRACTOR_JAVA_SOURCE_ARCHIVE_DIR=C:\Users\trganda\Documents\databases\app2\src;
CODEQL_DIST=C:\Program Files\codeql;
```

环境变量中出现了很多熟悉的面孔，在`java`的`extractor`中见过它们。由于前面执行的命令涉及到`tracer.exe`和`runner.exe`，如果直接以它们为目标进行分析需要借助其它逆向工具，导致问题过于复杂，先不走这条路。这里先通过`process hacker`查看这条命令执行过程中的变化

![](https://images.seebug.org/content/images/2022/06/7dc4e925-2076-42ab-b4c2-f6ca3a5da1e6.jpg-w331s)

从进程创建的结构看，后`3`个`java.exe`依次执行的命令如下

```
"C:\Program Files\Common Files\Oracle\Java\javapath\java.exe" -classpath "C:\Program Files\JetBrains\IntelliJ IDEA 2021.3\plugins\maven\lib\maven3\bin\..\boot\plexus-classworlds-2.6.0.jar"   "-Dclassworlds.conf=C:\Program Files\JetBrains\IntelliJ IDEA 2021.3\plugins\maven\lib\maven3\bin\..\bin\m2.conf"   "-Dmaven.home=C:\Program Files\JetBrains\IntelliJ IDEA 2021.3\plugins\maven\lib\maven3\bin\.."   "-Dlibrary.jansi.path=C:\Program Files\JetBrains\IntelliJ IDEA 2021.3\plugins\maven\lib\maven3\bin\..\lib\jansi-native"   "-Dmaven.multiModuleProjectDirectory=C:\Users\trganda\Documents\app"   org.codehaus.plexus.classworlds.launcher.Launcher clean compile

"C:\Program Files\Java\jdk-16.0.1\bin\java.exe" -classpath "C:\Program Files\JetBrains\IntelliJ IDEA 2021.3\plugins\maven\lib\maven3\bin\..\boot\plexus-classworlds-2.6.0.jar" "-Dclassworlds.conf=C:\Program Files\JetBrains\IntelliJ IDEA 2021.3\plugins\maven\lib\maven3\bin\..\bin\m2.conf" "-Dmaven.home=C:\Program Files\JetBrains\IntelliJ IDEA 2021.3\plugins\maven\lib\maven3\bin\.." "-Dlibrary.jansi.path=C:\Program Files\JetBrains\IntelliJ IDEA 2021.3\plugins\maven\lib\maven3\bin\..\lib\jansi-native" -Dmaven.multiModuleProjectDirectory=C:\Users\trganda\Documents\app org.codehaus.plexus.classworlds.launcher.Launcher clean compile

"C:\Program Files\Java\jdk-16.0.1\bin\java.exe" -Dfile.encoding=windows-1252 -Xmx1024M -Xms256M --add-opens java.base/sun.reflect.annotation=ALL-UNNAMED -classpath "C:\Program Files\codeql\java\tools\semmle-extractor-java.jar" com.semmle.extractor.java.JavaExtractor --jdk-version 16 --javac-args @@@C:\Users\trganda\Documents\databases\app\log\ext\javac.args
```

前两个是调用了`maven`工具链，而这里最引人注目的是最后一条命令的内容，它执行`semmle-extractor-java.jar`，并传入`javac.args`文件，这个文件的内容长这样

```
-Xprefer:source
-d
C:\Users\trganda\Documents\app\target\classes
-classpath
C:\Users\trganda\Documents\app\target\classes;
-sourcepath
C:\Users\trganda\Documents\app\src\main\java;C:\Users\trganda\Documents\app\target\generated-sources\annotations;
-s
C:\Users\trganda\Documents\app\target\generated-sources\annotations
-g
-nowarn
-target
1.7
-source
1.7
-encoding
UTF-8
C:\Users\trganda\Documents\app\src\main\java\org\example\App.java
```

这个文件称为[Command-Line Argument Files](https://docs.oracle.com/en/java/javase/13/docs/specs/man/javac.html?msclkid=bebf72cecf7611ecaa58ff68bdfe6baa#Command-Line%20Argument%20Files)，用于给`javac`传递参数，它应该是通过执行`maven`来生成的。

### Tracer

这里可能会疑惑`semmle-extractor-java.jar`是怎么被执行的，虽然并没有对`trace.exe`和`runner.exe`进行分析，但是可以从`javac.env`和环境变量`SEMMLE_JAVA_TOOL_OPTIONS`猜测出在`"C:\Program Files\codeql\tools\win64\tracer.exe" "C:\Program Files\codeql\tools\win64\runner.exe" cmd.exe /C type NUL && mvn clean compile`执行过程中时，通过`agent`的方式向`jvm`植入了`codeql-java-agent.jar`。

> 下面这一段内容是新加入的

在`$CODEQL_HOME/tools`目录下，有一个`tracer`目录，里面放着名为`base.lua`的问题，打开这个文件可以看到注释中大大方方的写着它的用途。

```
-- Overview:
-- Each traced language contains a `tracing-config.lua` file that defines two functions:
-- GetCompatibleVersions() -> [versionNumbers]. This function returns a list of major versions that
--   are compatible with this `tracing-config.lua` file.
-- RegisterExtractorPack(languageId) -> [matchers]. This function is called at by
--   the Lua tracer runtime. It returns a list of matchers for this language.
--   A matcher is a  function of the form function(compilerName, compilerPath, compilerArguments, languageID) -> Table | nil.
--   The return value of a matcher is either `nil` (no match) or a table with the following keys:
--     `trace`: True if the processes created by the compiler (and extractor) should be traced for the current language
--     `replace`: If true, then the compiler process is not run
--     `invocations`: A list of extractor invocations. Each invocation is a table with key `path` (absolute path to the executable)
--                    and key `arguments` XOR `transformedArguments` (see explanation below)
--   For convenience, the `CreatePatternMatcher` function is provided that deals with most of the low-level details
--   of creating matchers.
--
-- `compilerArguments` has the following structure:
-- {
--   "nativeArgumentPointer": Opaque pointer that can be used to create transformations of these command line arguments
--                        that are executed in C++. This is mostly necessary for Windows, where we want to
--                        prepend/append to the command line without parsing it
--   "argv": Posix-only, array of command line arguments passed to the compiler
--   "commandLineString": Windows-only, the string passed to CreateProcess*(), with the path to the compile removed (and converted to UTF-8).
--                  Can be parsed into an argv array using `NativeCommandLineToArgv`, but be warned, this is not
--                  a canonical interpretation of the command line.
-- }
-- The arguments for an extractor invocation have two possible shapes:
--   either, the invocation sets the key `transformedArguments` (like `BuildExtractorInvocation` does), which is a table with
--   the following keys:
--     `nativeArgumentPointer`: The same opaque pointer, copied from the compiler invocation
--     `prepend`: A list of arguments to prepend to the arguments from the compiler
--     `append`: A list of arguments to append to the arguments from the compiler
--   alternatively, it sets the key `arguments`, which is a table with the following keys:
--     `argv`: Posix-only: The command line arguments (without argv[0])
--     `commandLineString`: Windows-only: The command line string (without the leading path to the executable).
--                    This will be converted internally to UTF-16 before execution.
--
-- The user can specify an extra lua config file on the command line.
-- This is loaded after all enabled languages have been loaded. This file also needs to contain a `GetCompatibleVersions`
-- function, just like a regular tracing config.
-- Second, it is required to contain a function
-- RegisterExtraConfig() -> [{languageID -> [matchers]}], i.e. a function that returns a table
--   mapping language IDs to a list of matchers. For each language ID, these matchers will _overwrite_ the matchers
-- registered by that language.
-- Furthermore, this function has full access to the implementation details of `base.lua`. However, obviously
-- no guarantees about compatibility are made when accessing internal functions or state.
--
-- If tracing is enabled for multiple languages, the languages are processed in lexicographical order of the language ID.
-- For each language, the matchers are processed in the order supplied, until the first matcher returns non-nil.
-- Then, matching for that language is stopped.
-- Matchers between different languages are not allowed to cooperate - each language is supposed to be independent
-- of the other possibly active languages.
-- There is one exception, though: If two languages specify `replace=true` for the same compiler invocation,
-- then matching for the second language is aborted without action. In this case, a log message is emitted.
```

该文件配合`trace.exe`使用，每种语言的`extractor`下都有一个`tracing-config.lua`文件，它有点类似于插件，需要实现两个函数`GetCompatibleVersions`和`RegisterExtractorPack`。前者用于标识自身支持的版本，后者则会被`tracer`调用返回一个`matcher`，`matcher`可以用来标识编译器并插入参数。以`java`的`extractor`为例，它的`tracing-config.lua`文件如下

```
function RegisterExtractorPack(id)
    local pathToAgent = AbsolutifyExtractorPath(id, 'tools' .. PathSep ..
                                                    'codeql-java-agent.jar')
    -- inject our CodeQL agent into all processes that boot a JVM
    return {
        CreatePatternMatcher({'.'}, MatchCompilerName, nil, {
            jvmPrependArgs = {
                '-javaagent:' .. pathToAgent .. '=ignore-project,java,kotlin:experimental',
                '-Xbootclasspath/a:' .. pathToAgent
            }
        })
    }
end

-- Return a list of minimum supported versions of the configuration file format
-- return one entry per supported major version.
function GetCompatibleVersions() return {'1.0.0'} end
```

注释中已经写明，会向`jvm`中注入`agent`文件`codeql-java-agent.jar`。

> 以下为之前的理解

这个过程从`process hacker`中无法直接看到，但是任然有一些蛛丝马迹可以证明这一点。

*   日志文件`build-tracer.log`，有`Reading configuration file ...\working\tracing\compiler-tracing12908925883751484166.spec`
*   `compiler-tracing12908925883751484166.spec`来自`compiler-tracing.spec`，其中包含`agent`相应参数
*   `trace.exe`中包含`ODASA_TRACER_CONFIGURATION`字符串，指向`spec`文件

可以通过`jd-gui`打开`codeql-java-agent.jar`，阅读其中代码，在`com.semmle.extractor.java.Utils#loadClass`中看到

```
private static Class<?> loadClass(String name) {
    Class result;
    try {
        result = Class.forName(name);
    } catch (ClassNotFoundException var10) {
        String extractorTools = getExtractorTools();
        if (extractorTools == null) {
            throw new RuntimeException("Failed to determine SEMMLE_DIST", var10);
        }

        File extractorJar = new File(extractorTools, "semmle-extractor-java.jar");
        if (!extractorJar.exists() || !extractorJar.canRead()) {
            throw new RuntimeException("Cannot read semmle-extractor-java jar from " + extractorJar + " -- check SEMMLE_DIST", var10);
        }

        URL url;
        try {
            url = extractorJar.getAbsoluteFile().toURI().toURL();
        } catch (MalformedURLException var9) {
            throw new RuntimeException("Failed to convert " + extractorJar + " to URL", var9);
        }

        URLClassLoader loader = new URLClassLoader(new URL[]{url});

        try {
            result = loader.loadClass(name);
        } catch (ClassNotFoundException var8) {
            throw new RuntimeException("Failed to load " + name + " from " + extractorJar + " -- check SEMMLE_DIST", var8);
        }
    }

    return result;
}
```

会通过`Utils`加载`semmle-extractor-java.jar`，`codeql-java-agent.jar`的代码量不大，其大致逻辑可以通过静态代码阅读的方式来理解。

从前面的分析结果来看，`Extracotr`的操作位于`semmle-extractor-java.jar`中，根据`process hacker`的内容在IDEA中新增一个`Debug`配置

> 由于中途更换了机器，所以某些路径看上去会不一样，但不影响阅读。此外由于`semmle-extractor-java.jar`中没有清单文件`MAINFEST.MF`，无法直接运行该`jar`包，所以创建`Application`进行`Debug`即可。

```
Main class: com.semmle.extractor.java.JavaExtractor
Program arguments: --jdk-version 16 --javac-args @@@E:\Documents\databases\app\log\ext\javac.args
Enviroment variables: CODEQL_PLATFORM=win64;CODEQL_PLATFORM_DLL_EXTENSION=.dll;CODEQL_EXTRACTOR_JAVA_LOG_DIR=E:\Documents\databases\app2\log;CODEQL_JAVA_HOME=E:\Program Files\codeql\tools\win64\java;CODEQL_EXTRACTOR_JAVA_SCRATCH_DIR=E:\Documents\databases\app2\working;CODEQL_EXTRACTOR_JAVA_WIP_DATABASE=E:\Documents\databases\app2;CODEQL_EXTRACTOR_JAVA_ROOT=E:\Program Files\codeql\java;CODEQL_EXTRACTOR_JAVA_TRAP_DIR=E:\Documents\databases\app2\trap\java;CODEQL_TRACER_LOG=E:\Documents\databases\app2\log\build-tracer.log;CODEQL_EXTRACTOR_JAVA_SOURCE_ARCHIVE_DIR=E:\Documents\databases\app2\src;CODEQL_DIST=E:\Program Files\codeql
```

先不急着调试，直接运行看看它运行后`database/app`目录下有什么变化。注意要在`log/ext`目录下放入相应的文件，这个可以从正常创建数据库的步骤中获取到。运行后会增加两个目录`src`和`trap`，`src`中会放置项目中的源代码，`trap`用于存放`trap`文件。

`codeql`提供了相关命令导入`trap`文件并生成数据库，在前面列出的创建过程中，也有出现它的身影。

```
Usage: codeql dataset <command> <argument>...
[Plumbing] Work with raw QL datasets.
Commands:
  import   [Plumbing] Import a set of TRAP files to a raw dataset.
  upgrade  [Plumbing] Upgrade a dataset so it is usable by the current tools.
  cleanup  [Plumbing] Clean up temporary files from a dataset.
  check    [Plumbing] Check a particular dataset for internal consistency.
  measure  [Plumbing] Collect statistics about the relations in a particular
             dataset.
```

`trap`文件夹中列出了项目源码以及`jdk`依赖中类的信息，文件夹的结构如下

```
|-- Java
    |-- classes
    |-- diagnostics
    `-- E_\Projects\IdeaProjects\app\src\main\java\org\example\
```

项目源码对应的`trap`文件位于`E_\Projects\IdeaProjects\app\src\main\java\org\example\`中，里面有`3`个文件，`App.java.dep`，`App.java.set`，`App.java.trap.gz`。可以将`App.java.trap.gz`解压缩查看`trap`文件的内容。项目代码只是调用`System.out.println`输出`Hello, World!`，所以它的内容相对简单，如下

> `CodeQL`的`DB`架构是基于`Datalog`的，如果你熟悉`Datalog`，那理解这个文件的内容也会容易许多。

```
// Generated by the CodeQL Java extractor
#10000=@"E:/Projects/IdeaProjects/app/src/main/java/org/example/App.java;sourcefile"
files(#10000,"E:/Projects/IdeaProjects/app/src/main/java/org/example/App.java")
#10001=@"E:/Projects/IdeaProjects/app/src/main/java/org/example;folder"
folders(#10001,"E:/Projects/IdeaProjects/app/src/main/java/org/example")
#10002=@"E:/Projects/IdeaProjects/app/src/main/java/org;folder"
folders(#10002,"E:/Projects/IdeaProjects/app/src/main/java/org")
#10003=@"E:/Projects/IdeaProjects/app/src/main/java;folder"
folders(#10003,"E:/Projects/IdeaProjects/app/src/main/java")
#10004=@"E:/Projects/IdeaProjects/app/src/main;folder"
folders(#10004,"E:/Projects/IdeaProjects/app/src/main")
#10005=@"E:/Projects/IdeaProjects/app/src;folder"
folders(#10005,"E:/Projects/IdeaProjects/app/src")
#10006=@"E:/Projects/IdeaProjects/app;folder"
folders(#10006,"E:/Projects/IdeaProjects/app")
#10007=@"E:/Projects/IdeaProjects;folder"
folders(#10007,"E:/Projects/IdeaProjects")
#10008=@"E:/Projects;folder"
folders(#10008,"E:/Projects")
#10009=@"E:/;folder"
folders(#10009,"E:/")
containerparent(#10009,#10008)
containerparent(#10008,#10007)
containerparent(#10007,#10006)
containerparent(#10006,#10005)
containerparent(#10005,#10004)
containerparent(#10004,#10003)
containerparent(#10003,#10002)
containerparent(#10002,#10001)
containerparent(#10001,#10000)
#10010=@"loc,{#10000},0,0,0,0"
locations_default(#10010,#10000,0,0,0,0)
hasLocation(#10000,#10010)
numlines(#10000,9,8,0)
#10011=@"package;org.example"
packages(#10011,"org.example")
cupackage(#10000,#10011)
#10012=@"class;org.example.App"
#10013=@"loc,{#10000},3,14,3,16"
locations_default(#10013,#10000,3,14,3,16)
hasLocation(#10012,#10013)
numlines(#10012,6,6,0)
#10014=@"type;void"
primitives(#10014,"void")
#10015=@"unknown;sourcefile"
files(#10015,"")
#10016=@"loc,{#10015},0,0,0,0"
locations_default(#10016,#10015,0,0,0,0)
hasLocation(#10014,#10016)
#10017=@"callable;{#10012}.<init>(){#10014}"
locations_default(#10013,#10000,3,14,3,16)
hasLocation(#10017,#10013)
numlines(#10017,1,1,0)
#10018=*
stmts(#10018,0,#10017,0,#10017)
#10019=*
locations_default(#10019,#10000,3,14,3,16)
hasLocation(#10018,#10019)
numlines(#10018,1,1,0)
#10020=*
stmts(#10020,20,#10018,0,#10017)
#10021=*
locations_default(#10021,#10000,3,14,3,16)
hasLocation(#10020,#10021)
numlines(#10020,1,1,0)
#10022=@"class;java.lang.Object"
#10023=@"callable;{#10022}.<init>(){#10014}"
callableBinding(#10020,#10023)
#10024=@"class;java.lang.String"
#10025=@"array;1;{#10024}"
arrays(#10025,"String[]",#10024,1,#10024)
locations_default(#10016,#10015,0,0,0,0)
hasLocation(#10025,#10016)
#10026=@"field;{#10025};length"
#10027=@"type;int"
fields(#10026,"length",#10027,#10025,#10026)
#10028=@"modifier;public"
modifiers(#10028,"public")
hasModifier(#10026,#10028)
#10029=@"modifier;final"
modifiers(#10029,"final")
hasModifier(#10026,#10029)
#10030=@"callable;{#10025}.clone(){#10025}"
methods(#10030,"clone","clone()",#10025,#10025,#10030)
hasModifier(#10030,#10028)
extendsReftype(#10025,#10022)
#10031=@"class;java.lang.Cloneable"
implInterface(#10025,#10031)
#10032=@"class;java.io.Serializable"
implInterface(#10025,#10032)
#10033=@"callable;{#10012}.main({#10025}){#10014}"
#10034=@"loc,{#10000},5,24,5,27"
locations_default(#10034,#10000,5,24,5,27)
hasLocation(#10033,#10034)
numlines(#10033,4,4,0)
#10035=*
stmts(#10035,0,#10033,0,#10033)
#10036=*
locations_default(#10036,#10000,6,5,8,5)
hasLocation(#10035,#10036)
numlines(#10035,3,3,0)
#10037=*
exprs(#10037,62,#10014,#10033,-1)
callableEnclosingExpr(#10037,#10033)
#10038=*
locations_default(#10038,#10000,5,19,5,22)
hasLocation(#10037,#10038)
numlines(#10037,1,1,0)
#10039=@"params;{#10033};0"
params(#10039,#10025,0,#10033,#10039)
paramName(#10039,"args")
#10040=@"loc,{#10000},5,30,5,42"
locations_default(#10040,#10000,5,30,5,42)
hasLocation(#10039,#10040)
#10041=*
exprs(#10041,63,#10025,#10039,-1)
callableEnclosingExpr(#10041,#10033)
#10042=*
locations_default(#10042,#10000,5,30,5,37)
hasLocation(#10041,#10042)
numlines(#10041,1,1,0)
#10043=*
exprs(#10043,62,#10024,#10041,0)
callableEnclosingExpr(#10043,#10033)
#10044=*
locations_default(#10044,#10000,5,30,5,35)
hasLocation(#10043,#10044)
numlines(#10043,1,1,0)
#10045=*
stmts(#10045,14,#10035,0,#10033)
#10046=*
locations_default(#10046,#10000,7,9,7,45)
hasLocation(#10045,#10046)
numlines(#10045,1,1,0)
#10047=*
exprs(#10047,61,#10014,#10045,0)
callableEnclosingExpr(#10047,#10033)
statementEnclosingExpr(#10047,#10045)
#10048=*
locations_default(#10048,#10000,7,9,7,44)
hasLocation(#10047,#10048)
numlines(#10047,1,1,0)
#10049=*
#10050=@"class;java.io.PrintStream"
exprs(#10049,60,#10050,#10047,-1)
callableEnclosingExpr(#10049,#10033)
statementEnclosingExpr(#10049,#10045)
#10051=*
locations_default(#10051,#10000,7,9,7,18)
hasLocation(#10049,#10051)
numlines(#10049,1,1,0)
#10052=@"callable;{#10050}.println({#10024}){#10014}"
callableBinding(#10047,#10052)
#10053=*
exprs(#10053,22,#10024,#10047,0)
callableEnclosingExpr(#10053,#10033)
statementEnclosingExpr(#10053,#10045)
#10054=*
locations_default(#10054,#10000,7,29,7,42)
hasLocation(#10053,#10054)
numlines(#10053,1,1,0)
#10055=*
#10056=@"class;java.lang.System"
exprs(#10055,62,#10056,#10049,-1)
callableEnclosingExpr(#10055,#10033)
statementEnclosingExpr(#10055,#10045)
#10057=*
locations_default(#10057,#10000,7,9,7,14)
hasLocation(#10055,#10057)
numlines(#10055,1,1,0)
#10058=@"field;{#10056};out"
variableBinding(#10049,#10058)
namestrings("""Hello World!""","Hello World!",#10053)
```

它的内部并不会太难理解，首先这个文件是根据`semmlecode.dbscheme`文件所创建的，每种语言的`extractor`下都有一个这样的文件。

`#10000=@"E:/Projects/IdeaProjects/app/src/main/java/org/example/App.java;sourcefile"`

`#10000`可理解为一个标签，类似于数据库表格某一列的`id`，每个`trap`文件的标签都是独立的。

`files(#10000,"E:/Projects/IdeaProjects/app/src/main/java/org/example/App.java")`

这是一段声明，这个声明是按照`semmlecode.dbscheme`中的约定构建的，你可以在该文件中看到

```
folders(
  unique int id: @folder,
  string name: string ref
);
```

所以上面的内容表示了一个文件，它的`id`为`#10000`，路径为`E:/Projects/IdeaProjects/app/src/main/java/org/example/App.java`。

其余的声明都可以按相同的逻辑来理解。

下面跟进源码看看它具体做了什么。

在`com.semmle.extractor.java.JavaExtractor#main`打下断点，先根据传入的参数创建`JavaExtractor`对象再调用`runExtractor`执行`extractor`操作生成`trap`文件。`jarac-extractor*.log`日志文件对象由静态代码块中的`LOG_ID = MarkerFactory.getMarker("javac-extractor" + PID);`创建

```
public static void main(String[] args) {
    String allArgs = StringUtil.glue(" ", args);
    JavaExtractor extractor = new JavaExtractor(args);
    boolean hasJavacErrors = false;

    try {
        hasJavacErrors = !extractor.runExtractor();
    } catch (Throwable var8) {
        label102: {
            if (extractor.log != null) {
                extractor.log.error("Exception running the extractor with arguments: {}", allArgs);
                extractor.log.error("Exception: ", var8);
            }

            if (!(var8 instanceof Abort) && !(var8 instanceof FatalError)) {
                if (!(var8 instanceof OutOfMemoryError) && !(var8 instanceof UnknownError)) {
                    break label102;
                }

                throw var8;
            }

            throw var8;
        }
    } finally {
        extractor.close();
    }

    if (extractor.strictJavacErrors && hasJavacErrors) {
        throw new UserError("Compilation errors were reported by javac.");
    }
}
```

跟进`runExtractor`看看，代码内容很长，增加了一些注释以便理解

```
boolean runExtractor() {
    long time = System.nanoTime();
    long cpuTime = getCurrentThreadCpuTime();
    Context context = this.output.getContext();
    /* 创建日志对象，将内容写入javac-output+进程id文件 */
    Factory<PrintWriter> logFactory = new Factory<PrintWriter>() {
        public PrintWriter make(Context c) {
            return new PrintWriter(LogbackUtils.streamFor(JavaExtractor.this.log, "javac-output" + JavaExtractor.PID, false));
        }
    };
    context.put(Log.outKey, logFactory);
    context.put(Log.errKey, logFactory);
    JavacFileManager.preRegister(context, this.specialSourcepathHandling);
    /* javac 参数 */
    Arguments arguments = this.setupJavacOptions(context);
    Options.instance(context).put("ignore.symbol.file", "ignore.symbol.file");
    JavaFileManager jfm = (JavaFileManager)context.get(JavaFileManager.class);
    JavaFileManager bfm = jfm instanceof DelegatingJavaFileManager ? ((DelegatingJavaFileManager)jfm).getBaseFileManager() : jfm;
    JavacFileManager dfm = (JavacFileManager)bfm;
    dfm.handleOptions(arguments.getDeferredFileManagerOptions());
    arguments.validate();
    if (jfm.isSupportedOption(Option.MULTIRELEASE.primaryName) == 1) {
        Target target = Target.instance(context);
        List<String> list = List.of(target.multiReleaseValue());
        jfm.handleOption(Option.MULTIRELEASE.primaryName, list.iterator());
    }

    JavaCompiler compiler = JavaCompiler.instance(context);
    compiler.genEndPos = true;

    /* 列出待编译的文件 */
    Set<JavaFileObject> fileObjects = arguments.getFileObjects();
    /* DiagnosticTrapWriter类用于向trap/java/diagnostics中写入诊断信息（也就是日志） */
    DiagnosticTrapWriter diagWriter = this.dw.getDiagnosticTrapWriter();
    if (diagWriter != null) {
        Iterator var14 = fileObjects.iterator();

        while(var14.hasNext()) {
            JavaFileObject jfo = (JavaFileObject)var14.next();
            diagWriter.writeFileArgument(jfo);
        }
    }

    /* 通过javac解析源代码文件，拿到上下文信息 */
    javac_extend.com.sun.tools.javac.util.List<JCCompilationUnit> parsedFiles = compiler.parseFiles(fileObjects);
    compiler.enterTrees(compiler.initModules(parsedFiles));
    Queue<Queue<javac_extend.com.sun.tools.javac.comp.Env<AttrContext>>> groupedTodos = Todo.instance(context).groupByFile();
    long javacInitTime = System.nanoTime() - time;
    long javacInitCpuTime = getCurrentThreadCpuTime() - cpuTime;
    if (diagWriter != null) {
        diagWriter.writeCompilationFileTime((double)javacInitCpuTime / 1.0E9D, (double)javacInitTime / 1.0E9D, 0.0D, 0.0D);
    }

    int prevErr = 0;

    while(true) {
        long currJavacCpu;
        long cpu;
        long currJavacTime;
        while(true) {
            JCCompilationUnit cu;
            while(true) {
                Queue todo;
                do {
                    /* 检查待做事项，没有的话就返回 */
                    if ((todo = (Queue)groupedTodos.poll()) == null) {
                        long totalExtractorTime = System.nanoTime() - this.extractorStartTime;
                        this.log(String.format("Javac init time: %.1fs", (double)javacInitTime / 1.0E9D));
                        this.log(String.format("Javac attr time: %.1fs", (double)this.javacTime / 1.0E9D));
                        this.log(String.format("Extractor time: %.1fs", (double)this.extractorTime / 1.0E9D));
                        long otherTime = totalExtractorTime - javacInitTime - this.javacTime - this.extractorTime;
                        this.log(String.format("Other time: %.1fs", (double)otherTime / 1.0E9D));
                        this.log(String.format("Total time: %.1fs", (double)totalExtractorTime / 1.0E9D));
                        int totalErrors = compiler.errorCount();
                        compiler.close();
                        if (diagWriter != null) {
                            diagWriter.writeCompilationFinished((double)getCurrentThreadCpuTime() / 1.0E9D, (double)totalExtractorTime / 1.0E9D);
                        }

                        if (totalErrors != 0) {
                            this.log.error(LOG_ID, totalErrors + " errors were reported by javac.");
                            return false;
                        }

                        return true;
                    }

                    cu = null;
                    Iterator var23 = todo.iterator();

                    while(var23.hasNext()) {
                        javac_extend.com.sun.tools.javac.comp.Env<AttrContext> env = (javac_extend.com.sun.tools.javac.comp.Env)var23.next();
                        if (cu == null) {
                            cu = env.toplevel;
                        } else if (cu != env.toplevel) {
                            throw new CatastrophicError("Not grouped by file: CUs " + cu + " and " + env.toplevel);
                        }
                    }
                } while(cu == null);

                if (diagWriter != null) {
                    diagWriter.writeCompilationFileStart(cu);
                }

                cpu = getCurrentThreadCpuTime();
                time = System.nanoTime();

                try {
                    Queue<javac_extend.com.sun.tools.javac.comp.Env<AttrContext>> queue = compiler.attribute(todo);
                    String envFlowChecks = System.getenv("CODEQL_EXTRACTOR_JAVA_FLOW_CHECKS");
                    if (envFlowChecks == null || Boolean.valueOf(envFlowChecks)) {
                        compiler.flow(queue);
                    }
                    break;
                } catch (StackOverflowError | Exception var36) {
                    this.logThrowable(cu, var36);
                }
            }

            currJavacTime = System.nanoTime() - time;
            this.javacTime += currJavacTime;
            currJavacCpu = getCurrentThreadCpuTime() - cpu;
            cpu = getCurrentThreadCpuTime();
            time = System.nanoTime();

            try {
                CharSequence cachedContent = dfm.getCachedContent(cu.getSourceFile());
                if (cachedContent == null) {
                    try {
                        cachedContent = cu.getSourceFile().getCharContent(false);
                    } catch (IOException var37) {
                        this.logThrowable(cu, var37);
                        continue;
                    }
                }

                String contents = ((CharSequence)cachedContent).toString();
                /** 
                 * 根据compiler处理的结果，进行extractor操作
                 * this.output 存有`trap`和`src`文件的保存路径
                 *    trapFolder=E:\Documents\databases\app2\trap\java
                 *    sourceArchiveFolder=E:\Documents\databases\app2\src
                 */
                (new CompilationUnitExtractor(this.output, cu, this.dw)).process(contents);
            } catch (StackOverflowError | Exception var38) {
                this.logThrowable(cu, var38);
            }
            break;
        }

        long currExtractorTime = System.nanoTime() - time;
        this.extractorTime += currExtractorTime;
        long currExtractorCpu = getCurrentThreadCpuTime() - cpu;
        if (diagWriter != null) {
            diagWriter.writeCompilationFileTime((double)currJavacCpu / 1.0E9D, (double)currJavacTime / 1.0E9D, (double)currExtractorCpu / 1.0E9D, (double)currExtractorTime / 1.0E9D);
            int currErr = compiler.errorCount();
            int deltaErr = currErr - prevErr;
            if (deltaErr > 0) {
                String errorMsg = String.valueOf(deltaErr);
                diagWriter.writeDiagnostic(DiagSeverity.ErrorHigh, errorMsg, DiagKind.SOURCE, (Label)null);
            }

            prevErr = currErr;
        }
    }
}
```

`process`函数的内容如下，根据输入的源代码文件内容进行处理，而`CompilationUnitExtractor`在创建时传入的`cu(JCCompilationUnit)`对象，保存着编辑器处理后的上下文信息。

![](https://images.seebug.org/content/images/2022/06/8e909631-820d-4fdb-a489-a7f22f9b9741.png-w331s)

以`ClassDeclExtractor#visitClassDef`为例，会通过调用`this.onDemand.getClassKey`得到当前类的唯一标签，其它方法也是类似的。

```
public void visitClassDef(JCClassDecl that) {
    if (this.onDemand.getOutput().getTrackClassOrigins()) {
        this.attributeClassFile(that);
    }

    if (this.extractedClasses.add(that)) {
        this.enclosingCallables.push((Object)null);
        this.enclosingStatements.push((Object)null);
        if (that.type instanceof ClassType) {
            this.onDemand.extractPrivateMembers((ClassType)that.type);
        } else {
            this.log.error(DiagKind.SOURCE, this.treeUtil, "Unexpected type for class " + that.name + ": " + that.type, that);
        }

        Label classId;
        if (that.sym != null) {
            /* 获取标签#10012 */
            classId = this.onDemand.getClassKey(that.sym);
            this.treeUtil.writeKeyedLocation(this.writer, that, classId);
            this.treeUtil.writeJavadocAssociation(this.writer, classId, that);

    ...
}
```

整个`java extractor`的代码量太多，我没有深入研究各个部分。`Extractor`有用到名为`javac_extend.com.sun.tools.javac`的包来进行`javac`的操作，但是`jdk`中只有`com.sun.tools.javac`，并不清楚两者的差异具体体现在哪里，但可以看出是进行了一定修改的。整个`jar`包就像缝合怪，将很多功能修改后嵌入在里面。

这样整个`Extractor`的工作流程大概了解，

*   根据`javac`配置文件创建`javac compiler`对象
*   `javac`对源码一次进行预处理
*   根据前一步出的处理结果，构造`trap`文件

由于涉及到的内容较多且广泛，继续深入可能会让我陷入泥沼，了解其作用和用法即可，如果有缘会再回来看看。

从前面的分析大致能看出，数据的构建过程中，`codeql`并不需要完整的去编译源代码，只是借助`javac`从源码中那拿点东西。其次，只要能够根据源码文件构造正确的`javac.args`，就可以生成`trap`文件了。之后再通过`codeql database finalize`即可得到一个数据库。

这种想法在\[2\]中已经提及，只是可能由于反编译时代码的正确性无法保证完美，其次编译时各个文件编译的先后顺序不同都会导致构造`trap`出现错误。但另一种更简单直接的方式是根据反编译结果，构造编译命令，然后通过`codeql database create`并指定构造好的编译命令即可，在`github`中也有相关项目。

按照前面的分析结果，`CodeQL`创建数据库的过程中并不关心整个编译过程和结果，只是借用编译过程中的部分数据。那么对于任何`java`代码，无论其构建系统为何，只要能够让编译该`java`文件时，编译器不应错误而退出，那么数据库的创建过程就可以正常进行下去。

可以通过下面的脚本来创建数据库，这里以`dubbo`项目为实例，使用先需要下载好[ecj.jar](https://mvnrepository.com/artifact/org.eclipse.jdt.core.compiler/ecj/4.6.1)，这里使用`ecj`的目的是，相比`javac`而言，它更能容忍编译错误，从而避免创建数据库过程失败。

```
import pathlib
import os


def compile_cmd_file_create(save_path, ecj_path):
    with open("{}/file.txt".format(save_path), "w+") as f:
        for java_path in pathlib.Path(save_path).glob('**/*.java'):
            f.write(str(java_path) + "\n")
    ecj_absolute_path = pathlib.Path(ecj_path).resolve()
    compile_cmd = "java -jar {} -encoding UTF-8 -8 " \
                  "-warn:none -noExit @{}/file.txt".format(ecj_absolute_path, save_path)

    with open("{}/run.cmd".format(save_path), "w+") as f:
        f.write(compile_cmd)

    with open("{}/run.sh".format(save_path), "w+") as f:
        f.write(compile_cmd)


if __name__ == '__main__':
    self_ecj_path = os.getcwd() + r"/ecj-4.6.1.jar"

    compile_cmd_file_create(os.getcwd() + r"/dubbo", self_ecj_path)
```

运行后会在`os.getcwd() + r"/dubbo"`中生成`run.sh/run.cmd`文件，之后进入`os.getcwd() + r"/dubbo"`运行

```
codeql database create --language=java -c "bash run.sh"  <path to database>
```

~就可以快速创建数据库，当然它与通过正常方式创建的结果是否一致尚未验证。~ 这种不顾编译错误情况的方式创建的数据库，会丢失数据流的信息从而导致失去它存在的意义，因为当编译某个文件它的依赖未找到时，生成的`trap`文件也是不完整的。

References
----------

1.[https://help.semmle.com/lgtm-enterprise/user/help/generate-database.html](https://help.semmle.com/lgtm-enterprise/user/help/generate-database.html)  
2\. [https://testanull.com/build-codeql-db-without-source-code](https://testanull.com/build-codeql-db-without-source-code)  
3\. [https://paper.seebug.org/1324/](https://paper.seebug.org/1324/)

* * *

![](https://images.seebug.org/content/images/2017/08/0e69b04c-e31f-4884-8091-24ec334fbd7e.jpeg)
 本文由 Seebug Paper 发布，如需转载请注明来源。本文地址：[https://paper.seebug.org/1921/](https://paper.seebug.org/1921/)