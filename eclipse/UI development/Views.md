

Views must implement the org.eclipse.ui.**IViewPart** interface. Typically, views are subclasses of org.eclipse.ui.part.**ViewPart** and thus indirectly subclasses of org.eclipse.ui.part.**WorkbenchPart**, inheriting much of the behavior needed to implement the **IViewPart** interface.
Views are contained in a view site (an instance of the type org.eclipse.ui.**IViewSite**), which in turn is contained in a workbench page (an instance of org.eclipse.ui.**IWorkbenchPage**). In the spirit of lazy initialization, the IWorkbenchPage instance holds on to instances of
org.eclipse.ui.**IViewReference** rather than the view itself so that views can be enumerated and referenced without actually loading the plug-in that defines the view.

![[Views-20231126-1.png]]


Views share a common set of behaviors with editors via the superclass org.eclipse.ui.part.**WorkbenchPart** and the org.eclipse.ui.**IWorkbenchPart** interface but have some very important differences. Any action performed in a view should immediately affect the state of the workspace and the underlying resource(s), whereas editors follow the classic open-modify-save paradigm.

Editors appear in one area of Eclipse, while views are arranged around the outside of the editor area . Editors are typically resource-based, while views may show information about one resource, multiple resources, or even something totally unrelated to resources at all.

Because there are potentially hundreds of views in the workbench, they are organized into categories.

## View declarition

### Declaring a view category

![[Views-20231126-2.png]]

### Declaring a view

![[Views-20231126-3.png]]


# ViewPart
The code defining a view’s behavior is found in a class implementing the org.eclipse.ui.**IViewPart** interface, typically by subclassing the org.eclipse.ui.part.**ViewPart** abstract class.

## View methods
IViewPart and its supertypes define the following methods.
**createPartControl**(Composite)—This method is required because it creates the controls comprising the view. Typically, this method simply calls more finely grained methods such as **createTable**, **createSortActions**, **createFilters**, and so on.
**dispose**()—Cleans up any platform resources, such as images, clipboard, and so on, that were created by this class. This follows the if you create it, you destroy it theme that runs throughout Eclipse.
**getAdapter**(Class)—Returns the adapter associated with the specified interface so that the view can participate in various workbench behaviors.
**saveState**(IMemento)—Saves the local state of this view, such as the current selection, current sorting, current filter, and so on .
**setFocus**()—This method is required because it sets focus to the appropriate control within the view.

A view can participate in various workbench behaviors by either directly implementing or having a getAdapater(Class) method that returns an instance of a specific interface. A few of these interfaces are listed below:
**IContextProvider**—Dynamic context providers are used for providing focused dynamic help that changes depending on the various platform states .
**IContributedContentsView**—Used by PropertySheet used to identify workbench views which allow other parts (typically the active part) to supply their contents.
**IShowInSource**—Enables a view to provide a context when the user selects Navigate > Show In > ...
**IShowInTarget**—Causes a view to appear in the Navigate > Show In submenu and the **IShowInTarget**#show(ShowInContext) method to be called when the user makes a selection in that submenu.
**IShowInTargetList**—Specifies identifiers of views that should appear in the Navigate > Show In submenu.

## View controls
Views can contain any type and number of controls, but typically, a view contains a single table or tree control. 

## View model
A view can have its own internal model, it can use existing model objects such as an IResource and its subtypes, or it may not have a model at all.

## Content provider
When the model objects have been created, they need to be linked into the view. A content provider is responsible for extracting objects from an input object and handing them to the table viewer for displaying. Although the IStructuredContentProvider does not specify this, the content provider has also been made responsible for updating the viewer when the content of FavoritesManager changes.
## Label provider
The label provider takes a table row object returned by the content provider and extracts the value to be displayed in a column.

## Viewer sorter
Although a content provider serves up row objects, it is the responsibility of the ViewerSorter to sort the row objects before they are displayed.

## Viewer filters


## View selection


# View commands
A view command can appear as a menu item in a view’s context menu, as a toolbar button on the right side of a view’s title bar, and as a menu item in a view’s pull-down menu
## Model command handlers

## Context menu
Typically, views have context menus populated by commands targeted at the view or selected objects within it. There are several steps to create a view’s context menu programmatically. If you want other plug-ins to contribute commands to your view’s context menu via declarations in the plug-in manifest , then you must take several more steps to register your view.
### Creating contributions

### Creating the context menu

### Dynamically building the context menu

### Selection provider

### Filtering unwanted actions

## Toolbar buttons

## Pull-down menu

## Keyboard commands

## Global commands

## Clipboard commands

## Drag-and-drop support

## Inline editing


# Linking the view
In many situations, the current selection in the active view can affect the selection in other views, cause an editor to open, change the selected editor, or change the selection within an already open editor.  For example, in the Java browsing perspective, changing the selection in the Types view changes the selection in both the Projects and the Packages views, changes the content displayed in the Members view, and changes the active editor. For a view to both publish its own selection and to consume the selection of the active part, it must be both a selection provider and a selection listener.

## Selection provider
For a view to be a selection provider, it must register itself as a **selection provider** with the **view site**. In addition, each of the objects contained in the view should be adaptable so that other objects can adapt the selected objects into objects they can understand. Register the view as a selection provider:
`getSite().setSelectionProvider(viewer);`

## Adaptable objects
The org.eclipse.core.runtime.**IAdaptable** interface allows an object to convert one type of object that it may not understand to another type of object that it can interrogate and manipulate.

## Selection listener
For a view to consume the selection of another part, it must add a selection listener to the page so that when the active part changes or the selection in the active part changes, it can react by altering its own selection appropriately.
## Opening an editor

# Saving View state

## Saving local view information
Eclipse provides a memento-based mechanism for saving view and editor state information. In this case, this mechanism is good for saving the sorting and filter state of a view because that information is specific to each individual view.
To save the sorting state, two methods should be added. The first method saves the state as an instance of **IMemento** by converting the sort order and ascending/descending
state into an XML-like structure. The second method takes a very guarded approach to reading and resetting the sort order and ascending/descending state from IMemento so that the sort state will be valid even if IMemento is not what was expected.

### Saving global view information








































