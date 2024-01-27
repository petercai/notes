
## Using fragments
Fragments are used to extend the functionality of another plug-in. Fragments are typically used to supply alternative language packs, maintenance updates, and platform-specific implementation classes. When a fragment is loaded, its features and capabilities are merged with those of the base plug-in such that they appear to have come from the base plug-in itself.
Fragments are used to extend the functionality of another plug-in. Fragments are typically used to supply alternative language packs, maintenance updates, and platform-specific implementation classes. When a fragment is loaded, its features and capabilities are merged with those of the base plug-in such that they appear to have come from the base plug-in itself.


When neither referencing the code directly nor copying the code into your own plug-in will work, you can try using fragments. Fragments are chunks of code defined in a plug-in-like structure that Eclipse automatically attaches to an existing plug-in.
As far as the Eclipse system is concerned, code contributed by a fragment is treated exactly the same as code that exists in the target plug-in. Originally, fragments were created to insert different National Language Support (NLS) code into a plug-in based on the intended audience, but you can exploit this mechanism to solve your own problems. Using this technique, you cannot override classes that already exist in the plug-in, but you can insert new utility classes used to access methods that were previously not accessible because
they had default or protected visibility.


# Adapters
Eclipse provides an adapter framework for translating one type of object into a corresponding object of another type. This allows for new types of objects to be systematically translated into existing types of objects already known to Eclipse.

When a user selects elements in one view or editor, other views can request adapted objects from those selected objects that implement the **org.eclipse.core.runtime.IAdaptable** interface. This means that items selected in the Favorites view can be translated into resources and Java elements as requested by existing Eclipse views without any code modifications to them.

## IAdaptable
For objects to participate in the adapter framework, they must first implement the **IAdaptable** interface. The **IAdaptable** interface contains this single method for translating one type of object into another:
**getAdapter**(Class)—Returns an object that is an instance of the given class and is associated with this object. Returns null if no such object can be provided.

Implementers of the IAdaptable interface attempt to provide an object of the specified type. If they cannot translate themselves, then they call the adapter manager to see whether a factory exists for translating them into the specified type.

```
private IResource resource;
public Object getAdapter(Class adapter) {
	if (adapter.isInstance(resource))
		return resource;
	return Platform.getAdapterManager()
		.getAdapter(this, adapter);
}
```

## Using adapters
Code that needs to translate an object passes the desired type, such as IResource.class, into the getAdapter() method, and either obtains an instance of IResource corresponding to the original object or null indicating that such a translation is not possible.

```
if (!(object instanceof IAdaptable)) {
	return;
}
	
MyInterface myObject
	= ((IAdaptable) object).getAdapter(MyInterface.class);
	
if (myObject == null) {
	return;
}
... do stuff with myObject ...
```

## Adapter factory
Implementing the **IAdaptable** interface allows new types of objects, such as the Favorites items to be translated into existing types such as **IResource**, but how are existing types translated into new types? To accomplish this, implement the **org.eclipse.core.runtime.IAdapterFactory** interface to translate existing types into new types.
For example, in the Favorites product, you cannot modify the implementers of IResource, but can implement an adapter factory to translate IResource into IFavoriteItem. The getAdapterList() method returns an array indicating the types to which this factory can translate, while the getAdapter() method performs the translation. In this case, the factory can translate IResource and IJavaElement objects into IFavoriteItem objects so the getAdapterList() method returns an array containing the IFavoriteItem.class.

Adapter factories must be registered with the adapter manager before they are used. Typically, a plug-in registers adapters with adapter managers when it starts up and unregisters them when it shuts down. 

Using an adapter factory has a bit more overhead than referencing the FavoritesManager directly. When considering this for your own product, you will need to determine whether the advantage of looser coupling outweighs the additional complexity and slightly slower execution time inherent in this approach.

### IWorkbenchAdapter
Eclipse uses the **IWorkbenchAdapter** interface to display information. Many Eclipse views, such as the Navigator view, create an instance of **WorkbenchLabelProvider**. This label provider uses the IAdaptable interface to translate unknown objects into instances of IWorkbenchAdapter, and then queries this translated object for displayable information such as text and images.

