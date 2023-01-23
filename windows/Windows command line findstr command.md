#  Windows command line findstr command
Updated: 12/30/2021 by

![](https://www.computerhope.com/cdn/dos/findstr-command.gif)

The **findstr** (short for find string) command is used in MS-DOS to locate files containing a specific [string](https://www.computerhope.com/jargon/s/string.htm) of [plain text](https://www.computerhope.com/jargon/p/plaintex.htm).

Availability
------------

Findstr.exe is an [external command](https://www.computerhope.com/jargon/e/extecomm.htm) that is available for the following Microsoft operating systems.

*   [Windows 2000](https://www.computerhope.com/jargon/w/win2000.htm)
*   [Windows ME](https://www.computerhope.com/jargon/w/winme.htm)
*   [Windows XP](https://www.computerhope.com/jargon/w/winxp.htm)
*   [Windows Vista](https://www.computerhope.com/jargon/v/vista.htm)
*   [Windows 7](https://www.computerhope.com/jargon/w/windows7.htm)
*   [Windows 8](https://www.computerhope.com/jargon/w/windows8.htm)
*   [Windows 10](https://www.computerhope.com/jargon/w/windows-10.htm)
*   [Windows 11](https://www.computerhope.com/jargon/w/windows-11.htm)

Tip

Microsoft Windows and MS-DOS users who do not have support for this command can use the [find command](https://www.computerhope.com/findhlp.htm).

Syntax
------

### Windows Vista and later syntax

```
FINDSTR \[/B\] \[/E\] \[/L\] \[/R\] \[/S\] \[/I\] \[/X\] \[/V\] \[/N\] \[/M\] \[/O\] \[/P\] \[/F:file\] \[/C:string\] \[/G:file\] \[/D:dir list\] \[/A:color attributes\] \[/OFF\[LINE\]\] strings \[\[drive:\]\[path\]file name\[ ...\]\]
```

| /B | Matches pattern if at the beginning of a line. |
| /E | Matches pattern if at the end of a line. |
| /L | Uses search strings literally. |
| /R | Uses search strings as [regular expressions](https://www.computerhope.com/jargon/r/regex.htm). |
| /S | Searches for matching files in the [current directory](https://www.computerhope.com/jargon/c/currentd.htm) and all [subdirectories](https://www.computerhope.com/jargon/s/subdirec.htm). |
| /I | Specifies that the search is not to be [case-sensitive](https://www.computerhope.com/jargon/c/case.htm). |
| /X | Prints lines that match exactly. |
| /V | Prints only lines that do not contain a match. |
| /N | Prints the line number before each line that matches. |
| /M | Prints only the [file name](https://www.computerhope.com/jargon/f/filename.htm) if a file contains a match. |
| /O | Prints character offset before each matching line. |
| /P | Skip files with non-printable characters. |
| /OFF\[LINE\] | Do not skip files with offline attribute set. |
| /A:attr | Specifies color attribute with two hex digits. See "color /?" |
| /F:file | Reads file list from the specified file(/ stands for console). |
| /C:string | Uses specified string as a literal search string. |
| /G:file | Gets search strings from the specified file(/ stands for console). |
| /D:dir | Search a [semicolon-delimited](https://www.computerhope.com/jargon/s/semicolo.htm) list of directories. |
| strings | Text to be searched. |
| \[drive:\]  
\[path:\]  
file name | Specifies a file or files to search. |

You'll need to use spaces to separate multiple search strings unless the argument is prefixed with /C. For example, 'FINDSTR "hello there" x.y' searches for "hello" or "there" in file x.y. 'FINDSTR /C:"hello there" x.y' searches for "hello there" in file x.y.

Regular expression quick reference:

| . | [Wildcard](https://www.computerhope.com/jargon/w/wildcard.htm): any character. |
| * | Repeat: zero or more occurrences of previous character or class. |
| ^ | Line position: beginning of line. |
| $ | Line position: end of line. |
| \[class\] | Character class: any one character in set. |
| \[^class\] | Inverse class: any one character not in set. |
| \[x-z\] | Range: any characters in the specified range. |
| \\x | Escape: literal use of [metacharacter](https://www.computerhope.com/jargon/m/metachar.htm) x. |
| \\<xyz | Word position: beginning of word. |
| xyz\\> | Word position: end of word. |

### Windows XP and earlier syntax

```
FINDSTR \[/B\] \[/E\] \[/L\] \[/R\] \[/S\] \[/I\] \[/X\] \[/V\] \[/N\] \[/M\] \[/O\] \[/P\] \[/F:file\] \[/C:string\] \[/G:file\] \[/D:dir list\] \[/A:color attributes\]\[strings\] \[\[drive:\]\[path\]file name\[ ...\]\]
```

| /B | Matches pattern if at the beginning of a line. |
| /E | Matches pattern if at the end of a line. |
| /L | Uses search strings literally. |
| /R | Uses search strings as regular expressions. |
| /S | Searches for matching files in the current directory and all subdirectories. |
| /I | Specifies that the search is not to be case-sensitive. |
| /X | Prints lines that match exactly. |
| /V | Prints only lines that do not contain a match. |
| /N | Prints the line number before each line that matches. |
| /M | Prints only the file name if a file contains a match. |
| /O | Prints character offset before each matching line. |
| /P | Skip files with non-printable characters. |
| /A:attr | Specifies color attribute with two hex digits. See "color /?" |
| /F:file | Reads file list from the specified file(/ stands for console). |
| /C:string | Uses specified string as a literal search string. |
| /G:file | Gets search strings from the specified file(/ stands for console). |
| /D:dir | Search a semicolon-delimited list of directories. |
| strings | Text to be searched. |
| \[drive:\]  
\[path:\]  
file name | Specifies a file or files to search. |

You'll need to use spaces to separate multiple search strings unless the argument is prefixed with /C. For example, 'FINDSTR "hello there" x.y' searches for "hello" or "there" in file x.y. 'FINDSTR /C:"hello there" x.y' searches for "hello there" in file x.y.

Regular expression quick reference:

| . | Wildcard: any character. |
| * | Repeat: zero or more occurrences of previous character or class. |
| ^ | Line position: beginning of line. |
| $ | Line position: end of line. |
| \[class\] | Character class: any one character in set. |
| \[^class\] | Inverse class: any one character not in set. |
| \[x-z\] | Range: any characters in the specified range. |
| \\x | Escape: literal use of metacharacter x. |
| \\<xyz | Word position: beginning of word. |
| xyz\\> | Word position: end of word. |

Examples
--------

```
findstr "computer help" myfile.txt
```

In the example above, any lines containing "computer help" would be printed to the screen.

```
findstr /s "computer help" *.txt
```

Similar to the first example, the code above would find lines containing "computer help" in any txt file in the current directory and all subdirectories.

```
findstr /x /c:"computer help" *.txt
```

Match .txt files that contain an exact match on "computer help"; therefore, files that contain "computer helps" or other non-exact matches are not shown. Realize though that using /x must be a line that exactly matches "computer help"; in other words, if anything else is on the same line, it's not an exact match.

```
findstr /n /i /c:"computer help" *
```

Search for any file containing "computer help" regardless of its case and display the line where the text is found. Below is an example of how the results in the example above may look.

```
**tables.htm:20:** src="https://www.computerhope.com/logo.gif" width="96" height="69" alt="Computer Hopes free computer help." border="0"></a></font></td>
**v.htm:10:** content="computer help, computer, hardware, help, hardware help, support, video card, video card support, video card help, vlb, vesa, local, bus">
**wissues.htm:8:** <meta name="keywords" content="windows 95 help, dos help, computer help Windows issues, boot, ">
**xdoseror.htm:20:** src="https://www.computerhope.com/logo.gif" width="96" height="69" alt="Free computer help." border="0"></a></font></td>
```