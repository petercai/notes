
# SWT
SWT uses heavyweight components from the underlying platform. The communication between SWT/JFace and the operating system is performed using the Java Native Interface (JNI).  SWT/JFace relies on JNI to manage the operating system’s rendering instead of performing all the work by itself.

SWT/JFace lets you determine when your resources should be deallocated. The toolset simplifies this process by providing **dispose**() methods within the component classes. Also,
once you have freed a parent resource, its child resources will automatically be disposed of. This means that few explicit deallocation calls are necessary within most applications. You might call SWT/JFace’s resource management semi-automatic.

## Display & Shell
When the **Shell**’s **open**() method is invoked, the application’s main window takes shape and its children are rendered in the display. So long as the **Shell** remains open, the **Display** instance uses its **readAndDispatch**() method to keep track of relevant user events in the platform’s event queue. When one of these actions involves closing the window, the resources associated with the Display object (such as the **Shell** and its children) are deallocated.
![[Graphical Editing Framework-20231203-6.png]]
### Display 
Although the **Display** class has no visible form, it keeps track of the GUI resources and manages communication with the operating system. That is, it concerns itself with how its windows are displayed, moved, and redrawn. It also ensures that events such as mouse clicks and keystroke actions are sent to the widgets that can handle them.
#### Operation of the Display class
Although the **Display** class may only appear in a few lines of your GUI code, it’s important to respect and understand its operation. It’s the workhorse of any SWT/JFace application, and whether you work with SWT/JFace or SWT alone, you must include an instance of this class in your program. This way, your interface will be able to use the widgets and containers of the operating system and respond to the user’s actions. Although most applications do little more than create a **Display** object and invoke a few of its methods, the role played by this class is sufficiently important to be worth describing in detail.
The main task of the **Display** class is to<u> translate SWT/JFace commands from your code into low-level calls to the operating system</u>. This process comprises two parts and begins once the application creates an instance of the **Display** class: 
First, the **Display** object constructs an instance of the OS class, which represents the platform’s operating system (OS). This class provides access to the computer’s low-level resources through a series of special Java procedures called native methods.
Then, like a switchboard operator, this Display object uses these methods to direct commands to the operating system and convey user actions to the application.

#### Methods of the Display class
Table 2.1 identifies and describes a number of methods that belong to the **Display** class. This isn’t a full listing, but it shows the methods vital for SWT/JFace GUIs to function and those necessary to implement particular capabilities in an application:
![[Graphical Editing Framework-20231203-3.png]]
The first two methods must be used in any SWT-based GUI. The first, **Display**(), creates an instance of the class and associates it with the GUI. The second, **getCurrent**(), returns the primary thread of the application, the user-interface thread. This method is generally used with the **dispose**() method to end the operation of the Display.

The next two methods in the table enable the application to receive notifications from the operating system whenever the user takes an action associated with the GUI. it’s important to understand the **readAndDispatch**() method, which accesses the operating system’s event queue and determines whether any of the user’s actions are related to the GUI. Using this method, the Application class knows whether the user has decided to dispose of the Shell. If so, the method returns TRUE, and the application ends. Otherwise, the Display object invokes its **sleep**() method, and the application continues waiting.

Although the **Display** class is important, there is no way to directly see the effects of its operation. Instead, you need to use classes with visual representations. The most important of these is the **Shell** class.

### The Shell class
Just as the **Display** class provides window management, the **Shell** class functions as the <u>GUI’s primary window</u>. Unlike the **Display** object, a Shell **instance** has a visual implementation. The **Shell** class accesses the operating system through the OS class to an extent, but only to keep track of opening, activating, maximizing, minimizing, and closing the main window.
The main function of the **Shell** class is to provide a common connection point for the **containers**, **widgets**, and **events** that need to be integrated into the GUI. In this respect, Shell serves as the parent class to these components. Figure 2.2 shows the relationship between an application’s operating system, Display, Shell, and their widgets.
![[Graphical Editing Framework-20231203-4.png]]
Every SWT/JFace application bases its widgets on a main **Shell** object, but other shells may exist in an application. They’re generally associated with temporary windows or dialog boxes. Because <u>these shells aren’t directly attached to the Display instanc</u>e, they’re referred to as
secondary shells. <u>Shells that are attached to the Display are called top-level shells.</u>

