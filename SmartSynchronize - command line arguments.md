# SmartSynchronize - command line arguments

Command-Line Parameters[](#command-line-parameters)
---------------------------------------------------

Use one of these parameters to define what to open `--file-manager`, `--file-compare`, `--dir-compare` or `--file-merge`. Add further parameters specifying the file or directory paths: `--left=<path>`, `--right=<path>` or `--merge=<file>` (only for `--file-merge`). If relative paths are used, you can use the parameter `--root=<dir>` to specify to which directory the paths should be relative. If no such parameter is specified, the relative paths will be treated relative to the current directory.

### Examples[](#examples)

```
smartsynchronize.exe --file-compare --left=C:\path1\left.txt --right=C:\path2\right.txt 
```

This will open a file compare for the files `C:\path1\left.txt` and `C:\path2\right.txt`.

```
smartsynchronize.exe --file-manager --left=D:\my-projects 
```

This will open a file manager window with the directory `D:\my-projects` shown at the left side.

Alternatively, you can pass the file or directory names as command line parameters. If you pass three file paths, the 3-Way-Merge opens. If you pass two file paths, the File Compare opens, if you pass two directory paths, the Directory Compare opens. If you pass a file and a directory paths together, SmartSynchronize tries to take the opposite file for the File Compare from the specified directory using the name of the specified file. If SmartSynchronize could not find a suitable command to perform directly, the Welcome dialog is displayed with the specified files or directories in the file or directory input fields.

### Examples[](#examples-1)

```
smartsynchronize.exe C:\path1\left.txt C:\path2\right.txt C:\path\base.txt 
```

This will open the 3-Way Merge with `left.txt` on the left, `right.txt` on the right and `base.txt` as the base, where also the merge result will be stored.

```
smartsynchronize.exe C:\path1\file1.txt C:\path2\file2.txt 
```

This will open the File Compare with `file1.txt` on the left and `file2.txt` on the right.

```
smartsynchronize.exe C:\path1\dir1 C:\path2\dir2 
```

This will open the Directory Compare with `dir1` being the left and `dir2` the right directory.

```
smartsynchronize.exe C:\path1\file.txt C:\path2\dir2 
```

This will try to open the File Compare with `C:\path1\file.txt` on the left and `C:\path2\dir2\file.txt` on the right.

### Command-Line Usage on macOS[](#command-line-usage-on-macos)

To launch SmartSynchronize without passing command line parameters, just invoke the SmartSynchronize application (assuming SmartSynchronize is installed in `/Applications`):

```
$ open /Applications/SmartSynchronize.app 
```

To launch SmartSynchronize with passing command line parameters, invoke the following shell-script embedded in the SmartSynchronize application (assuming SmartSynchronize is installed in `/Applications`):

```
$ /Applications/SmartSynchronize.app/Contents/MacOS/Smartsynchronize file1.txt file2.txt 
```

To not have to type the full path each time, it is recommended to create a symbolic link to the shell-script in your path (write on one line):

```
$ sudo ln -s /Applications/SmartSynchronize.app/Contents/MacOS/Smartsynchronize /usr/local/bin/compare 
```

Then it becomes much less typing:

```
$ compare file1.txt file2.txt 
```