# Make App-Specific Hotkeys With AutoHotkey
Save From : [How to Make App-Specific Hotkeys With AutoHotkey](https://www.makeuseof.com/autohotkey-app-specific-hotkeys/) 

## Content
Time to Make a Script in AutoHotKey
-----------------------------------

Right-click on your script and choose **Edit Script** to open it in your default text editor. As you will see, it will be pre-populated with some values that help with compatibility and performance. Ignore them, press Enter one or two times, and target your app using:

#IfWinActive APP_IDENTIFIER

Replace APP_IDENTIFIER with the actual target that you copied from AutoHotkey's Window Spy. In our case, this translated to:

#IfWinActive ahk_exe Obsidian.exe

  

When writing AutoHotkey scripts, you can use the following symbols for the modifier keys on your keyboard:

googletag.cmd.push(function() { googletag.display('adsninja-ad-unit-characterCountRepeatable-636c2cc1cf2a8-REPEAT4'); });

*   ! for Alt
*   \+ for Shift
*   ^ for CTRL
*   \# for the Windows Key

Before creating your actual shortcuts, though, test if the script will indeed only work when your chosen application is active. The easiest way to do it is by using what AutoHotkey calls "a message box" or, rather, a "msgbox".

  

Type the following directly under the line where you targeted the application you chose:

 `^a::  
  
msgbox it works!  
  
return` 

If translated into plain English, this would look like:

*   When **CTRL + A** are pressed together on the keyboard...
*   ... show a message box on the screen that states "it works!".
*   When the user acknowledges that message box, return to the previous state.

Run your script, press **CTRL + A** on your keyboard, and nothing should happen. That's because you've targeted a specific application but haven't yet switched to it. So, activate that application's window, press the same combination, and you should see a message box pop up stating that "it works".

googletag.cmd.push(function() { googletag.display('adsninja-ad-unit-characterCountRepeatable-636c2cc1cf2a8-REPEAT5'); });

Now, switch back to any other application and retry your key combo. Hopefully, nothing should happen. If so, this means your MSGBOX activates only in your targeted app, which is the desired result we want from this script.

  

If the keybind does "leak" into other apps, double-check your syntax, and ensure there is no typo in your selected target.

How to Make Custom Keyboard Profiles for Your Apps
--------------------------------------------------

AutoHotkey makes it easy to remap what the keys on your keyboard do, both individually and when combined. Would you like to swap the A and B keys? In AutoHotkey syntax, this would look like this:

 `a::b  
  
b::a` 

However, you probably don't want to remap individual keys, but to have multi-key combinations, with one or more modifier keys, perform specific actions.

To build on the previous example, if you want B to appear when you press CTRL+A and, vice versa, A to pop up when pressing CTRL+B, try:

googletag.cmd.push(function() { googletag.display('adsninja-ad-unit-characterCountRepeatable-636c2cc1cf2a8-REPEAT6'); });

 `^a::b  
  
^b::a` 

Of course, this is merely an example. In real life, pressing multiple keys to type a single character is the very definition of counterproductive. In contrast, assigning text strings to key combinations can significantly speed up text entry. To have your name, email address, or any other piece of text typed when you press a key combination, you can use AutoHotkey's "send" command. This "tells" AutoHotkey, as its name states, "send" the string of text that follows it to the active window. In action, it may look like this:

 `^+O::  
send Odysseas  
return` 

  

In the above script:

*   We begin by "telling" AutoHotkey that it should do something when we press Shift + CTRL + O at the same time.
*   That "something" is sending the string "Odysseas", which happens to be this writer's name, to the active window.
*   Finally, with "return", we state the equivalent of "that will be all, thanks, AutoHotkey!".

googletag.cmd.push(function() { googletag.display('adsninja-ad-unit-characterCountRepeatable-636c2cc1cf2a8-REPEAT7'); });

  

Try experimenting with different key combinations and having AutoHotkey send various text strings to your chosen application. You can have multiple rules in the same script.

Using keyboard combinations to enter text strings may be helpful for instantly entering your name and email address. However, it isn't intuitive when typing. After a while, it becomes hard to keep track of what dozens of shortcuts do. That's where text expansion can help.

Instead of mapping specific key combinations to text strings, AutoHotkey allows you to define shortcodes. Then, when it detects that you typed one of them, it can automatically replace it with a longer text string. It's as simple as:

 `:*:MUO~::Make Use Of` 

*   The ":*:" at the beginning of the line states that this is a text expansion rule.
*   Then comes the shortcode, which in our case is "MUO~".
*   As with shortcuts, "::" are the logical equivalent of "=" in this scenario.
*   The final piece of the puzzle is the actual string of text with which we want to replace "MUO~".

googletag.cmd.push(function() { googletag.display('adsninja-ad-unit-characterCountRepeatable-636c2cc1cf2a8-REPEAT8'); });

With this rule, whenever we type **MUO~** in our targeted app, AHK will jump in and replace it with **Make Use Of**.

After you're done defining rules for an application, you can target another one in precisely the same way. Use "#IfWinActive APP_IDENTIFIER" again, this time targeting another app's window, and type your rules for it directly underneath.

Repeat as many times as you wish, creating app-specific profiles of shortcuts and shortcodes.

Since AutoHotkey scripts are basically text files, here's a nifty idea: incorporate other scripts in your own, and also make them app-specific! Check our list of [cool AutoHotkey Scripts](https://www.makeuseof.com/tag/10-cool-autohotkey-scripts-make/). Choose any that you like, but instead of using them as standalone scripts, open them in a text editor.

Copy their contents and add them under an app-targeting section of your script. Save and re-run your script, and theoretically, those scripts should work as part of your own when the app you've targeted is active.
## Note