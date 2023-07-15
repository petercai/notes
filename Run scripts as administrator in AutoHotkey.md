# Run scripts as administrator in AutoHotkey 
Save From : [Run scripts as administrator? - AutoHotkey Community](https://www.autohotkey.com/boards/viewtopic.php?t=21278) 

## Content
I would not recommend changing compatibility settings of AutoHotkey.exe.  
  
Running as administrator is not the only solution; see the [FAQ: How do I work around problems caused by User Account Control (UAC)?](https://autohotkey.com/docs/FAQ.htm#uac)  
  
A script can also run itself as admin by using Run *RunAs "%A\_AhkPath%" "%A\_ScriptFullPath%". (A_AhkPath is usually optional if the script has the .ahk extension.) You would typically check if not A_IsAdmin first.  
  
If you want a "Run as administrator" option for txt files, you can try copying the Shell\\RunAs\\Command subkey of HKCR\\AutoHotkeyScript into the text file key (usually HKCR\\txtfile), but I think you're better off just using the .ahk extension. If you want Edit to be the default (for double-click), you can change the default value of HKCR\\AutoHotkeyScript\\Shell to Edit (it is usually Open).
