An **Android app** consists of components like activities, services, broadcast receivers, and
content providers:
![[Pasted image 20230721225632.png]]

**An Android app** is a single zip archive file with the suffix .apk. The most important control artifact inside is the file AndroidManifest.xml describing the application and the components an application consists of.

A **task** is a group of activities interacting with each other in such a way that the end user
considers them as the elements of an application.  the main structure a task exhibits is its **back stack** or simply **stack**, where activities of an app pile up. The standard behavior for simple apps concerning this stack is the first activity when you launch an app building the root of this stack, the next activity launched from inside the app landing on top of it, another sub-activity landing on top of both, and so on. Whenever an activity gets closed because you
navigate back, the activity gets removed from the stack. And when the root activity gets removed, the stack gets closed as a whole, and your app is considered shut down.

**AndroidManifest.xml** describes the app and declares all the components that are part
of the app.
Every Android app must include a file called AndroidManifest.xml, which lives in the app/src/main folder of your project. The AndroidManifest.xml file contains essential information about your app that the Android operating system, the Android build tools, and the Google Play Store need to know about. This includes the app’s package name, details of any activities, and any permissions the app requires.

![[Android app-2023728-2.png]]

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

![[Android app-2023730-1.png]]

### So what’s R?
R refers to **R.java**. It’s a special file that’s automatically generated when you build the app. Android uses R.java to keep track of any resources used within the app such as layouts, String resources, and views.
When you pass the findViewById method a value of R.id.brands, Android looks up the brands ID from R, and uses this to return a reference to the brands view. Similarly, when you pass the setContentView method a value of R.layout.activity_main, Android looks up the correct layout from R, and assigns it to the activity. You don’t change any of the code in R.java yourself: Android Studio automatically generates it for you.

### Layout inflation
As well as displaying the layout on the device screen, the setContentView() method converts the views in the layout’s XML into objects. This process is called **layout inflation** because it inflates each view into an object



### Intent Filters
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

![[Android app-2023730-5.png]]

***When you override any activity lifecycle method in your activity, you need to call the Activity superclass method or the compiler will complain.***

The onRestart() method is called before onStart(), but only when the activity becomes visible again after previously losing visibility. It’s not called when the activity first becomes visible.
*onStart() gets called whenever the activity becomes visible.
onRestart() only gets called when the activity becomes visible again after losing visibility.*

![[Android app-2023730-4.png]]

1. The activity gets launched, and the onCreate() and onStart() methods run. At this point, the activity is visible, but it doesn’t have the focus.
2. The onResume() method runs. It gets called when the activity is about to move into the foreground. After the onResume() method has run, the activity has the focus and the user can interact with it.
3. The onPause() method runs when the activity stops being in the foreground. After the onPause() method has run, the activity is still visible but doesn’t have the focus.
4. If the activity moves into the foreground again, the onResume() method gets called. The activity may go through this cycle many times if the activity repeatedly loses and then regains the focus.
5. If the activity stops being visible to the user, the onStop() method gets called. After the onStop() method has run, the activity is no longer visible.
6. If the activity becomes visible to the user again, the onRestart() method gets called, followed by onStart() and onResume(). The activity may go through this cycle many times.
7. Finally, the activity is destroyed. As the activity moves from running to destroyed, the onPause() and onStop() methods get called before the activity is destroyed.

![[Android app-2023722-1.png]]
![[Android app-2023722-2.png]]

### Bundle
A Bundle is a type of object that’s used to hold key/value pairs. <u>Before the activity is destroyed, Android lets you put key/value pairs into a Bundle</u>. T<u>his Bundle is then picked up by the new instance of the activity when it’s recreated</u>. Using a Bundle therefore gives you a way of reinstating an activity’s state when you rotate the device screen

## Services
Services are components running without a user interface apart from notifications in
the status bar or per Toast and with a conceptual affinity toward long-running processes.


## View class
![[_assets/Android app-2023726-1.png]]

- A **spinner** provides a drop-down list of values. It allows you to choose a
single value from a set of values.

### Chronometer
A chronometer is a type of text view that acts as a simple timer. It has built-in methods you can use to start and stop it, and it displays the elapsed time.

### array
![[Android app-2023726-1 1.png]]
![[Android app-2023726-2.png]]

**Use array in spinner**

![[Android app-2023726-1 2.png]]
or
![[Android app-2023726-3.png]]
### Image
Android’s preferred image format is WebP, which combines a small file size with a minimum loss of quality. You can convert images to WebP in Android Studio by right-clicking on
the image, and choosing the “Convert to WebP” option.

The default folder for storing image resources in your app is app/src/main/res/drawable folder.

