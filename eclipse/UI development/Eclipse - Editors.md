
# Editors

Editors are the primary mechanism for users to create and modify resources (e.g., files).
Editors must implement the org.eclipse.ui.**IEditorPart** interface. Typically, editors are subclasses of org.eclipse.ui.part.**EditorPart** and thus indirectly subclasses of org.eclipse.ui.part.**WorkbenchPart**, inheriting much of the behavior needed to implement the IEditorPart interface.

![[Editors-20231125-1.png]]

Editors are contained in an org.eclipse.ui.**IEditorSite**, which in turn is contained in an org.eclipse.ui.**IWorkbenchPage**. In the spirit of lazy initialization, **IWorkbenchPage** holds on to instances of org.eclipse.ui.**IEditorReference** rather than the editor itself so that editors can be enumerated and referenced without actually loading the plug-in defining the editor.


![[Editors-20231125-2.png]]

**class** —“com.qualityeclipse.favorites.editors.PropertiesEditor”: The fully qualified name of the class defining the editor and implementing org.eclipse.ui.**IEditorPart** . Click the Browse… button to the right of the field to open a dialog and select an existing editor part. Click the class label on the left to generate a new one. The attribute’s class, command, and launcher are mutually exclusive. The class is instantiated using its no argument constructor, but can be parameterized using the **IExecutableExtension** interface.
**contributorClass**—“com.qualityeclipse.favorites.editors.PropertiesEditorContributor”: The fully qualified name of a class that implements org.eclipse.ui.IEditorActionBarContributor and adds new actions to the workbench menu and toolbar, reflecting the features of the editor type. This attribute should only be defined if the class attribute is defined. Click the Browse…
button to the right of the field to open a dialog for selecting an existing editor contributor. Click the contributorClass on the left to generate a new one.
**extensions**—“properties”: A string of comma-separated file extensions indicating file types understood by the editor.
**icon**—“icons/sample.gif”: The image displayed at the upper left corner of the editor. Similar to action images, this path is relative to the plug-in’s installation directory.
**id**—“com.qualityeclipse.properties.editor”: The unique identifier for this editor.
**name**—“Properties Editor”: The human-readable name for the editor.

Other attributes that are not used in this example include:
**command**—A command to run to launch an external editor. The executable command must be located on the system path or in the plug-in’s directory. The attribute’s class, command, and launcher are mutually exclusive. default—“true” or “false” (blank = false)
If true, this editor will be used as the default editor for the type. This is only relevant in the case where more than one editor is registered for the same type. If an editor is not the default for the type, it can still be launched using the Open with... submenu for the selected resource.
**filenames**—A string containing comma-separated filenames indicating filenames understood by the editor. For instance, an editor that understands plug-in and fragment manifest files can register plugin.xml, fragment.xml.
**launcher**—The name of a class that implements org.eclipse.ui.IEditorLauncher and opens an external editor. The attribute’s class, command, and launcher are mutually exclusive.
**matchingStrategy**—the name of a class that implements org.eclipse.ui.IEditorMatchingStrategy. This attribute should only be defined if the class attribute is defined and allows the editor extension to provide its own algorithm for matching the input of one of its editors to a given editor input. This is used to find a matching editor duringopenEditor() and findEditor().

# EditorPart

The code defining the editor’s behavior is found in a class implementing the org.eclipse.ui.IEditorPart interface, typically by subclassing one of the following concrete classes:
• org.eclipse.ui.part.**EditorPart**—Abstract base implementation of the org.eclipse.ui.IEditorPart interface
• org.eclipse.ui.part.**MultiPageEditorPart**—Abstract base implementation extending EditorPart for multi-page editors.
• org.eclipse.ui.forms.editor.**FormEditor**—Abstract base implementation extending MultiPageEditorPart for multi-page editors that typically use one or more pages with forms and one page for raw source of the editor input.
The Properties editor subclasses MultiPageEditorPart and provides two pages for the user to edit its content.