# Label Decorators
Label decorators visually indicate specific attributes of an object. For example, if a project is stored in a repository, then it has a small cylinder below and to the right of its folder icon in the Navigator view. The Navigator’s label provider returns a folder image, which is then decorated by the repository’s label decorator with a small cylinder. The final composite image is then rendered in the Navigator view. Label decorators are not restricted to decorating images only; an object’s text can be enhanced by adding characters to the beginning or end.
The **org.eclipse.ui.decorators** extension point provides a mechanism for adding new label decorators. Label decorators appear in the General > Appearance > Label Decorations preference page and can be enabled or disabled by the user. Behavior for a label decorator is supplied by implementing **ILabelDecorator**, and optionally **IFontDecorator** and/or
**IColorDecorator**. If the information to decorate an object is not immediately available, for example the type of decoration depends on a network query, then implement **IDelayedLabelDecorator**.

![[Eclipse Adv. topics-20231203-1.png]]

## ILightweightLableDecorator
Instances of **ILightweightLabelDecorator** can modify the image, text, font, and color displayed for an object. Create the class that contains the decorative behavior when you specify the class attribute by clicking the class label to the left of the class attribute’s value.

## Decorative label decorators
If you simply want to decorate a label by adding a static image in one of the quadrants without any text modifications, then you can specify the icon attribute instead of the class attribute. If the class attribute is not specified,Eclipse places the image specified by the icon attribute in the quadrant specified by the location attribute.
In this case, there is no need to create a class that implements ILightweightLabelDecorator because Eclipse provides this behavior for you. A read-only file decorator is one example of a decorative label decorator.

```
<decorator
	lightweight="true"
	location="BOTTOM_LEFT"
	label="Locked"
	icon="icons/locked_overlay.gif"
	state="true"
	id="com.qualityeclipse.favorites.locked">
	<description>
		Indicates whether a file is locked
	</description>
	<enablement>
		<and>
			<objectClass
				name="org.eclipse.core.resources.IResource"/>
			<objectState name="readOnly" value="true"/>
			</and>
	</enablement>
</decorator>
```

With this declaration in the plug-in manifest, a small lock icon appears in the lower left corner of the icon associated with any locked resource:

![[Eclipse Adv. topics-20231203-2.png]]

## IDecoratorManager
If you want to decorate your own view. Eclipse provides a **DecoratingLabelProvider** and a decorator manager via the **getDecoratorManager**() method in **IWorkbench**. 


# Background Tasks - Jobs API
Long-running operations should be executed in the background so that the UI stays responsive. One solution is to fork a lower-priority thread to perform the operation rather than performing the operation in the UI thread. But, how do you keep the user informed as to the progress of the background operation? Eclipse provides a Jobs API for creating, managing, and displaying background operations.

# Plug-in ClassLoaders
Most of the time you can easily ignore ClassLoaders, knowing that as long as your classpath is correct—or in this case, the dependency declaration in the plug-in manifest —class
loading will happen automatically, without intervention. But what if you want to load classes that are not known when a plug-in is compiled? Information about code developed by the user in the workspace is accessible via the JDT interfaces such as ICompilationUnit, IType, and IMethod; however, it is not normally on a plug-in’s classpath and thus cannot be executed. Normally, this is a good thing, because code under development can throw exceptions, or under rare circumstances, crash Eclipse without any warning.

The Eclipse debugger executes user-developed code in a separate VM to avoid these problems, but it is a heavyweight, involving the overhead of launching a separate VM and communicating with it to obtain results. If you need a quick way to execute user-developed code in the same VM as Eclipse and are willing to accept the risks involved in doing so, then you need to write a ClassLoader.


# Early Startup
Early plug-in startup use the **org.eclipse.ui.startup** extension point to ensure that your plug-in will be started when Eclipse starts. Doing so should not be done lightly because it defeats the Eclipse lazy-loading mechanism, causing Eclipse to always load and execute your plug-in thus consuming precious memory and startup time. If you must do this, then keep your early startup plug-in small so that it takes up less memory and executes quickly when it starts.

## Managing early startup
Eclipse does not provide a mechanism for programmatically specifying whether a plug-in should be started immediately when it is launched. If you have one or more plug-ins that may need early startup, then consider creating a small plug-in that manages early startup. For example, if you have a large plug-in that only needs early startup if the user has enabled
some functionality, then create a small early startup plug-in that determines whether that functionality has been enabled, and if so, starts the larger plug-in.
![[Eclipse Adv. topics-20231203-3.png]]




