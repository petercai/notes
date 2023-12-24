
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
1. The first method is to use the add() method of a **ContributionManager** subclass, as performed by the **MenuManager** and **ToolBarManager** in the Contributions application. 
2. The second is to use the **fill**() method associated with the **ActionContributionItem** class and add an SWT widget as its argument. If the argument is a **Composite**, as in **Contributions**, then the contributor will appear as determined by the **STYLE** property of the action. If the argument is an SWT **Menu** object, then the contributor will take the form of a menu item. Finally, if the argument is an SWT ToolBar object, then the contributor will appear as an item in a toolbar.

![[Graphical Editing Framework-20231209-1.png]]
An interesting characteristic of the ContributionManager class is that its **add**() method is overloaded to accept arguments of both **Action** and **ActionContributionItem** classes. So, you can associate a ContributionItem with a ContributionManager implicitly (with the Action) or explicitly (with the ActionContributionItem). But there’s a fundamental difference: You can perform implicit contribution association repeatedly with the same Action object, as shown in the Contributions class. Explicit contribution association can be performed only once.


#### Exploring the Action class
You need to keep in mind many more aspects of the Action class. The **Action** class contains a large number of methods to enhance the capability of your user interface. These have been divided into categories and listed in the tables that follow. The first set of methods is important in any implementation of the Action class. The first and most important method is **run**(). As we mentioned earlier, this is the single event-handling routine in an Action class, and it’s invoked every time the action is triggered. The next method in the table serves as the default constructor. In addition, constructor methods initialize the member fields associated with the Action class, which we’ll fully describe shortly.
![[Graphical Editing Framework-20231209-2.png]]
![[Graphical Editing Framework-20231209-3.png]]
![[Graphical Editing Framework-20231209-4.png]]
![[Graphical Editing Framework-20231209-5.png]]
![[Graphical Editing Framework-20231209-6.png]]
![[Graphical Editing Framework-20231209-7.png]]


# Layout
Layouts are associated with a composite and help organize the controls within it.

## The fill layout

```
public class FillLayoutComposite extends Composite {
	public FillLayoutComposite(Composite parent) {
		super(parent, SWT.NONE);
		FillLayout layout = new FillLayout( SWT.VERTICAL);
		setLayout(layout);
		for (int i = 0; i < 8; ++i) {
			Button button = new Button(this, SWT.NONE);
			button.setText("Sample Text");
			}
	}
}
```

The fill layout is a simple layout that takes the child controls and lays them out at equal intervals to fill the space in a composite. By default, the controls are stacked side by side, from left to right. Each control is given the space it needs, and any leftover space in the composite is divided among the child controls.

## The row layout

```
RowLayout layout = new RowLayout(SWT.HORIZONTAL);
```

The row layout arranges child controls into single row by default, you need to pass in SWT.WRAP to get the functionality of additional rows. The row layout provides additional customization options by giving you access to margin and spacing options. 
	(Note that the name row layout is a bit of a misnomer, because you can choose to use either a horizontal row or a vertical row. The vertical row layout is therefore really a column layout.)

Much of the other behavior in a RowLayout is specified through property values. We’re mainly concerned with the following properties:
- **wrap**—A boolean value that defaults to true. You’ll probably want to keep the default. Switching it off will result in all the controls staying on a single row, with the controls cut off at the end of the visible edge of the parent composite.
- **pack**—A boolean value that defaults to true. This property keeps child controls the same size, which typically is desirable in the context of a row layout. You get even rows of controls by setting pack; on the other hand, keeping pack off lets controls retain their natural sizing.
- **justify**—A boolean value that defaults to false. This property distributes controls evenly across the expanse of the parent composite. If justify is on and the parent is resized, then all the child controls pick up the slack and redistribute themselves evenly across the empty space.

## The grid layout
The grid layout builds on the row layout model by allowing you to explicitly create multiple rows and columns.  Factor in a flexible grid data object, and the end result is that the grid layout is the most useful and widely used layout. 
```
public class GridLayoutComposite extends Composite {
	public GridLayoutComposite(Composite parent) {
		super(parent, SWT.NONE);
		GridLayout layout = new GridLayout(4,false);
		setLayout(layout);
		for (int i = 0; i < 16; ++i) {
			Button button = new Button(this, SWT.NONE);
			button.setText("Cell " + i);
		}
	}
}
```
Note that the constructor takes two parameters: the number of columns and a boolean to indicate whether the columns should take up an even amount of space. By passing false, you tell the layout to only use the minimum amount of space needed for each column.

### GridData
The key to using the grid layout is understanding that a single child control can span more
than one grid cell at a time. You do this through layout data. In this case, let’s turn to the **GridData** object, which provides additional hints for the **GridLayout** on how to lay out a Control.

```
Text t = new Text(this, SWT.MULTI);
GridData data = new GridData(GridData.FILL_BOTH);
data.horizontalSpan = 2;
data.verticalSpan = 2;
t.setLayoutData(data);
```

#### Using GridData styles
The constructor takes a series of style constants, which when combined determine how the layout will position an individual widget. These styles fall into three categories: **FILL**, **HORIZONTAL_ALIGN**, and **VERTICAL_ALIGN**.
The various **FILL** styles determine whether the cell should be expanded to fill available space. Valid values include **FILL_HORIZONTAL**, which indicates that the cell should be expanded horizontally; **FILL_VERTICAL**, to expand the cell vertically; and **FILL_BOTH**, which effectively causes the cell to fill all the space available.
The **ALIGN** styles, on the other hand, determine where the control should be positioned in the cell. Values include **BEGINNING**, **END**, **CENTER**, and FILL. **BEGINNING** positions the control at the left or topmost edge of the cell, whereas **END** puts the control at the right or bottommost edge. **CENTER** centers the control, and **FILL** causes the control to expand to fill all available space.
![[Graphical Editing Framework-20231209-8.png]]

#### Using GridData size attributes
GridData also has a number of public attributes that can be set to control its behavior. Several of these are boolean values that are automatically managed when the different styles are set, so it isn’t typically necessary to manipulate them directly. Some, however, are integer values used to precisely control the size of individual cells.
![[Graphical Editing Framework-20231209-9.png]]

## The form layout
The form  layout lets you create a UI based on gluing together controls relative to each other or the parent composite.
This makes it much easier to create resizable forms with controls of differingsizes. A typical dialog box, for example, has a large central text area and two buttons located just below and to the right. In this case, the most natural way to think of how the controls should be positioned is by envisioning them relative to each other.
The **FormLayout** class is fairly simple. The only configuration options come from attributes that control the height and width of the margins around the edge of the layout, and the spacing attribute, which lets you specify the amount of space (in pixels) to be placed between all controls. 
```
public class FormLayoutComposite extends Composite {
	public FormLayoutComposite(Composite parent) {
		super(parent, SWT.NONE);
		
		FormLayout layout = new FormLayout();
		setLayout(layout);
		
		Text t = new Text(this, SWT.MULTI);
		FormData data = new FormData();
		data.top = new FormAttachment(0, 0); // Test goes at uppper left
		data.left = new FormAttachment(0, 0);
		data.right = new FormAttachment(100);
		data.bottom = new FormAttachment(75);
		t.setLayoutData(data);
		
		Button ok = new Button(this, SWT.NONE);
		ok.setText("Ok");
		Button cancel = new Button(this, SWT.NONE);
		cancel.setText("Cancel");
		
		data = new FormData();
		data.top = new FormAttachment(t); // Ok button positioned relative
										// to other widgets
		data.right = new FormAttachment(cancel);
		ok.setLayoutData(data);
		data = new FormData();
		data.top = new FormAttachment(t);
		data.right = new FormAttachment(100); // Cancel button on right side
		cancel.setLayoutData(data);
	}
}
```

### Using FormData
A **FormData** instance is typically associated with each child control in a composite. Even more so than with other layouts, it’s important to provide configuration data for each child, because the whole idea of a form layout is to specify positions of child controls relative to each other. If a given control doesn’t have a **FormData** instance describing it, it will default to being placed in the upper-right corner of the composite, which is rarely what you want.
The **width** and height attributes specify the dimensions of a control in pixels. More important are the **top**, **bottom**, **right**, and **left** attributes, each of which holds an instance of **FormAttachment**. These attachments describe the control’s relations to other controls in the composite.

### Specifying relations using FormAttachment
Understanding the **FormAttachment** class is the most important part of using a form layout. As mentioned earlier, each instance of **FormAttachment** describes the positioning of one side of a control. You can use FormAttachment two different ways.
First, you can specify a **FormAttachment** using a percentage of the parent composite. For example, if the left side of a FormData is set to a FormAttachmentwith 50%, then the left edge of the control will be placed at the horizontal middle of the parent. Likewise, setting the top edge to 75% positions the control three quarters of the way down the composite
![[Graphical Editing Framework-20231210-1.png]]
Specifying FormAttachments in terms of percentages can be useful, but you shouldn’t use this approach often. Specifying all your controls using percentages isn’t much different from assigning them absolute pixel positions: It quickly becomes difficult to visualize the positions of each element; and when the composite is resized, it’s unlikely that the controls will still be in the positions you desire. The point of using a FormLayout is to position controls relative to each other, which the second form of FormAttachment allows.

The second series of FormAttachment constructors are based on passing in other controls. They’re used to position the edge of one control next to another. By setting the right attribute of the **FormData** for button1 to a FormAttachment constructed with button2, you’re saying that button1 should always be positioned such that button2 is immediately to its right. Laying out most or all of your controls in this fashion has several benefits. The intent of your layout code becomes easier to understand: Instead of your having to guess which controls are meant to be next to each other based on percentages or pixels, it becomes obvious that, for example, control foo should always be below bar. Second, the form layout is also aware of your intent. However the composite may be resized, it will
always be able to maintain the correct relative positions.
![[Graphical Editing Framework-20231210-2.png]]


# Graphics

## The graphic context
The graphic context functions like a drawing board on top of a **Control**. It lets you add custom shapes, images, and multifont text to GUI components. It also provides event processing for these graphics by controlling when the Control’s visuals are updated.

In SWT/JFace, the graphic context is encapsulated in the **GC** class. GC objects attach to existing Controls and make it possible to add graphics. This section will deal with how this important class and its methods operate.

### Creating a GC object
The first step in building a graphically oriented application is creating a graphic context and associating it with a component. The GC constructor method performs both tasks.
![[Graphical Editing Framework-20231210-3.png]]

