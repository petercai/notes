# What is an extension point schema? - Eclipsepedia
Save From : [FAQ What is an extension point schema? - Eclipsepedia](https://wiki.eclipse.org/FAQ_What_is_an_extension_point_schema%3F) 

## Content
What is an extension point schema?
==================================

Each extension point has a _schema_ file that declares the elements and attributes that extensions to that point must declare. The schema is used during plug-in development to detect invalid extensions in the plugin.xml files in your workspace and is used by the schema-based extension wizard in the plug-in Manifest Editor to help guide you through the steps to creating an extension. Perhaps most important, the schema is used to store and generate documentation for your extension point. The schema is _not_ used to perform any runtime validation checks on plug-ins that connect to that extension point. In fact, extension point schema files don&#146;t even need to exist in a deployed plug-in.

The exact format of the schema file is an implementation detail that you probably don&#146;t want to become familiar with. Instead, you should use the graphical schema editor provided by the Plug-in Development Environment.
## Note