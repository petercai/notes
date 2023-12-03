# Graphical Editing Framework

## GEF Architecture
The GEF framework is designed using the Model-View-Controller (MVC) architecture. MVC
is comprised of three major components: the model, the view and the controller. The model holds the information to be displayed and is persisted across sessions. The view renders information on the screen and provides basic user interaction. The controller coordinates the activities of the model and the view, passing information between them as necessary.
**Model <—> Controller <—> View**
When an object is added to the GEF model, a corresponding controller is instantiated which then creates the view objects representing that model object. In the reverse direction, when the user interacts with the view, the controller updates the model with the new information.
- **model**—the underlying objects being represented graphically 
- **view**—a GEF canvas containing a collection of figures
- **controller**—a collection of GEF edit parts

GEF provides various edit part (controller) and figure (view) classes for you to extend, minimizing the effort necessary to use the graphical framework in your application.

The GEF project is made up of three subsections:
• **Draw2D**—(org.eclipse.draw2d.*) a lightweight framework layered on top of SWT for rendering graphical information.
• **GEF Framework**—(org.eclipse.gef.*) an MVC framework layered on top of Draw2D to facilitate user interaction with graphical information.
• **Zest**—(org.eclipse.zest.*) a graphing framework layered on top of Draw2D for graphing. 

## GEF Model
First, the model is responsible for providing all data that will be viewable and modifiable by the user; no data should be stored in the controller or view. Next, the model needs to provide a way for the data to be persisted across sessions. Finally, while the controller will have references to the model, the model should not reference the controller or view.
Instead, <u>the controller registers itself with the model as a listener so that model changes are communicated to the controller using the observer pattern.</u>

## GEF Controller
The GEF controller is <u>a collection of edit parts</u>. As<u> new objects are added to the model, an edit part factory creates edit parts corresponding to the new model objects</u>. Each <u>edit part is responsible for creating figures representing its model objects and giving itself characteristics by overriding methods which declare</u>:
• child edit parts
• connections
• figures
• policies

### EditPart classes
GEF provides various edit part classes for you to extend depending on the desired visual representation. **AbstractGraphicalEditPart** is subclassed for parts that will represent graphical data such as a figure, or collection of figures, while **AbstractConnectionEditPart** for parts that represent a connection between two figures. **AbstractEditPart** is a superclass for both of these edit parts.

#### AbstractEditPart
**activate**()—Called when the EditPart becomes an element in an active hierarchy of edit parts whose figures are displayed on a canvas.
**addEditPartListener**(EditPartListener listener)—Adds a listener to the EditPart.
**createEditPolicies**()—Creates the initial edit policies and/or reserves slots for dynamic ones.
**deactivate**()—Called when the EditPart is no longer an element in an active hierarchy of edit parts (see activate).
**getChildren**()—Returns the List of child EditParts. This method should only be called internally or by helpers such as edit policies.
**getCommand**(Request request)—Returns the GEF Command to perform the specified Request or null.
**getModel**()—Returns the primary model object that this EditPart represents. EditParts may correspond to more than one model object, or even no model object. In practice, the Object returned is used by other EditParts to identify this EditPart. In addition, edit policies probably
rely on this method to build Commands that operate on the model.
**getModelChildren**()—Returns a List containing the child model objects. If this EditPart’s model is a container, this method should be overridden to return its children. refresh() uses this to create child EditParts.
**getParent**()—Returns the parent EditPart. This method should only be called internally or by helpers such as edit policies.
**installEditPolicy**()—Installs an EditPolicy for a specified role. For example, use EditPolicy.LAYOUT_ROLE to install your own layout edit policy in your EditPart. Specify a null value to reserving a location.
**refresh**()—Refresh all properties visually displayed by this EditPart.
**refreshChildren**()—Updates the set of child EditParts so that it is in sync with the set of model children. Called from refresh(), and mayalso be called in response to notifications from the model.
**refreshVisuals**()—This method is called by refresh(), and may also be called in response to notifications from the model. This method does nothing by default and may be overridden by subclasses.
**removeEditPartListener**(EditPartListener listener)— Removes the first occurrence of the specified listener from the list of listeners. Does nothing if the listener is not present.

