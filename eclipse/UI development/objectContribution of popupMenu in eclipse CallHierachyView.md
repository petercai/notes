

# option 1

In Eclipse, the Call Hierarchy View doesn't directly offer an interface model equivalent to `IStackFrame` from the Debug view. However, you can achieve similar functionality by utilizing the `org.eclipse.jdt.ui.actions.OpenCallHierarchyAction` class.

Here's how you can create an object contribution for a popup menu in Eclipse that operates on the Call Hierarchy View:

```java
import org.eclipse.core.commands.AbstractHandler;
import org.eclipse.core.commands.ExecutionEvent;
import org.eclipse.core.commands.ExecutionException;
import org.eclipse.jdt.ui.actions.OpenCallHierarchyAction;
import org.eclipse.jface.viewers.IStructuredSelection;
import org.eclipse.jface.viewers.StructuredSelection;
import org.eclipse.ui.handlers.HandlerUtil;

public class OpenCallHierarchyHandler extends AbstractHandler {

    @Override
    public Object execute(ExecutionEvent event) throws ExecutionException {
        IStructuredSelection selection = HandlerUtil.getCurrentStructuredSelection(event);
        if (selection != null && !selection.isEmpty()) {
            OpenCallHierarchyAction action = new OpenCallHierarchyAction(HandlerUtil.getActiveWorkbenchWindow(event));
            action.run(new StructuredSelection(selection.getFirstElement()));
        }
        return null;
    }
}
```

And then, you can contribute this handler to the popup menu in your Eclipse plugin's `plugin.xml` file:

```xml
<extension point="org.eclipse.ui.popupMenus">
   <objectContribution
         id="com.example.OpenCallHierarchyObjectContribution"
         objectClass="org.eclipse.jdt.core.IJavaElement"
         adaptable="true"
         nameFilter="*">
      <menu
            label="Open Call Hierarchy"
            path="org.eclipse.jdt.ui.PackageExplorer#PopupMenu?after=additions">
         <command
               commandId="com.example.openCallHierarchyCommand"
               mnemonic="O"
               style="push">
         </command>
      </menu>
   </objectContribution>
   <command
         id="com.example.openCallHierarchyCommand"
         commandId="com.example.openCallHierarchyCommand"
         style="org.eclipse.ui.commands.CommandStyle"
         disabledIcon="icons/sample.gif"
         helpContextId="org.eclipse.ui.ui.workbenchWindow.help"
         label="Open Call Hierarchy"
         tooltip="Open Call Hierarchy">
      <commandParameter
            id="org.eclipse.ui.commands.toggleState">
      </commandParameter>
   </command>
   <handler
         class="com.example.OpenCallHierarchyHandler"
         commandId="com.example.openCallHierarchyCommand">
   </handler>
</extension>
```

In this example, when the user selects an element in the Package Explorer or any other view that provides `IJavaElement` instances, they can right-click and select "Open Call Hierarchy" from the context menu. This will then open the Call Hierarchy View with the selected element as the input.



# option 2

To add an object contribution popup menu action inside the Call Hierarchy View in Eclipse, you would typically extend the `org.eclipse.ui.popupMenus` extension point and target the Call Hierarchy View's context menu ID. Here's how you can achieve this:

1. **Create a Handler for your action**:
   First, create a handler for the action you want to perform when the menu item is clicked. This handler should extend `org.eclipse.core.commands.AbstractHandler` and override the `execute` method to define the action to be performed.

```java
import org.eclipse.core.commands.AbstractHandler;
import org.eclipse.core.commands.ExecutionEvent;
import org.eclipse.core.commands.ExecutionException;

public class MyCallHierarchyActionHandler extends AbstractHandler {
    @Override
    public Object execute(ExecutionEvent event) throws ExecutionException {
        // Your action code goes here
        return null;
    }
}
```

2. **Define the popup menu contribution in plugin.xml**:
   In your plugin's `plugin.xml`, you need to define the object contribution for the Call Hierarchy View's context menu.

```xml
<extension point="org.eclipse.ui.popupMenus">
   <objectContribution
         id="com.example.callHierarchyObjectContribution"
         objectClass="org.eclipse.jdt.internal.ui.callhierarchy.CallHierarchyViewPart"
         adaptable="false"
         nameFilter="*">
      <menu
            id="myContextMenuId"
            label="My Context Menu"
            path="additions">
         <command
               commandId="com.example.myCallHierarchyActionCommand"
               mnemonic="M"
               style="push">
         </command>
      </menu>
   </objectContribution>
</extension>
```

3. **Define the command and its associated parameters**:
   Define the command and its associated parameters in the plugin.xml.

```xml
<extension point="org.eclipse.ui.commands">
   <command
         id="com.example.myCallHierarchyActionCommand"
         name="My Call Hierarchy Action">
   </command>
</extension>
```

4. **Associate the command with the handler**:
   Associate the command with the handler you created earlier.

```xml
<extension point="org.eclipse.ui.handlers">
   <handler
         class="com.example.MyCallHierarchyActionHandler"
         commandId="com.example.myCallHierarchyActionCommand">
   </handler>
</extension>
```

With these steps, your popup menu action should now be added inside the Call Hierarchy View in Eclipse. You can replace `"My Call Hierarchy Action"` and `MyCallHierarchyActionHandler` with your actual action name and handler class.