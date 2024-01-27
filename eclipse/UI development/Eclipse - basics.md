from "Contributing to Eclipse principles, patterns & plug-ins"

# core runtime - IAdaptable
## Extension Object/Extension Interface

We need a mechanism that allows us to

- Add a service interface to a type without exposing it in that type
- Add behavior to preexisting types such as IFile
The pattern is called **Extension Object** and is also known as **Extension Interface**. To quote its intent: “Anticipate that an object’s interface needs to be extended in the future. Extension Object lets you add interfaces to a class and lets clients query whether an object has a particular extension.”

When implementing **Extension Object**, you have to answer several questions.
- Do you want to extend an individual object or a class? With a classbased mechanism you add behavior to an entire class. You can only add behavior but not state, that is, no fields.
- How is an extension described and identified?
The Eclipse extension support is **class-based**. That is, you can add behavior to existing classes, but not add state to its existing instances. The additional behavior is described by an interface. The key interface of the Eclipse extension support is **IAdaptable**. Classes that support adaptability implement this interface.

**IAdaptable** emphasizes the fact that the mechanism enables adapting an existing class to another interface.
**IAdaptable** is an interface with a single method that allows clients to dynamically query whether an object supports a particular interface:
```
org.eclipse.core.runtime/IAdaptable
public interface IAdaptable {
	public Object getAdapter(Class adapter);
}
```
The **getAdapter**() method answers a particular interface. Callers of **getAdapter**() pass in the class object corresponding to the interface.
![[Eclipse core runtime - IAdaptable-2024105-1.png]]
**GetAdapter**() returns an object castable to the given class (or null if the interface isn’t supported). 

The **IAdaptable** interface is used in two different ways in Eclipse.
- A class wants to provide additional interfaces without exposing them in the API—In this case, **getAdapter**() is implemented by the class itself. Adding a new interface requires changing the implementation of **getAdapter**(). This is useful when a class wants to support additional interfaces without changing its existing interface and thereby breaking the API.
- A class is augmented from the outside to provide additional services—In this case, no code changes to the existing class are required and the **getAdapter**() implementation is contributed by a factory.

### Surfacing Interfaces Using IAdaptable
Here is an implementation of a **getAdapter**() method in a class that supports the **IPropertySource** interface:
```
Object getAdapter(Class adapter) {
	if (adapter.equals(IPropertySource.class))
		return new PropertySourceAdapter(this);
	return super.getAdapter(adapter);
}
```

In **PropertySourceAdapter**, you implement the **IPropertySource**:

```
public class PropertySourceAdapter implements IPropertySource {
	private Object source;
	public PropertySourceAdapter(Object source) {
		this.source= source;
	}
	public IPropertyDescriptor[] getPropertyDescriptors() {
		// return the property descriptors.
	}
	//...
}
```
Implementing **getAdapter**() directly helps with the API evolution. If later on the class wants to provide an additional interface, say **IShowInSource**, then we only have to augment the **getAdapter**() method. No changes to the external interface of the class are required:

```
Object getAdapter(Class adapter) {
	if (adapter.equals(IPropertySource.class))
		return new PropertySourceAdapter(this);
	if (adapter.equals(IShowInSource.class))
		return new ShowInSourceAdapter(this);
	return super.getAdapter(adapter);
}

```
Implementing **getAdapter**() with conditional logic is simple. But since **Extension Object** is supposed to support unanticipated extension, conditional logic isn’t the final answer. The next step is externally augmenting the inter-faces supported by an existing type.
### AdapterFactories—Adding Interfaces to Existing Types
Let’s consider how to add property-sheet support to IFile without having to pollute IFile with UI-specific behavior. A layer-preserving solution is to introduce a property-sheet wrapper for an IFile. Let’s go down this path for a while to see its problems:
```
public class FileWithProperties implements IPropertySource {
	private IFile file;
	public IPropertyDescriptor[] getPropertyDescriptors() {...}
	public Object getPropertyValue(Object id) {...}
	public boolean isPropertySet(Object id) {...}
	public void resetPropertyValue(Object id) {...}
	public void setPropertyValue(Object id, Object value) {...}
	
	public IFile toFile() { return file; }
}
```