The **Shell** instance created in an application has a number of properties associated with it that allow users to alter its state or read information. These characteristics make up the component’s style. You can control a Shell’s style by adding a second argument to its constructor. If the only argument in an application **Shell** declaration is the display, it receives the default style for top-level windows, called **SHELL_TRIM**. This combines a number of individual style elements and tells the application that the window should have a title bar (SWT.**TITLE**) and that the user can minimize (SWT.**MIN**), maximize (SWT.**MAX**), resize (SWT.**RESIZE**), and close (SWT.**CLOSE**) the shell. The other default shell style, **DIALOG_TRIM**,
ensures that dialog shells have title bars, a border around the active area
(SWT.**BORDER**), and the ability to be closed.

### Model-based adapters
Eclipse documentation uses two terms to refer to JFace classes that work with SWT
widgets: **helper classes** and **model-based adapters**. We’ll call them model-based adapters, or
JFace adapters. These adapters can be split into four categories. 
![[Graphical Editing Framework-20231203-5.png]]
The first and most widely used category of model-based adapters includes the
**Viewer** classes. In SWT, the information and appearance of a GUI component are bound together. However, viewers separate these aspects and allow for the same information to be presented in different forms. For example, the information in an SWT tree can’t be separated from the tree object. But the same information in a JFace **TreeViewer** can be displayed in a **TableViewer** or a **ListViewer**.

The next category involves **Actions** and **Contributions**. These adapters simplify event handling, separating the response to a user’s command from the GUI events that result in that response. This can be best explained with an example. In SWT, if four different buttons will close a dialog box, then you must write four different event-handling routines even though they accomplish the same result. In JFace, these four routines can be combined in an
action, and JFace automatically makes the four buttons contributors to that action.

The third category involves **image and font registries**. In SWT, it’s important to keep the number of allocated fonts and images to a minimum, since they require operating system resources. But with JFace registries, these resources can be allocated and deallocated when
needed. Therefore, if you’re using multiple images and fonts, you don’t need to be concerned with manual garbage collection. 

The last group comprises **JFace dialogs and wizards**. These are the simplest adapters to understand, since they extend the capability of SWT dialogs. JFace provides dialogs that present messages, display errors, and show the progress of ongoing processes. In addition, JFace provides a specialized dialog called a wizard, which guides the user through a group of tasks,such as installing software or configuring an input file.

## Events
The JFace library replaces events and listeners with **actions** and **contributions**, which perform the same function as their SWT counterparts but in very different ways. These new classes simplify the process of event programming by separating the event-processing methods from the GUI’s appearance.

### Event processing in SWT
The SWT event-processing cycle is depicted in figure 4.1. It begins with the operating system’s event queue, which records and lists actions taken by the user. Once an SWT application begins running, its Display class sorts through this queue using its **readAndDispatch**() method and **msg** field, which acts as a handle to the underlying OS message queue. If it finds anything relevant, it sends the event to its **top-level Shell** object, which <u>determines which widget should receive the event</u>. The **Shell** then sends the event to the widget that the user acted on, which transfers this information to an associated interface called a **listener**. One of the **listener**’s methods performs the necessary processing or invokes another method to handle the user’s action, called an **event handle**r.
![[Graphical Editing Framework-20231203-7.png]]
When making a widget responsive to events, the main tasks of the GUI designer are determining which events need to be acted on, creating and associating listeners to sense these events, and then building event handlers to perform the necessary processing. This section will show how to accomplish these tasks using the SWT data structures contained in the **org.eclipse.swt.events** package.

