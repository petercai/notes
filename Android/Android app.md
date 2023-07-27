An **Android app** consists of components like activities, services, broadcast receivers, and
content providers:
![[Pasted image 20230721225632.png]]

**An Android app** is a single zip archive file with the suffix .apk. The most important control artifact inside is the file AndroidManifest.xml describing the application and the components an application consists of.

A **task** is a group of activities interacting with each other in such a way that the end user
considers them as the elements of an application.  the main structure a task exhibits is its **back stack** or simply **stack**, where activities of an app pile up. The standard behavior for simple apps concerning this stack is the first activity when you launch an app building the root of this stack, the next activity launched from inside the app landing on top of it, another sub-activity landing on top of both, and so on. Whenever an activity gets closed because you
navigate back, the activity gets removed from the stack. And when the root activity gets removed, the stack gets closed as a whole, and your app is considered shut down.

**AndroidManifest.xml** describes the app and declares all the components that are part
of the app.

## Activities
1. The main activity, as declared inside AndroidManifest.xml, gets started by launching the app.
2. All activities can be configured to be started by an explicit or implicit Intent, as configured inside AndroidManifest.xml.
3. Intents are both objects of a class and an outstanding concept in Android.
	1. For explicit Intents, by triggering an Intent, a component tells that it needs something to be done by a dedicated component of a dedicated app.
	2. For implicit Intents, the component just tells what needs to be done without specifying which component is supposed to do it – the Android OS or the user decides which app or component is capable of fulfilling such an implicit request.
4.  you can start the activity name with a dot, which leads to prepending the app’s package name:
	```
	<application ... >
	<activity android:name=".ExampleActivity" />
	...
	</application ... >
	```
### Starting activities
1. To declare an activity as a launchable main activity, inside the AndroidManifest.xml you’d write
	```
	<activity android:name="com.example.myapp.ExampleActivity">
	  <intent-filter>
	   <action android:name="android.intent.action.MAIN" />
	   <category android:name="android.intent.category.LAUNCHER" />
	  </intent-filter>
	</activity>
	```
	The “**android.intent.action.MAIN**” tells Android that it is the main activity and will go
	to the bottom of a task.
	The “**android.intent.category.LAUNCHER**” tells it must be listed inside the launcher.

2. an activity can be started by an Intent from the same app or any other app.
3. **AppCompatActivity** transforms your plain old Kotlin class into a full-fledged, card-carrying Android activity.  The **onCreate()** method. This method gets called when the activity object gets created, and it’s used to perform basic setup such as what layout the activity is associated with. This is done via a call to setContentView()
![[Android app-2023726-1 3.png]]
![[Android app-2023726-1 4.png]]


## Intent Filters
Intents are objects to tell Android that something needs to be done, and they can be
explicit by exactly specifying which component needs to be called or implicit if we don’t
precisely specify the called component but let Android decide which app and which
component can answer the request.

The `<intent-filter>` element can be a child of
 - activity and activity-alias
 - service
 - receiver

So Intents can be used to launch activities and services and to fire broadcast messages.

- Intent Action specifies the action to perform
	The complete list of generic actions is specified by the constants whose names	have ACTION_* inside the class android.content.Intent and shown in section “Intent Constituent Parts” in the online text companion.
- Intent Category  specifies a category for the filter
	This attribute may be used to specify the type of component that an Intent should address.

### Activities Lifecycle
- shut down
- created
- started
- resumed
- running
- paused
- stopped
- destroyed
![[Android app-2023722-1.png]]
![[Android app-2023722-2.png]]

## Services
Services are components running without a user interface apart from notifications in
the status bar or per Toast and with a conceptual affinity toward long-running processes.


## View class
![[_assets/Android app-2023726-1.png]]

- A **spinner** provides a drop-down list of values. It allows you to choose a
single value from a set of values.

### array
![[Android app-2023726-1 1.png]]
![[Android app-2023726-2.png]]

**Use array in spinner**

![[Android app-2023726-1 2.png]]
or
![[Android app-2023726-3.png]]

## Layout

- Using a **linear layout** means that UI components are displayed in a
vertical column or a horizontal row.
A linear layout is the most appropriate type of layout to use if you
want to arrange the components in a single row or column. 


## EventListener

1. The **findViewById** method lets you get a reference to any view in the layout that has an ID. 
	![[Android app-2023726-1 5.png]]
2. Calls view's **setOnClickListener()** method using lamda code to register a handler
	![[Android app-2023726-1 6.png]]