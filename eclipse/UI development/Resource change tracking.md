The Eclipse system generates resource change events indicating, for example, the files and folders that have been added, modified, and removed during the course of an operation. Interested objects can subscribe to these events and take whatever action is necessary to keep themselves synchronized with Eclipse.

# IResourceChangeListener
Eclipse uses the interface org.eclipse.core.resources.**IResourceChangeListener** to notify registered listeners when a resource has changed.

## IResourceChangeEvent

**PRE_BUILD**—Before-the-fact report of builder activity .
**PRE_CLOSE**—Before-the-fact report of the impending closure of a single project as returned by getResource().
**PRE_DELETE**—Before-the-fact report of the impending deletion of a single project as returned by getResource().
**PRE_REFRESH**—before-the-fact report of refreshing of a project.
**POST_BUILD**—After-the-fact report of builder activity.
**POST_CHANGE**—After-the-fact report of creations, deletions, and modifications to one or more resources expressed as a hierarchical resource delta as returned by getDelta().

The **IResourceChangeEvent** interface also defines several methods that
can be used to query its state.
**findMarkerDeltas**(String, boolean)—Returns all marker deltas of the specified type that are associated with resource deltas for this event. Pass true as the second parameter if you want to include subtypes of the specified type.
**getBuildKind**()—Returns the kind of build that caused the event.
**getDelta**()—Returns a resource delta, rooted at the workspace, describing the set of changes that happened to resources in the workspace.
**getResource**()—Returns the resource in question.
**getSource**()—Returns an object identifying the source of this event.
**getType**()—Returns the type of event being reported.

## IResourceDelta
Each individual change is encoded as an instance of a resource delta that is represented by the IResourceDelta interface. Eclipse provides several different constants that can be used in combination to identify the resource deltas handled by the system. Below is the list of valid constants as they appear in the IResourceDelta Javadoc.
**ADDDED**—Delta kind constant indicating that the resource has been added to its parent.
**ADDED_PHANTOM**—Delta kind constant indicating that a phantom resource has been added at the location of the delta node.
**ALL_WITH_PHANTOMS**—The bit mask that describes all possible delta kinds, including those involving phantoms.
**CHANGED**—Delta kind constant indicating that the resource has been changed.
**CONTENT**—Change constant indicating that the content of the resource has changed.
**COPIED_FROM**—Change constant indicating that the resource was copied from another location.
**DESCRIPTION**—Change constant indicating that a project’s description has changed.
**ENCODING**—Change constant indicating that the encoding of the resource has changed.
**LOCAL_CHANGED**—Change constant indicating that the underlying file or folder of the linked resource has been added or removed.
**MARKERS**—Change constant indicating that the resource’s markers have changed.
**MOVED_FROM**—Change constant indicating that the resource was moved from another location.
**MOVED_TO**—Change constant indicating that the resource was moved to another location.
**NO_CHANGE**—Delta kind constant indicating that the resource has not been changed in any way.
**OPEN**—Change constant indicating that the resource was opened or closed.
**REMOVED**—Delta kind constant indicating that the resource has been removed from its parent.
**REMOVED_PHANTOM**—Delta kind constant indicating that a phantom resource has been removed from the location of the delta node.
**REPLACED**—Change constant indicating that the resource has been replaced by another at the same location (i.e., the resource has been deleted and then added).
**SYNC**—Change constant indicating that the resource’s sync status has changed.
**TYPE**—Change constant indicating that the type of the resource has changed.