#### Using typed listeners and events
Most of the listener interfaces in SWT only react to a particular set of user actions. They’re called **typed listeners** for this reason, and they inherit from the **TypedListener** class. Similarly, the events corresponding to these specific actions are **typed events**, which subclass the **TypedEvent** class. For example, a mouse click or double-click is represented by a **MouseEvent**, which is sent to an appropriate **MouseListener** for processing. Keyboard actions performed by the user are translated into **KeyEvents**, which are picked up by **KeyListeners**. A full list of these typed events and listeners is shown below.

![[Graphical Editing Framework-20231203-8.png]]
In order to function, these listeners must be associated with components of the GUI. For example, a **TreeListener** will only receive **TreeEvents** if it’s associated with a **Tree** object. But not every GUI component can use each listener. For example, as shown in the GUI component column of the table, a **Control** component broadcasts many more types of events than a **Tracker** object. There are also listeners, such as **MenuListeners** and **TreeListeners**, that can only be attached to very specific widgets. This attachment is performed by invoking the component’s **add...Listener()** method with the typed listener as the argument.

##### Understanding Event classes
The Event column in table 4.1 lists the subclasses of **TypedEvent** that the **Display** and **Shell** objects send to **typed listeners**. Although programmers generally don’t manipulate these classes directly, the classes contain member fields that provide information regarding the event’s occurrence. This information can be used in event handlers to obtain information about the environment. These fields, inherited from the **TypedEvent** and **EventObject** classes:
![[Graphical Editing Framework-20231203-9.png]]
In addition to these, many event classes have other fields that provide more information about the user’s action. For example, the **MouseEvent** class also includes a button field, which tells which mouse button was pressed, and x and y, which specify the widget-relative coordinates of the mouse action. The **ShellEvent** class contains a boolean field called **doit**, which lets you specify whether a given action will result in its intended effect. Finally, the **PaintEvent** class provides additional methods.

##### Programming with listeners
There are two main methods of incorporating listeners in code. The first creates <u>an anonymous interface in the component’s add...Listener() method</u>, which narrows the scope of the listener to the component only. This method is shown in the following code snippet:

```
Button button = new Button(shell, SWT.PUSH | SWT.CENTER);
button.addMouseListener(new MouseListener()
{
	public void mouseDown(MouseEvent e)
	{
		clkdwnEventHandler();
	}
	public void mouseUp(MouseEvent e)
	{
		clkupEventHandler();
	}
	public void mouseDoubleClick(MouseEvent e)
	{
		dblclkEventHandler();
	}
});
```

#### Adapters
**Adapters** are **abstract classes** that implement **Listener** interfaces and provide default implementations for each of their required methods. This means that when you associate a widget with an adapter instead of a listener, you only need to write code for the method(s) you’re interested in. Although this may seem like a minor convenience, it can save you a great deal of programming time whenyou’re working with complex GUIs.
	 NOTE
		The adapters mentioned in this section are very different from the model-based adapters provided by the JFace library. Here, adapters reduce the amount of code necessary to create listener interfaces. 
**Adapters** are only available for events whose listeners have more than one member method. The full list of these classes is shown in table 4.3, along with their associated Listener classes.
![[Graphical Editing Framework-20231204-1.png]]
Adapter objects are easy to code and are created with the same add...Listener() methods. Two examples are shown here:

```
button.addMouseListener(new MouseAdapter()
{
	public void mouseDoubleClick(MouseEvent e)
	{
		dblclkEventHandler();
	}
)};
```

#### Keyboard events
Although most of the events in table 4.1 are straightforward to understand and use, the keyboard event classes require further explanation. Specifically, these events include the **KeyEvent** class, which is created any time a key is pressed, and its two subclasses, **TraverseEvent** and **VerifyEvent**. A **TraverseEvent** results when the user presses an arrow key or the Tab key in order to focus on the next widget. A **VerifyEvent** fires when the user enters text that the program needs to check before taking further action.

