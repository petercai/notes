# 25 Eclipse Shortcut Keys for Code Editing
Details

Written by  

Last Updated on 07 August 2019   |  [ Print ](https://www.codejava.net/ides/eclipse/25-eclipse-shortcut-keys-for-code-editing?tmpl=component&print=1&page= "Print") [Email](https://www.codejava.net/component/mailto/?tmpl=component&template=protostar&link=4557981ff8246d50f838692d2f7e11d45b62c7f6 "Email")

When using an IDE, you cannot be more productive without using its shortcut keys frequently as your habit. In this article, we summarize a list of shortcut keys which are useful for editing Java code in Eclipse IDE.

**NOTE:** Standard shortcuts are not covered, such as Ctrl + A (select all), Ctrl + Z (undo), etc.

1.  **Ctrl + D**: Deletes current line.
2.  **Ctrl + Delete:** Deletes next word after the cursor.
3.  **Ctrl + Shift + Delete:** Deletes from the cursor until end of line.
4.  **Ctrl + Backspace:** Deletes previous word before the cursor.
5.  **Shift + Ctrl + y:** Changes a selection to lowercase.
6.  **Shift + Ctrl + x:** Changes a selection to uppercase.
7.  **Alt + Up Arrow**: Moves up current line (or a selected code block) by one line:![](_assets/Move%20up%20a%20line.png)
    
8.  **Alt + Down Arrow**: Moves down current line (or a selected code block) by one line:![](_assets/Move%20down%20a%20line.png)
    
9.  **Ctrl + Alt + Up Arrow:** Copies and moves up current line (or a selected code block) by one line:![](_assets/Copy%20and%20move%20up%20a%20line.png)
    
10.  **Ctrl + Alt + Down Arrow:** Copies and moves down current line (or a selected code block) by one line:![](_assets/Copy%20and%20move%20down%20a%20line.png)
    
11.  **Shift + Enter:** Inserts a blank line after current line, regardless where the cursor is at the current line (very different from press **Enter** key alone):![](_assets/insert%20a%20blank%20line.png)
    
12.  **Ctrl + Shift + Enter:** works similar to the **Shift + Enter**, but inserts a blank line just before the current line.
13.  **Ctrl + Shift + O:** Organizes import statements by removing unused imports and sorts the used ones alphabetically. This shortcut also adds missing imports.![](_assets/organize%20imports.png)
    
14.  **Ctrl + Shift + M:** Adds a single import statement for the current error due to missing import. You need to place the cursor inside the error and press this shortcut:![](_assets/add%20single%20import.png)
    
15.  **Ctrl + Shift + F:** Formats a selected block of code or a whole source file. This shortcut is very useful when you want to format messy code to Java-standard code. Note that, if nothing is selected in the editor, Eclipse applies formatting for the whole file:![](_assets/format%20code.png)
    
16.  **Ctrl + I:** Corrects indentation for current line or a selected code block. This is useful as it helps you avoid manually using **Tab** key to correct the indentation:![](_assets/correct%20indentation.png)
    
17.  **Ctrl + /**or **Ctrl + 7**: Toggle single line comment. This shortcut adds single-line comment to current line or a block of code. Press again to remove comment. For example:![](_assets/single%20line%20comment%20code.png)
    
18.  **Ctrl + Shift + /:** Adds block comment to a selection.![](_assets/add%20block%20comment.png)
    
19.  **Ctrl + Shift + \\:** Removes block comment.  
    
20.  **Alt + Shift + S:** Shows context menu that lists possible actions for editing code:![](_assets/context%20menu%20for%20code.png)
      
    From this context menu, you can press another letter (according to the underscore letters in the names) to access the desired functions.
    
21.  **Alt + Shift + S, R:** Generates getters and setters for fields of a class. This is a very handy shortcut that helps us generate getter and setter methods quickly. The following dialog appears:![](_assets/Generate%20getters%20and%20setters.png)
    
22.  **Alt + Shift + S, O:** Generates constructor using fields. This shortcut is very useful when you want to generate code for a constructor that takes class’ fields as its parameters. The following dialog appears:![](_assets/generare%20constructor%20using%20fields.png)
    
23.  **Alt + Shift + S, C:** Generates Constructors from Superclass. A common example for using this shortcut is when creating a custom exception class. In this case, we need to write some constructors similar to the **Exception** superclass. This shortcut brings the _Generate Constructors from Superclass_ dialog which allows us to choose the constructors to be implemented in the subclass:![](_assets/generate%20constructors%20from%20superclass.png)
    
24.  **Alt + Shift + S, H:** Generates [hashCode() and equals() methods](https://www.codejava.net/java-core/collections/understanding-equals-and-hashcode-in-java), typically for a JavaBean/POJO class. The class must have non-static fields. This shortcut brings the _Generate hashCode() and equals()_ dialog as below:![](_assets/generate%20hashCode%20and%20equals.png)
    Select the fields to be used in **hashCode()** and **equals()** method, and then click OK. We got the following result (example):
    
    ```
    
    ```
    
     
25.  **Alt + Shift + S, S:** Generates **toString()** method. This shortcut comes in handy if we want to override **toString()** method that returns information of relevant fields of the class. This brings the _Generate toString()_ dialogas below:![](_assets/Generate%20toString%20method.png)
    And here’s sample of a generated **toString()** method:
    
    ```
    
    ```
    

  

### Related Eclipse Shortcut Keys Tutorials:

*   [16 Eclipse Shortcut Keys for Workspace and Project](https://www.codejava.net/ides/eclipse/16-eclipse-shortcut-keys-for-workspace-and-project)
*   [8 Eclipse Shortcut Keys for Code Refactoring](https://www.codejava.net/ides/eclipse/8-eclipse-shortcut-keys-for-code-refactoring)

 

### Other Eclipse Tutorials:

*   [How to use Eclipse IDE for Java EE Developers](https://www.codejava.net/ides/eclipse/how-to-use-eclipse-ide-for-java-ee-developers)
*   [How to create, build and run a Java Hello World program with Eclipse](https://www.codejava.net/ides/eclipse/how-to-create-build-and-run-a-java-hello-world-program-with-eclipse)
*   [How to change font for Java code in Eclipse](https://www.codejava.net/ides/eclipse/how-to-change-font-for-java-code-in-eclipse)
*   [How to Add Copyright License Header for Java Source Files in Eclipse](https://www.codejava.net/ides/eclipse/how-to-add-copyright-license-header-for-java-source-files-in-eclipse)
*   [How to generate Javadoc in Eclipse](https://www.codejava.net/ides/eclipse/how-to-generate-javadoc-in-eclipse)
*   [How to create JAR file in Eclipse](https://www.codejava.net/ides/eclipse/how-to-create-jar-file-in-eclipse)
*   [How to generate WAR file in Eclipse](https://www.codejava.net/ides/eclipse/eclipse-create-deployable-war-file-for-java-web-application)
*   [How to add custom Ant build script to Eclipse project](https://www.codejava.net/ides/eclipse/add-custom-ant-build-script-to-eclipse-project)
*   [How to pass arguments when running a Java program in Eclipse](https://www.codejava.net/ides/eclipse/how-to-pass-arguments-when-running-a-java-program-in-eclipse)
*   [How to create Java web project with Maven in Eclipse](https://www.codejava.net/ides/eclipse/how-to-create-java-web-project-with-maven-in-eclipse)

  

### About the Author:

![](_assets/NamAuthor.png)
[Nam Ha Minh](https://www.codejava.net/nam-ha-minh) is certified Java programmer (SCJP and SCWCD). He started programming with Java in the time of Java 1.4 and has been falling in love with Java since then. Make friend with him on [Facebook](https://www.facebook.com/namjavaprogrammer) and watch [his Java videos](https://www.youtube.com/codejava?utm_source=codejava&utm_campaign=aboutnamhm) you YouTube.