The **IResourceDelta** class also defines numerous useful APIs as follows:
**accept**(IResourceDeltaVisitor)—Visits resource deltas that are ADDED, CHANGED, or REMOVED. If the visitor returns true, the resource delta’s children are also visited.
**accept**(IResourceDeltaVisitor, boolean)—Same as above but optionally includes phantom resources.
**accept**(IResourceDeltaVisitor, int)—Same as above but optionally includes phantom resources and/or team private members.
**findMember**(IPath)—Finds and returns the descendant delta identified by the given path in this delta, or null if no such descendant exists.
**getAffectedChildren**()—Returns resource deltas for all children of this resource that were ADDED, CHANGED, or REMOVED.
**getAffectedChildren**(int)—Returns resource deltas for all children of this resource whose kind is included in the given mask.
**getFlags**()—Returns flags that describe in more detail how a resource has been affected.
**getFullPath**()—Returns the full, absolute path of this resource delta.
**getKind**()—Returns the kind of this resource delta.
**getMarkerDeltas**()—Returns the changes to markers on the corresponding resource.
**getMovedFromPath**()—Returns the full path (in the “before” state) from which this resource (in the “after” state) was moved.
**getMovedToPath**()—Returns the full path (in the “after” state) to which this resource (in the “before” state) was moved.
**getProjectRelativePath**()—Returns the project-relative path of this resource delta.
**getResource**()—Returns a handle for the affected resource.



# Processing Change Events
The POST_CHANGE resource change event is expressed not as a single change, but as a hierarchy describing one or more changes that have occurred. Events are batched in this manner for efficiency; reporting each change as it occurs to every interested object would dramatically slow down the system and reduce responsiveness to the user.


# Batching Change Event
Anytime a UI plug-in modifies resources, it should wrap the resource modification code by subclassing org.eclipse.ui.actions.**WorkspaceModifyOperation**. The primary consequence of using this operation is that events that typically occur as a result of workspace changes (e.g., the firing of resource deltas, performance of autobuilds, etc.) are deferred until the outermost operation has successfully completed.

# Progress Monitor
For long-running operations, the progress monitor indicates what the operation is doing and an estimate of how much more there is left to be done.

## IProgressMonitor
The org.eclipse.core.runtime.IProgressMonitor interface provides methods for indicating when an operation has started, how much has been done, and when it is complete.
**beginTask**(String, int)—Called once by the operation to indicate that the operation has started and approximately how much work must be done before it is complete. This method must be called exactly once per instance of a progress monitor.
**done**()—Called by the operation to indicate that it is complete.
**isCanceled**()—The operation should periodically poll this method to see whether the user has requested that the operation be cancelled.
**setCanceled**(boolean)—This method is typically called by UI code setting the cancelled state to true when the user clicks on the Cancel button during an operation.
**setTaskName**(String)—Sets the task name displayed to the user. Usually, there is no need to call this method because the task name is set by beginTask(String, int).
**worked**(int)—Called by the operation to indicate that the specified number of units of work has been completed.

The **IProgressMonitorWithBlocking** interface extends **IProgressMonitor** for monitors that want to support feedback when an activity is
blocked due to concurrent activity in another thread. If a running operation
ever calls the setBlocked method listed below, it must eventually call
clearBlocked before the operation completes.
clearBlocked()—Called by an operation to indicate that the operation
is no longer blocked.
setBlocked(IStatus)—Called by an operation to indicate that this
operation is blocked by some background activity.

## Classed for displaying progress
Eclipse provides several classes that either implement the **IProgressMonitor** interface or provide a progress monitor via the **IRunnableWithProgress** interface. These classes are used under different circumstances to notify the user of the progress of long-running operations.
**SubProgressMonitor**—A progress monitor passed by a parent operation to a suboperation so that the suboperation can notify the user of progress as a portion of the parent operation.
**NullProgressMonitor**—A progress monitor that supports cancellation but does not provide any user feedback. Suitable for subclassing.
**ProgressMonitorWrapper**—A progress monitor that wraps another progress monitor and forwards IProgressMonitor and **IProgressMonitorWithBlocking** methods to the wrapped progress monitor. Suitable for subclassing.
**WorkspaceModifyOperation**—An operation that batches resource change events and provides a progress monitor as part of its execution.
**ProgressMonitorPart**—An SWT composite consisting of a label displaying the task and subtask name, and a progress indicator to show progress.
**ProgressMonitorDialog**—Opens a dialog that displays progress to the user and provides a progress monitor used by the operation to relate that information.
**TimeTriggeredProgressMonitorDialog**—Waits for a specified amount of time during operation execution then opens a dialog that displays progress to the user and provides a progress monitor used by the operation to relate that information. If the operation completes before the specified amount of time, then no dialog is opened. This is an internal workbench class but is listed here because the concept and its functionality is interesting. 
**WizardDialog**—When opened, optionally provides progress information as part of the wizard. The wizard implements the **IRunnableContext**, and thus the operation can call run(boolean, boolean, **IRunnableWithProgress**) and display progress in the wizard via the provided progress monitor.