In addition to the fields inherited from the **TypedEvent** and **EventObject** classes, the **KeyEvent** class has three member fields that provide information concerning the key that triggered the event:
- **character**—Provides a char value representing the pressed key.
 - **stateMask**—Returns an integer representing the state of the keyboard modifier keys. By examining this integer, a program can determine whether any of the Alt, Ctrl, Shift, and Command keys are currently pressed.
 - keyCode—Provides the SWT public constant corresponding to the typed key, called the key code. These public constants are presented in table 4.4.
![[Graphical Editing Framework-20231204-2.png]]
The following code snippet shows how to use a **KeyListener** to receive and process a **KeyEvent**. It also uses the fields (character, stateMask, and keyCode) to acquire information about the pressed key:

```
Button button = new Button(shell, SWT.CENTER);
button.addKeyListener(new KeyAdapter()
{
	public void keyPressed(KeyEvent e)
	{
		String string = "";
		if ((e.stateMask & SWT.ALT) != 0) string += "ALT-";
		if ((e.stateMask & SWT.CTRL) != 0) string += "CTRL-";
		if ((e.stateMask & SWT.COMMAND) != 0) string += "COMMAND-";
		if ((e.stateMask & SWT.SHIFT) != 0) string += "SHIFT-";
		switch (e.keyCode)
		{
			case SWT.BS: string += "BACKSPACE"; break;
			case SWT.CR: string += "CARRIAGE RETURN"; break;
			case SWT.DEL: string += "DELETE"; break;
			case SWT.ESC: string += "ESCAPE"; break;
			case SWT.LF: string += "LINE FEED"; break;
			case SWT.TAB: string += "TAB"; break;
			default: string += e.character; break;
		}
		System.out.println (string);
	}
});
```

The **TraverseEvent** fires when the user presses a key to progress from one component to another, such as in a group of buttons or checkboxes. The two fields contained in this class let you control whether the traversal action will change the focus to another control, or whether the focus will remain on the widget that fired the event. 
- The simplest field, **doit**, is a boolean value that allows (TRUE) or disallows (FALSE) traversal for the given widget. 
-  The second field of the TraverseEvent class, detail, is more complicated. It’s an integer that represents the identity of the key that caused the event. For example, if the user presses the Tab key to switch to a new component, the detail field will contain the SWT constant **TRAVERSE_TAB_NEXT**.

Each type of control has a different default behavior for a given traversal key. For example, a TraverseEvent that results from a TRAVERSE_TAB_NEXT action will, by default, cause a traversal if the component is a radio button, but not if it’s a Canvas object. Therefore, by setting the doit field to TRUE, you override the default setting and allow the user to traverse. Setting the field to FALSE keeps the focus on the component.

The use of the **VerifyEvent** is similar to that of the TraverseEvent. The goal is to determine beforehand whether the user’s action should result in the usual or default behavior. In this case, you can check the user’s text to determine whether it should be updated or deleted in the application. Two of the class fields, start and end, specify the range of the input, and the text field contains the input String under examination. Having looked at the user’s text, you set the boolean doit field to allow (TRUE) or disallow (FALSE) the action.

#### Customizing event processing with untyped events
**Typed events and listeners** enable event processing with classes and interfaces expressly suited to their tasks. Further, typed listeners provide specific methods to receive and handle these events. By narrowing the scope of listeners and events to handle only particular actions, <u>the use of typed components reduces the possibility of committing coding errors</u>.
However, if you prefer coding flexibility over safety, SWT provides **untyped events and listeners**. When an **untyped listener**, represented by the Listener class, is associated with a GUI component, it <u>receives every class of event that the component is capable of sending</u>. Therefore, you have to manipulate the **catch-all event**, represented by the **Event** class, to determine which action the user performed. The proper event-handling method can then be invoked.

It’s important to note that Eclipse.org recommends **against** using **untyped events and listeners**. In fact, it mentions that they are “not intended to be used by applications.” These mechanisms also aren’t included with their typed counterparts in the **org.eclipse.swt.events** package. Instead, both the **untyped Listener interface and the Event class** are located in the **org.eclipse.swt.widgets** package.