The style constant mentioned in the second constructor determines how text appears in the display. The two values are **RIGHT_TO_LEFT** and **LEFT_TO_RIGHT**; the default style is **LEFT_TO_RIGHT**.
The first argument requires an object that implements the **Drawable** interface. This interface contains methods that relate to the internals of a graphic context. SWT provides three classes that implement **Drawable**: **Image**, **Device**, and **Control**. Unless you create your own **Drawable** objects, you can only add graphics to instances of these classes or their subclasses. 
![[Graphical Editing Framework-20231210-5.png]]
The **Device** class represents any mechanism capable of displaying SWT/JFace objects. This is easier to understand if you consider its two main subclasses: **Printer**, which represents print devices, and **Display**, which accesses a computer’s console. The **Display** class, the base class of any SWT/JFace application. But since this chapter deals with adding graphics to individual components, we’ll associate our GC with the third **Drawable** class, **Control**. 
A **Control** object is <u>any widget that has a counterpart in the underlying operating system. </u>I<u>nstances of this class and its subclasses can be resized, traversed, and associated with events and graphics</u>. Although of all them can contain graphics, only one class is particularly suited for GC objects: **Canvas**. This class not only provides the containment property of a Composite, but also can be customized with a number of styles that determine how graphics are shown in its region.


### Drawing shapes on a Canvas
A full graphical application, DrawExample.java, uses a GC object to draw lines and shapes on a Canvas instance.
```
public class DrawExample
{
	public static void main (String [] args)
	{
		Display display = new Display();
		Shell shell = new Shell(display);
		shell.setText("Drawing Example");
		
		Canvas canvas = new Canvas(shell, SWT.NONE); // Create Cavas object
		canvas.setSize(150, 150);
		canvas.setLocation(20, 20);
		
		shell.open ();
		shell.setSize(200,220);
		
		GC gc = new GC(canvas); // Create graphic context in Cavas
		gc.drawRectangle(10, 10, 40, 45);
		gc.drawOval(65, 10, 30, 35);
		gc.drawLine(130, 10, 90, 80);
		gc.drawPolygon(new int[] {20, 70, 45, 90, 70, 70});
		gc.drawPolyline(new int[] {10,120,70,100,100,130,130,75});
		gc.dispose(); // Deallocate Coor object when finished
		
		while (!shell.isDisposed())
		{
			if (!display.readAndDispatch())
			display.sleep();
		}
		display.dispose();
	}
}
```
This example demonstrates two important concerns to keep in mind when you work with
GCs. 
- First, the program constructs its **Canvas** object before invoking the **shell**.**open**() method; it creates and uses the **GC** object afterward. This sequence is necessary since **open**() clears the **Canvas** display. This also means that graphic contexts must be created in the same class as the **Shell** object. 
- Second, the program deallocates the GC object immediately after its last usage. Doing so frees up computer resources quickly without affecting the drawing process. Along with those used in DrawExample.java, the GC class provides a number of methods that draw and fill shapes on a Drawable object.
![[Graphical Editing Framework-20231210-6.png]]
One problem with DrawExample is that its shapes are erased whenever the shell is obscured or minimized. This is an important concern, since we need to make sure the graphics remain visible despite windowing changes. For this purpose, SWT lets you control when a **Drawable** object is refreshed. This updating process is called **painting**.

### Painting and PaintEvents
When a GC method draws an image on a **Drawable** object, <u>it performs the painting process only once</u>. If a user resizes the object or covers it with another window, its graphics are erased. Therefore, it’s important that an application maintain its appearance whenever its display is affected by an external event. 

These external events are called **PaintEvent**s, and the interfaces that receive them are **PaintListener**s. A **Control** triggers a **PaintEvent** any time its appearance is changed by the application or through outside activity. The following snippet shows an example; because a PaintListener has only one event handling method, no adapter class is necessary:

```
Canvas canvas = new Canvas(shell, SWT.NONE);
canvas.setSize(150, 150);
canvas.setLocation(20, 20);
canvas.addPaintListener(new PaintListener()
{
	public void paintControl(PaintEvent pe)
	{
		GC gc = pe.gc;
		gc.drawPolyline(new int[] {10,120,70,100,100,130,130,75});
	}
});
shell.open();
```
An interesting aspect of using **PaintListeners** is that each **PaintEvent** object contains its own GC. This is important for two reasons. 
- First, because the GC instance is created by the event, the PaintEvent takes care of its disposal. 
- Second, the application can create the GC before the shell is opened, which means that graphics can be configured in a separate class.

SWT optimizes painting in **PaintListener** interfaces, and its designers strongly recommend painting with Controls only in response to **PaintEvent**s. If an application must update its graphics for another reason, they recommend using the control’s **redraw**() method, which adds a paint request to the queue. Afterward, you can invoke the **update**() method to process all the paint requests associated with the object.

It’s important to remember that, although painting in a **PaintListener** is recommended for **Control** objects, **Device** and **Image** objects can’t use this interface. If you need to create graphics in an image or device, you must create a separate GC object and dispose of it when you’re finished.

### Clipping and Canvas styles
By default, the area available for drawing with a graphic context is the same as that of its associated **Control**. However, the GC provides methods that establish bounds for its own graphical region, called the <u>clipping region</u>. The **setClipping**() method specifies the limits for the GC’s graphics, and the **getClipping**() method returns the coordinates of the clipping region.
The concept of clipping is also important when you’re dealing with **PaintEvent**s. Not only do these events fire whenever a **Drawable** object is covered by another window, but they also keep track of the area being obscured. That is, if a user covers part of a Canvas with a second window, the **PaintEvent** determines which section has been clipped and sets its x, y, height, and width fields according to the smallest rectangle that encloses the concealed region. This is necessary since repainting refreshes only this clipped region, not the entire object.
If multiple sections of a **Control** object are obscured, then by default, the object merges these sections into a single region and requests that it be repainted. However, if an application requires that separate requests be made for each concealed area, then the **Control** should be constructed with the **NO_MERGE_PAINTS** style. This is the first of the styles associated with the **Composite** class but specifically intended for **Canvas** objects. 
![[Graphical Editing Framework-20231210-7.png]]
Normally, when a user clicks a window, any keyboard input is directed to it. This property is called<u> focus behavio</u>r, and you can remove it from a **Canvas** object by constructing the object with the **NO_FOCUS** style. Similarly, when a **Canvas** is resized, a **PaintEvent** is triggered by default and the display is repainted. You can change this default behavior by using the **NO_REDRAW_RESIZE** style. It’s important to note, though, that using this style may cause graphical artifacts during a resize operation.
Before a graphic context draws its images, its Canvas paints itself with the color of its shell, the default background color. These paint operations can cause screen flicker on certain displays. You can prevent this by using the **NO_BACKGROUND** style, which prevents the first painting. Without a background color, the graphic context must cover every pixel of the **Canvas**, or it will take the appearance of the screen behind the shell.

### Programming with colors
One of the fundamental aspects of any graphical toolset is its use of colors. The theory behind colors is straightforward, but their practical usage requires explanation. This section will discuss how colors are represented in SWT and how to allocate and deallocate them in a program. It will also discuss two classes provided by the JFace library that simplify the process of working with colors.

#### Color development with SWT
Because monitors use light to provide color, it makes sense to use light’s primary colors—red, green, and blue (RGB)—to represent the colors of a display. This color system is additive, which means that colors are generated by adding red, green, and blue elements to a black field. For example, if 24 bits are used to specify the RGB value at a point, then black (the absence of light) is represented in hexadecimal as 0x000000, and white (the combination of light) as 0xFFFFFF. SWT follows this course by providing classes and methods that access and use RGB objects.
This concept may seem simple, but SWT’s designers faced a serious challenge in implementing it on multiple platforms. The problem involved providing a standard set of colors despite variations in display resolution and color management policy. In the end, they decided on a two-part solution. 
First, SWT provides a set of 16 basic colors (called system colors) using the display’s **getSystemColor**() method. This method takes an integer representing one of SWT’s color constants and returns a Color object. 
![[Graphical Editing Framework-20231210-8.png]]
If you want to use colors that fall outside this set, you must allocate a Color object according to its RGB values. You do so by invoking one of two constructor methods associated with the Color class. If a display’s resolution is too low to show this color, then it will use the system color with the nearest RGB value.
![[Graphical Editing Framework-20231210-9.png]]
In both constructors, the first argument is an object of the Device class. Afterward, the color’s RGB value is set according to three integers between 0 and 255, or an instance of the RGB class. This RGB class, whose constructor is RGB(int, int, int), is used to describe a color according to the values of its elements. It’s important to remember that creating an RGB instance doesn’t create a color and that an RGB object doesn’t require disposal.

```
public class Colors extends Canvas
{
	public Colors(Composite parent)
	{
		super(parent, SWT.NONE);
		setBackground(this.getDisplay().
		getSystemColor(SWT.COLOR_DARK_GRAY)); // User system color for Cavas
		addPaintListener(drawListener);
	}
	
	PaintListener drawListener = new PaintListener()
	{
		public void paintControl(PaintEvent pe)
		{
			Display disp = pe.display;
			Color light_gray = new Color(disp, 0xE0, 0xE0, 0xE0); // Create color
												// object based on RGB value
			GC gc = pe.gc;
			gc.setBackground(light_gray);
			gc.fillPolygon(new int[] {20, 20, 60, 50, 100, 20});
			gc.fillOval(120, 30, 50, 50);
			light_gray.dispose(); // Deallocate Color object when finished
		}
	};
}
```
#### Additional color capability with JFace
JFace uses the same color methodology as SWT. It also provides two interesting classes to simplify color manipulation: **JFaceColors**, located in the **org.eclipse.jface.resource** package; and **ColorSelector**, located in the **org.eclipse.jface.preference** package.
##### The JFaceColors class
The **JFaceColors** class contains a number of static methods that you can use to obtain colors in an Eclipse Workbench application. **getBannerBackground**() returns the color of an application’s banner, and **getErrorBorder**() returns the border color of widgets that display errors. There are also methods that return colors of different kinds of text.

The **JFaceColors** class also provides a useful method that can be invoked in both SWT and JFace applications: **setColors**(), which you can use to set both the foreground and background colors of a widget at once. The following code snippet makes the button’s foreground color red and its background color green:
```
Button button = new Button(parent, SWT.NONE);
red = display.getSystemColor(SWT.COLOR_RED);
green = display.getSystemColor(SWT.COLOR_GREEN);
JFaceColors.setColors(button,red,green);
```
There is also a **disposeColors**() method, which despite its described capability of deallocating all colors at once, can’t replace the dispose() method in the Color class. Instead, it’s meant to perform additional tasks when the workbench disposes of its color resources.