## Editor methods

### Methods of EditorPart

**createPartControl**(Composite)—Creates the controls comprising the editor. Typically, this method simply calls more finely grained methods such as createTree, createTextEditor, and so on.
**dispose**()—This method is automatically called when the editor is closed and marks the end of the editor’s lifecycle. It cleans up any platform resources, such as images, clipboard, and so on, which were created by this class. This follows the if you create it, you destroy it theme that runs throughout Eclipse.
**doSave**(IProgressMonitor)—Saves the contents of this editor. If the save is successful, the part should fire a property changed event (PROP_DIRTY property), reflecting the new dirty state. If the save is canceled via user action, or for any other reason, the part should
invoke setCanceled on the IProgressMonitor to inform the caller.
**doSaveAs**()—This method is optional. It opens a Save As dialog and saves the content of the editor to a new location. If the save is successful, the part should fire a property changed event (PROP_DIRTY property),reflecting the new dirty state.
**gotoMarker**(IMarker)—Sets the cursor and selection state for this editor as specified by the given marker.
**init**(IEditorSite, IEditorInput)—Initializes this editor with the given editor site and input. This method is automatically called shortly after editor construction; it marks the start of the editor’s lifecycle.
**isDirty**()—Returns whether the contents of this editor have changed since the last save operation.
**isSaveAsAllowed**()—Returns whether the “Save As” operation is supported by this part.
**setFocus**()—Asks this part to take focus within the workbench. Typically, this method simply calls setFocus() on one of its child controls.

### Methods of MultiPageEditorPart

**addPage**(Control)—Creates and adds a new page containing the given control to this multipage editor. The control may be null, allowing it to be created and set later using setControl.
**addPage**(IEditorPart, IEditorInput)—Creates and adds a new page containing the given editor to this multipage editor. This also hooks a property change listener onto the nested editor.
**createPages**()—Creates the pages of this multipage editor. Typically,this method simply calls more finely grained methods such as createPropertiesPage, createSourcePage, and so on.
**getContainer**()—Returns the composite control containing this multipage editor’s pages. This should be used as the parent when creating controls for individual pages. That is, when calling addPage(Control), the passed control should be a child of this container.
**setPageImage**(int, Image)—Sets the image for the page with the given index.
**setPageText**(int, String)—Sets the text label for the page with the given index.

## Editor controls


## Editor model


## Content Provider
All these model objects are useless until they can be properly displayed in the tree. To accomplish this, you need to create a **content provider** and label provider. The content provider provides the rows appearing in the tree along with parent/child relationships, but not the actual cell content.

# Editing


# Editor Lifecycle

Typical editors go through an **open-modify-save-close** lifecycle. When the editor is opened, the **init**(IEditorSite, IEditorInput) method is called to set the editor’s initial content. When the user modifies the editor’s content, the editor must notify others that its content is now “dirty” by using the **firePropertyChange**(int) method. When a user saves the editor’s content,
the firePropertyChange(int) method must be used again to notify registered listeners that the editor’s content is no longer dirty. Eclipse automatically registers listeners to perform various tasks based on the value returned by the **isDirty**() method, such as updating the editor’s title, adding or removing an asterisk preceding the title, and enabling the Save menu. Finally, when the editor is closed, the editor’s content is saved if the isDirty() method returns true.

# Editor commands
Editor commands can appear as menu items in the editor’s **context menu**, as toolbar buttons in the **workbench’s toolbar**, and as menu items in the **workbench’s menu**.

## Context menu


## Editor contributor
An editor contributor manages the **installation** and **removal** of g**lobal menus**, **menu items**, and **toolbar buttons** for one or more editors. The need for an editor contributor is greatly reduced or eliminated by using commands instead.

### IEditorActionBarContributor

