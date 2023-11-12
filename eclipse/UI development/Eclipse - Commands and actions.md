
Commands and actions are two different APIs for accomplishing the same thing: declaring and implementing functions that manifest as menu items and toolbar buttons. The action API has been around since before Eclipse 3.0, while the commands API has only just solidified as of Eclipse 3.3 with small refinements in Eclipse 3.4.

Commands and actions are defined through various extension points so that new functionality can be easily added at various points throughout the Eclipse framework.

With actions, there are different extension points for each different area where one can appear within the UI, yet no separation of presentation from implementation.

The commands API separates presentation from implementation by providing one extension point for specifying the command, another for specifying where it should appear in the UI, and a third for specifying the implementation. This, along with a richer declarative expression syntax, makes the commands API more flexible than the actions API.
# 1. Commands

Declaring and implementing a menu or toolbar item using the command API involves declaring a **command**, at least one **menu contribution** for that command, and at least **one handler** for that command.

<u>The command declaration is the abstract binding point associating one or more menu contributions with one or more handlers.</u>

![[Eclipse - show actions in toolbar-20231022-1.png]]
A **menu contribution** declaration defines where in the user interface that command should appear, and the text and image associated with that representation. A **handler** declaration  associates a command with a concrete class implementing the behavior for that command.

## defining a command

The first step to add a menu or toolbar item is to declare the “intent” of that function using the **org.eclipse.ui.commands** extension point. Using this extension point, you declare both the **category of the command** and the command itself. <u>Categories are useful for managing large numbers of commands more easily.</u>

![[Eclipse - Commands and actions-20231030-1.png]]


## Menu and Toolbar contributions

Use the **menuContribution** element of the **org.eclipse.ui.menus** extension point to define where (and when) the command should appear in the user interface. The **locationURI** attribute specifies where the command should appear, while a **visibleWhen** child element specifies when the command should appear. Using these two aspects of menuContribution, you can display a command in any menu, context menu, or toolbar and limit its visibility
to when it is applicable.
**locationURI**—“menu:org.eclipse.ui.main.menu?after=additions” Identifies the location in the user interface where commands associated with this menu contribution will appear


## Handlers




# IAction versus IActionDelegate

An Eclipse action is composed of several parts, including the **XML declaration of the action** in the plug-in’s manifest, the **IAction** object instantiated by the Eclipse UI to represent the action, and the **IActionDelegate** defined in the plug-in library containing the code to perform the action:

![[Eclipse - Commands and actions-20231030-2.png]]

This separation of the **IAction** object, defined and instantiated by the Eclipse user interface based on the plug-in’s manifest and the **IActionDelegate** defined in the plug-in’s library, <u>allows Eclipse to represent the action in a menu or toolbar without loading the plug-in that contains the operation until the user selects a specific menu item or clicks on the toolbar</u>.
Again, this approach represents one of the overarching themes of Eclipse: **lazy plug-in initialization.**

There are several interesting subtypes of IActionDelegate:
- **IActionDelegate2**—Provides lifecycle events to action delegates; if you are implementing IActionDelegate and need additional information, such as <u>when to clean up before the action delegate is disposed</u>, then implement IActionDelegate2 instead. In addition, an action delegate implementing IActionDelegate2 will <u>have runWithEvent(IAction, Event) called instead of run(IAction).</u>
- **IEditorActionDelegate**—Provides lifecycle events to action delegates associated with an editor.
- **IObjectActionDelegate**—Provides lifecycle events to action delegates associated<u> with a context menu</u>.
- **IViewActionDelegate**—Provides lifecycle events to action delegates associated with <u>a view</u>.
- **IWorkbenchWindowActionDelegate**—Provides lifecycle events to action delegates associated with the<u> workbench window menu bar or toolbar.</u>

## Workbench window Actions - ActionSets

Where and when an action appears is dependent on the extension point and filter used to define the action. 
We can add a new menu to the workbench menu bar and a new button to the workbench toolbar using the **org.eclipse.ui.actionSets** extension point. These actions are very similar to menu contributions with a locationURI id  equal to “**org.eclipse.ui.main.menu**”.  The declaration must describe the location and content of the new menu and reference the action delegate class that performs the operation.

