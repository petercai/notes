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

The MainActivity’s layout file (activity_main.xml) is inflated into a hierarchy of View objects.
If the file describes a linear layout that contains a text view and a button, the layout gets inflated into LinearLayout, TextView, and Button objects. The LinearLayout is the **root** view of
this hierarchy.
![[Android app-2023807-1.png]]

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
- Bundles are passed from one instance of an activity to another using two methods: onSaveInstanceState() and onCreate(). 
- onSaveInstanceState() lets you add values to a Bundle before the activity is destroyed, and onCreate() picks the Bundle up again when the activity has been recreated.
- 
	![[Android app-2023805-2.png]]
	
```
override fun onSaveInstanceState(savedInstanceState: Bundle) {
	savedInstanceState.putInt("answer", 42)
	super.onSaveInstanceState(savedInstanceState)
}
```
	![[Android app-2023805-3.png]]
	
## JetPack

**Android Jetpack** is a collection of libraries that help you follow best practice, reduce boilerplate code, and make your coding life easier. It includes constraint layouts, navigation, the Room persistence library (which helps you build databases) and lots, lots more.

![[Android app-2023805-1.png]]

> Every project needs to know where to find any extra Jetpack libraries it needs, and this is done by adding a reference to the Google repository in the project’s build.gradle file. It's  in the repositories section under all projects:
> 
> ![[Android app-2023729-1.png]]

## Services
Services are components running without a user interface apart from notifications in
the status bar or per Toast and with a conceptual affinity toward long-running processes.


## View class

### View binding
View binding is a type-safe, more efficient alternative to calling findViewById(). To use view binding, you first need to enable it in the android section of the app’s build.gradle file. 
![[Android app-2023806-3.png]]

#### Enabling view binding generates code for each layout
When you enable view binding, it automatically creates a binding class for each of the layout files in your app.

#### Binding in Activity
An activity can interact with its views from onCreate() to onDestroy().
![[Android app-2023807-6.png]]
The activity has access to the views in its layout from onCreate() to onDestroy(), it can interact with them in any of its methods.  
 
 An activity first gets access to its layout when its onCreate() method runs. This is the first method in the activity’s lifecycle, and it’s used to inflate the layout (or bind to it with view binding) and perform any initial setup.

![[Android app-2023807-2.png]]
![[Android app-2023807-3.png]]

![[Android app-2023807-4.png]]

The above code declares a property named binding whose type is ActivityMainBinding. This property gets set in the activity’s onCreate() method using the following code:
![[Android app-2023807-5.png]]

This calls ActivityMainBinding’s inflate() method, which creates an ActivityMainBinding object that’s linked to the activity’s layout.

The code:
```
val view = binding.rootsetContentView(view)
```
gets a reference to the binding object’s root view, and uses the setContentView() method to display it. Once you’ve added view binding to an activity in this way, you can use the binding property to interact with the layout’s views

#### Binding in Fragment
A fragment can only interact with its views from onCreateView() to onDestroyView().
![[Android app-2023807-7.png]]
A fragment first gets access to its layout when its onCreateView() method runs. This gets called when an activity needs access to the fragment’s layout, so it’s used to inflate the layout and perform any initial setup such as setting OnClickListeners.

onCreateView(), however, isn’t the first method in the fragment’s lifecycle. There are other methods, such as onCreate(), that run before onCreateView(), so they can’t interact with the fragment’s views.

After its onCreateView() method has run, the fragment continues to have access to its views until its onDestroyView() method has finished running. This gets called when the activity no longer needs the fragment’s layout, which may be because the activity needs to navigate to a different fragment, or it’s being destroyed.

onDestroyView(), however, isn’t the final method in the fragment’s lifecycle. There are other methods, such as onDestroy(), that run after onDestroyView(), so these can’t interact with the fragment’s views either.
![[Android app-2023807-8.png]]


![[Android app-2023807-9.png]]


**Activities and fragments get access to their views at different points intheir lifecycles.**

An activity can interact with its views from when it’s been created and its onCreate() method is called. It continues to have access to these views until the activity is destroyed, which happens at the end of its lifecycle.

A fragment, however, can only interact with its views from when its onCreateView() method gets called. This means that the \_binding property can only be set to a view binding object in this method. 