##### The ColorSelector class
Another class offered by the JFace toolset lets the user choose colors in an application. Although the **ColorSelector** is part of JFace’s Preference framework, we felt it necessary to mention its capability here. In essence, this class adds a button to an instance of SWT’s **ColorDialog** class. 
The **ColorSelector** sets and retrieves the RGB value corresponding to the user’s selection. The **setColorValue**() method sets the default selection as the dialog box is created. The **getColorValue**() method converts the user’s selection into an RGB object that can be used to allocate the color. This is shown in the following code snippet:
```
ColorSelector cs = new ColorSelector(this);
Button button = cs.getButton();
RGB RGBchoice = cs.getColorValue();
```
Colors improve the appearance of a GUI, but they don’t convey useful information. A proper user interface needs to communicate with the user. This means adding text, which means working with resources that control how text is presented. These resources are called fonts.

## Displaying text with fonts
This section will present the means of choosing, allocating, and deallocating fonts with the SWT toolset. Then, we’ll show you how to simplify these tasks with JFace. In keeping with its goal of maintaining a native look and feel, the SWT/JFace toolkit relies primarily on fonts provided by the operating system. Unfortunately, these fonts vary from one platform to another. Therefore, when this section describes a font’s name, it means the name of one of the fonts installed on your system.

### Using fonts with SWT
SWT provides a number of font-related classes that perform one of three functions. The first involves font management—allocating and deallocating Font objects. The second function is implementing fonts in objects to change the display of their text. Finally, SWT contains methods that provide measurements of text dimensions for use in graphical applications.
#### Font management
**FontData** objects provide the basic data for creating Font instances. This data consists
of three parts, which are also the three arguments to the most common **FontData** constructor method:
```
FontData(String name, int height, int style)
```
The first argument represents the name of the font, such as Times New Roman or Arial. The height refers to the number of points in the font, and the style represents the type of font face: **SWT.NORMAL**, **SWT.ITALIC**, or **SWT.BOLD**.
In addition, you can customize a **FontData** object by specifying its locale (the application’s geographic location and the set of characters that should be used). A font’s locale can be determined by invoking the **getLocale**() method and specified with **setLocale**().
Neither **RGB** or **FontData** instances need to be disposed of, but Font objects require allocation and deallocation. 
![[Graphical Editing Framework-20231210-10.png]]
There is only one deallocation method for the Font class: **dispose**(). You should invoke it shortly after the Font’s last usage.

##### Implementing fonts in objects
In SWT, fonts are generally associated with one of two GUI objects: **Controls** and **GCs**. When you use the **setFont**() method associated with Controls, any text presented with a **setText**() method is displayed with the specified font. Graphic contexts also use the **setFont**() method, but they provide a number of different methods for painting text in its clipping region. 
![[Graphical Editing Framework-20231210-11.png]]
Because of the overloaded **drawString**() and **drawText**() methods, this table requires some explanation. Although implementations of **drawString**() and **drawText**() have the same argument types and functions, the difference is that <u>drawText() processes carriage returns and tab expansions</u>, whereas <u>drawString() disregards them</u>. Also, the two integers following the String argument represent the coordinates of the text display.

The Boolean argument in the second and fourth methods <u>indicates whether the text background should be transparent</u>. If this value is set to TRUE, then the color of the rectangle containing the text won’t be changed. If it’s FALSE, the color of the rectangle will be set to that of the graphic context’s background.

The third integer in the last drawText() method represents a flag that changes the text display. These flags are as follows:
- DRAW_DELIMITER—Displays the text as multiple lines if necessary
- DRAW_TAB—Expands tabs in the text
- DRAW_MNEMONIC—Underlines accelerator keys
- DRAW_TRANSPARENT—Determines whether the text background will be the same color as its associated object
Many of these flags are implemented in the code that follows.

##### Measuring font parameters
When incorporating text in GUIs, you may want to know the text’s dimensions, which means knowing the measurements of a given font. SWT provides this information through its **FontMetrics** class, which contains a number of methods for determining these parameters. 
![[Graphical Editing Framework-20231210-12.png]]
This class has no constructor methods. Instead, the GC object must invoke its **getFontMetrics**() method. It returns a **FontMetrics** object for the font used in the graphic context and lets you use the listed methods. Each returns an integer that measures the given dimension according to the number of pixels.


```
public class Fonts extends Canvas
{
	static Shell mainShell;
	static Composite comp;
	FontData fontdata;
	public Fonts(Composite parent)
	{
		super(parent, SWT.BORDER);
		parent.setSize(600, 200);
		addPaintListener(DrawListener);
		comp = this;
		mainShell = parent.getShell();
		
		Button fontChoice = new Button(this, SWT.CENTER);
		fontChoice.setBounds(20,20,100,20);
		fontChoice.setText("Choose font");
		fontChoice.addMouseListener(new MouseAdapter()
		{
			public void mouseDown(MouseEvent me)
			{
				FontDialog fd = new FontDialog(mainShell); // Open FontDialog box
				fontdata = fd.open();
				comp.redraw();
			}
		});
	}
	PaintListener DrawListener = new PaintListener()
	{
		public void paintControl(PaintEvent pe)
		{
			Display disp = pe.display;
			GC gc = pe.gc;
			gc.setBackground(pe.display.getSystemColor(SWT.COLOR_DARK_GRAY));
			if (fontdata != null)
			{
				Font GCFont = new Font(disp, fontdata);// Create Fond based on user
				gc.setFont(GCFont);
				FontMetrics fm = gc.getFontMetrics(); // Measure properties of font
				gc.drawText("The average character width for this font is " +
				fm.getAverageCharWidth() + " pixels.", 20, 60);
				gc.drawText("The ascent for this font is " +
				fm.getAscent() + " pixels.", 20, 100, true);
				gc.drawText("The &descent for this font is " 
					+ fm.getDescent()
					+ " pixels.", 20, 140, SWT.DRAW_MNEMONIC|SWT.DRAW_TRANSPARENT);
				GCFont.dispose();
			}
		}
	};
}
```
Once the user clicks the Choose Font button, the MouseEvent handler creates an instance of a FontDialog and makes it visible by invoking the dialog’s open() method. This method returns a FontData object, which is used in the DrawListener interface to create a font for the graphic context. This GC object, created by the PaintEvent, then invokes its getFontMetrics() method to measure the parameters of the font.

When the graphic context sets its foreground color to SWT.COLOR_DARK_GRAY, this usually means that all text created by the GC will be surrounded by this color. However, only the first drawText() method is surrounded by the foreground color; this is because the second and third invocations are considered transparent and take the color of the underlying Canvas. The third drawText() method also enables mnemonic characters, which means that an ampersand (&) before a letter results in the display underlining this character.

In the DrawListener interface, a great deal of the processing is performed only after the FontData object has been set by the FontDialog. This is necessary since errors will result if the FontData argument is null. Also, since the graphic context only draws its text after a PaintEvent, the MouseAdapter ends by invoking the redraw() method, which causes the Canvas to repaint itself.

### Improved font management with JFace
One of the main functions of SWT’s graphics is to provide font management—the allocation and deallocation of font resources. This can be accomplished with the Font constructor and dispose() methods, but there is no efficient way to manage multiple fonts in a single
application. JFace provides this capability with its **FontRegistry** class, located in the **org.eclipse.jface.resource** package.
By using a **FontRegistry**, you don’t need to worry about the creation or disposal of Fonts. Instead, the **FontRegistry**’s **put**() method lets you match a String value with a corresponding **FontData**[] object. This method can be invoked multiple times to add more Fonts to the registry. Then, when the application needs a new Font to change its text display, it calls the registry’s **get**() method, which returns a Font object based on the argument’s String value. 
```
FontRegistry fr = JFaceResources.getFontRegistry();
fr.put("User_choice", fontdialog.getFontList());
fr.put("WingDings", WDFont.getFontData());
Font choice = fr.get("User_choice");

```
Rather than create an empty registry, this code uses the preexisting **FontRegistry** associated with JFace and adds two more fonts. The first font is placed in the registry from the result of a **FontDialog** selection, and the second is taken from a font that existed previously in the application. In the last line, the **FontRegistry** converts the **FontData**[] object associated with the **FontDialog** into a Font instance. Just as the **FontRegistry** manages the creation of this font, it also performs its disposal, as well as that of every font in its registry.

The fonts in the **FontRegistry** include those used in the Eclipse Workbench’s banners and dialog boxes. However, you need the **JFaceResources** class to access them. The following code shows how this can be performed. It’s important to note that the Strings used to invoke the registry’s get() method, as well as the **FontRegistry** itself, are member fields in JFaceResources:
```
FontRegistry fr = JFaceResources.getFontRegistry();
Font headFont = fr.get(JFaceResources.HEADER_FONT);
Font dialogFont = fr.get(JFaceResources.DIALOG_FONT);
```
![[Graphical Editing Framework-20231210-13.png]]


## Incorporating images in graphics
SWT provides classes and methods for image management and integration in much the same way that it provides for font handling. Also, JFace provides built-in resources and registries that reduce the amount of complexity involved with image management.
### Allocating images
Most applications only create Image objects to add existing image files to a user
interface. In this case, you should use the first and simplest of the Image constructor methods:
```
Image(Device, String)
```
Applications seeking to present the image in a GUI invoke this method using the Display object as the first argument and the image file’s pathname as the second. As of the time of this writing, SWT accepts .jpg, .gif, .png, .bmp, and .ico file types.
If the image file resides in the same directory as a known class, then an InputStream can be generated by invoking the class’s getResourceAsStream() method. With this InputStream, you can use the second constructor method:
```
InputStream is = KnownClass.getResourceAsStream("Image.jpg");
Image Knownimage = new Image(Display, is);
```
![[Graphical Editing Framework-20231210-14.png]]
The third and fourth constructor methods create empty Image instances with dimensions set by the method’s arguments. The two integers specify the x and y parameters of the image, and the Rectangle object in the fourth method frames the image according to its boundaries. The fifth creates an Image based on a second Image instance and an integer flag that determines whether the image should appear disabled or in grayscale.

The last two constructor methods construct Image instances using objects of the **ImageData** class. This class provides device-independent information about an Image object and contains methods to manipulate the image. Like the FontData class, instances of **ImageData** don’t use operating system resources and don’t require deallocation. Image instances, however, need to invoke their **dispose**() methods when they’re no longer in use.