### ActionSet
- **id**—“com.qualityeclipse.favorites.workbenchActionSet” The unique identifier used to reference the action set.
- **label**—“Favorites ActionSet” The text that appears in the Customize Perspective dialog.
- **visible**—“true” Determines whether the action set is initially visible. The user can show or hide an action set by selecting Window > Customize Perspective..., expanding the Other category in the Customize Perspective dialog, andchecking or unchecking the various action sets that are listed.
### Menu
- **id**—“com.qualityeclipse.favorites.workbenchMenu” The unique identifier used to reference this menu.
- **label**—“Fa&vorites” The name of the menu appearing in the workbench menu bar. The “&” is for keyboard accessibility.
- **path**—“additions” The insertion point indicating where the menu will be positioned in the  menu bar. 
##### Groups in a menu
	Actions are added not to the menu itself, but to groups within the menu, so first, some groups need to be defined by adding "**New > groupMarker**" under the menu. Select the new groupMarker and change the name to “content” to uniquely identify that group within the  menu. Add a second group to the menu; however, this time select "**New > separator**" and give it the name “**additions**”. A separator group has a horizontal line above the first menu item in the group, whereas a groupMarker does not have any line. The additions group is not used here, but it exists  in case another plug-in wants to contribute actions to the plug-in’s menu.

### Action - menu item and toolbar button
- **id**—“com.qualityeclipse.favorites.openFavoritesView” The unique identifier used to reference the action.
- **label**—“Open Favo&rites View” The text appearing in the Favorites menu. The “&” is for keyboard accessibility.
- **menubarPath**—“com.qualityeclipse.favorites.workbenchMenu/content” The insertion point indicating where the action will appear in the menu.
- **toolbarPath**—“Normal/additions” The insertion point indicating where the button will appear in the toolbar.
- **tooltip**—“Open the favorites view in the current workbench page” The text that appears when the mouse hovers over the action’s icon in the workbench toolbar.
- **allowLabelUpdate**—Optional attribute indicating whether the retarget action allows the handler to override its label and tooltip. Only applies if the retarget attribute is true.
- **class**—The **org.eclipse.ui.IWorkbenchWindowActionDelegate** delegate used to perform the operation. If the pulldown style is specified, then the class must implement the **org.eclipse.ui.IWorkbenchWindowPulldownDelegate** interface. The class is instantiated using its no argument constructor, but may be parameterized using the **IExecutableExtension** interface.
- **definitionId**—The command identifier for the action, which allows a key sequence to be associated with it.
- **disabledIcon**—The image displayed when the action is disabled. 
- **enablesFor**—An expression indicating when the action will be enabled. If blank, then the action is always active unless overridden programmatically via the **IAction** interface.
- **helpContextId**—The identifier for the help context associated with the action.
- **hoverIcon**—An image displayed when the cursor hovers over the action without being clicked. 
- **icon**—The associated image. 
- **retarget**—An optional attribute to retarget this action. When true, view and editor parts may supply a handler for this action using the standard mechanism for setting a global action handler on their site using this action’s identifier.<u> If this attribute is true, the class attribute should not be supplied.</u>
- **state**—For an action with either the radio or toggle style, set the initial state to true or false.
- **style**—An attribute defining the visual form of the action and having one of the following values:
	- push—A normal menu or toolbar item (the default style).
	- radio—A radio button-style menu or toolbar item where only one item at a time in a group of items all having the radio style can be active. See the state attribute.
	- toggle—A checked menu item or toggle tool item. See the state attribute.
	- pulldown—A submenu or drop-down toolbar menu. See the class attribute.

#### Insertion points
Because Eclipse is composed of multiple plug-ins—each one capable of contributing actions but not necessarily knowing about one another at build-time—the absolute position of an action or submenu within its parent is not known until runtime. Even during the course of execution, the position might change due to a sibling action being added or removed as the user changes a selection. For this reason, Eclipse uses identifiers to reference a menu, group,
or action, and a path, known as an **insertion point**, for specifying where a menu or action will appear.

Every insertion point is composed of one or two identifiers separated by a forward slash, indicating the parent (a menu in this case) and group where the action will be located. 

In some cases, such as when the parent is the workbench menu bar or a view’s context menu, the parent is implied and thus only the group is specified in the insertion point.

Typically, plug-ins make allowances for other plug-ins to add new actions to their own menus by defining an empty group labeled “**additions**” in which the new actions will appear. The “**additions**” identifier is fairly standard throughout Eclipse, indicating where new actions or menus will appear, and is included in it as the **IWorkbenchActionConstants.MB_ADDITIONS** constant.

**Nested ActionSet** Problem
Defining an action in an actionSet that contributes to a menu defined in a different actionSet can result in the following error in the Eclipse log file: Invalid Menu Extension (Path is invalid): some.action.id To work around this issue, define the menu in both actionSets. 

The **toolbarPath** attribute is also an insertion point and has a structure identical to the **menubarPath** attribute, but indicates where the action will appear in the workbench toolbar rather than the menu bar. 