The fragment continues to have access to its views until its onDestroyView() method runs. At this point, the fragment’s layout is discarded, so \_binding needs to be set to null. This prevents the fragment from trying to use the view binding object when it can no longer interact with views.


### View Model
A view model is a separate class that sits alongside the activity or fragment code. It’s responsible for all of the data that needs to be displayed on the screen, along with any business logic.
The view model can survive configuration changes.
A view model holds business logic.

#### 1. Add a view model dependency to the app build.gradle file
![[Android app-2023807-10.png]]

#### 2. extend ViewModel class
![[Android app-2023807-11.png]]

#### 3. Use ViewModeProvider to create model instances
ViewModelProvider is a special class whose job is to provide activities and fragments with view models. It ensures that a new view model object only gets created if one doesn’t already exist.
When the screen is rotated, any fragments that are displayed on the screen get destroyed and recreated. When this happens, the view model provider makes sure that the same view model object continues to be used. The view model keeps its state, so any properties that are usedby the fragment don’t get reset.

![[Android app-2023807-12.png]]
The view model provider keeps hold of the view model so long as the activity or fragment stays alive. When a fragment is detached or removed from its activity, for example, the view model provider lets go of the fragment’s view model. The next time the view model provider is asked to provide a view model object, it creates a new one.

### Live Data
Live data lets the view model tell interested parties—such as fragments and activities—when its property values have been updated. They can then react to these changes by updating views, or calling other methods.

#### 1. add library dependencies
![[Android app-2023807-13.png]]

#### 2. use MutableLiveData
You specify that a property uses live data by changing its type to MutableLiveData\<Type\>, where Type is the type of data the property should hold.
![[Android app-2023807-15.png]]

#### 3. Live data objects use a value property
When you use MutableLiveData properties, you update their values using a property named value. E.g.
```
secretWordDisplay.value = deriveSecretWordDisplay()
```

#### 4. The fragment observes the changes
![[Android app-2023807-16.png]]

### Data binding
#### 1. enable data binding
![[Android app-2023807-17.png]]
#### 2. Add \<layout\> and \<data\> elements
![[Android app-2023807-18.png]]

#### 3. Set the layout’s data binding variable

```
<data>
	<variable
		name="resultViewModel"
		type="com.hfad.guessinggame.ResultViewModel" />
</data>
```
![[Android app-2023807-19.png]]

#### 4.  Use the layout’s data binding variable to access the view model
![[Android app-2023807-20.png]]





### Spinner
A **spinner** provides a drop-down list of values. It allows you to choose a
single value from a set of values.

### Chronometer
A chronometer is a type of text view that acts as a simple timer. It has built-in methods you can use to start and stop it, and it displays the elapsed time.

### Array

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
![[_assets/Android app-2023726-1.png]]


### EventListener
The Fragment class isn’t a subclass of Activity.
#### In Activity
1. The **findViewById** method lets you get a reference to any view in the layout that has an ID. 
	![[Android app-2023726-1 5.png]]
2. Calls view's **setOnClickListener()** method using lamda code to register a handler. The **setOnClickListener** method takes one parameter—a lambda—which describes what should happen when the button is clicked.
	![[Android app-2023726-1 6.png]]

#### In Fragment

The **first** difference is that you add an OnClickListener to a fragment’s button in the fragment’s onCreateView() method, not onCreate(). This is because a fragment first has access to its views in onCreateView(), so it’s the best place to set any OnClickListeners.

The **second** difference is the Fragment class doesn’t include a findViewById() method, so you can’t directly call it to get a reference to any views. You can, however, call findViewById() on the fragment’s root view instead.

![[Android app-2023802-1.png]]

## Database
Most Android databases use SQLite. Android Jetpack includes a persistence library
named **Room** that sits on top of SQLite.
**Room** apps are usually structured using MVVM, which is an architectural design pattern that’s used to structure apps. It stands for Model-View-ViewModel.
![[Android app-2023807-21.png]]

### 1. build.gradle
- project
	![[Android app-2023807-22.png]]
- app
	![[Android app-2023807-23.png]]

### Room database
- data classes for the tables
	![[Android app-2023807-25.png]]
	
