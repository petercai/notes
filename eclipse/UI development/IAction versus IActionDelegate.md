# Brief
在 Eclipse 中，`IAction` 和 `IActionDelegate` 都是用于处理用户界面中的操作的接口，但它们有着不同的职责和用途：

- **`IAction`**：`IAction` 接口表示可以在用户界面中执行的操作。它提供了设置和获取操作文本、图像、工具提示、启用状态以及执行逻辑的方法。通常情况下，`IAction` 实例用于定义菜单、工具栏或其他 UI 组件中的操作。开发者可以通过子类化 `Action` 或使用其他提供的实现来创建自定义操作。
    
- **`IActionDelegate`**：`IActionDelegate` 接口用于将操作的行为委托给另一个对象。当操作的行为或执行逻辑需要与操作本身分离时，通常会使用 `IActionDelegate`。`IActionDelegate` 实例通常与 `IAction` 实例结合使用，以分离关注点，允许将操作的行为定义在不同的类中。
    

典型的使用方式涉及将 `IActionDelegate` 设置到 `IAction` 实例上，通过 `setActionDelegate(IActionDelegate delegate)` 方法进行关联。然后，`IActionDelegate` 定义了当关联的操作被触发时应该执行的行为。

基本的工作流程如下：

1. `IActionDelegate` 定义操作的行为：它实现了当关联的操作被触发时应该执行的逻辑。
    
2. `IAction` 表示操作本身：它持有与操作在 UI 中的可视化表示相关的属性，并将实际的行为委托给关联的 `IActionDelegate`。
    

这种关注点的分离允许更模块化和灵活的代码编写，因为操作的行为可以通过简单地更换关联的委托来动态地改变，而无需修改操作本身。

总而言之，`IAction` 负责在 UI 中表示操作，而 `IActionDelegate` 则定义了操作在触发时应该执行的行为。

当使用 Eclipse 平台开发插件时，你可以使用 `IAction` 和 `IActionDelegate` 来创建和执行自定义的操作。

举个例子，假设你正在开发一个插件，需要在工具栏或菜单中添加一个按钮，点击该按钮时会执行某项操作，比如打开一个新窗口或执行特定的任务。

首先，你会创建一个实现了 `IActionDelegate` 接口的类，用于定义该操作的具体行为：

```
public class CustomActionDelegate implements IActionDelegate {      
	
	@Override 
	public void run(IAction action) {         
		// 执行你想要的操作         
		// 例如打开一个新窗口、执行特定任务等         
		System.out.println("Custom action executed!");     
	}      
	
	@Override     
	public void selectionChanged(IAction action, ISelection selection) {         
		// 在此方法中处理与选择相关的变化     
	} 
}
```


然后，在另一个地方，你会创建一个实现了 `IAction` 接口的类，用于表示并展示这个操作：
```
import org.eclipse.jface.action.Action;  
public class CustomAction extends Action {      
	public CustomAction() {         
		// 设置操作的文本、图标、工具提示等信息         
		setText("Custom Action");         
		setToolTipText("Execute custom action");         
		// 设置与该操作关联的委托         
		setActionDelegate(new CustomActionDelegate());     
	} 
}
```

在上面的例子中，`CustomAction` 类表示一个自定义操作，它设置了该操作的文本、工具提示等信息，并将一个 `CustomActionDelegate` 的实例设置为该操作的委托。当用户触发这个操作时，委托中的 `run()` 方法会被调用，执行相应的逻辑（在这个例子中，输出 "Custom action executed!" 到控制台）。

在 Eclipse 插件开发中，通常会将这些操作添加到工具栏、菜单或视图中，以便用户可以方便地执行它们。

# IAction versus IActionDelegate
An Eclipse action is composed of several parts, including the XML declaration of the action in the plug-in’s manifest, the **IAction** object <span style="background:#fff88f">instantiated by the Eclipse UI to represent the action</span>, and the **IActionDelegate** <span style="background:#fff88f">defined in the plug-in library containing the code to perform the action</span>:
![[IAction versus IActionDelegate-20231225-1.png]]
This separation of the **IAction** object, defined and instantiated by theEclipse user interface based on the plug-in’s manifest and the **IActionDelegate** defined in the plug-in’s library, allows Eclipse to represent the action in a menu or toolbar without loading the plug-in that contains the operation until the user selects a specific menu item or clicks on the toolbar.
Again, this approach represents one of the overarching themes of Eclipse: lazy plug-in initialization.
There are several interesting subtypes of **IActionDelegate**.
- **IActionDelegate2**—Provides lifecycle events to action delegates; if you are implementing **IActionDelegate** and need additional information, such as when to clean up before the action delegate is disposed, then implement **IActionDelegate2** instead. In addition, an action delegate implementing **IActionDelegate2** will have **runWithEvent(IAction, Event)** called instead of **run(IAction)**.
- **IEditorActionDelegate**—Provides lifecycle events to action delegates associated with an editor .
- **IObjectActionDelegate**—Provides lifecycle events to action delegates<span style="background:#fff88f"> associated with a context menu </span>.
- **IViewActionDelegate**—Provides lifecycle events to action delegates associated with a view.
- **IWorkbenchWindowActionDelegate**—Provides lifecycle events to action delegates associated with the workbench window menu bar or toolbar.