First, it’s important for you to understand how images are integrated in applications. The process of adding an Image to a GUI begins with creating a graphic context. This GC object then calls its **drawImage**() method, which takes one of two forms, based on whether the image will be presented with its original dimensions

```
public class ImageTest extends Composite
{
	public ImageTest(Composite parent)
	{
		super(parent, SWT.NONE);
		parent.setSize(320,190);
		
		// create ImageData object from file
		InputStream is = getClass().getResourceAsStream("eclipse_lg.gif");
		final ImageData eclipseData = new ImageData(is).scaledTo(87,123);
		
		this.addPaintListener(new PaintListener()
		{
			public void paintControl(PaintEvent pe)
			{
				GC gc = pe.gc;
				
				// Create Image from ImageData
				Image eclipse = new Image(pe.display, eclipseData);
				gc.drawImage(eclipse, 20, 20);
				
				gc.drawText("The image height is: " 
					+ eclipseData.height 
					+ " pixels.",120,30);
				gc.drawText("The image width is: " 
					+ eclipseData.width 
					+ " pixels.",120,70);
				gc.drawText("The image depth is: " 
					+ eclipseData.depth 
					+ " bits per pixel.",120,110);
				eclipse.dispose();
			}
		});
	}
}
```
This code begins by constructing an **ImageData** object using an InputStream. In this case, it makes sense to start with an **ImageData** instance since Image objects can’t be resized or recolored. This resizing process is performed using the **scaleTo**() method, which shrinks the image for the GUI.
When a **PaintEvent** occurs, the program invokes the **paintControl**() method. This method creates the window’s graphic context and an Image object based on the **ImageData**. To the right of the image, three statements provide information regarding the fields of the ImageData instance. It’s worth noting that by changing the coordinates, you can superimpose the text (or any graphic) on the image.

### Creating a bitmap with ImageData
The easiest way to learn about ImageData is to design, build, and display an instance of this class. In this case, you’ll create a bitmap and use it to form an Image. Doing so will introduce many of the fields and methods associated with the ImageData class and provide a better idea why this class is necessary.
The first step involves determining which colors will be used.  To tell ImageData about the colors you’ll be using, you need to create an instance of the PaletteData class. This object contains an array of the RGB values in the image. It consists of three elements: 0x000000 (black), 0x808080 (gray), and 0xFFFFFF (white).
Each pixel in an image has three pieces of information: its x coordinate, or offset; its y coordinate, or scanline; and the pixel’s color, which is referred to as its value or index. Because this image contains only three colors, you don’t need to assign each pixel its full RGB value (0x000000, 0x808080, 0xFFFFFF). Instead, it’s simpler and less memory-intensive to use the color’s index in the **PaletteData** array and assign pixel values of (0, 1, 2). This simplified mapping between a pixel’s value and color is referred to as an indexed palette. For example, because the eclipse_lg.gif file used in the last code snippet has a depth of only 8 bits per pixel, each pixel is assigned a value between 1 and 28 (255).
However, for images with depth greater than 8 bits per pixel, the additional processing needed to translate between an index and its color isn’t worth the reduced memory. These images use a direct palette, which directly assigns a pixel’s value to its RGB value. The **isDirect**() method in the **PaletteData** class tells whether an instance uses direct or indexed conversion.

```
public class FlagGraphic
{
	public FlagGraphic()
	{
		int pix = 20;
		int numRows = 6;
		int numCols = 11;
		PaletteData pd = new PaletteData(new RGB[] // Create color palette
		{
			new RGB(0x00, 0x00, 0x00),
			new RGB(0x80, 0x80, 0x80),
			new RGB(0xFF, 0xFF, 0xFF)
		});
		final ImageData flagData = new ImageData(pix*numCols, pix*numRows, 2, pd);
		
		for(int x=0; x<pix*numCols; x++) // Set value of each pixel in image
		{
			for(int y=0; y<pix*numRows; y++)
			{
				int value = (((x/pix)%3) + (3-((y/pix)%3))) % 3;
				flagData.setPixel(x,y,value);
			}
		}
	}
}
```
This example begins by creating a PaletteData object as an array of RGB objects corresponding to black, gray, and white colors. Then, an ImageData instance is constructed with the dimensions of the image, the depth of the image, and the PaletteData. Because there are three possible colors, the depth is set to 2, which provides support for up to 2^2 (4) colors. If an image’s color depth is 1, 2, 4, or 8, then the application creates an indexed palette for the ImageData object during its allocation process. For images with greater depth, the application will create a direct palette.
	NOTE
	If the user attempts to create a palette with a depth outside the set of {1,
	2, 4, 8, 16, 24, 32}, the compiler will throw an ERROR_INVALID_ARGUMENT.
![[Graphical Editing Framework-20231210-15.png]]
It’s important to remember that, because **flagData** only works with RGB, **PaletteData**, and **ImageData** objects, no **dispose**() methods need to be invoked. Deallocation is only necessary when an **ImageData** instance is used to create an Image or if the RGB values are used to create Colors.

### Manipulating images with ImageData
Along with the bitmap methods described so far, the **ImageData** class also contains methods that provide graphical effects. You can set pixels in an image to provide transparency instead of a color. Using alpha blending, two images can be combined into an image that contains elements of both. Finally, using the **ImageData** and **ImageLoader** classes, you can sequence images into animated GIF files.
#### Transparency
With sufficient color depth, the RGB system can provide any color in the visible spectrum. However, this won’t help if you want sections of the image to be transparent. No combination of red, green, and blue elements will add up to a see-through color, so you need to set a specific pixel value to represent transparency. This way, any pixel with this value will instead take the color of the background behind it. This capability is provided with the transparentPixel field of the ImageData class. This is simple to use in code, as the following snippet shows:
```
flagData.transparentPixel = 2;
Image flagImage = new Image(pe.display, flagData);
gc.drawImage(flagImage, 20, 20);
```
In this code, any pixels in **FlagImage** with the value of 2 (representing white) take the color of the image’s background. 
The right image is the result of setting the transparentPixel value to 1 (representing gray).
In addition to the transparentPixel field, the ImageData class contains a number of methods that provide information about transparency. The getTransparencyMask() method returns an ImageData object with its transparent pixels separated in a mask array. The **getTransparencyType**() method returns an integer representing the type of transparency used. In many image-editing toolsets, a program can specify different degrees of transparency in an image. However, since there is no **setTransparencyType**() method at the time of this writing, this feature has yet to be integrated in SWT.
**Transparency** is a helpful capability, but it’s still static. It would be much more striking to create a series of images and display them at short time intervals to provide the illusion of continuous motion. 

#### Saving and animating images
Of the many common types of images, the Graphics Interchange Format (GIF) is the only one that supports animation. Therefore, before we can discuss animation in depth, we need to describe how SWT’s Image objects are saved as image files. This means looking into SWT’s **ImageLoader** class.
Like the **ImageData** constructors, the **ImageLoader** class contains methods that accept image files and streams and return ImageData[] objects. However, this class’s main purpose involves converting ImageData[] instances into OutputStreams and image files. This way, graphics can be persisted instead of being disposed with their Image objects. 
![[Graphical Editing Framework-20231210-16.png]]

### Managing images with JFace
The ImageRegistry class lets you incorporate multiple images in an application without worrying about resource deallocation. It also uses the same access methods as the FontRegistry class. To place an image in the registry, you use the put() method with an
Image object and a String. When the image needs to be displayed, the get() method returns the image based on the key. 
```
ImageRegistry ir = new ImageRegistry();
ir.put("Eclipse", new Image(display, "eclipse_lg.gif"));
Image eclipse = ir.get("Eclipse");
```
In this case, you still need to allocate resources for the Image object. This may cause a problem if the application places many images in the registry but only needs to display a few. For this reason, JFace created the **ImageDescriptor** class. Like SWT’s **ImageData** class, the **ImageDescriptor** contains the information needed for an image without requiring allocation of system resources. The **get**() and **put**() methods associated with the **ImageRegistry** class are also available for **ImageDescriptor** objects:
```
ImageRegistry ir = new ImageRegistry();
ImageDescriptor id = createFromFile(getClass(),"eclipse_lg.gif");
ir.put("Eclipse", id);
Image eclipse = ir.get("Eclipse");
```
If you use **ImageDescriptors**, the **put**() and **get**() operations can be performed without allocating for Image objects. This way, you can add a large number of **ImageDescriptors** to an application’s **ImageRegistry** without worrying about image creation. Finally, since an **ImageRegistry** disposes of its contents when its **Display** object is closed, you don’t need to concern yourself with image deallocation.



# Draw2D
Draw2D relies on many constructs from SWT (not JFace), but you must draw and design most of the graphical components within it. You also need to specify their event responses and any drag-and-drop capability. Essentially, the Draw2D library serves as a complete graphics package, with the additional feature that these graphics can be moved and associated with events.
### Using Draw2D’s primary classes
If you’ve understood our discussion of SWT so far, then Draw2D won’t present much difficulty. As shown in table C.1, the two libraries use related classes and provide similar structures for drawing, event handling, and component layout. In fact, all Draw2D GUIs must be added to an SWT Canvas. The first difference is that whereas a normal Canvas adds a GC object to provide graphics, a Canvas in a Draw2D application uses an instance of a LightweightSystem.
![[Graphical Editing Framework-20231203-1.png]]
**LightweightSystems** function similarly to **Display** objects in SWT. <u>They have no visual representation but provide event handling and interaction with the external environmen</u>t. As the name implies, **LightweightSystems** operate at a level removed from the operating system. This means you lose the advantages of SWT/JFace’s heavyweight rendering, such as its rapid execution and native look and feel. However, now you can truly customize the appearance and operation of your components.
Without question, the most important class in Draw2D is the **Figure**, and the majority of our Draw2D discussion will focus on its methods and subclasses. Like an SWT **Shell**, it must be added to a **LightweightSystem** in order to provide a basis for the GUI’s appearance. Like an SWT Control, it provides for resizing and relocating, adding **Listeners** and **LayoutManagers**, and setting colors and fonts. It also functions like a **Composite** in SWT, providing methods for adding and removing other Figure **objects**—children—within its frame. However, unlike Widget and Control objects, you can easily subclass Figures. Their graphical aspects can be represented by a drawing or image. 
![[Graphical Editing Framework-20231203-2.png]]
Not only can **Figures** use separate **Listener** interfaces, they can also perform the majority of
their event handling by themselves. <u>Figures can even initiate specific kinds of events to alert other objects in the GUI</u>.
To add images and drawings to **Figures**, you need to use instances of the **Graphics** class. It functions similar to SWT’s **GC** (graphics context) class, and it provides methods for incorporating graphics in a given area. It also contains many of the same methods as GC, particularly for drawing lines and shapes, displaying images, and working with fonts. However, the **Graphics** class provides one very different capability: Its objects can be moved, or translated, within the **LightweightSystem**. This means that when you want to change the position of a graphical component, Draw2D provides its own drag-and-drop capability to translate a **Figure** to the desired location.