The class **FileWithProperties** wraps an existing **IFile** and implements the **IPropertySource** interface. As an improvement, we could also implement the **IFile** interface (which the API contract doesn’t allow) and implement the wrapper using the Decorator pattern. This would make the use of the wrapper more transparent to clients. However, there is another problem. We now end up with two different objects representing the same **IFile**. This introduces
subtle complexities when it comes to testing files for equality. Life would be much simpler without any wrapping. Clients can just deal with IFile but extend its behavior. This is exactly the purpose of adapter factories. Here are the steps for how to extend IFile:
1. Implement an **AdapterFactory** with the adapters you want to add to a particular type.
2. Register the factory for a specific type with the **AdapterManager**. The **AdapterManager** is provided by the Platform class, a façade with static methods for general platform services.
In the implementation you have to declare which adapters the factory provides. This is done in the **getAdapterList**() method. The purpose of this declarative method is to enable a quick lookup for an adapter. With this information, Eclipse can quickly decide whether a type supports a particular adapter.
An **AdapterFactory** encapsulates the extensions you want to add for a particular type:
```
class FileAdapterFactory implements IAdapterFactory {
	public Class[] getAdapterList() {
		return new Class[] {
			IPropertySource.class
		}
	};
	//...
}
```

In our case, the factory contributes a single adapter for **IPropertySource**. The other method is **getAdapter**(). In contrast to the **IAdaptable.getAdapter**(), it has an additional argument for the object that originally receivedthe request:
```
public Object getAdapter(Object o, Class adapter) {
	if (adapter == IPropertySource.class)
		return new FilePropertySource((IFile)o);
	return null;
}
```
Next we have to register the factory for our desired type (IFile) with the **AdapterManager**. This has to occur early enough so that calls to **getAdapter**() return the correct result. This is commonly done in a plug-in’s **startup**() method or in a static initializer. Here is a corresponding code snippet:

```
IAdapterManager manager = Platform.getAdapterManager();
IAdapterFactory factory = new FileAdapterFactory();
manager.registerAdapters(factory, IFile.class);
```
![[Eclipse core runtime - IAdaptable-2024105-2.png]]
The class **File** descends from **Resource**, which descends from the class **PlatformObject**. **PlatformObject** implements **IAdaptable** and forwards **getAdapter**() invocations to the Platform’s adapter manager. If you cannot derive from **PlatformObject**, you invite others to contribute adapters by calling:
```
Platform.getAdapterManager().getAdapter(this, adapter);
```
![[Eclipse core runtime - IAdaptable-2024105-3.png]]

Here are some points about the extension mechanism.
- Multiple adapters for the same type—What happens when the same adapter is registered more than once in a type’s hierarchy? The rule is that the most specific adapter wins. Most specific means the first adapter in the base class chain followed by a depth-first order search in the interface hierarchy.
 - Stateless adapters—Adapters that have no state are the most space-efficient and easiest to manage. You store a single instance of the adapter in a field of the adapter factory or a static variable and return it when it is requested. To reuse a single instance of the adapter for all adapted objects, the adapter methods need to support passing in the adapted object. **IWorkbenchAdapter** is an interface that enables a stateless implementation. **IPropertySource** is an interface that does not.
- Instance based extensions—**IAdaptable** supports class extensions. How can you extend instances? **IResource** supports dynamic-state extension with properties. A property is identified by a qualified name and can be managed either per session or persistently. Refer to the API specification of **IResource** for more details.
- Which adapters are supported?—You cannot determine the available adapters from just looking at a class interface. You have to read the API documentation to find out which adapters are expected by a service (see for example, org.eclipse.ui.views.properties.PropertySheet). Alternatively, search for references to IAdaptable.getAdapter() to uncover all uses of adapter interfaces.
- Adapter negotiation—An **IAdaptable** client can ask for different interfaces until a suitable one is found. For example, a property sheet can be populated from an **IPropertySource** adapter. However, a client can also get full control over the Property Sheet view contents by providing an **IPropertySheetPage** adapter. The Property Sheet view first queries for an **IPropertySheetPage** adapter and if there isn’t one then it uses a default Property Sheet page that queries for the **IPropertySource** adapter.
- Reduced programming comfort—Programming with adapters is more bulky than programming with interfaces directly. With an adapter a direct method call is replaced by multiple statements:
```
IPropertySource source=
	(IPropertySource) object.getAdapter(IPropertySource.class);
if (source != null)
	return source.getPropertyDescriptors();
```
**Extension Object**—implemented as **IAdaptable**, **IAdapterManager**, and **IAdapterFactory**—furthers Eclipse’s goal of supporting unanticipated extension by allowing contributors to extend the classes an object can pretend to be. While more complicated than just using objects, the extra complexity is balanced by the additional flexibility.