# Workbench Window Actions
Where and when an action appears is dependent on the extension point and filter used to define the action. This section discusses adding a new menu to the <span style="background:#fff88f">workbench menu bar</span> and a new button to the workbench toolbar using the **org.eclipse.ui.actionSets extension point**. These actions are very similar to menu contributions with a locationURI id  equal to “org.eclipse.ui.main.menu”.

## Defining a workbench window menu

## Groups in a menu
Actions are added not to the menu itself, but to groups within the menu. A **separator** group has a horizontal line above the first menu item in the group, whereas a **groupMarker** does not have any line.

## Defining a menu item and toolbar button


## Action images

## Insertion points
Because Eclipse is composed of multiple plug-ins—each one capable of contributing actions but not necessarily knowing about one another at build time—the absolute position of an action or submenu within its parent is not known until runtime. Even during the course of execution, the position might change due to a sibling action being added or removed as the user changes a selection. For this reason, Eclipse uses identifiers to reference a menu, group,
or action, and a path, known as an insertion point, for specifying where a menu or action will appear.
Every insertion point is composed of one or two identifiers separated by a **forward slash,** indicating the parent (a menu in this case) and group where the action will be located. 

Typically, plug-ins make allowances for other plug-ins to add new actions to their own menus by defining an empty group labeled “**additions**” in which the new actions will appear. The “additions” identifier is fairly standard throughout Eclipse, indicating where new actions or menus will appear, and is included in it as the **IWorkbenchActionConstants.MB_ADDITIONS** constant. 

## Creating an action delegate

### selectionChanged method
While the action declaration in the plug-in manifest provides the initial state of the action, the selectionChanged() method in the action delegate provides an opportunity to dynamically adjust the state, enablement, or even the text of the action using the **IAction** interface.
In order for the action delegate’s **selectionChanged**() method to be called, you need to call **getViewSite().setSelectionProvider(viewer)** in your view’s **createPartControl**() method.

### run method
The **run**() method is called when a user selects an action and expects an operation to be performed. Similar to the **selectionChanged**() method, the **IAction** interface can be used to change the state of an action dependent on the outcome of an operation.

# Object Actions
Suppose you want to make it easy for the user to add files and folders to the your view. **Object contributions** are ideal for this because they appear in context menus only when the selection in the current view or editor contains an object compatible with that action. In this manner, an **object contribution** is available to the user when he or she needs the action, yet not intrusive when the action does not apply. Object actions are very similar to menu contributions that have a locationURI id equal to “org.eclipse.ui.any.popup.”

## Defining an object-based action
Defining a workbench window menu use the Extensions page of the plug-in manifest editor to create the new object contribution. Click on the Add button to add an **org.eclipse.ui.popupMenus extension**, then add an **objectContribution** with the following attributes:
- **adaptable**—“true”: Indicates that objects that adapt to **IResource** are acceptable targets.
- **id**—“com.qualityeclipse.favorites.popupMenu”: The unique identifier for this contribution.
- **nameFilter**—Leave blank: A wildcard filter specifying the names that are acceptable targets. For example, entering  `*`.java would target only those files with names ending with .java..
- **objectClass**—“org.eclipse.core.resources.IResource”: The type of object that is an acceptable target. Use the Browse... button at the right of the objectClass field to select the existing **org.eclipse.core.resources.IResource** class. If you want to create a new class, then click the objectClass: label to the left of the objectClass field.
Next, add an action to the new **objectContribution** with the following attribute values.
![[IAction versus IActionDelegate-20231225-2.png]]

## Action filtering and enablement
In keeping with the lazy loading plug-in theme, Eclipse provides multiple declarative mechanisms for filtering actions based on the context, and enabling visible actions only when appropriate. Because they are declared in the plug-in manifest, these mechanisms have the advantage that they do not require the plug-in to be loaded for Eclipse to use them.
### Basic filtering and enablement
The **enablesFor** attribute determines when an action will be enabled. When the context menu
is activated, if a selection does not contain objects with names that match the wildcard **nameFilter** or are not of a type specified by the **objectClass** attribute, none of the actions defined in that object contribution will appear in the context menu. In addition, the  **enablesFor** attribute uses the syntax below to define exactly how many objects need to be selected for a particular action to be enabled: 
![[IAction versus IActionDelegate-20231225-3.png]]
The techniques listed in this table represent those most commonly used for limiting **visibility** and the **enablement** of actions; occasionally, a more refined approach is needed. The **visibility** and **filter** elements provide an additional means to limit an action’s visibility, while the **selection** and **enablement** elements provide a more flexible way to specify when an action is enabled.
Still further refinement of action enablement can be provided by using the **selectionChanged**() method in the action delegate.

### The visibility element
The visibility element provides an alternate and more powerful way to specify when an object contribution’s actions will be available to the user as compared with the object contribution’s nameFilter and objectClass. For example, an alternate way to specify filtering for the object contribution just described would be:

```
<objectContribution ...>
	<visibility>
		<objectClass name="org.eclipse.core.resources.IResource"/>
	</visibility>
	...the other stuff here...
</objectContribution>
```

If the action is to be visible only for resources that are not read-only, then the visibility object contribution might look like this:

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
- adapt—Adapts the selected object to the specified type then uses the new object in any child expressions. For example, if you wanted to adapt the selected object to a resource and then test some resource object state, the expression would look like this:

```
<adapt type="org.eclipse.core.resources.IResource">
	<objectState name="readOnly" value="false"/>
</adapt>
```