- interface for data classes (DAO)
	![[Android app-2023807-26.png]]

- Database class
	![[Android app-2023807-27.png]]
	**<u>entities</u>** specifies any classes—marked with @Entity—that define the 	tables you want Room to add to the database. For the Tasks app, this is the 	Task data class.
	**<u>version</u>** is an Int that specifies the database version. In this case, it’s 1,  as this is the first version of the database.
	**<u>exportSchema</u>** tells Room whether to export the database schema into a folder so that you can record its version history. 

	![[Android app-2023807-28.png]]






## Layout
All layouts are a type of **ViewGroup** and a ViewGroup is a type of **View**
![[Android app-2023728-11.png]]

- Every UI component you add to a layout is a type of View: an object that takes up space on the screen.
- Every layout is a type of ViewGroup: a type of View that can contain other Views.

### Linear loyout
Using a **linear layout** means that UI components are displayed in a vertical column or a horizontal row. A linear layout is the most appropriate type of layout to use if you want to arrange the components in a single row or column. 
	![[Android app-2023728-1.png]]

### EditText

#### 1. add weight to make multi-line textbox
	![[Android app-2023728-3.png]]
	![[Android app-2023728-4.png]]

#### 2. Gravity
- The **gravity** attribute controls the position of a view’s contents. `android:gravity` lets you say where you want the view’s contents to appear inside the view.
	![[Android app-2023728-7.png]]
	You can also apply multiple gravities to a view by separating each value with a “|”. To sink a view’s contents to the bottom-end corner, for example, you’d use:
	```
	android:gravity="bottom|end"
	```


	![[Android app-2023728-5.png]]
	![[Android app-2023728-6.png]]
#### 3. Use margins to add space between views
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

#### FragmentContainerView

![[Android app-2023730-7.png]]

You add a fragment to a layout using a **FragmentContainerView**. This is a type of FrameLayout that’s used to display fragments.
![[Android app-2023730-8.png]]
You specify which fragment you want to display by setting the FragmentContainerView’s android:name attribute to the fully qualified fragment name, including its package.

When Android creates the activity’s layout, it populates the FragmentContainerView with the View object returned by the fragment’s onCreateView() method. This View is the fragment’s user interface.


### ScrollView
![[Android app-2023728-14.png]]

### ConstraintLayout

The constraint layouts are specifically designed to work with Android Studio’s design editor.  Constraint layouts can be built visually. You drag and drop views into the design editor’s blueprint, and give it instructions for how each view should be displayed. Constraint layouts are part of a suite of libraries known as **Android Jetpack**.
> 
> To use constraint layouts, a reference to its library needs to be included in the app’s build.gradle file.
> ![[Android app-2023729-2.png]]

- A **constraint** is a connection or attachment that tells the layout where the view should be positioned. You can use a constraint to attach a view to the start edge of the layout, or underneath another view.

## Fragment
A fragment is like a kind of subactivity that’s displayed inside an activity’s layout. It has Kotlin code that controls its behavior, and an associated layout that defines its appearance. Fragment class is part of Android Jetpack. 
A fragment has Kotlin code and a layout.
The Fragment class doesn’t extend Activity.
Fragments don’t have a findViewById() method.
#### OnCreateView()
onCreateView() gets called each time Android needs the fragment’s layout. Fragment code looks similar to activity code:
![[Android app-2023730-6.png]]

- The first parameter is a **LayoutInflater** that you use to inflate the fragment’s layout - inflating a layout turns its XML views into objects.
- The second parameter is a **ViewGroup**?. This is the ViewGroup in the activity’s layout that displays the fragment.
- The final parameter is a **Bundle**?. This is used if you’ve previously saved the fragment’s state, and want to reinstate it.
- The onCreateView() method returns a **View**?, which is an inflated version of the fragment’s layout.
#### LayoutInflater.inflate()
The inflate() is the fragment equivalent of activity’s setContentView() method, as it’s used to inflate the fragment’s layout into a hierarchy of View objects.


## Navigation

Navigate between screens using the Navigation component.  The Navigation component is part of Android Jetpack. It’s extremely flexible, and simplifies many of the complexities of fragment navigation— such as fragment transactions and back stack manipulation.

