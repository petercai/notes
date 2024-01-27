# Empower Eclipse’s Extension-points with Guice 
Eclipse Equinox is an implementation of the OSGi core specification that allows Eclipse applications to dynamically install, activate, de-activate, update and/or uninstall components and services (plugins) that extend/enhance the application’s capabilities. A _plugin_ is a modular, self-contained unit that provides an API that can be used within an Eclipse application or by other plugins. A plugin can provide enhancements to the Eclipse application UI (e.g. additional menus, buttons, dialogs, etc), can provide Java classes that allow the user to perform tasks (e.g. commit to VCS repositories, run Unit tests, deploy web applications to a server, etc.) or both. Additionally, plugins can provide extension points so users can extend the plugin by adding missing/additional functionality. For example, the [Mylyn](https://www.eclipse.org/mylyn/) plugin defines extension points that allow users to connect to task services not covered out of the box.

Simply put, an extension point defines a contract between a plugin and other plugins that wish to extend its functionality. This will usually include a set of parameters that must be configured and/or the qualified name of a class (or classes) that implements a specific interface (or interfaces — class extension is also supported). Extending plugins must provide all this information. At runtime, the Equinox api can be used to find which plugins are extending a specific extension point, retrieve the parameter values and instantiate the appropriate classes.

![](_assets/1!g3D_fKOQukGxrtyyzTLhew.png)

The figure above presents how a simple extension point might be defined. This extension points allows plugins to contribute their own implementations of the API defined in the the class org.eclipse.epsilon.IModuleTraceRepository. As the description indicates, this class allows other plugins to contribute implementations of the API that provide access to repositories in different persistence technologies. Imagine we would like to provide an implementation that can work with text files and another that can work with a database. In order to access each of them, a different set of information is needed; e.g. for the file a path/location is enough, for the database we might need url, login credentials, and target database. Hence, each implementation requires a separate configuration dialog to provide the necessary information. We can solve this by adding another attribute to our extension point through which the specific configuration dialog to use is provided too, as presented in the next figure.

![](_assets/1!otZGJ5B0wH2tg64yVN9S6w.png)

Extension point information is stored in an extension point schema (xml files with _.exsd_ extension) that are referenced from the plugin’s _plugin.xml_ metadata file. This means that although when creating the extension points content assist or class selection facilities help us pick existing classes, _extends/implements_ information is stored as plain text. The biggest consequence of this is that the information in the extension point schemes is disconnected form the java code. As a result, refactoring such as changing a class name or package will break the extension point. However, this errors wont manifest until the extending plugin is loaded into the Eclipse application.

Further, extension points have a lot of documentation that needs to be provided. Since the schema is not part of the main code base it is easier to let this information get stale (or just never provided at all). Moreover, adding/removing APIs becomes time consuming. Create the new element/attribute, define its properties, point to the correct class, add the description, update the extension point documentation… Moreover, the extension point becomes an intrinsic part of your API and making any changes to it must be considered when determining if a major, minor or patch bump is needed.

Ideally, we would like to define our extension points once and have them change as little as possible. This would require us to foresee possible extension points and prepare for them. Realistically, not until new features/functionally is added, and as the application architecture changes we are able to determine what extension points are needed. For example, in the plugin we introduced perviously, we might preemptively guard against other types of repositories … but what if we want to also allow extending plugins to contribute factories, task managers, etc?

**_A neat way, IMHO, is to solve this via dependency injection._**

[Guice](https://github.com/google/guice) is a dependency injection framework developed by Google.

> … injection is just a design pattern. The core principle is to _separate behaviour from dependency resolution_.

That is, classes are no longer responsible for looking up objects they need for their execution, these should be provided during construction. The Guice framework handles the “provision during construction” (a.k.a injection), all you need to do is **a)** tell Guice what dependencies it is responsible of injecting (via annotations) and **b)** what implementations to use during injection. The latter is done by extending the **AbstractModule** class.

If we delegate all dependency resolution to Guice, it means that we can simplify our extension point enormously. How? In the previous example we could change our extension point to accept classes that extend Guice’s AbstractModule. That is, extending plugins provide a Guice module that binds the required implementations.

I will assume that we modified our base code so that now the IModuleTraceRepository and friends are provided by injection. Then, we change the extension point to accept an AbstractModule instead:

![](_assets/1!GOQoM_10M8jGnkYZWxKnIQ.png)

The TraceRepository implementation is replaced by a Guice module that can provide all bindings.

Next, extending plugins can provide their own AbstractModule implementation:

```
**public** **class** MyTraceGuiceModule **extends** AbstractModule {@Override  
**protected** **void** configure() {  
    bind(IModuleTraceRepository.**class**).to(MyModuleTraceRepo.**class**);  
}
```

And provide it in their extensions configuration.

Now, if in the future we need other types of repositories, perhaps factories or other implementations, we don’t need to change the extension point definition and extending plugins just need to add more bindings:

```
**public** **class** MyTraceGuiceModule **extends** AbstractModule {@Override  
**protected** **void** configure() {  
    bind(IModuleTraceRepository.**class**).to(MyModuleTraceRepo.**class**);  
    bind(IModelTraceRepository.**class**).to(MyModelTraceRepo.**class**);  
    bind(IElementFactory.**class**).to(MyElementFactory.**class**);  
    ...  
}
```

This still does not solve the problem of documentation. If the required bindings are listed in the extension point definition, then this information can become stale after some time. A solution for this is to provide your own AbstractModule extension with Javadoc comments that list the required bindings. Further, this class can have a method that can check if all the required bindings have been provided and issue warnings/exceptions to inform the developers. Something in the lines of:

```
class YourModule extends AbstractModule {@Override protected void configure() { }

  void validateBindings(Injector injector) {

Set<Class<?>> reqInterfaces = **new** HashSet<>(  
      Arrays._asList_(  
        IModuleTraceRepository.**class,** IModelTraceRepository.**class,** IElementFactory.**class,  
        ...));** req.removeAll(injector.getAllBindings().keySet().stream()  
      .map(k -> k.getTypeLiteral().getRawType())  
      .collect(Collectors._toSet_()));  
    if (!req.isEmpty()) {  
      throw new IllegalStateException("Missing bindings: " + req);  
    }  
  }  
}
```

Guice injection can simplify your extension point management. Give it a try and let me know how it went on the comments below.

[

![](_assets/1!SGCT6C60o4t58wRqeU2viQ.png)

](https://www.buymeacoffee.com/KinoriTech)