## Draw2D Figures
As you’ll see, it takes a fair amount of code to build a Draw2D GUI. However, unlike those in an SWT/JFace GUI, Draw2D elements can be moved and manipulated. These components, including the overall container, are descendents of Draw2D’s main class, **Figure**. This class contains a number of subclasses that produce the visual aspects of the toolset’s GUI. 
![[Graphical Editing Framework-20231216-1.png]]
### Figure methods
Like the SWT **Control** class, the **Figure** class contains many methods for manipulating its properties. We can’t describe all 137 of them, but we can divide them into four main categories:
■ Working with the Figure’s visual aspects
■ Event handling
■ Keeping track of parents and children
■ Managing graphics

#### Working with a Figure’s visual aspects
The methods in the first category are exactly like those in SWT. These include getting and setting the Figure’s bounds, location, and size. The Figure’s maximum and minimum size can be controlled as well as its border size and visible area. This category also provides methods for changing the Figure’s foreground and background color and getting/setting its focus and visibility parameters.
#### Event handling in Draw2D
The process of handling events in Draw2D is also similar to SWT, but it provides a few new events and listeners. Unlike SWT, **Figures** can handle many of their own events. 
![[Graphical Editing Framework-20231216-2.png]]
The first five methods look and act just like those in SWT, with **addListener**() receiving untyped Events. But the last three methods are unique to Draw2D. The **addAncestorListener**() method responds to any changes to the Figure’s ancestors and functions like the Swing implementation. Similarly, the **FigureListener** responds whenever the **Figure** is moved.
The last method, **addPropertyChangeListener**(), lets you create your own events. This process starts by associating a property, such as a Figure’s location, with a String descriptor. Then, when **firePropertyChange**() is invoked with this String, any PropertyChangeListeners listening for this property change respond. We’ll revisit this subject in greater depth when we discuss the GEF and its model classes.
#### Parent and child Figures
SWT and JFace allow components to be included in other components by providing a **Composite** class. But in Draw2D, any **Figure** can be a container, and any **Figure** can be contained. Therefore, Draw2D uses the term parent to refer to the outer graphic and child to refer to the graphic contained within. You create and manipulate these relationships with the methods listed below.
![[Graphical Editing Framework-20231216-3.png]]
In SWT, Buttons and Labels add themselves to Composites by identifying parents in their constructors. In Draw2D, a parent **Figure** uses its **add**() method to include **Figures** in its List of children. This method can include an optional constraint (such as the child’s size or location) and/or an index in the parent’s List. Parent **Figures** can obtain this List with the **getChildren**() method, and children can access their parent with **getParent**(). Parents can also enable or disable children with **setChildrenEnabled**() or alter an aspect of the child with **setConstraint**().
### Managing graphics
Although Draw2D provides a **Graphics** class that performs most of the duties of SWT’s GC, **Figures** have a few graphical methods of their own. Not only can they control their display with **paint**(), they can also use **paintBorder**() and **paintClientArea**() to select which section to show. **Figures** can display their children with **paintChildren**() or use **paintFigure**() to only show themselves. There are also a number of **repaint**() methods available, which work similarly to those in SWT.
Draw2D contains methods for finding information about the user’s selection location. This is different from SWT because Draw2D is particularly concerned with precise mouse movements, whereas an SWT GUI is only concerned about the selected **Control**. These methods include **FindMouseEventAt**(), **FindFigureAt**(), and **FindFigureAt**().

## Labels and Clickables
The first **Figure** subclasses for our investigation are the simplest: **Labels** and **Clickables**. These objects look and act similarly to their SWT counterparts, but there are a few interesting concerns that you need to keep in mind.
### Labels
Draw2D **Labels** resemble those of SWT but contain more methods for <u>text measurement</u> and <u>image location</u>. You can measure the parameters of the Label’s String through **getTextBounds**() and **getTextLocation**(). Similarly, if the Label is associated with an **Image**, then **getIconBounds**() and **getIconAlignment**() will provide information about the Image.

### Clickables
This class, which includes **Buttons** and **Toggles**, provides binary information concerning the user’s preferences. Like SWT Buttons, they can be configured with style bits to appear like toggle buttons, checkboxes, or regular pushbuttons. You can also control their selection and add images or text. But there are two main differences between Draw2D Clickables and SWT Buttons. The first is that **Clickables** can take the appearance of any Draw2D Figure. The second has to do with **Clickable** event handling.
Draw2D user interfaces are generally more complex than those created with SWT and JFace. Therefore, a Clickable’s state information is managed by a **ButtonModel** or **ToggleModel** object. This separates the component’s appearance from its behavior and enables you to develop the two aspects independently. You can also contain these **Model** objects in a **ButtonGroup**, which manages multiple **Clickables** at once.
**Clickables** update their **Model** objects by calling **fireChangeEvent**(), which works like the **Figure**’s **firePropertyChangeEvent**(). **Clickable** properties, such as **MOUSEOVER_PROPERTY** and **PRESSED_PROPERTY**, are represented by constants in the **Model** class. When they change, the **Model** may fire a number of Draw2D events or inform its **ButtonGroup** by default.

```
public class Draw2D_Example
{
public static void main(String args[])
{
	final Label label = new Label("Press a button!");
	Shell shell = new Shell();
	LightweightSystem lws = new LightweightSystem(shell);
	Figure rootFigure = new Figure();
	rootFigure.setLayoutManager(new XYLayout());
	lws.setContents(rootFigure);
	
	Clickable above = new CheckBox("I'm above!");
	rootFigure.add(above, new Rectangle(10,10,80,20));
	ButtonModel aModel = new ToggleModel();
	aModel.addChangeListener(new ChangeListener()
		{
		public void handleStateChanged(ChangeEvent e)
		{
		label.setText("Above");
		}
		});
	above.setModel(aModel);
	
	Clickable below = new CheckBox("I'm below!");
	rootFigure.add(below, new Rectangle(10,40,80,20));
	ButtonModel bModel = new ToggleModel();
	bModel.addChangeListener(new ChangeListener()
		{
		public void handleStateChanged(ChangeEvent e)
		{
			label.setText("Below");
		}
		});
	below.setModel(bModel);
	
	ButtonGroup bGroup = new ButtonGroup();
	bGroup.add(aModel);
	bGroup.add(bModel);
	bGroup.setDefault(bModel);
	rootFigure.add(label, new Rectangle(10,70,80,20));
	
	shell.setSize(130,120);
	shell.open();
	shell.setText("Example");
	Display display = Display.getDefault();
	while (!shell.isDisposed())
	{
		if (!display.readAndDispatch())
		{
			display.sleep ();
		}
	}
}
```
A Draw2D application is just an SWT Shell with a **LightweightSystem** and **Figures**. It’s important to understand that the **ChangeListener** is created by the **button’s Model** and responds to **any mouse action**, including clicks and hovering. Also, because the two Models are added to the ButtonGroup, only one of them can be selected at a time.
In order for the parent **Figure** to understand the **Rectangle** constraints of its children, you must configure it with a **LayoutManager** called **XYLayout**. We’ll now focus on **LayoutManagers** and how they enable you to determine how children are arranged within **Figures**.

## Using LayoutManagers and panes
**LayoutManagers**, like SWT’s Layout classes, specify how child components should be positioned and sized in a container. This section describes **LayoutManager**’s subclasses and how you can use them.
In addition, we’ll go over Draw2D’s panes: **ScrollPanes**, **LayerPanes**, and their subclasses. Draw2D has no Composite class, but these panes generally serve as background containers for its GUIs. As you’ll see, these window-like classes provide a number of capabilities that make it simple to build graphical editors. 

### Understanding LayoutManager subclasses
In SWT, containers rely on a default layout policy for their children; but Draw2D demands that you choose a **LayoutManager** subclass. Two of these, **FlowFigureLayout** and **ScrollBarLayout**, are only useful for specific Figures.
![[Graphical Editing Framework-20231216-4.png]]

### LayeredPanes
Since Draw2D applications can become very complex, a **LayeredPane** provides many levels for displaying **Figures**. Using **transparent Layers**, you can separate the graphical aspects of your GUIs. Different Layers can have different properties, including separate **LayoutManagers**. This will be important for our editor, because we add not only **Figures** but also **Connections** and **feedback**.
The first step in understanding how **LayeredPanes** work is learning about **Layers**. These transparent objects can be manipulated as individual **Figures**, and the **Layer** class contains two methods of its own: **containsPoint**() and **findFigureAt**(). These objects have fixed boundaries, but the **FreeformLayer** subclass can be extended in all directions. This is necessary for a graphical editing application whose drawings are larger than the window’s visible region.
A **LayeredPane** adds new Layers with its **add**() method, which specifies the **Layer** object, a key to identify it, and an index representing its position. It can also remove **Layers** or change their positions in its stack.
By choosing subclasses of **LayeredPane**, you can increase its capabilities. If you’d like to be able to zoom in on sections of the pane, choose **ScalableLayeredPane**. Use **FreeformLayeredPane** if you’d like to extend the window in all directions. If you want both capabilities, use **ScalableFreeformLayeredPane**.

### ScrollPanes and Viewports
ScrollPanes classes are easy to understand and function by creating Scrollbars on top of another Figure. The bars’ visibility can be configured so that they are always showing, never showing, or shown only when needed.
At any given time, only a section of the ScrollPane can be seen. This visible region is called a **Viewport**. These work similarly to **Layers** but provide more methods for controlling size and shape. There is also a **FreeformViewport** for panes that can be extended in all directions.

#### Using the Graphics class to create Shapes
In SWT, graphic contexts (GCs) can either be created as separate objects or obtained as part of a PaintEvent. But in Draw2D, a **Figure** can acquire a **Graphics** object by one of the paint methods. The vast majority of the **Graphics** methods are exactly the same as those in GC; the only important difference is that Draw2D allows a **Graphics** object to move through its **translate**() method.