### ImageView
![[Android app-2023728-12.png]]
### Button
Buttons can include attributes such as `android:drawableStart` and `android:drawableEnd`, which allow you to position an image at the start or end of the button. The following code, for example, adds a duck image at the start of a button:
```
android:drawableStart="@drawable/duck"
```
Android also includes an image button view: a button with an image but no text.



### EventListener

1. The **findViewById** method lets you get a reference to any view in the layout that has an ID. 
	![[Android app-2023726-1 5.png]]
2. Calls view's **setOnClickListener()** method using lamda code to register a handler. The **setOnClickListener** method takes one parameter—a lambda—which describes what should happen when the button is clicked.
	![[Android app-2023726-1 6.png]]
## Layout
All layouts are a type of **ViewGroup** and a ViewGroup is a type of **View**
![[Android app-2023728-11.png]]

- Every UI component you add to a layout is a type of View: an object that takes up space on the screen.
- Every layout is a type of ViewGroup: a type of View that can contain other Views.

### Linear loyout
Using a **linear layout** means that UI components are displayed in a vertical column or a horizontal row. A linear layout is the most appropriate type of layout to use if you want to arrange the components in a single row or column. 
	![[Android app-2023728-1.png]]


- add weight to view
	![[Android app-2023728-3.png]]
	![[Android app-2023728-4.png]]

- The **gravity** attribute controls the position of a view’s contents. `android:gravity` lets you say where you want the view’s contents to appear inside the view.
	![[Android app-2023728-7.png]]
	You can also apply multiple gravities to a view by separating each value with a “|”. To sink a view’s contents to the bottom-end corner, for example, you’d use:
	```
	android:gravity="bottom|end"
	```


	![[Android app-2023728-5.png]]
	![[Android app-2023728-6.png]]
- Use margins to add space between views
	![[Android app-2023728-8.png]]
	![[Android app-2023728-9.png]]
	- padding vs. margin
		You can use padding attributes such as paddingTop with views, but it has a different effect. As you’ve seen, layout_marginTop increases the amount of space above the view. paddingTop, however, increases the height of the view by adding extra space inside it:
		![[Android app-2023728-10.png]]

- Another example
	![[Android app-2023728-15.png]]
- The layout’s views are inflated into objects

### FrameLayout
a **frame layout** stacks its views, one on top of another. If you want a layout whose views can overlap, a simple option is to use a frame layout. Instead of displaying its views in a single row or column, it stacks them, one on top of another. It’s often used to hold just a single view.
- A frame layout stacks views in the order they appear in the layout XML
	The first view is displayed first, the second is stacked on top of it, and so on.
![[Android app-2023728-13.png]]

##### ScrollView
![[Android app-2023728-14.png]]

### ConstraintLayout

The constraint layouts are specifically designed to work with Android Studio’s design editor.  Constraint layouts can be built visually. You drag and drop views into the design editor’s blueprint, and give it instructions for how each view should be displayed. Constraint layouts are part of a suite of libraries known as **Android Jetpack**.
> **Android Jetpack** is a collection of libraries that help you follow best practice, reduce boilerplate code, and make your coding life easier. It includes constraint layouts, navigation, the Room persistence library (which helps you build databases) and lots, lots more.
> Every project needs to know where to find any extra Jetpack libraries it needs, and this is done by adding a reference to the Google repository in the project’s build.gradle file. It's  in the repositories section under allprojects:
> ![[Android app-2023729-1.png]]
> To use constraint layouts, a reference to its library needs to be included in the app’s build.gradle file.
> ![[Android app-2023729-2.png]]

- A **constraint** is a connection or attachment that tells the layout where the view should be positioned. You can use a constraint to attach a view to the start edge of the layout, or underneath another view.

### Navigation and Fragment
A fragment is like a kind of subactivity that’s displayed inside an activity’s layout. It has Kotlin code that controls its behavior, and an associated layout that defines its appearance.
Fragment class is part of Android Jetpack. 
#### OnCreateView()
Fragment code looks similar to activity code:
![[Android app-2023730-6.png]]

- The first parameter is a **LayoutInflater** that you use to inflate the fragment’s layout - inflating a layout turns its XML views into objects.
- The second parameter is a **ViewGroup**?. This is the ViewGroup in the activity’s layout that displays the fragment.
- The final parameter is a **Bundle**?. This is used if you’ve previously saved the fragment’s state, and want to reinstate it.
- The onCreateView() method returns a **View**?, which is an inflated version of the fragment’s layout.
#### LayoutInflater.inflate()
The inflate() is the fragment equivalent of activity’s setContentView() method, as it’s used to inflate the fragment’s layout
into a hierarchy of View objects.
Navigate between screens using the Navigation component.  The Navigation component is part of Android Jetpack