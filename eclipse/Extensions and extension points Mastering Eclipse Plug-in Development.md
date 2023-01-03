# Extensions and extension points | Mastering Eclipse Plug-in Development
The first thing to understand in the registry is the terminology. An **extension** is a contributed functionality that is often found in the `plugin.xml` file as an `<extension>` element. The extension itself provides some configuration or customization that can be processed appropriately. An extension is like a USB device, such as a mouse or keyboard. For example, the new feed wizard was added as an extension in the previous chapter:

An **extension point** defines the contract of an extension, along with any required arguments or attributes that an extension must provide. An extension point is like a USB hub that allows extensions (USB devices) to be plugged in. For example, the `newWizards` extension point is defined in the `plugin.xml` file of the `org.eclipse.ui` plug-in as follows:

This refers to an XML schema document that defines the extension content, as shown in the following snippet:

The schema defines the point for the `org.eclipse.ui.newWizards` extension (the ID is the concatenation of the values of `meta.schema plugin` and `id`). It declares that the extension has a category, which has required `id` and `name` attributes.

The schema also allows the PDE to verify whether elements are missing when editing a `plugin.xml` file, or provide code completion to insert required or optional elements.

Fortunately, PDE comes with good support to build this schema via a graphical user interface, so the XML can remain hidden.

### Creating an extension point

To demonstrate the process of creating a new extension point in Eclipse, a feed parser will be created. This takes a `Feed` instance (which contains a URL) and returns an array of `FeedItem` instances. Extensions can be contributed to provide different feed parsers; this allows a `MockFeedParser` instance to be initially created that can then be substituted for other implementations in future.

Executable extension points tend to have a `class` attribute, whose class typically implements a particular interface. An `IFeedParser` interface will be created to represent the abstract API of all feed parsers; extensions that provide a feed parser will be expected to implement this interface.

#### Creating an IFeedParser interface