### Draw2D geometry and graphs
For higher-precision measurements, Draw2D provides **PrecisionPoints**, **PrecisionRectangles**, and **PrecisionDimensions**. It contains **Ray** objects that function like mathematical vectors and a **Transform** class to translate, rotate, and scale graphical **Points** and dimensions.
The **org.eclipse.draw2d.graph** package contains a number of useful classes for creating and analyzing directed graphs. Along with basic **Nodes** and **Edges**, this package also provides a **DirectedGraphLayout** for arranging them. 
Draw2D’s Figures aren’t connected by **Edges**, but by **Connection** objects. 

## Understanding Connections
The **FixedAnchor** class has figured prominently in our code listings so far. These objects (subclasses of **AbstractConnectionAnchor**) enable you to add lines, or **Connections**, between two **Figures**. Because Connections create relationships between components, they’re fundamental in system models and diagrams. However, managing **Connections** and their **ConnectionAnchors** can be complicated, so it’s important that you understand how they function.
### Working with ConnectionAnchors
**ConnectionAnchors** don’t have a visual representation. Instead, they specify a point on a **Figure** that can receive **Connections**. You add them by identifying the **Figure** in the **ConnectionAnchor**’s constructor method. This Figure is called the anchor’s owner, not its parent.
The difficulty in working with anchors isn’t adding them, but placing them appropriately. For this reason, the only method required by the **ConnectionAnchor** interface is **getLocation**(). The **getLocation**() method is called whenever the owner’s location changes. The **Point** returned by the method tells the GUI where the anchor should be positioned. 
### Adding Connections to the GUI
Working with **Connections** is easier than dealing with their anchors, because Draw2D takes care of drawing the line. Draw2D’s implementation of the **Connection** interface is **PolylineConnection**, which is a connected line. **Connections** are more than connected lines. We need to set their source and target **Figures**, and we can configure their appearance and routing. With regard to appearance, you can add decorations to the start and end of the
**Connection** by calling **setSourceDecoration**() or **setTargetDecoration**(). 
In addition to decorators, you can add **Labels** or other **Figures** to **Connections** by using a **ConnectionEndpointLocator**. These objects are created with a **Connection** object and a boolean value representing whether the **Figure** should be added at the start or end. Then, **setVDistance**() tells the new **Figure** how far away it should be from the **Connection**, and **setUDistance**() specifies the distance to the **Connection**’s source or target.
The **Connection**’s router refers to the path it takes from one anchor to the next. The four subclasses of **AbstractConnectionRouter** are listed below:
![[Graphical Editing Framework-20231216-5.png]]

## Drag-and-drop in Draw2D


# Zest

## GraphViewer
The Zest GraphViewer class provides similar services to the various JFace viewers. Create the following methods to initialize the viewer field and set the viewer’s input:

```
private GraphViewer viewer;

private Control createDiagram(Composite parent) {
	viewer = new GraphViewer(parent, SWT.NONE);
	return viewer.getControl();
}
private void setModel(GenealogyGraph newGraph) {
	viewer.setInput(newGraph);
}
```

## Content Provider
Before the Zest diagram can be displayed, we must specify a content provider. Because Zest is a diagram with connections or “relationships,” we cannot implement **IStructuredContentProvider** from JFace but instead must implement one of the four Zest subinterfaces that have specialized methods for specifying both the objects in the diagram and the relationships between those objects. Each of these four interfaces takes a different approach to adapting a model to the diagram and is appropriate for adapting different
types of models.
-  **IGraphEntityContentProvider**—best for models that do not have elements representing the connections between other model objects 
-  **IGraphEntityRelationshipContentProvider**—best for models that have elements representing both concrete objects and the connections between those objects 
- **IGraphContentProvider**—best for models that are relationship-centric and have elements representing the connections between concrete objects rather than the concrete objects themselves 
-  **INestedContentProvider**—used in conjunction with one of the content providers above to display nested content 

### Presentation
To make diagrams look better and be more informative, the **GraphViewer**’s label provider can implement any combination of the following interfaces to adjust the presentation:
• ILabelProvider (required)
• IColorProvider
• IFigureProvider
• ISelfStyleProvider
• IEntityStyleProvider
• IEntityConnectionStyleProvider
• IConnectionStyleProvider

#### Label Provider
The GraphViewer’s default label provider uses the toString() method to provide text for both nodes and connections. 

# Graphical Editing Framework

## GEF Architecture
The GEF framework is designed using the Model-View-Controller (MVC) architecture. MVC
is comprised of three major components: the model, the view and the controller. The **model** holds the information to be displayed and is persisted across sessions. The **view** renders information on the screen and provides basic user interaction. The controller coordinates the activities of the model and the view, passing information between them as necessary.
**Model <—> Controller <—> View**
When an object is added to the GEF model, a corresponding **controller** is instantiated which then creates the **view** objects representing that **model** object. In the reverse direction, when the user interacts with the view, the controller updates the model with the new information.
- **model**—the underlying objects being represented graphically 
- **view**—a GEF canvas containing a collection of figures. The View aspect is generally implemented as a **Figure** or **Image**.
- **controller**—a collection of GEF edit parts. The Controller is a subclass of **AbstractGraphicalEditPart**

GEF provides various **edit part** (controller) and **figure** (view) classes for you to extend, minimizing the effort necessary to use the graphical framework in your application.

The GEF project is made up of three subsections:
• **Draw2D**—(org.eclipse.draw2d.*) a lightweight framework layered on top of SWT for rendering graphical information.
• **GEF Framework**—(org.eclipse.gef.*) an MVC framework layered on top of Draw2D to facilitate user interaction with graphical information.
• **Zest**—(org.eclipse.zest.*) a graphing framework layered on top of Draw2D for graphing. 


## GEF Model
First, the **model** is responsible for <u>providing all data that will be viewable and modifiable by the user</u>; no data should be stored in the controller or view. Next, the model needs to provide a way for the data to be persisted across sessions. Finally, while the controller will have references to the model, the model should not reference the controller or view.
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
When rendering a figure, the Graphics class provides many drawing primitives for things such as displaying text, drawing lines, and drawing and filling polygons. Some of the useful Graphics methods include:
**drawFocus**(Rectangle)—Draws a focus rectangle.
**drawImage**(Image, Point)—Draws the given image at a point.
**drawLine**(Point, Point)—Draws a line between the two specified points using the current foreground color.
**drawPolygon**(PointList)—Draws a closed polygon defined by the given PointList containing the vertices. The first and last points in the list will be connected.
**drawRectangle**(Rectangle)—Draws the given rectangle using the current foreground color.
**drawRoundRectangle**(Rectangle, int, int)—Draws a rectangle with rounded corners using the foreground color. The last two arguments represent the horizontal and vertical diameter of the corners.
**drawText**(String, Point)—Draws the given string at the specified location using the current font and foreground color. The background of the text will be transparent. Tab expansion and carriage return processing are performed.
**fillPolygon**(PointList)—Fills a closed polygon defined by the given PointList containing the vertices. The first and last points in the list will be connected.
**fillRectangle**(Rectangle)—Fills the given rectangle using the current background color.
**fillRoundRectangle**(Rectangle, int, int)—Fills a rectangle with rounded corners using the background color. The last two arguments represent the horizontal and vertical diameter of the corners.
**fillText**(String, Point)—Draws the given string at the specified location using the current font and foreground color. The background of the text will be filled with the current background color. Tab expansion and carriage return processing are performed.
**getBackgroundColor**()—Returns the background color.
**getForegroundColor**()—Returns the foreground color.
**getLineStyle**()—Returns the line style.
**getLineWidth**()—Returns the current line width.
**rotate**(float)—Rotates the coordinates by the given counter-clock-wise angle. All subsequent painting will be performed in the resulting coordinates. Some functions are illegal when a rotated coordinates system is in use. To restore access to those functions, it is necessary to call restore() or pop() to return to a non rotated state.
**setBackgroundColor**(Color)—Sets the background color.
**setBackgroundPattern**(Pattern)—Sets the pattern used for fill-type graphics operations. The pattern must not be disposed while it is referenced by the Graphics object.
**setForegroundColor**(Color)—Sets the foreground color.
**setForegroundPattern**(Pattern)—Sets the foreground pattern for draw and text operations. The pattern must not be disposed while it is referenced by the Graphics object.
**setLineStyle**(int)—Sets the line style to the argument, which must be one of the constants SWT.LINE_SOLID, SWT.LINE_DASH, SWT.LINE_DOT, SWT.LINE_DASHDOT or SWT.LINE_DASHDOTDOT.
**setLineWidth**(int)—Sets the line width.
**translate**(Point)—Translates the receiver’s coordinates by the specified x and y amounts. All subsequent painting will be performed in the resulting coordinate system. Integer translation used by itself does not require or start the use of the advanced graphics system in SWT. It is emulated until advanced graphics are triggered.

GEF provides a number of basic figure classes that can be extended or composed to build more complex figures. These classes include:
**Button**—A figure typically having a border that appears to move up and down in response to being pressed.
**Ellipse**—A figure that draws an ellipse filling its bounds.
**ImageFigure**—A figure containing an image. Use this Figure, instead of a Label, when displaying Images without any accompanying text.
**Label**—A figure that can display text and/or an image.
**Panel**—A figure container which is opaque by default.
**Polygon**—A figure similar to a Polyline except it is closed and filled.
**Polyline**—A figure rendered as a series of line segments.
**PolylineConnection**—A figure used to visually connect other figures.
**RectangleFigure**—A rectangular figure with square corners.
**RoundedRectangle**—A rectangular figure with round corners.
**TextFlow**—A figure for displaying a string of text across multiple lines.
**Thumbnail**—A figure displaying an image at reduced size with the same aspect ratio as the original figure.
**Triangle**—A figure rendering itself as a triangle.

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
( Appendix D of SWT/JFace in Action)
### MVC interaction with Palette
The GEF editing modification requires a complex data exchange involving **Tools**, **Requests**, **EditPolicys**, **Commands**, and **PropertyChangeEvents**.  When the user causes an event, a
**Tool** object sends a **Request** to the selected **EditPart**. This **EditPart** uses a List of **EditPolicys** to create a **Command** that updates the **Model** class. When the **Model** changes, it fires a **PropertyChangeEvent**. After receiving this event, the **EditPart** modifies the component’s **Figure** (View) by invoking one of its **refresh**() methods.

![[Graphical Editing Framework-20231216-6.png]]

