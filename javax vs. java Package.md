# javax vs. java Package 
  

If you have a few years of experience in the Java ecosystem and you'd like to share that with the community, have a look at our [**Contribution Guidelines**](https://www.baeldung.com/expand-java-contribution-guidelines).

1\. Overview[](#overview)
-------------------------

**A package in Java is a scope that groups a set of related interfaces and classes**. Applications can contain hundreds or thousands of individual classes. Placing classes and interfaces into packages makes it easy to work it.

In this tutorial, we'll explore a range of examples of both _java_ and _javax_ packages. Additionally, we'll delve into the key differences between them and have a better understanding of their functions.

2\. _java_ Package[](#java-package)
-----------------------------------

The _java_ package holds the core classes or interfaces in the Java programming language. **Most classes required to write basic Java programs are in the _java_ package**.

**One of the most commonly used _java_ packages is _java.lang_**. **It contains the key classes that are essential to the design of the Java programming language itself**.

The most important class in this package is the [_Object_](https://www.baeldung.com/java-classes-objects) class, which is the foundation of the class hierarchy. Also, we have the [_Math_](https://www.baeldung.com/java-lang-math) class which provides mathematical functions like cosine, sine, square root, etc.


Additionally, another example of a _java_ package is  _java.net_. The package contains classes for developing networking applications. An example class in this package is _Inet4Address _which represents an Internet Protocol version 4 (IPv4) address.

Furthermore, **_java.awt _(Abstract Window Toolkit) package provides classes and interfaces for developing graphical user interfaces (GUIs) in desktop applications**. This package is the foundation for the _javax.swing_ package. _TextField _is an example class. It's a GUI component that provides an area for user input.

Finally, **another common example is _java.io_. It's a set of classes and interfaces that provide functionality for input/output (I/O) operations in Java**. Also, it helps to handle data from diverse sources, such as files, user input/output, etc. One example class from _java.io_ package is _FileInputStream. _The class is essential to read any type of file including image, text, and binary files.

3\. _javax_ Package[](#javax-package)
-------------------------------------

**The _javax_ package contains classes or interface that extend the functionality of _java_ packages**. **It's also known as an extension package**.

It provides additional functions to some classes already in the core _java_ package. There are several _javax_ packages, but we'll review a few examples.


One of the most common javax packages is _javax.swing_. It provides a set of classes for creating graphical user interfaces (GUIs) in Java. It's built on top of the [Abstract Window Toolkit (AWT)](https://www.baeldung.com/java-images#:~:text=AWT%20is%20a%20built%2Din,it%20is%20shipped%20with%20Java.). Some of the key classes in this package are _JPanel_, _JFrame_, _JComponent_, _JButton,_ and _JLabel_. It helps us create GUI components, customize their appearance and add event listeners.

Another example is _javax.tool_. This package contains interfaces and classes which can be invoked from a Java program. A common interface is _JavaCompiler,_ which helps to invoke Java compiler from programs. Also, we have _StandardJavaFileManager_, which is based on _java.io.File. _It helps to manage files.

Additionally, **_javax.net_ is another _javax_ package.** **This package provides classes for network communication using the Java Secure Socket Extension (JSSE) framework**. Some of its classes help secure socket communication.

Furthermore, **_[javax.servlet](https://www.baeldung.com/intro-to-servlets) _is another popular example. This package provides a set of classes and interfaces for developing web applications that run within a web server**. It contains classes that can handle requests and responses. An example class is _ServletInputStream_, which helps to read binary data from the client's request.

Finally, _**javax.crypto**_ **provides a set of classes and interfaces for cryptographic processes**. An example class is _[Cipher](https://www.baeldung.com/java-cipher-class). _The _Cipher _provides functionality for encryption and decryption. It's the core of the Java Cryptographic Extension (JCE) framework.


4\. Comparison Between _java_ and _javax_ Package[](#comparison-between-java-and-javax-package)
-----------------------------------------------------------------------------------------------

_javax_ and _java_ package both provide classes and interfaces to write effective Java programs.

**_java_ package contains the core Java Application Programming Interfaces (APIs)**. It helps to bootstrap any Java program and serves as the foundation for most Java APIs.

On the other hand, **the _javax_ package is an extension of the core _java_ package**. **It provides additional classes built on top of the _java_ package to add advanced features and functionalities**.

As Java evolves, new features that require modifications to the core _java_ package are introduced as an extension package. Java is a language that's backward compatible. Introducing new features as an extension makes old programs run.

5\. Conclusion[](#conclusion)
-----------------------------

In this article, we saw different examples of _java_ and _javax_ packages. We deep-dived into examples of the packages and looked at their uses case.

**We learned that the _javax_ package is an extension of the core _java_ package**. Both packages can be imported smoothly into any Java program.