### Action delegate
The action is almost complete except for the action delegate, which contains
the behavior associated with the action

#### **selectionChanged** method
While the action declaration in the plug-in manifest provides the initial state of the action, the **selectionChanged**() method in the action delegate provides an opportunity to dynamically adjust the state, enablement, or even the text of the action using the IAction interface.
The **enablesFor** attribute is used to specify the number of objects to select for an action to be enabled, but further refinement of this enablement can be provided by implementing the **selectionChanged**() method. This method can interrogate the current selection and call the **IAction.setEnabled**() method as necessary to update the action enablement.
In order for the action delegate’s selectionChanged() method to be called, you need to call getViewSite().setSelectionProvider(viewer) in your view’s createPartControl() method.

#### **run** method
The **run**() method is called when a user selects an action and expects an operation to be performed. Similar to the **selectionChanged**() method, the **IAction** interface can be used to change the state of an action dependent onthe outcome of an operation.
	<u>Guard Code Needed</u>
	Be aware that if the plug-in is not loaded 	and the user selects a menu option causing the plug-in to be loaded,	the **selectionChanged**() method may not be called before the
	**run**() method, so the run() **method** still needs the appropriate guard code. In addition, the **run**() method executes in the main UI thread,	so consider pushing long running operations into a background thread

### Object Actions
Suppose you want to make it easy for the user to add files and folders to the your view. Object contributions are ideal for this because they appear in context menus only when the selection in the current view or editor contains an object compatible with that action. In this manner, an **object contribution** is available to the user when he or she needs the action,
yet not intrusive when the action does not apply. **Object actions** are very similar to **menu contributions** that have a **locationURI** id equal to “**org.eclipse.ui.any.popup**.”

#### Defining an object-based action
In an **org.eclipse.ui.popupMenus** extension,  add an **objectContribution** with the following attributes:
- **adaptable**—“true”: Indicates that objects that adapt to **IResource** are acceptable targets.
- **id**—“**com.qualityeclipse.favorites.popupMenu**”: The unique identifier for this contribution.
- **nameFilter**—Leave blank: A wildcard filter specifying the names that are acceptable targets. For example, entering “.java” would target only those files with names ending with .java. 
- **objectClass**—“org.eclipse.core.resources.**IResource**”: The type of object that is an acceptable target. Use the Browse... button at the right of the **objectClass** field to select the existing org.eclipse.core.resources.**IResource** class. If you want to create a new class, then click the **objectClass**: label to the left of the **objectClass** field.

Next, add an action to the new **objectContribution** with the following attribute values, which are very similar to the action attributes:
![[Eclipse - Commands and actions-20231104-1.png]]

- **class**—“com.qualityeclipse.favorites.actions. XxxActionDelegate”: The action delegate for the action that implements the org.eclipse.ui.IObjectActionDelegate interface. The class is instantiated using its no argument constructor, but can be parameterized using the IExecutableExtension interface. The class can be specified in one of three different ways.
- **enablesFor**—“+”: An expression indicating when the action will be enabled .
- **id**— The unique identifier for the action.
- **label**—: The text that appears in the context menu for the action.
- **menubarPath**—“additions”: The insertion point for the action .
- **tooltip**— The text that appears when the mouse hovers over the menu item in the context menu.
- **helpContextId**—The identifier for the help context associated with the action .
- **icon**—The associated image .
- **overrideActionId**—An optional attribute specifying the identifier for an action that the action overrides.
- **state**—For an action with either the radio or toggle style, set the initial state to true or false .
- **style**—An attribute defining the visual form of the action. The pulldown style does not apply to object contributions.

#### Action filtering and enablement
In keeping with the lazy loading plug-in theme, Eclipse provides multiple
declarative mechanisms for filtering actions based on the context, and
enabling visible actions only when appropriate. Because they are declared in
the plug-in manifest, these mechanisms have the advantage that they do not
require the plug-in to be loaded for Eclipse to use them.

##### Basic filtering and enablement
The **nameFilter** and **objectClass** attributes are examples of filters, while the **enablesFor** attribute determines when an action will be enabled. When the context menu is activated, if a selection does not contain objects with names that match the wildcard **nameFilter** or are not of a type specified by the **objectClass** attribute, none of the actions defined in that object contribution will appear in the context menu. In addition, the **enablesFor** attribute uses the syntax in Table 6–1 to define exactly how many objects need to be selected for a particular action to be enabled:
![[Eclipse - Commands and actions-20231104-2.png]]
The **visibility** and filter **elements** provide an additional means to limit an action’s visibility, while the **selection** and **enablement** elements provide a more flexible way to specify when an action is enabled. Still further refinement of action enablement can be provided by using the **selectionChanged**() method in the action delegate