Since the feed parser could be used outside of a UI, it makes sense to create a new plug-in project called `com.packtpub.e4.advanced.feeds` and to refactor the `Feed` instance from the UI package (created in [Chapter 1](https://subscription.packtpub.com/book/application-development/9781783287796/1), _Plugging in to JFace and the Common Navigator Framework_) into this package as well.

### Tip

When you refactor the `Feed` class, ensure that **Update fully qualified names in non-Java text files** option is selected, or remember to refactor the name in the fully qualified names in the `plugin.xml` file, since the class name is used in several places in enablement tests.

Note that you will also need to ensure that the `com.packtpub.e4.advanced.feeds` package is exported from the plug-in (from the **Runtime** tab of the manifest editor) and the package is imported by the `com.packtpub.e4.advanced.feeds.ui` plug-in (from the **Dependencies** tab of the manifest editor).

One this is done, an interface `IFeedParser` will be created to parse a feed, as shown in the following code snippet:

The intent is that this will return a list of items parsed from the feed. To do this, a `FeedItem` class will be needed as well. Each `FeedItem` instance will have an associated parent `Feed`, along with some other metadata.

### Note

It would be possible to create a mutable `FeedItem` instance with getter/setter pairs for each attribute. However, this leads to the possibility that a feed might be inadvertently mutated after it has been constructed.

A second approach is to use the constructor to add all arguments. Unfortunately, this prevents evolution of the class; as new parameters are added, more constructors need to be created with the values in place.

A better solution is to use the **builder pattern**, which allows a separate object to assemble the instance. This way, the object can be created but not mutated after it is returned. Visit [http://en.wikipedia.org/wiki/Builder_pattern](http://en.wikipedia.org/wiki/Builder_pattern) for more information.

To instantiate a `FeedItem` class, an inner `Builder` class will be used. This has access to the private fields of the `FeedItem` class, but permits the object to be returned without a means of mutating it afterwards:

The preceding example shows how the builder pattern is used, in this case, for two fields: a parent `feed` and `date`. To extend the `FeedItem` class, add accessors in the builder to set other elements such as the following:

A `FeedItem` class can now be instantiated using the following code:

### Note

Note that the builder pattern is fairly common in Java, as is the literate programming style used; by returning instances of `Builder` at the end of each setter method, this allows chaining of method calls into a single expression. The `build` method can also perform any necessary validation to verify that all mandatory fields have been assigned and any optional fields are assigned default values if necessary.

#### Creating a MockFeedParser class

To provide some feed data that can be used by a parser without having to make a network connection, a `MockFeedParser` class can be created. This will take a `Feed` instance and return a set of hardcoded `FeedItems`, allowing further testing to be done.

Because this class isn't intended to be directly visible to downstream users, put the class in a different package, `com.packtpub.e4.advanced.feeds.internal`. This way, the package will be hidden by the OSGi runtime and so dependent classes won't be able to see or instantiate it. The following code illustrates the creation of the `MockFeedParser` class:

### Tip

By default, PDE and the Maven `maven-bundle-plugin` hide packages with `internal` in their name. This allows the public API to be separated from the internal implementation details to downstream clients.

The mock can be populated with more data, such as an HTML body or different dates, if these are desired.

#### Creating the extension point schema

The extension point for the feed will be called `feedParser`, and it will use the `IFeedParser` interface.

To create an extension point, open up the plug-in's manifest by double-clicking on the `plugin.xml` or `MANIFEST.MF` files, or by navigating to **Plug-in tools** | **Open Manifest** from the project. Switch to the **Extension Points** tab, click on **Add**, and enter `feedParser` for both the ID and the name in the dialog that shows up. This is shown in the following screenshot:

![](https://static.packt-cdn.com/products/9781783287796/graphics/7796OS_02_01.jpg)

After clicking on **Finish**, the schema editor will be shown:

![](https://static.packt-cdn.com/products/9781783287796/graphics/7796OS_02_02.jpg)

The **Description**, **Since**, **Examples**, **API Information**, **Supplied Implementation**, and **Copyright** are all text-based fields that are used to generate the documentation and can be left blank. However, this documentation will be shown to users in the future and is used to generate the information as seen in the Eclipse help center and at [http://help.eclipse.org](http://help.eclipse.org/).

Switching to the **Definition** tab allows the contents of the extension point to be modified. Select the **extension** element, click on the **New Element** button to its right, and give it the name `feedParser`. This will be the name of the XML element that is expected by clients. To give it an attribute value, ensure **feedParser** is selected, click on **New Attribute**, and give it the name `class`. Its type will be **java** and it will be **required**; use the **Browse...** button next to the **Implements** textbox to select the `IFeedParser` created earlier.

The resulting schema definition will now look something like what is shown in the following screenshot:

![](https://static.packt-cdn.com/products/9781783287796/graphics/7796OS_02_03.jpg)

Under the covers, the extension is represented in two different files. The first is the `plugin.xml` file, which includes the following:

The schema reference points to the schema definition, which was created by the UI previously.

### Tip

Note that the `build.properties` file should be updated to include the `schema` directory in the binary output; otherwise, implementors of the plug-in won't be able to verify whether the `feedParser` element is correctly provided or not.

The schema is not necessary for the extension mechanism to work; it is mainly used by PDE when allowing the element to be created in the `plugin.xml` file. However, it has value in communicating documentation to other users of the extension point in both its intent and its requirements, and so providing the schema is considered best practice.

There are other values that can be defined on the extension point. For example, each element has one of the following values:

*   **Name**: This is the name that will be used for the element or attribute. This must be a valid XML name for elements and attributes.
    
*   **Deprecated**: This is `false` by default, but can be changed to `true`. This is used to indicate to clients that an extension point should no longer be used; it is often combined with the description to suggest an alternative or replacement function.
    
*   **Translatable**: If an attribute has a value that can be translated (such as a label or another human-readable string), then this value should be `true`. If so, the value in the `plugin.xml` may be a percent string such as `%description`, and the value will be pulled out automatically from the localized `plugin.properties` file by Eclipse when the extension point is loaded.
    
*   **Description**: This is a human-readable description that can be shown by PDE or converted into a help document that indicates how the point should be used.
    
*   **Use**: The value for this can be `optional`, `required`, or `default`. If an attribute is marked as `optional`, then it does not need to be present. If an attribute is marked as `required`, then it must be present. If it is marked as `default`, then a default value box is shown that allows the default value to be defined, which is used when the attribute is missing.
    
*   **Type**: This is the attribute type. The attribute value can be of one of the following types:
    
    *   **Boolean**: This specifies that the attribute can have the value `true` or `false`.
        
    *   **String**: This specifies that the attribute value can be a string. Strings may be translatable, and they can have **restrictions** that are preset values that the string can take (such as `North`, `South`, `East`, and `West` or `UP` and `DOWN`).
        
    *   **Java**: This specifies that the attribute must be a type that either extends the specified class or implements the specified interface.
        
    *   **Resource**: This specifies that the attribute can have a resource type.
        
    *   **Identifier**: This specifies that the attribute might reference another ID in another schema document using an XPath-like expression of the form `org.eclipse.jdt.ui.javaDocWizard/@point`, where `org.eclipse.jdt.ui` is the plug-in namespace, `javaDocWizard` is an extension, and `@point` is the attribute `point` within that element.
        
    
    ### Note
    
    There is a **DTD approximation** that is used to show an approximate Document Type Definition of the children. If an element has no children, then it will show `EMPTY`; for a text element, it will show `(#PCDATA)`.
    
    **PCDATA**, which stands for Parsed Character Data, is used in HTML and originally came from SGML.
    

In addition, elements can be repeated. The schema editor permits a **sequence** of elements (in other words, a list) or a **choice** of items (one of a set). These are known as **compositors** and can be switched between using the **Type** drop-down. Compositors have a minimum value and a maximum value; if the minimum is zero, then it is effectively optional. The maximum value, if specified, allows a fixed number to be specified (for example `months=12`); but if the **unbounded** option is checked, then the compositor can have any number of child elements.

Typically, an extension point will permit more than one element to be added. To enable this, it is necessary to add a `Sequence` element underneath the `extension` element. The sequence is necessary to permit more than one element to be provided.

In the PDE schema editor, click on the **extension** element and choose **New Sequence**. The minimum value should be `1` and the sequence should be **unbounded**; these are the typical defaults, as shown in the following screenshot:

![](https://static.packt-cdn.com/products/9781783287796/graphics/7796OS_02_04.jpg)

To add the `feedParser` point to the extension, drag-and-drop the **feedParser** element underneath the **Sequence** element, as shown in the following screenshot:

![](https://static.packt-cdn.com/products/9781783287796/graphics/7796OS_02_05.jpg)

The `feedParser.esxd` schema should be similar to the following:

Now the schema definition is complete.

#### Using the extension point

As with other extensions, they are added to a plug-in's `plugin.xml` file. It's not uncommon for a plug-in to define both the extension point and an extension in the same file.

### Tip

Note that defining an extension point in the same file as an extension means that there is no way of removing that extension from the platform if that extension point is used elsewhere. Providing a plug-in that defines the extension point and then separate plug-ins for the extensions allows the extensions to be individually removed from the platform.

To add `MockFeedParser` to the `plugin.xml` file, add the following:

In order to provide an easy way for clients to obtain a list of feed parsers, a class `FeedParserFactory` will be created in the `feeds` plug-in. This class will be used to provide a list of `IFeedParser` instances without having a specific API dependency on the extension registry itself. The code is as follows:

The registry is managed with an `IExtensionRegistry` interface, which can be accessed via the `org.eclipse.equinox.registry` bundle. It is possible to dynamically register extension elements, but the most common practice is to read the extensions that exist in the runtime. To do this, add the `org.eclipse.equinox.registry` bundle to the list of imported bundles in the manifest as follows:

The extension registry manages a set of extension points, which are identified with an ID—typically consisting of the contributing bundle ID and the specific ID of the extension point. From the preceding definition, the values will be `com.packtpub.e4.advanced.feeds` and `feedParser`, respectively. Add the following to the `FeedParserFactory` class:

### Note

`IExtensionPoint` represents the extension point definition itself. This might return `null` if there is no such extension point, so it should be checked before use. In previous versions of Eclipse, the identifiers used to be stored as a single string, such as `com.packtpub.e4.advanced.feeds.feedParser`. However, this resulted in many thousands of strings that took up a lot of space in the PermGen area of the JVM. By splitting them into two separate strings, many extensions in the same plug-in share the same namespace, which results in just a single entry in the PermGen area. Note that PermGen has been removed in the latest JVM versions.

If the preceding code returns a non-null value, it can be interrogated further. The most common call is `getConfigurationElements`, which allows the extensions to be parsed. This gives a tree-like view of the content of the extension, mapping closely to the structure of the entries in the `plugin.xml` file:

If the extension point contained only textual information (for example, Mylyn's use of the registry to store URLs such as [http://bugs.eclipse.org](http://bugs.eclipse.org/) to report Eclipse bugs), then the element could be interrogated to return the actual textual value. In the `feedParser` example, the attribute containing the class name is the one of interest.

In this case, the extension point defines a class to be instantiated. To do this, there is a method called `createExecutableExtension` that takes an attribute name and then instantiates a class using that name from the appropriate bundle. In effect, this is similar to `class.forName(extension.getAttribute("class"))`, but uses the correct `ClassLoader`.

### Note

Although it might be tempting to think that using `Class.forName()` would work on the returned class name, this doesn't work in the case where the class comes from outside the current plug-in. Since each bundle has its own `ClassLoader` and the plug-in that uses the extension is almost always not the bundle that provides the extension, it would not work in most cases.

Since it's possible that the extension has a semantic error, or that the plug-in might not be loaded successfully, the instantiation of the class should `try` and `catch CoreException`. If an error occurs, then the extension won't be useful; the runtime might choose to log the error (for further diagnostics) if appropriate. Don't forget to check whether the returned instance is of the correct type using `instanceof`; this will also look for `null`.

The object is instantiated with a zero-argument constructor and returned to the caller, shown as follows:

Should the return values of the extension be cached? It depends on what the use case is likely to be. If they are only going to be used transiently, then there might be no point in creating them each time. On the other hand, if they are cached, then additional code will need to be created to register listeners to note when the plug-ins are uninstalled.

The extension registry does a reasonable job of caching and returning values from the calls, so these can be assumed to be fast. However, for the instantiated objects, if identity is important, then it might be necessary to arrange some kind of cache of the executable extensions.

If the return result is cached, then any new plug-ins that are subsequently installed might not be seen.

The extension registry also has listener support; calling `addListener` on the registry will provide a way of picking up changes to a specific extension point. This can be used to update any caches when changes occur.

#### Integrating the extension with the content and label providers

Having defined an extension point with a `FeedItem` provider, the next step is to integrate it with the `FeedLabelProvider` and the `FeedContentProvider` classes created in the previous chapter. Integrating it into the `FeedLabelProvider` class is really simple because this is just an addition of a couple of lines:

When a `FeedItem` element is seen in the tree, its title will be used as the label.

Integrating with the `FeedContentProvider` class is the next step. To start off, declare that all `Feed` elements have children and the parent of the `FeedItem` element is `Feed` itself:

To parse the feed URL, perform the following steps:

1.  Acquire the `IFeedParser` list (which comes from the extension registry).
    
2.  Iterate through the list and attempt to acquire the `FeedItem` list.
    
3.  If the value is non-null, return.
    

The code will be similar to the following:

This pattern allows the `FeedParserFactory` class to return items in the order of preference such that the first parser that handles the feed can return a value.

Run the Eclipse application, create a feed (the URL and title won't matter at this point), and then drill down into the feeds and then into a single feed. The test data that was used in the `MockFeedParser` class should be shown in the list, as shown in the following screenshot:

![](https://static.packt-cdn.com/products/9781783287796/graphics/7796OS_02_10.jpg)

#### Showing a feed in the browser

Now that feeds are showing as individual elements in the content provider, the `ShowFeedInBrowserHandler` class can be copied and modified to allow individual `FeedItem` entries to be added. Refer to the _Adding commands to the common navigator_ section in [Chapter 1](https://subscription.packtpub.com/book/application-development/9781783287796/1), _Plugging in to JFace and the Common Navigator Framework_.

Copy the `ShowFeedInBrowserHandler` class to `ShowFeedItemInBrowserHandler`. The only change that's needed is the change from `Feed` to `FeedItem` in the `instanceof` test and subsequent cast:

It will also be necessary to duplicate both the handler and command entries in the `plugin.xml` file, along with the change to the feed item. This is shown in the following:

Enabling the menu item is necessary to show the command associated with any selected `FeedItem` instances as follows:

If the application is run, then a `MalformedURLException` will be generated, since the `MockFeedParser` doesn't have any URLs set on it. Modify it as follows:

Now running the application will show the URLs when selected, as shown in the following screenshot:

![](https://static.packt-cdn.com/products/9781783287796/graphics/7796OS_02_09.jpg)

#### Implementing a real feed parser

The `MockFeedParser` can be replaced with an implementation to parse RSS feeds. This requires parsing some simple XML to understand the feed reference.

An **RSS** feed looks like:

Unfortunately, since this is XML, it will require parsing. There are several ways of doing this, but using a `DocumentBuilder` instance will allow the elements to be iterated through and pull out the `title` and `link` elements, along with `pubDate`.

Create an `RSSFeedParser` class (with a couple of helper methods) to parse a date from the **RFC822** format and a mechanism to parse a text value from an `Element`, as shown in the following code snippet:

These helper methods can be used to parse the elements out of the feed by parsing the URL of the `Feed` with a `DocumentBuilder` instance and then using `getElementsByName` to find the `item` elements. The `parseFeed` method will be similar to the following code:

This looks for elements called `item`, finds child text elements with `title` and `link`, and sets them into the feed.

To add this to the Eclipse instance, add `RSSFeedParser` to the `plugin.xml` filein the `feedParser` extension point:

If the `MockFeedParser` instance is still present, the ordering in the `plugin.xml` file will be important. By default, the order returned in the array is the same as they are in the file. `FeedParserFactory` returns the first parser that successfully parses the feed; therefore, if the `MockFeedParser` is first, then the real content of the feed will not be returned.

### Tip

The feed parser is non-optimal; the feed will be parsed and potentially reacquired multiple times. It would be desirable to have the contents of the source document cached, but this is an optimization left for you.

Run the Eclipse application and use the add feed wizard to add a feed for the Packt Publishing RSS feed, [http://www.packtpub.com/rss.xml](http://www.packtpub.com/rss.xml).

Not all feeds use RSS as a feed type, partly because RSS was incompletely specified and had several slightly different incompatible feed formats. **Atom** was designed to resolve the issues with RSS but, in practice, there are approximately an equal number of feeds specified in RSS and Atom.

An Atom feed looks like:

Parsing it will not be significantly different from before; the structure can be used to pull out the `entry` and the contained `title`, `link`, and `updated` references. However, there are a couple of points that are worth noting in this example:

*   The Java Date APIs do not understand colons in time zones, so `+01:00` must be converted to `+0100` to be parsed
    
*   The reference for the link is stored inside an `href` attribute instead of as a text node
    

An `AtomFeedParser` class can be created as follows:

The `parseDate` method is similar to the following code:

To parse an attribute value out from XML, an additional helper method can be created as follows:

Adding the class into the extension points will allow the element to be parsed if the feed is not an RSS feed as follows:

Now when the feed is downloaded, it will attempt to parse it as RSS first and then fall back to Atom afterwards.

#### Making the parser namespace aware

The Atom specification uses **XML namespaces**. To parse the feed properly, the document builder must be specified as **namespace aware** and the elements lookup needs to use the equivalent `getElementsByTagNameNS`.

In the `AtomFeedParser` class, define a `static` constant to hold the Atom namespace:

Then, replace the calls to `getElementsByTagName` with the namespace aware equivalent `getElementsByTagNameNS`, as shown in the following code:

Finally, to ensure that the document builder is using namespace aware parsing, the `DocumentBuilderFactory` instance needs to be appropriately configured before `DocumentBuilder` is instantiated. The code is as follows:

When the Atom feed is parsed, it will be correctly represented if there are multiple namespaces or a default namespace is not specified.

The order of the `IConfigurationElement` instances returned by the registry should not be relied upon for consistency. Relying on a specific order will prevent others from easily being able to contribute additional implementations from other plug-ins. If an ordering is desired, additional metadata should be added to the extension to allow the ordering to be calculated after retrieval.

In this case, `MockFeedParser` should have the lowest priority and `RSSFeedParser` should have a higher priority than `AtomFeedParser`. To implement this, an extra attribute `priority` will be added to the extension point so that the results can be processed after loading.

Although any ordering can be used (for example, `high`/`medium`/`low`, or `top`/`bottom`), it is easier to deal with numerical priorities and perform an integer sort. Using both positive and negative numbers allows a full range of priorities and also allows some extensions to register themselves as less desirable than the default, which can remain at zero.

Modify the extension point schema `feedParser.esxd` and add a new attribute under the **feedParser** element called `priority`. Since the XML schema for extension points does not permit numeric values, use **string** as the type and the value can be parsed afterwards. This is shown in the following screenshot:

![](https://static.packt-cdn.com/products/9781783287796/graphics/7796OS_02_06.jpg)

Now, in the `plugin.xml` file where the feeds are defined, add a priority of `-1` to `MockFeedParser` and a priority of `1` to `RSSFeedParser`:

Since the `IFeedParser` interface doesn't have a priority attribute that can be set, the `IConfigurationElement` instances must be sorted after retrieval, but before iteration. To do this, the `Arrays` class can be used by calling `sort` with an appropriate `Comparator`.

Create a class called `FeedParserConfigurationComparator` in the `internal` package, and then make it implement `Comparator` with a target type of `IConfigurationElement`. Write a helper method to parse an integer from a string, treating a missing value (`null`) or integers that cannot be parsed as a zero value.

The class will be similar to the following code:

This will sort the extensions based on the numerical value of the `priority` attribute. To invoke it, call a `sort` method after the configuration elements are accessed in `FeedParserFactory`:

Now when the extensions are processed, they will be done in priority order.

### Note

**Using a singleton for comparators?**

It's possible to use the singleton pattern for the comparator, since it uses no instance variables. By making the constructor private and instantiating a `public static final` constant, it can be referred to with `FeedParserConfigurationComparator.INSTANCE`. This is useful if the comparator is being used in a lot of places, since the same instance will be reused. However, if it is used infrequently, creating and disposing the comparator will be fairly fast and will not permanently stay in memory when not in use. The tradeoff between memory and CPU utilization will depend on the expected use cases.

#### Executable extensions and data

When an extension point is created, it is called with a zero-argument constructor. The result is that most extensions in Eclipse are pre-configured with whatever data they need with no further customization.

It is possible to pass through additional configuration data in the `plugin.xml` file and have that parsed at start-up. To do this, the `IExecutableExtension` interface provides a `setInitializationData` method that passes in information defined statically within the `plugin.xml` file.

By adding the `IExecutableExtension` interface to the concrete feed parser instance, it's possible to have additional data from the `plugin.xml` file passed into the class itself. This may allow the same implementation to perform in different ways based on values contained within the `plugin.xml` file; for example, a limit could be placed on the parser to indicate how many feed entries would be shown in the list.

Add the `IExecutableExtension` interface to the `AtomFeedParser` class, and then add following code:

The `max` field can be used to limit the number of elements returned from the feed:

The data can be passed in a couple of different ways. The easiest way (if it's a single value or can be represented as a string) is to pass it after the class name of the attribute with a colon. Modify the `plugin.xml` as shown:

Note the `:2` at the end of the class name. Upon initialization of the feed parser, the value will be passed in as the `data` object.

### Note

The value of `propertyName` in this case will be `class`, which is taken from the `createExecutableExtension` method call previously and refers to the XML attribute `class` in this entry.

Now when the application is run, Atom feeds such as [http://alblue.bandlem.com/atom.xml](http://alblue.bandlem.com/atom.xml) will only return a maximum of two values. The same change can be applied to `RSSFeedParser`.

### Note

There are several ways of pulling configuration information out from an extension. The simplest way is to parse the string appropriately after the class name, and this suffices for most cases.

More complex cases can simply parse the `IConfigurationElement` interface (shown in the next example).

There is also an ancient plug-in configuration process that can be invoked if `null` is passed as an attribute name and where elements and parameters use a specific hardcoded pattern. Parameters are parsed from name/value pairs and passed into the data as a `Hashtable`:

This is for compatibility with older versions of Eclipse and should not be used.

#### Executable extension factories

Sometimes, it's not possible to add an interface to the class that requires instantiation. This is either because it doesn't make sense for the component itself to implement `IExecutableExtension`, or because it's a closed source component, such as a database driver that cannot be modified.

### Tip

Note that the `DBFactory` example is not used or needed by the `Feed` example; it's used to demonstrate the need to introduce factories where the user has no ability to change the class being instantiated. It is provided here just as an example.

This issue can be resolved by using an **executable extension factory**. This permits a factory to be used to instantiate the extension. For example, if a database connection was defined in an extension point, it might be similar to the following:

The desired result of this would be to provide an instantiated JDBC `Connection` object of the right type. Clearly, the driver itself cannot be modified to implement the `IExecutableExtension` interface; but it is possible to provide it as an extension with a factory:

The preceding example shows another way of getting the configuration data—by direct parsing of the `IConfigurationElement` class. Of course, placing user IDs and passwords hardcoded into an extension point is not good practice; the example here is used because the `Connection` interface will be familiar to many readers.

Note that since the `DriverManager` class does a lookup based on the URL to acquire the database connection, it will be necessary to have those driver classes available on the bundle's classpath—either as a direct import or as a bundle dependency.

### Tip

It's fairly common for an `IExecutableExtensionFactory` instance to also inherit `IExecutableExtension`, as that is the only way of receiving data if it is required. If the factory does not need such data (for example, it is instantiating an in-memory structure, or using external file configuration), it might be possible to have an `IExecutableExtensionFactory` interface that does not implement `IExecutableExtension`.

### Using the extension registry outside of OSGi

Although the extension registry can work outside of OSGi, it's not easy to do so. This is partly is due to the set of libraries that are required and partly because there is additional setup required in order to provide the registry with the right state.

There are a number of dependencies that need to be provided for the extension registry to work, including the **Equinox Supplemental** bundle. The supplemental bundle provides commonly used features such as `NLS` and `Debug`, which are liberally spread around the Equinox codebase and are required for running outside of Equinox. In addition (in Kepler and below), the `Debug` class transitively depends on the OSGi `ServiceTracker`, which means that even when running outside of OSGi, this package is required.

The minimum setup for a Java application (running outside of OSGi) to use the extension registry is as follows:

*   `org.eclipse.equinox.registry`
    
*   `org.eclipse.equinox.common` (provides `CoreException`)
    
*   `org.eclipse.equinox.supplement` (provides `NLS` and `Debug`)
    
*   `org.osgi.core` (provides `ServiceTracker`)
    

Note that if running in a non-Equinox OSGi container, `org.osgi.core` will not be needed. If running in Equinox, then the supplemental bundle is not needed, as this provides a copy of the public classes and interfaces used.

In Eclipse Luna and above (`org.eclipse.equinox.supplement` version 3.5.100 or higher), the dependency on `org.osgi.core` has been removed so this particular dependency is no longer necessary.

To configure the registry to run outside of OSGi, it is necessary to set up a registry instance. There is an `IRegistryProvider` interface that can be used to define what registry instance should be returned when `RegistryFactory.getRegistry` is called. By default, this will return `null` until a registry has been set; in an OSGi runtime, this is done by the registry bundle itself starting.

To create a `Registry` instance directly, there is a `RegistryFactory.create` method that takes a `RegistryStrategy` instance along with a couple of tokens. The tokens can be used in secure environments to prevent callers from modifying or adding to the registry. All three elements can be left as `null`.

The net effect is that adding this to the start-up of a Java application, which needs the registry, will work:

Once the registry has been set, it can be acquired from the `RegistryFactory` interface:

However, unlike OSGi (where the registry automatically scans for bundles as they are inserted and registers the extension elements), in a standalone Java application this needs to be done manually.

To load a single `plugin.xm`l file into the registry, it is necessary to first create a contributor (which in OSGi is the bundle, but in a Java application can be a different mechanism) and then add a contribution. The contribution itself is an `InputStream` that represents the `plugin.xml` file, called `feeds.xml` here:

This loads the file `feeds.xml` from the classpath and uses that to register the feeds plug-in mentioned previously. This XML file may be built in-memory or loaded from other input sources, but has the same effect and content as the `plugin.xml` of the original bundle.

### Tip

Unfortunately, it isn't possible to use `plugin.xml` as the filename, as the `org.eclipse.equinox.registry` bundle also has a file called `plugin.xml`, and the standard Java `getResourceAsStream` will load the first one that it sees. As a result, depending on whether your class or the registry JAR is first on the path, you might see a different result. See the _Loading all extensions from the classpath_ section to find out how to handle this.

The extension can then be acquired as usual or via the `FeedParserFactory` method defined previously.

#### Using the extension registry cache

In typical Eclipse usage, the extension registry remains the same between Eclipse restarts, and adding (or removing) plug-ins updates the registry appropriately. To save time at start-up, the **extension registry cache** is used to store its contents at shutdown, and if available, loads it at start-up.

Contributions can be **persistent** or **non-persistent**. A persistent registration is kept such that a restart of the application will have the same value; a non-persistent registration is for this JVM only and will be lost on restart. Eclipse uses this mechanism at start-up to ensure a faster start-up time and to avoid having to reparse the `plugin.xml` files each time.

To take advantage of the cache, a `RegistryStrategy` instance must be provided with one or more cache directories to store the content. If at least one directory is writable, the registry can save new extensions; if the list of directories is empty or all of the directories are read-only, then the registry will not persist content between restarts.

Modify the `IRegistryProvider` interface in the `NonOsgi` class to return a directory, and then pass that into `RegistryStrategy` along with an array of `false` values:

The second parameter to `RegistryStrategy` is an array of `boolean` values (one per entry in the `File` array) that indicates whether the cache is read-only or not. If this parameter is `null`, then the cache directories are all considered read-only.

Finally, note that if using the registry in caching mode, it is necessary to stop the registry after use. This causes the data entries to be persisted to disk:

To enable the contribution to be added persistently, the `boolean persist` parameter should be specified as `true`:

Now, if the contribution is added and the line is commented out, then re-running the application will load the entry from the cache.

If the cache needs to be rebuilt, then the `clearRegistryCache` method of `ExtensionRegistry` will need to be called at start-up. This is equivalent to passing the `-clean` parameter to Eclipse at start-up. Since this is not an interface method, the call will need to be cast to the explicit `ExtensionRegistry` class. Alternatively, the cache directory can be deleted prior to starting the registry, and it will be rebuilt automatically.

#### Loading all extensions from the classpath

The `getResourceAsStream` method returns the first element found in the classpath. For applications that can span many JAR files, it's desirable to be able to find all the files with that name, not just the first one. Fortunately, `ClassLoader` has a means to scan all of the individual elements with the `getResources` method, which returns an enumeration of URLs from which streams can be individually obtained.

The solution is to call `getResources` with an argument of `plugin.xml` and then iterate through each of those JARs to instantiate the required contributions. By default, the contribution name is the name of the bundle, so it is necessary to parse out the `Bundle-SymbolicName` header from the manifest. There are standard OSGi classes to do this, but the `Manifest` class from the standard JDK libraries can be used.

### Tip

The `Bundle-SymbolicName` header can have additional metadata after the bundle name, separated by a semicolon (which delimits OSGi directives). The most common one is `;singleton=true`, but others can also exist such as `;mandatory` and `;fragment-attachment`.

Instead of adding a single `feeds.xml` file into the registry, all JARs can have their bundles scanned by changing the following code in `NonOsgi`:

After running this code, all JARs on the classpath will have extensions from their `plugin.xml` files registered.

Unfortunately, there's a minor problem with any projects that are open in the Eclipse workbench. An open project in PDE doesn't put the `plugin.xml` or `MANIFEST.MF` files available on the project's classpath, which means that the classpath loading mechanism doesn't work. This is generally a problem with PDE and means that (for example) `bundle.getEntry` and `class.getResourceAsStream` in a PDE project differ from how they will run when exported and installed into a runtime as a JAR.

Fortunately, there is a way of fixing this: adding the `plugin.xml` and `META-INF/**` files as explicit sources in JDT, which copies them to the output directory. As a result, the `getResource` methods work in both runtime and outside of runtime.

Right-click on the project and go to the **Java Build Path**. In the **Source** tab, add a folder by clicking on **Add Folder** and selecting the root of the project. Click on **Edit** and then add `plugin.xml` and `META-INF/**` as entries to be included. This is shown in the following screenshots:

![](https://static.packt-cdn.com/products/9781783287796/graphics/7796OS_02_07.jpg)

![](https://static.packt-cdn.com/products/9781783287796/graphics/7796OS_02_08.jpg)

Now when the application is run, `plugin.xml` from the `org.eclipse.equinox.registry` JAR and `plugin.xml` from the `com.packtpub.e4.advanced.feeds` JAR will both be registered and the default feed parsers will be able to return all of the values.