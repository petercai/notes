

Views must implement the org.eclipse.ui.**IViewPart** interface. Typically, views are subclasses of org.eclipse.ui.part.**ViewPart** and thus indirectly subclasses of org.eclipse.ui.part.**WorkbenchPart**, inheriting much of the behavior needed to implement the **IViewPart** interface.
Views are contained in a view site (an instance of the type org.eclipse.ui.**IViewSite**), which in turn is contained in a workbench page (an instance of org.eclipse.ui.**IWorkbenchPage**). In the spirit of lazy initialization, the IWorkbenchPage instance holds on to instances of
org.eclipse.ui.**IViewReference** rather than the view itself so that views can be enumerated and referenced without actually loading the plug-in that defines the view.

![[Views-20231126-1.png]]