## Workbench window status bar
The workbench window provides a progress display area along the bottom edge of the window. Use the **IWorkbenchWindow**.run() method to execute the operation and the progress monitor passed to **IRunnableWithProgress** will be the progress monitor in the status bar.
## IProgressService
Yet another mechanism for displaying progress in the workbench is using the **IProgressService** interface. While the **run**() method in **IWorkbenchWindow** displays progress in the status bar, the IProgressService **interface** displays progress using a subclass of **ProgressMonitorDialog** named **TimeTriggeredProgressMonitorDialog**. Although you could use a **ProgressMonitorDialog**, **IProgressService** only opens a progress dialog if the operation takes longer to execute than a specified amount of time(currently 800 milliseconds).

```
window.getWorkbench().getProgressService().run(true, true,
	new IRunnableWithProgress() {
		public void run(IProgressMonitor monitor)
			throws InvocationTargetException, InterruptedException
	{
		monitor.beginTask("Simulated long running task #1", 60);
		for (int i = 60; i > 0; --i) {
			monitor.subTask("seconds left = " + i);
			if (monitor.isCanceled()) break;
			Thread.sleep(1000);
			monitor.worked(1);
		}
		monitor.done();
	}
});
```
Typically, jobs are executed in the background, but the **IProgressService** provides the **showInDialog**() method and the **UIJob** class for executing them in the foreground.

# Delayed Changed Events
Eclipse uses lazy initialization—only load a plug-in when it is needed. Lazy initialization presents a problem for plug-ins that need to track changes. How does a plug-in track changes when it is not loaded?
Eclipse solves this problem by queuing change events for a plug-in that is not loaded. When the plug-in is loaded, it receives a single resource change event containing the union of the changes that have occurred during the time it was not active. To receive this event, your plug-in must register to be a save participant when it is started up, as follows.

```
public static void addSaveParticipant() {
	ISaveParticipant saveParticipant = new ISaveParticipant(){
		public void saving(ISaveContext context)
		throws CoreException
		{
		// Save any model state here.
		context.needDelta();
		}
		public void doneSaving(ISaveContext context) {}
		public void prepareToSave(ISaveContext context)
			throws CoreException {}
		public void rollback(ISaveContext context) {}
	};
	
	ISavedState savedState;
	try {
		savedState = ResourcesPlugin
			.getWorkspace()
			.addSaveParticipant (
		FavoritesActivator.getDefault(),
				saveParticipant);
	}
	catch (CoreException e) {
		FavoritesLog.logError(e);
		// Recover if necessary.
		return;
	}
	if (savedState != null)
		savedState.processResourceChangeEvents(
			FavoritesManager.getManager());
}
```

Tip: Even though Eclipse is based on lazy plug-in initialization, it does provide a mechanism for plug-ins to start when the workbench itself starts. To activate at startup, the plug-in must extend the **org.eclipse.ui.startup** extension point and implement the **org.eclipse.ui.
IStartup** interface. Once the plug-in is started, the workbench will call the plug-in’s **earlyStartup**() method. A workbench preference option gives the user the ability to prevent a plug-in from starting early, so make sure that if your plug-in takes advantage of this extension point, it degrades gracefully in the event that it is not started early.