#  Core Workspace - Resources
In Eclipse, the file system rules. An Eclipse workspace is mapped to the file system directly and there is no intermediate repository. Users can either change a resource on the file system directly or from inside Eclipse. The resources plugin provides support for managing a workspace and its resources.
![[Eclipse core-2024105-1.png]]

## Accessing File-System Resources—Proxy and Bridge
Clients need a way to track a resource in the file system. **Resources** change during their lifetime: they are created, their contents change, they are replaced with another version, they are deleted, and sometimes recreated. The information about a resource changes over its lifetime, but its identity doesn’t. Clients need a simple way to refer to a resource independent of the resource’s state in the workspace. We don’t want to hold onto stale state, for example, when a file is deleted.
Eclipse addresses this problem by only giving clients access to a **handle** for a resource, not the full resource. This design is best described as the conjunction of the two structural patterns: **Proxy** and **Bridge**. Neither of the patterns alone captures the full intent of the design. The **Proxy** pattern tells us how to control access to an object, and the **Bridge** pattern tells us how to separate an interface from its implementation. To quote their intent:
- **Proxy**—“Provide a surrogate or place holder for another object to control access to it”
- **Bridge** (also known as Handle/Body)—“Decouple an abstraction from its implementation so that the two can vary independently”
The relevant aspect from **Proxy** is controlling the access to an object. This is the key to avoid clients holding on to stale state. The relevant aspect from **Bridge** is the strong separation between an interface and its implementation. Both patterns address their problems by introducing a level of indirection. Applied to file-system resources, this gives us
- The handle, which acts like a key for a resource.
- An info object storing the representation of the file’s state. There is only one implementor for each handle and it is therefore an example of a degenerate **Bridge**.
The **handles** are defined as interfaces **IFile**, **IFolder**, **IProject**, and **IWorkspaceRoot**
![[Eclipse core-2024105-2.png]]

![[Eclipse core-2024105-3.png]]
![[Eclipse core-2024105-4.png]]
From the diagram we can see that a **File handle** knows only the path to the **resource** and its **containing workspace**. It acts like a key to access the file. Here are some interesting characteristics of the **resource handles**.
- They are Value Objects—small objects whose equality isn’t based on identity. Once created, none of their fields ever change. This enables clients to store them in hashed data structures like a Map. More than one handle can refer to the same resource. Always use IResource.equals() when comparing handles.
- **Handles** define the behavior of a resource, but they don’t keep any resource state information.
- A handle can refer to non-existing resources.
- Some operations can be implemented from information stored in the handle only (handle operations). A resource need not exist to successfully execute such an operation. Examples of handle operations are: **getFullPath**(), **getParent**(), and **getProject**(). The existence of a **resource** can be tested with **exists**(). Operations that depend on the existence of the resource throw a **CoreException** when the resource doesn’t exist.
- **Handles** are created from a parent handle:
	```
	IProject project;
	IFolder folder= project.getFolder("someFolder");
	```
- **Handles** are used to create the underlying resource:
	```
	folder.create(...);
	```