#### AbstractGraphicalEditPart
**AbstractGraphicalEditPart**, a subclass of **AbstractEditPart**, provides additional methods to handle the figure and connections:
**createFigure**()—Creates the Figure to be used as this part’s visuals. This is called from getFigure() if the figure has not been created.
**getFigure**()—Returns the Figure to be used as this part's visuals. Calls createFigure() if the figure is currently null.
**getModelSourceConnections**()—Returns the List of the connection model objects for which this EditPart’s model is the source. **refreshSourceConnections**() calls this method. For each connection model object, **createConnection**(Object) will be called automatically to obtain a corresponding ConnectionEditPart.
**getModelTargetConnections**()—Returns the List of the connection model objects for which this EditPart’s model is the target. **refreshTargetConnections**() calls this method. For each connection model object, **createConnection**(Object) will be called automatically to obtain a corresponding ConnectionEditPart.
**setLayoutConstraint**(EditPart, IFigure, Object)—Sets the specified constraint for a child’s Figure on content pane figure for this GraphicalEditPart.

#### Top Level EditPart
As a rule of thumb, each model object to be shown on the GEF canvas has a corresponding edit part.

#### Connection EditParts
There are edit parts to represent all the figures, except the connection figures between the edit parts to the resource parts. In order to create the connection edit parts, the existing edit parts need to have some model object to pass back to GEF to notify the framework that the connections exist. For the model, all the information needed to draw the connections already exists in the model, but there is no model object that represents a connection. Instead of modifying the model to include a new model object, we create a new transient object representing connections between model objects.

#### EditPartFactory
Every GEF canvas has exactly one edit part factory. When objects are added to the model, GEF uses the associated edit part factory to create edit parts corresponding to the new model objects.

## GEF Figures
The GEF canvas, an instance of **FigureCanvas**, contains a collection of figures that visually represent the underlying model. It is the responsibility of each edit part to create and manage figures representing the model object associated with that edit part.

### IFigure
Each figure in the GEF canvas must implement the IFigure interface. The Graphics class, the IFigure interface and many of the concrete figure classes are defined in the Draw2D part of GEF. Some of the useful methods declared by IFigure include:
**add**(IFigure)—Adds the given IFigure as a child of this IFigure, similar to calling add(IFigure, Object, int).
**containsPoint**(int, int)—Returns true if the point (x, y) is contained within this IFigure’s bounds.
**getBackgroundColor**()—Returns the background color.
**getBounds**()—Returns the smallest rectangle completely enclosing the figure. Callers of this method must not modify the returned Rectangle.
**getClientArea**()—Returns the rectangular area within this Figure's bounds in which children will be placed (via LayoutManagers) and to which the painting of child figures will be clipped.
**getForegroundColor**()—Returns the foreground color.
**getParent**()—Returns the IFigure which contains this IFigure or null if there is no parent.
**getToolTip**()—Returns an IFigure that is the tooltip for this IFigure.
**hasFocus**()—Returns true if this IFigure has focus.
**isOpaque**()—Returns true if this IFigure is opaque, and false if this figure is transparent.
**isVisible**()—Returns true if this figure’s visibility flag is set to true, without recursively checking the visiblity of the figure’s parent.
**paint**(Graphics)—Paints this IFigure and its children.
**setBackgroundColor**(Color)—Sets this figure’s background color.
**setConstraint**(IFigure, Object)—Convenience method to set the constraint of the specified child in the current LayoutManager.
**setForegroundColor**(Color)—Sets this figure’s foreground color.
**setLayoutManager**(LayoutManager)—Sets the LayoutManager.
**setOpaque**(boolean)—Sets this IFigure to be opaque if the argument is true or transparent if the argument is false.
**setToolTip**(IFigure)—Sets a tooltip that is displayed when the mouse hovers over this IFigure.
**setVisible**(boolean)—Sets this IFigure’s visibility.