### PaletteViewer (Tool, ToolEntry, TemplateEntry, PaletteRoot)
The PaletteViewer class builds components and handles events for the lefthand section of the Canvas, whose default width is 125 pixels. Like the viewer on the right-hand side, it implements the GraphicalViewer interface. But PaletteViewer has a number of distinguishing features:
■ The ability to add buttons that provide editing functions, called **ToolEntry**s
■ The ability to add buttons that create new components, called **TemplateEntry**s
■ Configuration information contained in a single object, the **PaletteRoot**
This **PaletteRoot** is important for a PaletteViewer because it provides the entries that will be available to the user. Like a Model class, it contains the palette’s information.
![[Graphical Editing Framework-20231216-7.png]]

```
public class FlowchartPalette
{
	public static final String
			TERMINATOR_TEMPLATE = "TERM_TEMP",
			DECISION_TEMPLATE = "DEC_TEMP",
			PROCESS_TEMPLATE = "PROC_TEMP";
	
	public static PaletteRoot getPaletteRoot()
	{
		PaletteRoot root = new PaletteRoot();
		
		PaletteGroup toolGroup = new PaletteGroup("Chart Tools");
		List toolList = new ArrayList();
		
		ToolEntry tool = new SelectionToolEntry(); // Create palette entries
											// that activate tools
		toolList.add(tool);
		root.setDefaultEntry(tool);
		
		tool = new MarqueeToolEntry(); // 2nd palette entry
		toolList.add(tool);
		
		tool = new ConnectionCreationToolEntry( // 3rd palette entry
			"Connection_Tool", "Used to connect multiple components", null,
		ImageDescriptor.getMissingImageDescriptor(),
		ImageDescriptor.getMissingImageDescriptor() );
		toolList.add(tool);
		toolGroup.addAll(toolList); // add tool entry list to group
		
		PaletteGroup templateGroup =
			new PaletteGroup("Chart Templates");
		List templateList = new ArrayList();
		
		CombinedTemplateCreationEntry entry = new
			CombinedTemplateCreationEntry(
			"Terminator", "Start or End Component", TERMINATOR_TEMPLATE,
			new ModelFactory(TERMINATOR_TEMPLATE),
			ImageDescriptor.getMissingImageDescriptor(),
			ImageDescriptor.getMissingImageDescriptor());
		templateList.add(entry);
		entry = new CombinedTemplateCreationEntry(
			"Process", "Action within a flowchart", PROCESS_TEMPLATE,
			new ModelFactory(PROCESS_TEMPLATE),
			ImageDescriptor.getMissingImageDescriptor(),
			ImageDescriptor.getMissingImageDescriptor());
		templateList.add(entry);
		
		entry = new CombinedTemplateCreationEntry(
			"Decision", "Choosing between 'Yes' and 'No'", DECISION_TEMPLATE,
			new ModelFactory(DECISION_TEMPLATE),
			ImageDescriptor.getMissingImageDescriptor(),
			ImageDescriptor.getMissingImageDescriptor());
		templateList.add(entry);
		templateGroup.addAll(templateList);
		
		List rootList = new ArrayList();
		rootList.add(toolGroup);
		rootList.add(templateGroup);
		
		root.addAll(rootList);
		return root;
	}
}
```
A **PaletteRoot** is a collection of **PaletteGroups**, which are collections of entries. These entries, represented by the **ToolEntry** and **CombinedTemplateCreationEntry** classes, create buttons that will be added to the palette. Except for the **SelectionTool** and the **MarqueeSelectionTool**, these entries are initialized with descriptors, images, and classes that implement the **CreationFactory** interface. 

#### Handling events with the ToolEntry and Tool classes
The first three entries in the palette are **ToolEntry**s, which provide the ability to select and connect existing components. Whenever any of these entries are clicked, they create a new **Tool** object. The top tool entry, labeled **Select**, creates a **SelectionTool**; the **Marquee** entry creates a **MarqueeSelectionTool**; and the **Connection_Tool** entry creates a **ConnectionCreationTool**.
The entries are simple to understand, but what exactly is a Tool? A **Tool** provides a distinctive means of interpreting keyboard and mouse events. For example, when you click a component with the **SelectionTool** activated, there will be a different result than if the **ConnectionCreationTool** had been activated. This is because the two Tools have different handlers for **MouseEvents**; they also interact differently with components and their **EditPart**s. These Tool/EditPart interactions are accomplished through **Request**s. Table below lists a group of important Tools provided in GEF, their functions, and the **Requests** they send to **EditParts** during activation.
![[Graphical Editing Framework-20231216-8.png]]
It’s important to note that **DragTrackers** such as ResizeTracker and **ConnectionBendpointTracker** are also GEF Tools. They determine how **DragEvent**s affect an
**EditPart**. Whereas a Tool’s activation is independent of the components in the editor, **DragTrackers** are specified in **EditParts**. In many cases, a high-level **Tool** transfers its event-handling authority to the **DragTracker** associated with the **EditPart**.
For example, when a user clicks a Decision component, the **SelectionTool** will activate. But if the user presses the button and moves the mouse, then the **SelectionTool** delegates authority to the **DecisionPart**’s **DragTracker**.

#### Requests
Requests are the means by which Tools perform editing. When a component is clicked with the **SelectionTool** activated, the Tool sends a **SelectionRequest** or a **LocationRequest** to the **EditPart**. Similarly, when the **ConnectionCreationTool** is activated, the tool sends a **CreateConnectionRequest**. In each case, the **Request** provides the **EditPart** with information about the event, such as which key was pressed or which button was clicked.
When an **EditPart** receives a **Request**, it responds with a **Command** object if one is available. This **Command** tells the **Tool** how the **EditPart** should be altered when the event occurs. Once the **Tool** receives a **Command**, it takes responsibility for modifying the **EditPart** by invoking the **Command**’s **execute**() method. 
We need to mention two points about Requests. 
- First, there is no subpackage (such as com.swtjface.flowchart.requests) for **Request**s because GEF already provides all the classes that will be needed in most editors. However, users can create their own **Tool**s and **Requests** as needed.
- Second, although **Requests** are generally categorized according to their class, they’re further distinguished by a TYPE field. This integer makes the Request’sfunction more specific. For example, when the mouse hovers over an **EditPart**, the **SelectionTool** sends a **LocationRequest** to the component. Because this **LocationRequest**’s **TYPE** field is **REQ_SELECTION_HOVER**, the part knows that the mouse is hovering above it.
#### EditDomains and Tools
Although **EditDomain** objects operate behind the scenes, it’s important to know what they are and how they relate to palettes and palette entries. **EditDomains** keep track of an editor’s state information, and part of this information consists of which **Tool** is currently active. The **EditDomain** sets the editor’s default Tool, which is usually the **SelectionTool**, and manages the process of switching from one **Tool** to the next. It ensures that only one **Tool** is active at any time and directs events to it.
This object also plays a role in creating an editor’s palette. The **EditDomain** ensures that the **PaletteViewer** receives the **PaletteRoot** containing the List of **ToolEntry**s and **TemplateEntrys**. Then, it determines which **Tool** in the List is the default tool and activates this **Tool** when the editor initializes.
####  Creating components with templates
A template is any object that can be matched to a component. If you’d like to number your components, then you can use ints or chars as your templates.
#### TemplateEntry objects
Just as a **ToolEntry** creates **Tool** objects, **TemplateEntrys** create **templates**. After clicking a **TemplateEntry** in the palette, the user can either drag the template to the editor or click an area in the editor. Once the drop location is determined, the editor’s **TemplateTransferDropTargetListener** determines how the component will be created.
In our example, the entry list consists of **CombinedTemplateCreationEntrys**, which create both a **template** and a **CreationEntryTool** to create the template’s matching component. The constructor for these entries requires a number of fields to specify its appearance and operation:
■ **label** (a String)—The entry’s name
■ **shortDesc** (a String)—Description when the mouse hovers over the entry
■ **template** (an Object; a String, in our case)—The component to be created
■ **factory** (a CreationFactory)—The class that builds the component
■ **iconSmall** (an ImageDescriptor)—The image shown near the label
■ **iconLarge** (an ImageDescriptor)—The image shown during the drag

#### Converting templates to Model classes with a CreationFactory
Components are created when the user drags a template from the palette and drops it in a container. The **CreationTool** tells the editor which component to build by identifying a class that implements the **CreationFactory** interface.
This important interface contains only two methods: **getNewObject**() receives a template object and returns a newly created Model object for the requested component, and **getObjectType**() returns the template created by the palette tool. 

### Model classes
The role of a Model class is to keep track of a component’s editable characteristics, called properties. The Model class also fires events when these properties change. It’s important to
note that although the methods are contained in the Model class, the **EditPart** invokes them.
![[Graphical Editing Framework-20231216-9.png]]
First, a Model object creates a **PropertyChangeSupport** instance during its creation. This enables an **EditPart** to listen to its **PropertyChangeEvents**. Then, when the **EditPart** is created, it calls the Model’s **addPropertyChangeListener**() method to be able to respond to these events. Whenever a property changes, the Model calls its **firePropertyChange**() method, and the **EditPart** handles the Event with its **propertyChange**() method. When the component is deallocated, the **EditPart** calls the Model’s **removePropertyChangeListener**() method.

### Changing Model properties with Commands
##### Commands and CommandStacks
It’s possible to extend the Request class, but doing so is generally unnecessary since GEF provides so many subclasses of its own. However, you’ll have to build all your own **Command** classes; you must do so because **Commands** interact with Model objects, which have no standard structure beyond accessor/mutator methods. But GEF does provide an abstract **Command** class.
The **execute**() method performs the real work of the **Command** by invoking the mutator methods (setXYZ()) of the Model class. This **execute**() method is called by the active Tool after it receives the Command in response to its Request. **Commands** can also have other methods and fields associated with them, and these are generally used to provide information for the **execute**() method.
Two other important **Command** methods are **undo**() and **redo**(). As expected, these methods are used to reverse the results of **execute**() or reinvoke it. For these methods to work, the editor must keep track of which **Commands** have been executed. The data structure used for this task is the **CommandStack**, which is initialized during the editor’s construction. This object pushes executed **Commands** onto its Undo stack and pushes undone **Commands** onto its **Redo** stack. The Redo stack is cleared whenever a new **Command** is executed.