An instance of org.eclipse.ui.**IEditorActionBarContributor** is associated with one or more editors, adding or removing top level menus and toolbar elements. The plug-in manifest specifies which editor contributor, typically a subclass of org.eclipse.ui.part.**EditorActionBarContributor** or org.eclipse.ui.part.**MultiPageEditorActionBarContributor**, is
associated with which editor type. The platform then sends the following events to the contributor, indicating when an editor has become active or inactive, so that the contributor can install or remove menus and buttons as appropriate.

**dispose**()—This method is automatically called when the contributor is no longer needed. It cleans up any platform resources, such as images, clipboard, and so on, which were created by this class. This follows the if you create it, you destroy it theme that runs throughout Eclipse.
**init**(IActionBars, IWorkbenchPage)—This method is called when the contributor is first created.
**setActiveEditor**(IEditorPart)—This method is called when an associated editor becomes active or inactive. The contributor should insert and remove menus and toolbar buttons as appropriate.

### EditorActionBarContributor
The **EditorActionBarContributor** class implements the **IEditorActionBarContributor** interface, caches the action bar and workbench page, and provides two new accessor methods.
**getActionBars**()—Returns the contributor’s action bars provided to the contributor when it was initialized.
**getPage**()—Returns the contributor’s workbench page provided to the contributor when it was initialized.

### MultiPageEditorActionBarContributor

The **MultiPageEditorActionBarContributor** class extends **EditorActionBarContributor**, providing a new method to override instead of the setActiveEditor(IEditorPart) method.
**setActivePage**(IEditorPart)—Sets the active page of the multipage editor to the given editor. If there is no active page, or if the active page does not have a corresponding editor, the argument is null.
## Global actions
By borrowing from org.eclipse.ui.editors.text.TextEditorActionContributor and
org.eclipse.ui.texteditor.BasicTextEditorActionContributor, you will create your own contributor for the Editor. This contributor hooks up global actions (e.g., cut, copy, paste, etc. in the Edit menu) appropriate not only to the active editor but also to the active page within the editor.

### Top-level menu
Instead of referencing the action directly as done with the context menu , you will use an instance of org.eclipse.ui.actions.**RetargetAction**, which references the remove action indirectly via its identifier. You’ll be using the ActionFactory.DELETE.getId() identifier, but could use any identifier so long as setGlobalActionHandler(String, IAction) is used to associate the identifier with the action.

## Editor commands rather than editor contributor
The commands API is the preferred over actions, and reduces or eliminates the need for an editor contributor.  You can add a top level menu item or top level tool bar  and limit its visibility  so that it appears only when your editor is active. 

## Global commands
To provide support for the Edit > Delete command, hook **DeletePropertiesHandler** to the global menu item using the following plug-in declaration.
```
<handler
	class="com.qualityeclipse.favorites.handlers.DeletePropertiesHandler"
	commandId="org.eclipse.ui.edit.delete">
	<activeWhen>
		<with variable="activeEditorId">
			<equals
				value="com.qualityeclipse.properties.editor">
			</equals>
		</with>
	</activeWhen>
</handler>
```

# Undo/Redo
Adding the capability for a user to **undo and redo** commands involves separating user edits into commands visible in the user interface and the underlying operations that can be executed, undone, and redone. Typically each command will instantiate a new operation every time the user triggers that command. The command gathers the current application state, such as the currently selected elements, and the operation caches that state so that it can be executed, undone, and redone independent of the original command. An instance of **IOperationHistory** manages the operations in the global undo/redo stack. Each operation uses one or more associated undo/redo contexts to keep operations for one part separate from operations for another.
![[Editors-20231125-3.png]]

# Linking the Editor
The selection in the active editor can be linked to the views that surround it in a technique similar to linking view selections. In addition, the editor can provide content for the Outline
view by implementing the getAdapter(Class) method.


```
private PropertiesOutlinePage outlinePage;

public Object getAdapter(Class adapter) {
	if (adapter.equals(IContentOutlinePage.class)) {
		if (outlinePage == null)
			outlinePage = new PropertiesOutlinePage();
		return outlinePage;
	}
	return super.getAdapter(adapter);
}
```