Despite this, the SWT code snippets provided by the Eclipse website use untyped listeners and events exclusively. This makes coding convenient, since you can create a customized listener that reacts to a specified set of events. An example is shown here:

```
Listener listener = new Listener ()
{
	public void handleEvent (Event event)
	{
	switch (event.type)
	{
		case SWT.KeyDown:
			if (event.character == 'b')
			System.out.println("Key"+event.character);
			break;
		case SWT.MouseDown:
			if (event.button == 3)
			System.out.println("Right click");
			break;
		case SWT.MouseDoubleClick:
			System.out.println("Double click");
			break;
		}
	}
};
Button button = new Button(shell, SWT.CENTER);
button.addListener(SWT.KeyDown, listener);
button.addListener(SWT.MouseDown, listener);
button.addListener(SWT.MouseDoubleClick, listener);
```

In order to take the place of typed events, the Event class contains all the fields in each typed event. It has the same character field as a KeyEvent and the same button field as a MouseEvent. As shown in the previous code, it also has a field called type, which refers to the nature of the event. 
![[Graphical Editing Framework-20231204-3.png]]

### Event processing in JFace
A listener interface can provide the same event handling for different controls, but its usage depends on the component that launched the event. Listeners that receive MouseEvents can’t be used for menu bar selections. Even untyped Events are only useful after the program determines which type of control triggered the event.

But when you’re dealing with complex user interfaces, it’s helpful to separate the event-handling capability from the GUI components that generated the event. 
 - This allows one group to work on a GUI’s event handling independently from the group designing its appearance. 
 -  Also, if a listener’s capability can be attached to any component, then its code can be reused more often. 
 - Finally, if one section of a program deals strictly with the GUI’s view and another is concerned only with event processing, then the code is easier to develop and understand.

JFace provides this separation with its **Action** and **ActionContributionItem** classes. Put simply, an **ActionContributionItem** combines the function of a GUI widget and its attached listener class. Whenever the user interfaces with it, it triggers its associated **Action** class, which takes care of handling the event. Although this may seem similar to SWT’s listener/event model, these classes are more abstract, simpler to use, and narrower in scope.
#### Understanding actions and contributions
Although it’s interesting to know that you can handle **TraverseEvents** and **ArmEvents** if they occur, few applications use them. Also, it may be fascinating to attach multiple listeners and event handlers to a widget, but GUI components usually perform only a single function in response to a single input type. Because SWT’s structure provides for every conceivable component and combination of events, even the simplest listener/event code requires complexity.

It would make event programming easier if a toolset concentrated on only those few widgets and events that are used most often and made their usage as simple as possible. JFace’s event-processing structure does exactly this: Its goal is to make event processing more straightforward, allowing programmers to receive and use common events with fewer lines of code. In reaching this goal, JFace makes three assumptions:
 -  The user’s actions will involve buttons, toolbars, and menus.
 -  Each component will have only one associated event.
 -  Each event will have only one event handler.
By taking these assumptions into account, JFace simplifies event processing considerably. The first assumption means that contributions only need to take one of three forms. The second assumption provides the separation of contributions from their associated actions; that is, if each contributing component triggers only one event, then it doesn’t matter what action is triggered or which component fired the event. The third assumption means that each action needs only one event-handling routine. This simplified event model for SWT/JFace.
![[Graphical Editing Framework-20231204-4.png]]
Like the SWT event model, the interface process begins with the **Display** class keeping track of the operating system’s event queue. This time, though, it passes information to the **ApplicationWindow**, which contains the Display’s **Shell** object.
The **ApplicationWindow** creates an **Action** class and sends it to the contribution that generated the original event. The **contribution** then invokes the **run**() method of the **Action** class as the single event handler.