### Graphics
When rendering a figure, the Graphics class provides many drawing primi-
tives for things such as displaying text, drawing lines, and drawing and filling
polygons. Some of the useful Graphics methods include:
drawFocus(Rectangle)—Draws a focus rectangle.
drawImage(Image, Point)—Draws the given image at a point.
drawLine(Point, Point)—Draws a line between the two specified
points using the current foreground color.
drawPolygon(PointList)—Draws a closed polygon defined by the
given PointList containing the vertices. The first and last points in the
list will be connected.
drawRectangle(Rectangle)—Draws the given rectangle using the cur-
rent foreground color.
drawRoundRectangle(Rectangle, int, int)—Draws a rectangle
with rounded corners using the foreground color. The last two argu-
ments represent the horizontal and vertical diameter of the corners.
drawText(String, Point)—Draws the given string at the specified
location using the current font and foreground color. The background of
the text will be transparent. Tab expansion and carriage return process-
ing are performed.
fillPolygon(PointList)—Fills a closed polygon defined by the given
PointList containing the vertices. The first and last points in the list
will be connected.
fillRectangle(Rectangle)—Fills the given rectangle using the cur-
rent background color.
fillRoundRectangle(Rectangle, int, int)—Fills a rectangle with
rounded corners using the background color. The last two arguments
represent the horizontal and vertical diameter of the corners.
fillText(String, Point)—Draws the given string at the specified
location using the current font and foreground color. The background of
the text will be filled with the current background color. Tab expansion
and carriage return processing are performed.
getBackgroundColor()—Returns the background color.
getForegroundColor()—Returns the foreground color.
getLineStyle()—Returns the line style.
getLineWidth()—Returns the current line width.
rotate(float)—Rotates the coordinates by the given counter-clock-
wise angle. All subsequent painting will be performed in the resulting
coordinates. Some functions are illegal when a rotated coordinates sys-
tem is in use. To restore access to those functions, it is necessary to call
restore() or pop() to return to a non rotated state.
setBackgroundColor(Color)—Sets the background color.
setBackgroundPattern(Pattern)—Sets the pattern used for fill-type
graphics operations. The pattern must not be disposed while it is refer-
enced by the Graphics object.
setForegroundColor(Color)—Sets the foreground color.
setForegroundPattern(Pattern)—Sets the foreground pattern for
draw and text operations. The pattern must not be disposed while it is
referenced by the Graphics object.
setLineStyle(int)—Sets the line style to the argument, which must be
one of the constants SWT.LINE_SOLID, SWT.LINE_DASH, SWT.LINE_DOT,
SWT.LINE_DASHDOT or SWT.LINE_DASHDOTDOT.
setLineWidth(int)—Sets the line width.
translate(Point)—Translates the receiver’s coordinates by the speci-
fied x and y amounts. All subsequent painting will be performed in the
resulting coordinate system. Integer translation used by itself does not
require or start the use of the advanced graphics system in SWT. It is
emulated until advanced graphics are triggered.

GEF provides a number of basic figure classes that can be extended or com-
posed to build more complex figures. These classes include:
Button—A figure typically having a border that appears to move up and
down in response to being pressed.
Ellipse—A figure that draws an ellipse filling its bounds.
ImageFigure—A figure containing an image. Use this Figure, instead of
a Label, when displaying Images without any accompanying text.
Label—A figure that can display text and/or an image (see Section
20.4.3.1, Composing Figures, on page 747).
Panel—A figure container which is opaque by default.
Polygon—A figure similar to a Polyline except it is closed and filled.
Polyline—A figure rendered as a series of line segments.
PolylineConnection—A figure used to visually connect other figures.
RectangleFigure—A rectangular figure with square corners.
RoundedRectangle—A rectangular figure with round corners (see Sec-
tion 20.4.3.1, Composing Figures, on page 747).
TextFlow—A figure for displaying a string of text across multiple lines.
Thumbnail—A figure displaying an image at reduced size with the same
aspect ratio as the original figure.
Triangle—A figure rendering itself as a triangle.

### Complex Figures
Figures must be created for each of the edit parts. One approach is to build complex figures by composing simpler figures without creating any custom figures. In other cases, it is more useful to create a specialized figure class for each edit part class to be rendered on the screen. 
#### Composing Figures
Figures can have child figures, making it possible to build complex figures by assembling simpler ones. Edit parts can directly compose figures by using **IFigure**’s **add**(IFigure) method. In addition, an edit part’s figure is automatically a child of that edit part’s parent figure.

#### Custom Figures

#### Tooltips

#### Connection Figures
By default, connections are displayed as thin solid lines without any decorators at the ends. You can change the line’s width, style, fill and endpoint decoration. To display connections as arrows pointing from the favorites item to the resource, add the following method to FavoriteConnectionEditPart:
```
protected IFigure createFigure() {
	PolylineConnection connection =
			(PolylineConnection) super.createFigure();
	connection.setTargetDecoration(new PolygonDecoration());
	return connection;
}
```


#### LayoutManager
Any figure which has children must declare a LayoutManager, which is responsible for positioning and sizing the child figures. GEF provides many general purpose layout managers such as
• BorderLayout
• FreeformLayout
• FlowLayout
• GridLayout
• StackLayout

## GEF in an Eclipse View

At the highest level, GEF provides the **ScrollingGraphicalViewer** class, a type of edit part, for creating and managing a GEF canvas. Using the ScrollingGraphicalViewer.**createControl**() method, the canvas can be added to any SWT Composite.