Navigating between fragments is comprised of three main parts:
- A **navigation grap**
	The navigation graph holds all of the navigation-related information that your app requires, and describes the possible paths the user can take when navigating the app.
	A navigation graph describes possible destinations and navigation paths. It uses actions to describe these paths.
- A **navigation host**
	A navigation host is **an empty container** that’s used to display the **fragment** you navigate to. You add the navigation host to your activity’s layout. The Navigation component comes with a default navigation host named NavHostFragment, which extends the Fragment class and implements the NavHost interface.
- A **navigation controller**
	The navigation controller controls which fragment is displayed in the navigation host as the user navigates through the app. A navigation controller uses actions to control which fragment is displayed in the navigation host.
	
![[Android app-2023801-2.png]]
#### Destination
A destination is a screen in the app—usually a fragment—which the user can navigate to.
![[Android app-2023731-1.png]]


#### Action
Actions are used to connect destinations in the navigation graph, and they define possible paths the user can take when navigating through the app. Every action must have a unique ID. Android uses this ID to determine which destination needs to be displayed as the user navigates through the app.
![[Android app-2023801-1.png]]
#### Host (or NavHost)
the Navigation component comes with a built-in one named **NavHostFragment** . It’s a subclass of Fragment that implements the NavHost interface.
![[Android app-2023801-3.png]]
NavHostFragment ca be added it to a layout file using a FragmentContainerView .

```
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"  
    xmlns:app="http://schemas.android.com/apk/res-auto"  
    xmlns:tools="http://schemas.android.com/tools"  
    android:layout_width="match_parent"  
    android:layout_height="match_parent"  
    tools:context=".MainActivity">  
  
    <androidx.fragment.app.FragmentContainerView  
        android:id="@+id/fragmentContainerView"  
        android:name="androidx.navigation.fragment.NavHostFragment"  
        android:layout_width="match_parent"  
        android:layout_height="match_parent"  
        app:defaultNavHost="true"  
        app:navGraph="@navigation/nav_graph" />  
</androidx.constraintlayout.widget.ConstraintLayout>
```

The **app:navGraph** attribute tells the navigation host which navigation graph to use, in this case nav_graph.xml. The navigation graph specifies which fragment to display first (its start destination) and lets the user navigate between its destinations.
The **app:defaultNavHost** attribute lets the navigation host interact with the device back button.

We’ve now created a navigation graph, and linked it to a navigation host that’s held in a FragmentContainerView  in MainActivity’s layout. When the app runs, WelcomeFragment—which is the navigation graph’s start destination—will be displayed.
#### NavController
Each time you want to navigate to a new fragment, 
you **first** need to get a reference to a navigation controller. You do this by calling the findNavController() method on its root View object.
```
val navController = view.findNavController()
```
Once you have a navigation controller, you ask it to navigate to a new
destination by calling its navigate() method. This method takes one
parameter: a navigation action ID.
![[Android app-2023802-2.png]]
Big picture:

![[Android app-2023802-4.png]]

### Safe Args
You navigate from one destination to another by passing a **navigation action** to the navigation controller. If you want to pass an argument to a destination, you do so by passing its value to the navigation action. 
When the navigation controller receives an action that includes an argument, it navigates to the appropriate fragment, and passes along the argument’s value.

You can add arguments to navigation actions using a **Directions** class.
*build.gradle(project)*
```
buildscript {  
    def nav_version = '2.6.0'  
    repositories {  
        google()  
        jcenter()  
        maven { url 'https://jitpack.io' }  
    }    dependencies {  
        classpath "androidx.navigation:navigation-safe-args-gradle-plugin:$nav_version"  
    }  
}
```

*build.gradle(module)*
```
plugins {  
    ...  
    id 'androidx.navigation.safeargs.kotlin'  
}
```
The **Safe Args plug-in** generates **Directions** and Args **classes**. Use the **Directions** class to pass arguments to a destination. Use the **Args** class to retrieve them.

Each Args class includes a fromBundle() method that you use to retrieve any arguments that have been passed to the fragment.
![[Android app-2023806-1.png]]

### "back" button - back stack
if you want, you can pop destinations off the back stack as the user navigates through the app. You do this by specifying pop behavior in the navigation graph.

![[Android app-2023806-2.png]]

