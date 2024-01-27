# When to Extend Which Extension Point in Eclipse Plugin - DZone
there’s no need to tell how much popular and widely used the eclipse platform is. and the huge success of it lies in its extensibility.  

it is extensible through the means of **extension points** (well defined exposed places/hooks for others to provide extended functionality) and **plugins** ( providing extended functionality using existing extension points and optionally exposing new extension points). eclipse itself is made up of many and many of plugins built around the small core runtime engine capable of dynamic discovery, loading, and running of plug-ins.

in this post i’m not going to provide a hello-world plugin tutorial or introduction of eclipse platform (as you can find many good ones on net). i’m going to share what difficulty i faced when i started plugin development. after reading some introduction tutorials i wanted to **_know quickly which extension points i need to use for some particular tasks_** . once we know the name of extension point we can find the details in official eclipse documentation ( [extension points reference](http://help.eclipse.org/juno/topic/org.eclipse.platform.doc.isv/reference/extension-points/index.html?cp=2_1_1) ) or even better within platform itself (plugin development environment) when trying to add an extension point it gives the description and sample implementation (if available) to understand.

 [![](_assets/13990368-plugin1.png)](http://javacurious.files.wordpress.com/2013/01/plugin1.png) 

so i’ll focus on quick introduction (to help you get started with further links) of some common extension points and when we can use them (not how) (considered release is eclipse juno). if you need introduction for eclipse plugin development, you can read first [http://www.vogella.com/articles/eclipseplugin/article.html](http://www.vogella.com/articles/eclipseplugin/article.html) for example or request google for other ones.

but before understanding the various extension points, we need to understand the platform structure and few terms.

 [![](_assets/13990369-eclipse1.png)](http://javacurious.files.wordpress.com/2013/01/eclipse1.png) 

the various subsystems (on top of platform runtime) as described above define many extension points to provide the additional related functionality.

below are the various components of the workbench ui.

 [![](_assets/13990370-workbench.png)](http://javacurious.files.wordpress.com/2013/01/workbench.png) 

now below describes the common functionality you want to extend/customize and which extensions points are your friends.

**add menus/buttons** :

 [![](_assets/13990371-menus.jpg)](http://javacurious.files.wordpress.com/2013/01/menus.jpg) 

declare an abstract semantic behavior of an action (command) with optional default handler: org.eclipse.ui. [commands](http://help.eclipse.org/juno/topic/org.eclipse.platform.doc.isv/guide/workbench_cmd_commands.htm?cp=2_0_4_1_2)

add menus, menu items and toolbar buttons in ui: org.eclipse.ui. [menus](http://help.eclipse.org/juno/topic/org.eclipse.platform.doc.isv/guide/workbench_cmd_menus.htm?cp=2_0_4_1_3)

add specific handlers to a command: org.eclipse.ui. [handlers](http://help.eclipse.org/juno/topic/org.eclipse.platform.doc.isv/guide/workbench_cmd_handlers.htm?cp=2_0_4_1_4)

declare key bindings (or set of them called as schemes) to commands: org.eclipse.ui. [bindings](http://help.eclipse.org/juno/topic/org.eclipse.platform.doc.isv/guide/workbench_cmd_bindings.htm?cp=2_0_4_1_5)

more help at: [configuring and _adding menu_ items in _eclipse_ v3.3](http://www.ibm.com/developerworks/library/os-eclipse-3.3menu/index.html)

**add views:**

 [![](_assets/13990372-view.png)](http://javacurious.files.wordpress.com/2013/01/view.png) 

define additional views  for the workbench: org.eclipse.ui. [views](http://help.eclipse.org/juno/topic/org.eclipse.platform.doc.isv/guide/workbench_basicext_views.htm?cp=2_0_4_1_0)

more help at: [creating an eclipse view](http://www.eclipse.org/articles/viewarticle/viewarticle2.html) (old)

**add editors:**

 [![](_assets/13990373-editor.png)](http://javacurious.files.wordpress.com/2013/01/editor.png) 

add new editors to the  workbench: org.eclipse.ui. [editors](http://help.eclipse.org/juno/topic/org.eclipse.platform.doc.isv/guide/workbench_basicext_editors.htm?cp=2_0_4_1_1)

more help at: [eclipse editor plugin tutorial](http://www.vogella.com/articles/eclipseeditors/article.html)

**configure the launching of applications** :

 [![](_assets/13990374-launch1.gif)](http://javacurious.files.wordpress.com/2013/01/launch1.gif) 

 [![](_assets/13990375-launch2.gif)](http://javacurious.files.wordpress.com/2013/01/launch2.gif) 

define/configure a type for launching applications : org.eclipse.debug.core. [launchconfigurationtypes](http://help.eclipse.org/juno/topic/org.eclipse.platform.doc.isv/guide/debug_launch_adding.htm?cp=2_0_17_0_0)

associate an image with a launch configuration type: org.eclipse.debug.ui. [launchconfigurationtypeimages](http://help.eclipse.org/juno/topic/org.eclipse.platform.doc.isv/guide/debug_launch_uiimages.htm?cp=2_0_17_0_3)

define group of tabs for a launch configuration dialog: org.eclipse.debug.ui. [launchconfigurationtabgroups](http://help.eclipse.org/juno/topic/org.eclipse.platform.doc.isv/guide/debug_launch_ui.htm?cp=2_0_17_0_2)

define shortcut for launching application based on current selection/ perspective: org.eclipse.debug.ui. [launchshortcuts](http://help.eclipse.org/juno/topic/org.eclipse.platform.doc.isv/guide/debug_launch_uishortcuts.htm?cp=2_0_17_0_4)

more help at: [launching framework in eclipse](http://www.eclipse.org/articles/article-launch-framework/launch.html)

**add resource markers** :

 [![](_assets/13990376-markers.png)](http://javacurious.files.wordpress.com/2013/01/markers.png) 

add marker (additional information to tag to resources like projects, files, folders): org.eclipse.core.resources. [markers](http://help.eclipse.org/juno/topic/org.eclipse.platform.doc.isv/guide/resadv_markers.htm?cp=2_0_10_8)

more help at: [using markers to tell users about problems and tasks](http://www.eclipse.org/articles/article-mark%20my%20words/mark-my-words.html)

**add preferences:**

 [![](_assets/13990377-preference.png)](http://javacurious.files.wordpress.com/2013/01/preference.png) 

add pages to the preference dialog box: org.eclipse.ui. [preferencepages](http://help.eclipse.org/juno/topic/org.eclipse.platform.doc.isv/guide/preferences_prefs_contribute.htm?cp=2_0_4_3_0)

more help at: [eclipse preferences tutorial](http://www.vogella.com/articles/eclipsepreferences/article.html)

**add wizards** :

 [![](_assets/13990378-wizard.png)](http://javacurious.files.wordpress.com/2013/01/wizard.png) 

add wizard to create a new resource: org.eclipse.ui. [newwizards](http://help.eclipse.org/juno/topic/org.eclipse.platform.doc.isv/guide/dialogs_wizards_newwizards.htm?cp=2_0_5_4_0)

add wizard to import a resource: org.eclipse.ui. [importwizards](http://help.eclipse.org/juno/topic/org.eclipse.platform.doc.isv/guide/dialogs_wizards_importwizards.htm?cp=2_0_5_4_1)

add wizard to export a resource: org.eclipse.ui. [exportwizards](http://help.eclipse.org/juno/topic/org.eclipse.platform.doc.isv/guide/dialogs_wizards_exportwizards.htm?cp=2_0_5_4_2)

more help at: [creating eclipse wizards tutorial](http://www.vogella.com/articles/eclipsewizards/article.html)

**contribute help:**

 [![](_assets/13990379-expanded_book.jpg)](http://javacurious.files.wordpress.com/2013/01/expanded_book.jpg) 

add help for an individual plugin: org.eclipse. [help](http://help.eclipse.org/juno/topic/org.eclipse.platform.doc.isv/guide/ua_help.htm?cp=2_0_19_1) .toc

more help at: [contributing a little help](http://www.eclipse.org/articles/article-online%20help%20for%202_0/help1.htm)

**define specialized searches:**

register search pages: org.eclipse.search. [searchpages](http://help.eclipse.org/juno/topic/org.eclipse.platform.doc.isv/guide/search_page.htm?cp=2_0_14_0)

register search result pages: org.eclipse.search. [searchresultviewpages](http://help.eclipse.org/juno/topic/org.eclipse.platform.doc.isv/guide/search_result.htm?cp=2_0_14_1)

more help at: [custom search page](http://bassistance.de/2009/11/17/eclipse-dev-custom-search-page/)

_i hope it helps someone in the similar need. let me know if it is useful or if you have any suggestions for improvement_ .

happy eclipse, bye  

Opinions expressed by DZone contributors are their own.