The root edit part for graphical data is either **ScalableRootEditPart** or **ScalableFreeformRootEditPart**. Both provide the same functionality for drawing, scrolling and zooming, but the **freeform** version allows figures to have negative coordinates and is the most commonly used root edit part. These edit parts manage a series of layers within the GEF canvas, each layer having a different purpose. One layer contains all of the favorites figures while another contains the figures representing the connections between the favorites figures.

![[Graphical Editing Framework-20231202-2.png]]


When the view first becomes visible, the edit parts and figures are instantiated for the view. Typically, as the model changes, listeners notify the edit parts of the changes which in turn update the figures so that the visual representation stays in sync with the model. For now, we add a setFocus method to FavoritesGEFView so that the edit parts and figures are refreshed each time the Favorites GEF View gets focus. This is not optimal, but is useful for debugging purposes. 

	Tip: If a child figure isn’t drawn in the GEF canvas, check that ...
	1) the parent model object contains the expected child model object
	2) getModelChildren() returns the expected child model object
	3) the edit part factory converts the child model object to the correct edit part
	4) the parent figure has a layout manager
	5) the child figure’s constraint or bounds are properly set


### Listening to Model changes
As the model changes, we would like the view to automatically adjust so that it always displays the current model state. To accomplish this, create a new listener in the XxxEditPart to refresh the edit parts based upon the model changes.

When a model change occurs, the edit part should act accordingly by refreshing whatever has changed from the model. For simplicity, the model listener described uses the refresh() method, which is appropriate for small to medium sized models. As models become larger, the time to execute the refresh() method increases in a linear fashion. If at some point performance becomes sluggish, consider using explicit calls to addChild() and removeChild() rather than calling refresh().

Edit parts have the methods activate() and deactivate() which are called when the edit part is added and removed. These methods provide the ideal place to add and remove model listeners. Add the following methods to XxxEditPart so that it will receive notifications as the model changes.

## GEF in an Eclipse Editor
Per Editor Lifecycle, each editor is initialized with an editor input.

Editors are typically built to modify some file type. But if the model is NOT stored in a workspace file and does not appear in the Package Explorer view. In the editor input class, the exists() method returns false indicating the the input is not from a workspace file, and the getPersistable() method returns null indicating that the model is not persisted in a workspace file.

###  Graphical Editor classes
GEF provides three classes upon which editors can be built. **GraphicalEditor** is the superclass of both **GraphicalEditorWithPalette** and **GraphicalEditorWithFlyoutPalette** which, as their names suggest, implement extra methods which place a palette on the editor. If you do not need a palette in your editor, then extend GraphicalEditor rather than one of its subclasses.

#### GraphicalEditor
Some of the interesting methods provided by this superclass include:
**createActions**()—Creates actions for this editor. Subclasses should override this method to create and register actions with the ActionRegistry.
**getCommandStack**()—Returns the command stack.
**getEditDomain**()—Returns the edit domain.
**initializeGraphicalViewer**()—Override to set the contents of the GraphicalViewer after it has been created.
**setEditDomain**(DefaultEditDomain)—Sets the EditDomain for this EditorPart.

#### GraphicalEditorWithPalette
This class extends **GraphicalEditor** to provide a fixed palette.
**getPaletteRoot**()—Must be implemented by the subclass to return the **PaletteRoot** for the palette viewer.

#### GraphicalEditorWithFlyoutPalette
Similar to **GraphicalEditorWithPalette**, this class extends **GraphicalEditor** to provide a palette, but this palette can be moved and collapsed by the user.
**getPaletteRoot**()—Must be implemented by the subclass to return the **PaletteRoot** for the palette viewer.
**getPalettePreferences**()—By default, this method returns a **FlyoutPreferences** object that stores the flyout settings in the GEF plugin. Sub-classes may override.

#### Edit domain
A GEF editor not only displays information graphically, but also facilitates user manipulation of that information. To facilitate editor changes to the model, each GEF editor must have an edit domain. The edit domain provides the interface for all user actions, tracks of the command stack, the active tool and the palette. 


### User interaction with GEF
As the user manipulates the figures, the GEF system breaks the continuous flow of user interaction into discrete GEF requests. These requests are forwarded to the appropriate GEF policy registered by the edit parts whose figure is being manipulated. Policies consume requests and produce GEF commands. Each command represents a specific change to the underlying model. By having user interaction broken into GEF commands, the GEF system can maintain a command stack and facilitate undo and redo operations in the typical Eclipse manner.

#### GEF requests
GEF requests encapsulate specifics about a particular user interaction such as selecting, dragging and connecting figures. Requests are used by the GEF system to capture the interaction and communicate this information to the appropriate GEF policy. The type of user interaction (the role) and the figure being manipulated is used to determine which GEF policy should receive the request.

