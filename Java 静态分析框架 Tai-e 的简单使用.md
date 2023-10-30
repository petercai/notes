# Java 静态分析框架 Tai-e 的简单使用
**作者：y4er  
原文链接：[https://y4er.com/posts/simple-use-of-the-java-static-analysis-framework-tai-e](https://y4er.com/posts/simple-use-of-the-java-static-analysis-framework-tai-e)**

### 前言

在做代码审计的时候，总是遇到一些批量垃圾洞，或者是遇到需要自动化批量找调用链验证的工作，一直想着解决这个问题，后来发现tabby，用了一段时间，总觉得不太舒服，配置不足oom异常加上非人的neo4j的语法，加上太多的toString、equals等无用调用关系，配合上杂乱的neo4j的图，有点扰乱审计思路。

自己照着tabby抄了一个poop出来，发现自己的问题并没有解决，只是熟悉了一下soot的基础用法，会抽取类信息了而已，在此期间狠狠补了一下soot，逐字逐句翻译啃完了英文的《soot存活指南》 https://www.brics.dk/SootGuide/sootsurvivorsguide.pdf 然后发现soot出了一个新版本的sootup，自己试了试ifds污点分析。

对于指针分析、污点分析还是一知半解，中间尝试过bytecodedl、doop这种声明式的分析工具，然后发现suffle语法更变态，鬼画符，加上没有详细的文档和对应的规则，自己想要进一步拓展过于困难。

思考很久，发现还是自己底子不扎实，于是学了很长一段时间的静态软件分析，看了很多的论文（~折磨~）和视频，其中包括南京大学谭添、李樾两位老师的课，北大熊英飞老师的课等等，今天就简单写一下谭添、李樾两位老师开发的tai-e指针分析框架的简单使用。

防喷：我只是看了课，并不代表我会了，很惭愧的是两位老师的课我看了第二遍有些地方还是不太理解，但是每看一遍总有新收获，所以本文有错实属正常不过，望读者赐教。

### 配置tai-e

GitHub的wiki给了[配置教程](https://github.com/pascal-lab/Tai-e/wiki/Setup-Tai%E2%80%90e-in-IntelliJ-IDEA)

```
git clone https://github.com/pascal-lab/Tai-e
cd Tai-e
git submodule update --init --recursive
```

idea打开 需要jdk17

![](_assets/b7757ffb-f097-41e6-9b35-7d998fe31874.png-w331s.png)

gradle也需要jdk17

![](_assets/1695290373000-2mrqch.png-w331s.png)

然后运行`pascal.taie.Main`，配置下主类，加一个jvm options `Xmx`防止oom异常

![](_assets/1695290392000-3mjdim.png-w331s.png)

输出

```
Tai-e starts ...
Writing options to output\options.yml
Usage: Options [-gh] [-ap] [--[no-]native-model] [-pp] [--pre-build-ir] [-cp=<classPath>] [-java=<javaVersion>]
               [-m=<mainClass>] [--options-file=<optionsFile>] [-p=<planFile>] [-scope=<scope>]
               [--world-builder=<worldBuilderClass>] [-a=<String=String>]... [--input-classes=<inputClass>[,
               <inputClass>...]]...
Tai-e options
  -a, --analysis=<String=String>      Analyses to be executed
      -ap, --allow-phantom            Allow Tai-e to process phantom references, i.e., the referenced classes that are
                                        not found in the class paths (default: false)
      -cp, --class-path=<classPath>   Class path. Multiple paths are split by system path separator.
  -g, --gen-plan-file                 Merely generate analysis plan
  -h, --help                          Display this help message
      --input-classes=<inputClass>[,<inputClass>...]
                                      The classes should be included in the World of analyzed program (the classes can
                                        be split by ',')
      -java=<javaVersion>             Java version used by the program being analyzed (default: 6)
  -m, --main-class=<mainClass>        Main class
      --[no-]native-model             Enable native model (default: true)
      --options-file=<optionsFile>    The options file
  -p, --plan-file=<planFile>          The analysis plan file
      -pp, --prepend-JVM              Prepend class path of current JVM to Tai-e's class path (default: false)
      --pre-build-ir                  Build IR for all available methods before starting any analysis (default: false)
      -scope=<scope>                  Scope for method/class analyses (default: APP, valid values: APP, REACHABLE, ALL)
      --world-builder=<worldBuilderClass>
                                      Specify world builder class (default: pascal.taie.frontend.soot.SootWorldBuilder)
--------------------
Version 0.1
--------------------
```

### 运行参数

列举几个关键参数，或者直接[看文档](https://github.com/pascal-lab/Tai-e/wiki/How-to-Run-Tai%E2%80%90e%3F-(command%E2%80%90line-options))

| 参数 | 示例 | 用途 |
| --- | --- | --- |
| `-ap` | `-ap` | 允许虚引用，等同于soot的`--allow-phantom` |
| `-java` | `-java 8` | 指定java版本为jre8 tai-e会从`java-benchmarks/JREs`加载对应的jdk lib |
| `-pp` | `-pp` | 将当前jvm的类路径添加到分析类路径中 和`-java`选项冲突 |
| `-m` | `-m com.example.demo.Main` | 指定主类 表示程序入口 必选参数 |
| `--input-classes` | `--input-classes=com.example.demo.controller.TestController,javax.servlet.ServletRequestWrapper` | 当main函数无法调用到TestController时，可以用这个参数把TestController强制加进来，类似于强制分析？ |
| `-cp` | `-cp E:\demo5\target\classes;.\lib\test.jar` | 类路径 和soot差不多 支持jar文件或者`.java`、`.class`文件目录 **在Windows中多个jar以`;`分隔，unix以`:`分隔** |
| `-g` | `-g` | 仅生成选项配置文件`output/options.yml`不执行分析 |
| `--options-file` | `--options-file=output/options.yml` | 解析配置文件作为选项配置 |
| `--pre-build-ir` | `--pre-build-ir` | 分析之前为所有的method构建IR |
| `-scope` | `-scope=APP` | 指定分析类和方法的分析范围APP, REACHABLE, ALL |
| `-a` | `-a <id>[=<key>:<value>;...]` | 指定分析选项，`-a`可重复指定多个分析，选项会保存在`output/tai-e-plan.yml`文件中 |
| `-p` | `-p output/tai-e-plan.yml` | 用文件指定分析选项 yaml语法 |

举一个污点分析的例子

```
-cp
E:\tools\code\demo5\target\classes;E:\tools\code\demo5\target\demo5-0.0.1-SNAPSHOT\BOOT-INF\lib\spring-aop-5.3.23.jar;E:\tools\code\demo5\target\demo5-0.0.1-SNAPSHOT\BOOT-INF\lib\spring-beans-5.3.23.jar;E:\tools\code\demo5\target\demo5-0.0.1-SNAPSHOT\BOOT-INF\lib\spring-boot-2.7.4.jar;E:\tools\code\demo5\target\demo5-0.0.1-SNAPSHOT\BOOT-INF\lib\spring-boot-autoconfigure-2.7.4.jar;E:\tools\code\demo5\target\demo5-0.0.1-SNAPSHOT\BOOT-INF\lib\spring-boot-jarmode-layertools-2.7.4.jar;E:\tools\code\demo5\target\demo5-0.0.1-SNAPSHOT\BOOT-INF\lib\spring-context-5.3.23.jar;E:\tools\code\demo5\target\demo5-0.0.1-SNAPSHOT\BOOT-INF\lib\spring-core-5.3.23.jar;E:\tools\code\demo5\target\demo5-0.0.1-SNAPSHOT\BOOT-INF\lib\spring-expression-5.3.23.jar;E:\tools\code\demo5\target\demo5-0.0.1-SNAPSHOT\BOOT-INF\lib\spring-jcl-5.3.23.jar;E:\tools\code\demo5\target\demo5-0.0.1-SNAPSHOT\BOOT-INF\lib\spring-web-5.3.23.jar;E:\tools\code\demo5\target\demo5-0.0.1-SNAPSHOT\BOOT-INF\lib\spring-webmvc-5.3.23.jar;E:\tools\code\demo5\target\demo5-0.0.1-SNAPSHOT\BOOT-INF\lib\tomcat-embed-core-9.0.65.jar;E:\tools\code\demo5\target\demo5-0.0.1-SNAPSHOT\BOOT-INF\lib\tomcat-embed-el-9.0.65.jar;E:\tools\code\demo5\target\demo5-0.0.1-SNAPSHOT\BOOT-INF\lib\tomcat-embed-websocket-9.0.65.jar
--input-classes=com.example.demo.controller.TestController,javax.servlet.ServletRequestWrapper,javax.servlet.ServletResponseWrapper,org.apache.catalina.connector.Request
-java
8
-m
com.example.demo.Main
-ap
-a pta=action:dump;action-file:result.txt;taint-config:src\test\resources\pta\taint\taint-config.yml
```

直接配置在idea的参数中就行，表示在给定的几个jar包和class中做pta，指定了`taint-config`文件表示做p/taint污点分析，强制指定`com.example.demo.controller.TestController`控制器和几个引用类，允许虚类，使用的污点配置文件为`src\test\resources\pta\taint\taint-config.yml`，结果导出到result.txt文件中。

需要重点讲一下`-a`参数，tai-e有三大类参数

1.  Program options 指定程序执行的参数
2.  Analysis options 指定执行代码分析时的参数
3.  其他选项 `-h`这类

`-a`就是第二种，涉及到代码分析时需要指定的参数，在`src/main/resources/tai-e-analyses.yml`中，tai-e作为一个插件式的框架将分析插件模块化，对应的配置放在了这个文件中。

拿出来一段配置来看

```
- description: whole-program pointer analysis
  analysisClass: pascal.taie.analysis.pta.PointerAnalysis
  id: pta
  options:
    cs: ci # | k-[obj|type|call][-k'h]
    only-app: false # only analyze application code
    implicit-entries: true # analyze implicit entries
    merge-string-constants: false
    merge-string-objects: true
    merge-string-builders: true
    merge-exception-objects: true
    handle-invokedynamic: false
    advanced: null # specify advanced analysis:
    # zipper | zipper-e | zipper-e=PV
    # scaler | scaler=TST
    # mahjong | collection
    action: null # | dump | compare
    action-file: null # path of file to dump/compare
    reflection-log: null # path to reflection log
    taint-config: null # path to config file of taint analysis,
    # when this file is given, taint analysis will be enabled
    plugins: [ ] # | [ pluginClass, ... ]

- description: call graph construction
  analysisClass: pascal.taie.analysis.graph.callgraph.CallGraphBuilder
  id: cg
  requires: [ pta(algorithm=pta) ]
  options:
    algorithm: pta # | cha
    dump: null # path of file to dump reachable methods and call edges
    dump-methods: null # path of file to dump reachable methods
    dump-call-edges: null # path of file to dump to call edges
```

id作为插件的唯一标识，当指定`-a pta`时表明用`pascal.taie.analysis.pta.PointerAnalysis`类进行分析，options指定了对应类的相关选项，其中id为cg的插件有一个选项为`requires: [ pta(algorithm=pta) ]`表示调用图构造需要用到pta的分析结果，相当于依赖。

tai-e实现了很多的分析插件，列一下

1.  whole-program pointer analysis 全程序指针分析
2.  call graph construction 调用图
3.  identify casts that may fail 识别可能失败的强制类型转换
4.  identify polymorphic callsites 识别多态callsites
5.  throw analysis 异常分析
6.  intraprocedural control-flow graph 过程内控制流图
7.  interprocedural control-flow graph 过程间控制流图
8.  live variable analysis 存活变量分析
9.  available expression analysis 有效表达分析
10.  reaching definition analysis 可达分析
11.  constant propagation 常量传播
12.  inter-procedural constant propagation 过程间的常量传播
13.  dead code detection 死代码检查
14.  process results of previously-run analyses 结果分析处理
15.  dump classes and Tai-e IR 导出类和IR
16.  null value analysis 空指针分析
17.  Null pointer and redundant comparison detector 空指针和冗余比较检查
18.  find clone() related problems clone相关的问题
19.  find the method that may drop or ignore exceptions 找到可能删除或忽略异常的方法

基本上用到的都在里面了。

### 使用tai-e分析javase程序

javase程序指的是命令行/桌面程序，javaee指的是web程序。

两者区别在于程序入口点不同，在javaee中存在多个servlet/controller，需要将多个路由对应的方法加入到静态软件分析的入口点entrypoint，而javase只有一个main函数，整个数据流是从main函数进去然后流向其他函数最终到sink点。

tai-e中需要指定`-m`参数指定程序主类，对于javaee来说需要增加分析入口点。接下来先以一个简单的javase项目为例学习tai-e的污点分析用法

新建一个maven空项目，创建主类 `org.example.Main`

```
package org.example;

import java.io.IOException;

public class Main {
    public static void main(String[] args) throws IOException {
        Main main = new Main();
        String source = main.source(args[0]);
        main.sink(source);
    }

    public String source(String s) {
        return s;
    }

    public String sink(String s) throws IOException {
        Runtime.getRuntime().exec(s);
        return "ok";
    }
}
```

修改`src/test/resources/pta/taint/taint-config.yml`污点规则文件

```
sources:
  - { method: "<org.example.Main: java.lang.String source(java.lang.String)>", type: "java.lang.String" }

sinks:
  - { method: "<java.lang.Runtime: java.lang.Process exec(java.lang.String)>", index: 0 }

transfers:
  - { method: "<java.lang.String: java.lang.String concat(java.lang.String)>", from: base, to: result, type: "java.lang.String" }
  - { method: "<java.lang.String: java.lang.String concat(java.lang.String)>", from: 0, to: result, type: "java.lang.String" }
  - { method: "<java.lang.String: char[] toCharArray()>", from: base, to: result, type: "char[]" }
  - { method: "<java.lang.String: void <init>(char[])>", from: 0, to: base, type: "java.lang.String" }
  - { method: "<java.lang.StringBuffer: java.lang.StringBuffer append(java.lang.String)>", from: 0, to: base, type: "java.lang.StringBuffer" }
  - { method: "<java.lang.StringBuffer: java.lang.StringBuffer append(java.lang.Object)>", from: 0, to: base, type: "java.lang.StringBuffer" }
  - { method: "<java.lang.StringBuffer: java.lang.String toString()>", from: base, to: result, type: "java.lang.String" }
  - { method: "<java.lang.StringBuilder: java.lang.StringBuilder append(java.lang.String)>", from: 0, to: base, type: "java.lang.StringBuilder" }
  - { method: "<java.lang.StringBuilder: java.lang.StringBuilder append(java.lang.Object)>", from: 0, to: base, type: "java.lang.StringBuilder" }
  - { method: "<java.lang.StringBuilder: java.lang.String toString()>", from: base, to: result, type: "java.lang.String" }
```

运行tai-e给定如下参数

```
-cp
E:\tools\code\aa\src\main\java
-java
8
-m
org.example.Main
-ap
-a
pta=action:dump;action-file:result.txt;taint-config:src\test\resources\pta\taint\taint-config.yml
```

指定`E:\tools\code\aa\src\main\java`为classpath，指定java版本为8，指定主类，允许幻象引用，启用指针分析和污点分析，并将污点分析结果导出到result.txt文件中。

查看txt文件会发现 tai-e列出了污点的信息流

```
Detected 1 taint flow(s):
TaintFlow{<org.example.Main: void main(java.lang.String[])>[5@L8] temp$4 = invokevirtual main.<org.example.Main: java.lang.String source(java.lang.String)>(temp$3); -> <org.example.Main: java.lang.String sink(java.lang.String)>[1@L17] invokevirtual temp$0.<java.lang.Runtime: java.lang.Process exec(java.lang.String)>(s);/0}
```

### 分析java web

以一个java web tomcat servlet项目为例

存在如下的servlet

```
package com.example.demo6;

import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@WebServlet(name = "helloServlet", value = "/hello-servlet")
public class HelloServlet extends HttpServlet {
    public void doGet(HttpServletRequest request, HttpServletResponse response) throws IOException {
        String source = request.getParameter("source");
        Runtime.getRuntime().exec(source);
    }
}
```

在上面说到分析java web项目需要我们自己定义分析入口点，并且模拟参数对象，所以我们需要修改下污点分析的处理类`pascal.taie.analysis.pta.plugin.taint.TaintAnalysis`

tai-e实现了插件式编程，将分析拆成小模块，官方wiki中提到了[《如何写一个分析插件》](https://github.com/pascal-lab/Tai-e/wiki/Pointer-Analysis-Framework#an-example-of-plugin)，除了写分析插件以外，还可以[《创建新的分析》](https://github.com/pascal-lab/Tai-e/wiki/How-to-Develop-A-New-Analysis-on-Tai%E2%80%90e%3F)

接下来我们将针对java web修改TaintAnalysis类，使污点分析时识别路由并将其加入到entrypoint中，即增加tai-e分析入口点。

TaintAnalysis类实现了`pascal.taie.analysis.pta.plugin.Plugin`接口，该接口有几个生命周期函数

![](_assets/1695290401000-4rjlai.png-w331s.png)

我们增加程序分析入口点肯定是在onStart函数中，所以在TaintAnalysis类重写onStart函数

```
@Override
public void onStart() {
}
```

那么如何添加entrypoint呢？我搜了issue，发现有人有和我同样的问题，[官方也给出了解决方案](https://github.com/pascal-lab/Tai-e/issues/9#issuecomment-1251836524)

添加entrypoint需要调用`pascal.taie.analysis.pta.core.solver.Solver#addEntryPoint`函数，该函数需要一个EntryPoint对象，EntryPoint构造函数中需要两个参数`JMethod method, ParamProvider paramProvider`，分别对应了入口点函数的JMethod对象，和入口点函数的参数处理器。其中ParamProvider接口有几个实现类

![](_assets/1bdf98f7-f940-497d-bd7c-b614af668101.png-w331s.png)

分别对应不同情况下的参数提供器。其中MainEntryPointParamProvider就是对应的Main函数的参数处理器

![](_assets/b9472563-72cf-47a4-b230-6d7753b138fa.png-w331s.png)

其中getParamObjs调用getMainArgs拿到模拟的main函数的参数`String[] args`，模拟参数用了`heapModel.getMockObj()`。

[官方的代码中ThreadHandler的onStart函数](https://github.com/pascal-lab/Tai-e/blob/d9784e3/src/main/java/pascal/taie/analysis/pta/plugin/ThreadHandler.java#L124-L129)是一个非常好的参数模拟并添加入口点的参考例子

![](_assets/1695290490000-7tgzyq.png-w331s.png)

参考这个我们来照猫画虎，首先我们需要拿到`com.example.demo6.HelloServlet`的JMethod对象，很简单，直接用tai-e的类型系统就行

```
JClass controller = World.get().getClassHierarchy().getClass("com.example.demo6.HelloServlet");
JMethod method = controller.getDeclaredMethod("doGet");
```

然后我们需要模拟参数对象，doGet函数有两个参数`HttpServletRequest request, HttpServletResponse response`，HttpServletRequest和HttpServletResponse都是一个接口类，我们模拟对象必须是模拟具体的类，所以这里用HttpServletRequest和HttpServletResponse的实现类HttpServletRequestWrapper和HttpServletResponseWrapper，代码如下

```
JClass requestWrapper = World.get().getClassHierarchy().getClass("javax.servlet.http.HttpServletRequestWrapper");
JClass responseWrapper = World.get().getClassHierarchy().getClass("javax.servlet.http.HttpServletResponseWrapper");
HeapModel heapModel = solver.getHeapModel();
Obj mockRequest = heapModel.getMockObj("EntryPointObj", "<http-request-wrapper>", requestWrapper.getType(), method);
Obj mockResponse = heapModel.getMockObj("EntryPointObj", "<http-response-wrapper>", responseWrapper.getType(), method);
Obj mockServlet = heapModel.getMockObj("EntryPointObj", "<http-controller>", servlet.getType());
SpecifiedParamProvider paramProvider = new SpecifiedParamProvider.Builder(method)
        .addThisObj(mockServlet)
        .addParamObj(0, mockRequest)
        .addParamObj(1, mockResponse)
        .build();
solver.addEntryPoint(new EntryPoint(method, paramProvider));
```

很简单，到这我们就把doGet加入到了entrypoint中，然后我们将`javax.servlet.ServletRequest#getParameter`加入到sink中

```
JMethod getParameter = requestWrapper.getDeclaredMethod("getParameter");
if (getParameter == null) {
    getParameter = requestWrapper.getSuperClass().getDeclaredMethod("getParameter");
}
sources.put(getParameter, getParameter.getReturnType());
sources.forEach((k, v) -> System.out.println(k.getMethodSource() + "\t" + v.getName()));
System.out.println();
```

运行一下试试，修改idea参数，加上javax.servlet-api-4.0.1.jar的lib包，并且要强制指定`--input-classes=com.example.demo6.HelloServlet,javax.servlet.http.HttpServletRequestWrapper,javax.servlet.http.HttpServletResponseWrapper`把两个warpper类引入进来。

```
-cp E:\tools\code\demo6\target\classes;C:\Users\xxx\.m2\repository\javax\servlet\javax.servlet-api\4.0.1\javax.servlet-api-4.0.1.jar -java 8 --input-classes=com.example.demo6.HelloServlet,javax.servlet.http.HttpServletRequestWrapper,javax.servlet.http.HttpServletResponseWrapper -m com.example.demo6.Main -ap -a pta=action:dump;action-file:result.txt;taint-config:src\test\resources\pta\taint\taint-config.yml
```

查看result.txt中，发现成功检测出来一条污点传播的路径来

```
Detected 1 taint flow(s):
TaintFlow{<com.example.demo6.HelloServlet: void doGet(javax.servlet.http.HttpServletRequest,javax.servlet.http.HttpServletResponse)>[1@L12] $r1 = invokeinterface request.<javax.servlet.http.HttpServletRequest: java.lang.String getParameter(java.lang.String)>(%stringconst0); -> <com.example.demo6.HelloServlet: void doGet(javax.servlet.http.HttpServletRequest,javax.servlet.http.HttpServletResponse)>[3@L13] invokevirtual $r2.<java.lang.Runtime: java.lang.Process exec(java.lang.String)>($r1);/0}
```

### 文末

对我而言，静态软件分析是一门高深的学问，涉及到的算法以及理论知识都比较晦涩，好在有tai-e这种开箱即用的框架。

回顾来看，tai-e在指针分析的能力和算法设计上毋庸置疑是很优秀的，而且整个框架的设计和插件式编程等设计思维远超soot这种单例模式一把梭的框架。但是其架构对我们做审计java web自动化并不友好：

1.  需要指定--input-classes参数强制引入对应的路由
2.  main函数限制
3.  指定entrypoint入口点和参数mock比较麻烦

### 参考

1.  南京大学谭添、李樾两位老师的课 [https://tai-e.pascal-lab.net/](https://tai-e.pascal-lab.net/)
2.  北大熊英飞老师的课 [https://liveclass.org.cn/cloudCourse/#/courseDetail/8mI06L2eRqk8GcsW](https://liveclass.org.cn/cloudCourse/#/courseDetail/8mI06L2eRqk8GcsW)
3.  tai-e Github开源地址 [https://github.com/pascal-lab/Tai-e](https://github.com/pascal-lab/Tai-e)
4.  两位老师的pascal课题组开源的一些指针分析的代码 [https://pascal-group.bitbucket.io/code.html](https://pascal-group.bitbucket.io/code.html)
5.  《soot存活指南》 [https://www.brics.dk/SootGuide/sootsurvivorsguide.pdf](https://www.brics.dk/SootGuide/sootsurvivorsguide.pdf)
6.  [fynch3r师傅的Soot知识点整理](https://fynch3r.github.io/soot%E7%9F%A5%E8%AF%86%E7%82%B9%E6%95%B4%E7%90%86/)
7.  [How to analyze java web or spring project with tai-e](https://github.com/pascal-lab/Tai-e/issues/19)

文笔垃圾，措辞轻浮，内容浅显，操作生疏。不足之处欢迎大师傅们指点和纠正，感激不尽。

* * *

![](_assets/0e69b04c-e31f-4884-8091-24ec334fbd7e.jpeg.jpg)
 本文由 Seebug Paper 发布，如需转载请注明来源。本文地址：[https://paper.seebug.org/3040/](https://paper.seebug.org/3040/)