The **Action** class behaves similarly to SWT’s **Event** class, but the contribution capability is more complicated. The two main contribution classes are the **ContributionItem** class and the **ContributionManager** class. The **ContributionItem** class provides individual GUI components that trigger actions, and the **ContributionManager** class produces objects capable of containing **ContributionItems**. Because these are both abstract classes, event handling is performed with their subclasses.
![[Graphical Editing Framework-20231204-5.png]]
Although the **ActionContributionItem** class is one of many concrete subclasses of **ContributionItem**, it’s the most important. This class is created and implemented in an **ApplicationWindow** to connect an action to the GUI. It has no set appearance, but instead takes the form of a button, menu bar item, or toolbar item, depending on your use of the **fill()** method.
The second way to incorporate contributions in an application involves the use of a **ContributionManager** subclass. These subclasses serve as containers for **ContributionItems**, combining them to improve GUI organization and simplify programming. The **MenuManager** class combines **ContributionItems** in a window’s top-level menu, and the **ToolBarManager** class places these objects in a toolbar located just under the menu.
#### Creating Action classes
Code below creates a subclass of the abstract Action class called StatusAction. This class functions by sending a String to an ApplicationWindow’s status line whenever it triggers.
Because this class will be implemented in a toolbar, it needs an associated image. 

```
public class StatusAction extends Action
	StatusLineManager statman;
	short triggercount = 0;
	public StatusAction(StatusLineManager sm)
	{
		super("&Trigger@Ctrl+T", AS_PUSH_BUTTON);
		statman = sm;
		setToolTipText("Trigger the Action");
		setImageDescriptor(ImageDescriptor.createFromFile
			(this.getClass(),"eclipse.gif"));
	}
	public void run()
	{
		triggercount++;
		statman.setMessage("The status action has fired. Count: " +
			triggercount);
	}
}
```

#### Implementing contributions in an ApplicationWindow
Because actions and contributions can only be associated with buttons, toolbar items, and menu items, any application demonstrating their capability must rely on these components. Code below shows how **ContributionItem** and **ContributionManager** classes are
added to a window. Three contributor classes, **ActionContributionItem**, **MenuManager**, and **ToolBarManager**, all trigger the StatusAction when acted on. This action sends a message to the status line at the bottom of the window.


```
public class Contributions extends ApplicationWindow
	StatusLineManager slm = new StatusLineManager();
	StatusAction status_action = new StatusAction(slm);
	ActionContributionItem aci = new
		ActionContributionItem(status_action); // 1. Assign contribution
	public Contributions()
	{ // 2. add resources to ApplicationWindow
		super(null);
		addStatusLine();
		addMenuBar();
		addToolBar(SWT.FLAT | SWT.WRAP);
	}
	protected Control createContents(Composite parent)
	{
		getShell().setText("Action/Contribution Example");
		parent.setSize(290,150);
		aci.fill(parent); //3. create button within window
		return parent;
	}
	public static void main(String[] args)
	{
		Contributions swin = new Contributions();
		swin.setBlockOnOpen(true);
		swin.open();
		Display.getCurrent().dispose();
	}
	protected MenuManager createMenuManager()
	{
		MenuManager main_menu = new MenuManager(null);
		MenuManager action_menu = new MenuManager("Menu");
		main_menu.add(action_menu);
		action_menu.add(status_action); // 1. assign contribution
		return main_menu;
	}
	protected ToolBarManager createToolBarManager(int style)
	{
		ToolBarManager tool_bar_manager = new ToolBarManager(style);
		tool_bar_manager.add(status_action); //1.
		return tool_bar_manager;
	}
	protected StatusLineManager createStatusLineManager()
	{
		return slm;
	}
}
```
1. Beneath the class declaration, the program constructs an instance of the StatusAction with a StatusLineManager object as its argument. Then, it creates an ActionContributionItem object and identifies it with the StatusAction instance. This contribution has no form yet, but is simply a high-level means of connecting an action to the user interface.
2. The constructor method creates an **ApplicationWindow** object and adds a menu, toolbar, and status line.
3. The **createContents**() method sets the title and size of the window and then invokes **aci.fill()**. This method is important since it places the **ActionContributionItem** object in the GUI. In this case, because the **fill()** argument is a **Composite** object, the contributor takes the form of a button that triggers a **StatusEvent** whenever it’s pressed.
The last three methods in Contributions are also straightforward. The main() method takes care of creating and opening the window and then disposing of the GUI resources. Then, the **createMenuManager**() method creates a menu instance at the top of the window. Because it’s a subclass of **ContributionManager**, an **Action** object can be associated with it, and the **status_action** object is added with the **add()** method. This method is also used in the **createToolBarManager**() method to associate the action instance. In both cases, an **ActionContributionItem** is implicitly created and added to the menu in the form of a menu item and to the toolbar as a toolbar item.
#### Interfacing with contributions
There are two main ways of incorporating an **ActionContributionItem** in a GUI. 
1. The first method is to use the add() method of a ContributionManager subclass, as performed by the MenuManager and ToolBarManager in the Contributions application. 
2. The second is to use the fill() method associated with the ActionContributionItem class and add an SWT widget as its argument. If the argument is a Composite, as in Contributions, then the contributor will appear as determined by the STYLE property of the action. If the argument is an SWT Menu object, then the contributor will take the form of a menu item. Finally, if the argument is an SWT ToolBar object, then the contributor will appear as an item in a toolbar.