- Since a **handle** is independent of the state in the file system, there is no way for the client to keep a reference to stale state, for example state about a deleted file. Every time you want state for the handle you have to fetch it anew.
Figure 32.5 illustrates the handle/body separation used for resources. In the following explanations we will use UML diagrams to illustrate the pattern stories. We annotate the UML with shaded rounded rectangles identifying the pattern being applied.
![[Eclipse core-2024105-5.png]]
The handle **IResource** is API and the **ResourceInfo** and **Resource** are internal classes. **ResourceInfo** is extended to store additional state for projects (**ProjectInfo**) and the workspace root. This illustrates the strong separation suggested by the **Bridge** pattern. Eclipse has a lot of freedom when resolving handles to find the corresponding resource state.
When an **IResource** is created, the resource information is retrieved from the workspace and the path information is stored inside the handle. 
<span style="background:#fff88f">The Eclipse workspace stores the resource info as a complete tree in memory</span>. This tree is referred to as the element tree. The resource info object corresponding to a handle is retrieved by traversing the element tree using the path stored inside the handle. A big advantage of this workspace implementation is that you can navigate the tree of files in almost no time. When you use the file system you have to make many queries to the underlying operating system (OS).
The resources plug-in not only applies handle/body separation for resources, it also applies the same separation for **markers**. <span style="background:#fff88f">A marker associates attributes with a resource</span>. A marker doesn’t store the attributes directly, but only stores a reference to their marker attribute info object. For markers the info object corresponds to the class **MarkerInfo**. The same coding idioms used for resources are consistently applied to markers.
- Markers are created from a handle.
- Accessing a marker attribute can throw a CoreException.

# The Workspace—Composite
The Eclipse workspace provides resources stored in the file system. A workspace consists of projects containing folders containing files.
You access the **Singleton** workspace instance from the static accessor **ResourcesPlugin.getWorkspace**(). An **IWorkspaceRoot** represents the top of the resource hierarchy in a workspace.
The workspace is a hierarchical structure and it therefore matches the intent of the **Composite** pattern well: “<span style="background:#fff88f">Compose object into tree structures to represent part/whole hierarchies. Composite lets clients treat individual objects and compositions of objects uniformly.</span>”
![[Eclipse core-2024105-6.png]]
Some observations on the workspace implementation.
- **IResource** provides access to its parent. The **getParent**() method is a handle-only operation that can derive the parent from the path stored in the handle.
- **IContainer** is the common base interface for the different composite classes. It provides a method **members**() that returns its children as a typed **IResource** array.
You can traverse a resource tree using the **members**() method provided by**IContainer**.

## Traversing the Resource Tree—Visitor
Traversing a resource tree manually using the **members**() method results in a lot of control-flow code in clients. The control flow to traverse a resource tree can be extracted with a **visitor**. When we check the intent of **Visitor** we find, “<span style="background:#fff88f">Represent an operation to be performed on the elements of an object structure. Visitor lets you define a new operation without changing the classes of the elements on which it operates</span>.” The main purpose here is to extract the common control flow and make it generally reusable.
**IResourceVisitor** is the **visitor** interface, which is accepted by **IResource**.
![[Eclipse core-2024105-7.png]]
The **accept**() method implements the resource traversal and calls back the **visitor** for each resource. The **IResourceVisitor** interface isn’t type-specific; there are no separate methods for visiting a file or a folder. If you need to distinguish between the visited resource types, you can do this inside the **visit**() method using the **getType**() method. The code snippet from the Eclipse resource implementation illustrates the use of visitor:
```
org.eclipse.core.internal.resources/ResourceTree
private void addToLocalHistory(IResource root, int depth) {
	IResourceVisitor visitor = new IResourceVisitor() {
		public boolean visit(IResource resource){
		if (resource.getType() == IResource.FILE)
			addToLocalHistory((IFile) resource);
		return true;
	  };
	}
	try {
	root.accept(visitor, depth, false);
	} 
	catch (CoreException e) {
	}
}

```
Returning **true** from **visit**() indicates that the children of a resource should be visited. Returning **false** stops the traversal at the current resource.
While performance tuning, it was discovered that for some common traversals only a subset of the resource information is actually needed by the **visitor**. **IResourceProxyVisitor** was introduced to reduce the information fetched from the file system. It doesn’t pass an **IResource** to the **visit**() method but an **IResourceProxy**.
```
org.eclipse.core.resources/IResourceProxyVisitor
public interface IResourceProxyVisitor {
	public boolean visit(IResourceProxy proxy) throws CoreException;
}
```
**IResourceProxy** is an example of a **virtual proxy**. It creates an expensive object on demand. The expensive object is in this case the full workspace path of a resource. The proxy is only valid during the call of the **visit**() method.