#### GEF roles
A user interacts with a figure in one of several predefined ways called roles. GEF defines a number of roles, such as **LAYOUT_ROLE**, which indicates how a figure may be moved and resized by the user, and **COMPONENT_ROLE**, which indicates how a figure may be deleted. Each edit part may register a policy with each of these roles providing the specific operations to be performed when the user interacts with that edit part’s figure. The set of predefined GEF roles includes:
EditPolicy.**COMPONENT_ROLE**—The key used to register a policy defining “fundamental” operations that modify or delete model elements. Generally, the policy associated with this role knows only about the model and performs operations on the model. Figure changes are not made directly by this policy, but rather are the result of the edit parts listening for and responding to model change events. We can subclass **ComponentEditPolicy** to create a policy for deleting favorites items.
EditPolicy.**CONNECTION_ROLE**—The key used to register a policy defining connection operations such as creating a connection. **GraphicalNodeEditPolicy** provides much of the underlying functionality for implementing your own connection policy.
EditPolicy.**LAYOUT_ROLE**—The key used to register a policy defining operations on the edit part’s figure such as creating, moving and resizing. We can subclass **XYLayoutEditPolicy** to create a policy for moving figures around the GEF editor.

#### GEF policies
If an edit part wants the user to interact with the edit part’s figure, then the edit part implements the **createEditPolicies**() method to register one or more GEF policies using the **installEditPolicy**() method. E.g., we want the ability to resize and reposition figures on the screen. To accomplish this, implement the createEditPolicies() method in XxxEditPart to register a LAYOUT_ROLE. The GEF system forwards requests to the policy associated with the LAYOUT_ROLE when the children of XxxEditPart are moved or resized.

```
protected void createEditPolicies() {
	installEditPolicy(EditPolicy.LAYOUT_ROLE,
		new FavoritesLayoutEditPolicy());
}
```

#### GEF commands
GEF commands encapsulate changes to the model that can be applied and undone. As subclasses of **org.eclipse.gef.commands.Command**, each command has a number of methods it can override to provide the necessary behavior.
**canExecute**()—True if the command can be executed.
**canUndo**()—True if the command can be undone. This method should only be called after execute() or redo() has been called.
**chain(Command)**—Returns a Command that represents the chaining of a specified Command to this Command. The Command being chained will execute() after this command has executed, and it will undo() before this Command is undone.
**execute**()—Executes the Command. This method should not be called if the Command is not executable.
**redo**()—Re-executes the Command. This method should only be called after undo() has been called.
**setLabel**()—Sets the label used to describe this command to the User.
**undo**()—Undoes the changes performed during execute(). This method should only be called after execute has been called, and only when canUndo() returns true.

Each GEF command represents a specific change to the underlying model. GEF commands should only reference and modify the model and not the figures or edit parts. Changes to the figures should only be a result of edit parts responding to model change events. 

#### Edit menu
GEF tracks model changes using GEF commands, but to undo a command, we must hook GEF into the Eclipse edit framework. Hooking the undo/redo commands requires an editor contributor, while hooking the delete command can be accomplished using the new command infrastructure.

#### Z-order
In GEF, the x-axis stretches from left to right (x-coordinate values increase as you move to the right) and the y-axis from top to bottom (y-coordinate values increase as you move down), with the origin (0, 0) typically in the upper left corner of the GEF canvas. There is no z-axis in GEF, but conceptually the z-axis would stretch out from your monitor at right angles to both the x-axis and y-axis. Since GEF figures can overlap one another, there is conceptually a z-order to the figures determining which figure is in front of another.

Each edit part in GEF can has a list of zero or more child edit parts. The children first or earlier in the list are considered to have a lower z-order and thus are drawn behind those children later in the list. When the refreshChildren() method is called, either when the parent edit part becomes active or when refresh() is explicitly called, the list of child edit parts is reorderedto match the order of the model objects returned by getModelChildren().

As model objects are added, a typical user would expect the figures representing those model objects to appear on top of the already existing objects. If the getModelChildren() method in EditPart is backed by a HashSet, and thus the order of the model objects returned by that method is undetermined. When a new object is added to the model, the model listener calls refresh() which in turn calls getModelChildren(), thus the figures representing those new model objects appear in a random z-order rather than on top. In addition, the constraints for those new figures has not been set so they appear stacked on top of one another in the upper left corner of the editor.

One way to solve the z-order and location issues just described is to update the child edit parts ourselves using addChild() and removeChild() rather than calling refresh(). 


## Palette