##### The **visibility** element
The **visibility** element provides an alternate and more powerful way to specify when an **object contribution**’s actions will be available to the user as compared with the object contribution’s **nameFilter** and **objectClass**. For example, an alternate way to specify filtering for the object contribution just describedwould be:
```
<objectContribution ... >
	<visibility>
		<objectClass name="org.eclipse.core.resources.IResource"/>
	</visibility>
	...the other stuff here...
</objectContribution>
```
If the action is to be visible only for resources that are not read-only, then the **visibility** object contribution might look like this:

```
<objectContribution ...>
	<visibility>
    	<and>
    		<objectClass name="org.eclipse.core.resources.IResource"/>
    		<objectState name="readOnly" value="false"/>
    	</and>
	</visibility>
	... the other stuff here ...
</objectContribution>
```
As part of the `<visibility>` element declaration, you can use nested `<and>`, `<or>`, and `<not>` elements for logical expressions, plus the following Boolean expressions.
- **adapt**—Adapts the selected object to the specified type then uses the new object in any child expressions. For example, if you wanted to adapt the selected object  to a resource and then test some resource object state, the expression would look like this:
```
	<adapt type="org.eclipse.core.resources.IResource">
		<objectState name="readOnly" value="false"/>
	</adapt>
```
The children of an **adapt** expression are combined using the **and** operator. The expression returns EvaluationResult.NOT_LOADED if either the adapter or the type referenced isn’t loaded yet. It throws an **ExpressionException** during evaluation if the type name doesn’t exist.
- **and**—Evaluates true if all subelement expressions evaluate true.
- **instanceof**—Compares the class of the selected object against a name. This is identical to the objectClass element, except that instanceof can be combined with other elements using the and and or elements.
- **not**—Evaluates true if its subelement expressions evaluate false.
- **objectClass**—Compares the class of the selected object against a name as shown above.
- **objectState**—Compares the state of the selected object against a specified state similar to the filter element.
- **or**—Evaluates true if one subelement expression evaluates true.
- **pluginState**—Compares the plug-in state, indicating whether it is installed or activated. For example, an expression such as `<pluginState id="org.eclipse.pde" value="installed"/>` would cause an object contribution to be visible only if the org.eclipse.pde plug-in is installed, and an expression such as `<pluginState id="org.eclipse.pde" value="activated"/>` would cause an object contribution to be visible only if the org.eclipse.pde plug-in has been activated in some other manner.
- **systemProperty**—Compares the system property. For example, if an object contribution should only be visible when the language is English, then the expression would be: ``<systemProperty name="user.language" value="en"/>`
- **systemTest**—Identical to the systemProperty element, except that systemTest can be combined with other elements using the and and or elements.
- **test**—Evaluate the property state of the object. For example, if an object contribution should only be visible when a resource in a Java project is selected, then the expression would be:
```
<test 
	property="org.eclipse.debug.ui.projectNature" 
	value="org.eclipse.jdt.core.javanature"/>
```
The **test** expression returns EvaluationResult.**NOT_LOADED** if the property tester doing the actual testing isn’t loaded yet. The **set** of testable properties can be extended using the org.eclipse.core.expressions.**propertyTesters** extension point. One example of this is the org.eclipse.debug.internal.ui.**ResourceExtender** class.`

##### The **filter** element
The **filter** element is a simpler form of the **objectState** element discussed previously. For example, if the object contribution was to be available for any filethat is not read-only, then the object contribution could be expressed like this:
```
<objectContribution ...>
	<filter name="readOnly" value="false"/>
	... the other stuff here ...
</objectContribution>
```
As with the **objectState** element, the **filter** element uses the **IActionFilter** interface to determine whether an object in the selection matches the criteria. Every selected object must either implement or adapt to the **IActionFilter** interface and implement the appropriate behavior in the testAttribute() method to test the specified name/value pair against the state of the specified object. For resources, Eclipse provides the following built-in state comparisons as listed in the **org.eclipse.ui.IResourceActionFilter** class:
name—Comparison of the filename. “*” can be used at the start or end
to represent “one or more characters.”
extension—Comparison of the file extension.
path—Comparison against the file path. “*” can be used at the start or
end to represent “one or more characters.”
readOnly—Comparison of the read-only attribute of a file.
projectNature—Comparison of the project nature.
persistentProperty—Comparison of a persistent property on the selected
resource. If the value is a simple string, then this tests for the existence of
the property on the resource. If it has the format propertyName=
propertyValue, this obtains the value of the property with the specified
name and tests it for equality with the specified value.