### The Controller aspect: EditPart classes
The  Controllers, represented by EditPart objects, whose activity may be divided into
three main functions:
■ Respond to Model changes and update the component’s View
■ Keep track of the component’s **Connections** and **ConnectionAnchors**
■ Activate **EditPolicys** and associate them with their editing roles
The first function is important to understand. An **EditPart**’s **activate**() method is called during its creation. This invokes the Model’s **addPropertyChangeListener**() method and enables the **EditPart** to respond to property changes in the Model. This response is performed with the **propertyChange**() method, which updates the **View** with the new information from the Model.
For example, when a **SetConstraintCommand** is executed, it calls the Model’s **setSize**() and **setLocation**() methods. When these properties change, the two **PropertyChangeEvents** are received by the **EditPart**, which modifies the Figure’s size and location.
In addition to updating size and location, many **EditParts** must manage **Connection** objects. These **EditParts** need to implement the **NodeEditPart** interface, which provides methods for matching **Connections** with **ConnectionAnchors**. When a **PathCommand** needs to know where to attach a new **Connection**, it communicates with a **NodeEditPart**.
The last **EditPart** function deals with **EditPolicys**, which determine the different ways the component can be edited. These policies function by creating **Command** objects in response to **Requests**. An **EditPart** specifies its applicable policies with the **createEditPolicies**() method and contains them in a List. 
Although we refer to these objects as EditParts, all of the GEF Controllers are really extensions of the **AbstractGraphicalEditPart** class. This class provides many of the methods needed for **Connection** management and child/parent interaction. It also creates a default **DragTracker** object to handle **DragEvents**.

### Creating new EditParts with EditPartFactory objects
When a new child Model is created from its template, the parent Model fires a **PropertyChangeEvent**. In response, the parent **EditPart** invokes its **refreshChildren**() method. If any child Models exist without matching EditParts, then this method calls the **createChild**() method.
**createChild**() accesses the editor’s **EditPartFactory** object. Just as our **ModelFactory** created Model classes according to templates, our **PartFactory** creates **EditParts** according to their Models. The interface specifies only one method, **createEditPart**()

## Creating Commands with EditPolicy objects
Let’s say that you has a **ChangeColorCommand**, and you want this Command to execute whenever the component receives a **DirectEditRequest**. For another component, you want the **ChangeShapeCommand** to be executed for this **Request**. Clearly, you need to modify the **EditPart** classes of these components, since these are the objects that receive **Requests**. But how do you make this distinction? The answer involves **EditPolicys**, which make it possible for **EditParts** to control how **Commands** are created in response to **Requests**. 
### The getCommand() method
Just as **execute**() makes **Command** operation possible, the **getCommand**() method
performs the work for **EditPolicys**. Once a **Tool** determines which **EditPart** should receive a given **Request**, it calls the part’s **getCommand**() method with the **Request** object as its parameter. Then, the **EditPart** iterates through its list of **EditPolicy** objects and invokes each of their **getCommand**() methods. In each case, if the policy can respond to the **Request**, it returns a **Command** object to the **Tool**, which executes it.
In addition, **EditPolicys** provide information to the **Command** that specify how its execution should be performed. That is, they tell the **Command** which Model properties should be modified and provide any parameters associated with the **Request**.
##### GEF policies
Although you can create customized **EditPolicys** from scratch, the GEF toolset provides a number of helpful abstract classes. To build concrete **EditPolicys**, you only need to specify which Model objects and properties should be changed when the **Command** executes.
■ ComponentPolicy—Removes Activity objects from the Chart
■ NodePolicy—Manages the process of adding new Paths
■ LayoutPolicy—Creates and arranges Activity objects in the Chart
![[Graphical Editing Framework-20231216-10.png]]The **ComponentEditPolicy** creates **Commands** when an **EditPart** is deleted or orphaned from a container. The **GraphicalEditPolicy** contains a number of subclasses for creating, deleting, and adding children to a **LayoutManager**. The **GraphicalNodeEditPolicy** subclass creates **Commands** for creating connections to and from a component. 


## Adding Actions to the editor
Users expect multiple ways to accomplish an editing task. For example, the Eclipse Workbench lets users copy GUI elements with a keystroke (Ctrl-C), a menu item (Edit->Copy), or an option in a context menu. GEF enables you to provide these capabilities through Actions.
They can be associated with text, images, and accelerator keys; and they can appear as buttons, toolbar items, or options in a menu. However, GEF’s Actions have two important differences.
- First, they’re subclasses of **WorkbenchPartAction**
-  Second, although they invoke **run**() when activated, their operation varies from class to class. 
A **SelectionAction** functions like a **Tool** by sending the selected component a **Request**. **StackActions** function by accessing the **CommandStack**, and **EditorPartActions** work by accessing the **EditorPart** itself.
In order to function, WorkbenchPartActions need to notify the editor of their presence. This is accomplished by adding them to the editor’s ActionRegistry.

### The ActionRegistry and ContextMenus
Every **GraphicalEditor** creates an **ActionRegistry** as part of its initialization and populates it by calling **createActions**(). 

```
action = new UndoAction(this);
registry.registerAction(action);
getStackActions().add(action.getId());

action = new RedoAction(this);
registry.registerAction(action);
getStackActions().add(action.getId());

action = new SelectAllAction(this);
registry.registerAction(action);

action = new DeleteAction((IWorkbenchPart)this);
registry.registerAction(action);
getSelectionActions().add(action.getId());

action = new SaveAction(this);
registry.registerAction(action);
getPropertyActions().add(action.getId());

action = new PrintAction(this);
registry.registerAction(action);
```
it’s helpful to know which **Actions** have already been added to your **GraphicalEditor** by default. It also shows that, in addition to the **ActionRegistry**, the editor keeps track of certain groups of **Action** classes.
These groups include **PropertyActions**, which deal with changes to the editor’s state; **StackActions**, which manipulate the **CommandStack**; and **SelectionActions**, which communicate with selected components. If you intend to add custom **Actions** to your editor, you need to register them in the **ActionRegistry** and add them to any groups to which they belong. You do so by implementing the **createActions**() method in your editor.
After you add your **Actions** to the editor, you need to make them available to the user. One of the simpler ways to do this involves a ContextMenu. This menu is easily accessible by right-clicking in the editor’s main window.
![[Graphical Editing Framework-20231216-11.png]]

### Redirecting Workbench actions with RetargetAction
The Eclipse Workbench contains a number of global **Actions** that can be triggered through its main menu and toolbar. We’d like to redirect them to allow users to perform editing in our editor and use Actions such as Undo, Redo, and Delete.
The process is simple. First, we need to create a set of **RetargetActions** and add them to the editor’s **ActionRegistry**. Then, these **Actions** must be made available to the user. All of this can be accomplished with one class: **ActionBarContributor**.
#### RetargetActions
**RetargetActions** function by directing the effects of the Workbench’s internal **Actions** to an application. When we use one of these **Actions**, its corresponding menu option is available for our editor. For example, when we add the **DeleteRetargetAction**, the Workbench’s Edit->Delete can be used to delete an EditPart.
![[Graphical Editing Framework-20231216-12.png]]
The **Label** column in the table refers to whether the **RetargetAction** can be given a custom label, which works like a tooltip. Only **UndoRetargetAction** and **RedoRetargetAction** are subclasses of **LabelRetargetAction**; the rest of the **RetargetActions** retain the label specified by the Workbench.

## Introducing the EditorPart
#### Working with EditorParts and GraphicalEditors
Graphical components in the Eclipse Workbench are divided into **views** and **editors**. **Views**, which extend the **ViewPart** class, organize and display information about the Workbench. Editors function by allowing the user to manipulate and save files; they descend from the **EditorPart** class.
Both **EditorPart** and its superclass, **WorkbenchPart**, are abstract classes with abstract methods. Therefore, if you’re seeking to build an Eclipse editor, you must
provide code for each of these methods, shown below:
![[Graphical Editing Framework-20231216-13.png]]
The two most important of these are **init**() and **createPartControl**(). The Workbench calls **init**() when the user opens a file with the supported extension. Then, to display the editor, the workbench calls **createPartControl**(). Like the **createContents**() method of a JFace Window, this method embodies the editor’s appearance within a **Composite** object.
The nature of this **Composite** determines how the editor looks and acts. For text editors, this is one large Text box. Graphical editors use a **Canvas**, but there’s much more to a graphical editor than this object. To provide added functionality, GEF supplies an **EditorPart** subclass, **GraphicalEditor**.
The documentation recommends using the GraphicalEditor class as a reference. This class provides a number of important capabilities to the editor, such as an **ActionRegistry**, a **CommandStack**, an **EditDomain**, and a **SelectionSynchronizer** to coordinate selections across multiple viewers

### Understanding the GraphicalViewer
You can think of a GEF editor as a **GraphicalViewer** on top of a **Canvas** object. It handles events, keeps track of the user’s selections, and creates the basis for the editor’s **EditPart** hierarchy.
You can extend **GraphicalViewerImpl**, a concrete class that implements the **GraphicalViewer** interface. 
![[Graphical Editing Framework-20231216-14.png]]
The first two methods create the structure underlying the GEF editor. First, the **createControl**() method builds the SWT **Canvas** object. Afterward, it creates a **LightWeightSystem** to hold the editor’s Draw2D **Figures**. These objects also provide a channel through which the Viewer can receive Events.
The next two methods provide a basis for the editor’s MVC structure. First, the **setRootEditPart**() creates a new top-level parent for the **EditPart** hierarchy. It’s important to understand that this is not a part that we’ve created previously. It doesn’t perform event handling or specify **EditPolicys**; it does interact with the editor’s Layers, but its main purpose is to manage child **EditParts**.
Another purpose of the **RootEditPart** is to specify a **Figure** to create when **setRootFigure**() is invoked. Again, this isn’t a **Figure** that we’ve coded. The nature of this **Figure** depends on the **RootEditPart**. 
The next two methods continue this MVC development. The **getContents**() method initializes the Viewer by providing the top-level Model class of the Viewer.
The next four methods handle events in the Viewer. The first method directs Events from the LightWeightSystem to the editor’s **EditDomain**. The Viewer creates Listeners for **DragEvents** and **DropTargetEvents** and also provides a **KeyHandler** to respond to keyboard actions. It’s important to note that the Viewer doesn’t respond to Events itself, but sends them to the object best suited to handle them.
The last three methods in the table deal with the Viewer’s management of selections. The Viewer listens for the user’s selections and calls the **findObjectAtExcluding**() method to see if a selection location matches that of an **EditPart**. If so, the **EditPart** is added to the List of **EditParts** returned by **getSelectedEditParts**(). Even though the **SelectionTool** responds to the user’s selection, it gets its information from the Viewer.

