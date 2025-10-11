# The Eclipse Feature Structure
Eclipse contains three large features which correspond to the three layers of Eclipse :
❍ Platform—The Eclipse language-neutral base
❍ JDT—The Java development tooling
❍ PDE—The plug-in development tooling
Eclipse is packaged this way so you can build applications using only the platform and ignoring all the Java tools.
The source is bundled as separate features. Again, this allows you to distribute Eclipse with or without source

The platform consists of several key components that are layered into a user interface (UI)-independent Core and a UI layer
![[Eclipse Basics-20231226-1.png]]
- Runtime—The runtime component defines the plug-in infrastructure. It discovers the available plug-ins on start-up and manages the plug-in loading.
- Workspace—A workspace manages one or more top-level projects. A project consists of file and folders that map onto the underlying file system.
- Standard Widget Toolkit (SWT)—The SWT provides graphics and defines a standard set of widgets.
- JFace—A set of smaller UI frameworks built on top of SWT supporting common UI tasks.
- Workbench—The workbench defines the Eclipse UI paradigm. It centers around editors, views, and perspectives.
![[Eclipse Basics-20231226-3.png]]


## Eclipse 4 IDE

![[Eclipse Basics-20231231-1.png]]

## E4 RCP
![[Eclipse Basics-20231231-2.png]]

# EditorPart vs. EditPart

The main differences between an EditorPart and an EditPart in Eclipse are: [[IB from chatgpt]](https://stackoverflow.com/questions/3259490)

- EditorPart represents an editor in the Eclipse workbench. It is responsible for editing a model object and handling save/load operations.
- EditPart on the other hand represents a part of the model being edited in the editor. It is responsible for visualizing and editing a particular part of the model.
- An EditorPart can contain multiple EditParts to represent different parts of the model being edited. So an EditorPart manages one or more EditParts.
- EditorParts are associated with file types using the IEditorInput interface. EditParts don't have direct file associations. [[2]](https://stackoverflow.com/questions/1929961)
- EditorParts handle dirty state and saving of the model. EditParts contribute to editing the model but don't directly handle saving.
- EditorParts are registered with the workbench and opened through the workbench API. EditParts are created and managed by the GraphicalViewer of the EditorPart.

So in summary, an EditorPart manages the overall editing of a model through associated EditParts which visualize and allow editing individual parts of the model. EditorParts provide the high-level editing functionality while EditParts focus on visual representation and interaction aspects.

# Bundle
  
In Eclipse, a bundle refers to a fundamental unit of modularity used in the Eclipse Equinox OSGi framework. <span style="background:#fff88f">Bundles are essentially JAR</span> (Java Archive) files that comply with the OSGi specifications, containing Java classes, resources, and metadata necessary for modular development.

Each bundle in Eclipse typically includes:

1. **Java Classes:** The actual Java code that provides specific functionality.
2. **Resources:** Such as images, XML files, configuration files, etc., needed for the bundle's operation.
3. **Manifest File:** A crucial file named `MANIFEST.MF` that contains metadata about the bundle, including its dependencies, exported packages, version information, and more.

Bundles in Eclipse are managed by the OSGi runtime, which facilitates their dynamic loading, activation, deactivation, and intercommunication at runtime. The OSGi framework allows bundles to interact with each other through well-defined APIs and services, promoting modularity, reusability, and maintainability in Eclipse-based applications.

Eclipse plugins themselves are essentially collections of bundles. These bundles work together to provide the functionalities that users interact with within the Eclipse Integrated Development Environment (IDE). Bundles enable Eclipse's extensibility by allowing developers to add, remove, update, or replace functionalities without affecting the entire system.