# Draw2D
Draw2D relies on many constructs from SWT (not JFace), but you must draw and design most of the graphical components within it. You also need to specify their event responses and any drag-and-drop capability. Essentially, the Draw2D library serves as a complete graphics package, with the additional feature that these graphics can be moved and associated with events.
### Using Draw2D’s primary classes
If you’ve understood our discussion of SWT so far, then Draw2D won’t present much difficulty. As shown in table C.1, the two libraries use related classes and provide similar structures for drawing, event handling, and component layout. In fact, all Draw2D GUIs must be added to an SWT Canvas. The first difference is that whereas a normal Canvas adds a GC object to provide graphics, a Canvas in a Draw2D application uses an instance of a LightweightSystem.
![[Graphical Editing Framework-20231203-1.png]]
**LightweightSystems** function similarly to Display objects in SWT. They have no visual representation but provide event handling and interaction with the external environment. As the name implies, **LightweightSystems** operate at a level removed from the operating system. This means you lose the advantages of SWT/JFace’s heavyweight rendering, such as its rapid execution and native look and feel. However, now you can truly customize the appearance and operation of your components.
Without question, the most important class in Draw2D is the **Figure**, and the majority of our Draw2D discussion will focus on its methods and subclasses. Like an SWT **Shell**, it must be added to a **LightweightSystem** in order to provide a basis for the GUI’s appearance. Like an SWT Control, it provides for resizing and relocating, adding **Listeners** and **LayoutManagers**, and setting colors and fonts. It also functions like a **Composite** in SWT, providing methods for adding and removing other Figure **objects**—children—within its frame. However, unlike Widget and Control objects, you can easily subclass Figures. Their graphical aspects can be represented by a drawing or image. 
![[Graphical Editing Framework-20231203-2.png]]
Not only can Figures use separate Listener interfaces, they can also perform the majority of
their event handling by themselves. Figures can even initiate specific kinds of events to alert other objects in the GUI.
To add images and drawings to **Figures**, you need to use instances of the **Graphics** class. It functions similar to SWT’s **GC** (graphics context) class, and it provides methods for incorporating graphics in a given area. It also contains many of the same methods as GC, particularly for drawing lines and shapes, displaying images, and working with fonts. However, the **Graphics** class provides one very different capability: Its objects can be moved, or translated, within the **LightweightSystem**. This means that when you want to change the position of a graphical component, Draw2D provides its own drag-and-drop capability to translate a **Figure** to the desired location.

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