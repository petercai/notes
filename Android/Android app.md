An **Android app** consists of components like activities, services, broadcast receivers, and
content providers:
![[Pasted image 20230721225632.png]]

**An Android app** is a single zip archive file with the suffix .apk. The most important control artifact inside is the file AndroidManifest.xml describing the application and the components an application consists of.

A **task** is a group of activities interacting with each other in such a way that the end user
considers them as the elements of an application.  the main structure a task exhibits is its **back stack** or simply **stack**, where activities of an app pile up. The standard behavior for simple apps concerning this stack is the first activity when you launch an app building the root of this stack, the next activity launched from inside the app landing on top of it, another sub-activity landing on top of both, and so on. Whenever an activity gets closed because you
navigate back, the activity gets removed from the stack. And when the root activity gets removed, the stack gets closed as a whole, and your app is considered shut down.

**AndroidManifest.xml** describes the app and declares all the components that are part
of the app. 