## Tracking Resource Changes—Observer
Resources in the workspace can change either as a result of manipulating them inside Eclipse or from resynchronizing them with the local file system. In both cases, observing clients need precise change information so that they can update themselves efficiently. To observe changes, the workspace provides a **resource listener**, which is an **Observer** variation. <span style="background:#fff88f">Observers register with the workspace, which acts as the subject to be notified about changes.</span>
Internally the notification mechanism is implemented by a **NotificationManager**.
![[Eclipse core-2024105-8.png]]Digging into the implementation of **NotificationManager**, we find that it is very careful in handling the case of modifications to the listener list during notification. By copying the listener list at the beginning of a notification, it ensures that any modifications to the listener list during the notification will have no effect on the ongoing notification. This is also referred to as making a **Safe Copy**. The internal class **ResourceChangeListenerList** implements the listener management.
```
org.eclipse.core.internal.events/ResourceChangeListenerList
public ListenerEntry[] getListeners() {
	if (size == 0)
		return EMPTY_ARRAY;
	ListenerEntry[] result = new ListenerEntry[size];
	System.arraycopy(listeners, 0, result, 0, size);
	return result;
}
```
The **observer** pattern asks us to decide how to provide details about a change notification. To quote the Observer pattern: “<span style="background:#fff88f">At one extreme, which we call the push model, the subject sends observers detailed information about the change, whether they want it or not. At the other extreme is the pull model; the subject sends nothing but the most minimal notification, and observers ask for details explicitly thereafter.</span>” Changes in a resource tree can be complex, so Eclipse uses the push model and provides detailed information about a change to all observers. Eclipse calls the change information **resource deltas**.
![[Eclipse core-2024105-9.png]]A **resource delta** describes the change between two states of the workspace tree. It is itself a tree of nodes. Each delta node describes how a resource has changed and provides delta nodes describing the changes to its children. A delta tree is rooted at the workspace root.

A resource delta has several interesting properties.
- A **resource delta** describes a single change and multiple changes using the same structure.
- A **resource delta** describes complete change information including information about moved resources and marker changes. The method **getMarkerDeltas()** returns the changes to markers.
- It is easy to process a resource delta recursively top-down when updating an observer.
- Because a resource delta can be expensive, it is only valid during the call of **resourceChanged**().
- Because resources are just handles to the real resources, a delta can easily describe deleted resources as well.
You can reuse the traversal logic of a resource delta with an **IResourceDeltaVisitor**:

```
org.eclipse.core.resources/IResourceDeltaVisitor
public interface IResourceDeltaVisitor {
	public boolean visit(IResourceDelta delta) throws CoreException;
}
```

## Batching Changes—Execute Around Method
Any system based on change events is vulnerable to being flooded with resource change events. The common practice to avoid this flooding is to **batch changes** wherever possible. Changes should be grouped together so that only a single notification is sent out at the end of a single logical change. 
In Eclipse the batching is achieved using an **IWorkspaceRunnable** that is passed to the workspace for execution. The action specified by the runnable is then run as an atomic workspace operation. The deltas are accumulated during the operation and broadcast at the end. The snippet below illustrates how to create a marker and set its attributes with an **IWorkspaceRunnable** so that only one instead of two (creation, setting attributes) notifications are sent out:

```
org.eclipse.ui.texteditor/MarkerUtilities
public static void createMarker(final IResource resource,
	final Map attributes, final String markerType) throws CoreException {
		IWorkspaceRunnable r= new IWorkspaceRunnable() {
		public void run(IProgressMonitor monitor) throws CoreException{
			IMarker marker= resource.createMarker(markerType);
			marker.setAttributes(attributes);
		}
	};
	resource.getWorkspace().run(r, null);
}
```
An **IWorkspaceRunnable** is an example of the **Execute Around Method** pattern from Smalltalk2 adapted to Java. **IWorkspace.run**() is the **execute-around** method. Before the runnable is executed, it informs the workspace to start batching. Then the **runnable** is invoked. Finally, when the runnable is done, **IWorkspace.run**() informs the workspace to end batching. Without an **execute-around** method, clients need to explicitly invoke the begin and end methods in the right order. This is error prone. Another benefit of an **execute-around** method is that the begin and end batching methods do not have to be published as API.
Another example of an **execute-around** method in Eclipse is **Platform.run(ISafeRunnable runnable)**. It invokes the runnable in a protected mode and catches exceptions.
