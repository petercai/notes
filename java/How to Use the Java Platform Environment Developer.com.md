# How to Use the Java Platform Environment | Developer.com
![](_assets/java-programming-tutorials-tips-300x200.png)

When programmers create a Java program, it runs in an environment that includes your operating system, the Java Virtual Machine (JVM), and Java class libraries. Developers can configure different aspects of their program’s development environment from the command line.

This programming tutorial takes you through the different ways in which you can use and configure the Java environment on your system.

**Read:** [Top Java Online Training Courses and Bundles](https://www.developer.com/java/java-online-training-courses/)

Java Configuration Utilities
----------------------------

Below, we will examine several of the configuration utilities available in the Java Platform Environment.

### Java Command Line Arguments

Developers can pass one or more arguments to the class of your program. Recall that your program’s **main()** function contains the **String\[\] args** array. If you have **n** arguments, you can access them by using **args\[0\].. arg\[n-1\]**.

See the sample code below, which shows how to take in user input and then print it on the screen using Java:

class JavaArgs{


   public static void main(String\[\] args) {
       if (args.length >= 1){
           for(String item: args){
               System.out.println(item);
           }
       } else{
           System.out.println("No input from user");
       }     
   }


}

### Java Environment Variables

Java _environment variables_ are variables which are specific to your computing environment. For example, the **home** directory in which Linux user **A** exists is different from that of Linux user **B**.

In Linux, programmers can set an environment variable from the terminal, using the **export** command:

$ export MY_VARIABLE=value

To output the value of your variable, you can use the **echo** command:

$ echo $MY_VARIABLE

Developers can also use the **printenv** command to do print the value of a variable as well:

$ printenv MY_VARIABLE

Notice that, when using **printenv**, you do not need to include the **$** symbol before the variable name.

It is important to note that variables you set from the terminal are temporary (i.e they only last while your system is on).

To make these variables persistent, programmers need to define them in the **.bashrc** file. Note that this is a hidden file, so it will not show up when you search for it on your Linux system. The **.bashrc** file is located in your **home** directory.

You can add an environment variable in this file by simply adding a line with the **export** command as shown earlier.

**Read:** [Working with Java Variables](https://www.developer.com/java/java-variables/)

Java System Utilities
---------------------

The following subsections discuss functionality that the **System** class provides in Java. This class is part of the **java.lang** package and it provides the following three streams in its static fields:

*   **in:** The standard input stream (your keyboard)
*   **output:** The standard output stream (the screen)
*   **err:** The standard error stream

The **in** variable is an **InputStream** object, while the **output** and **err** variables are **PrintStream** objects.

### Input/ Output Objects

This section discusses how you can use the **System** I/O objects.

To output data on your screen, there are a number of methods that the **out** field provides, including:

*   print()
*   println()

Both of these methods can be used to display a numerical data type, a boolean value, a String, an array or even an object on your screen. These values are passed as an argument to these methods.

The difference between them is that **println()** terminates with a line separator while **print()** ends with the element in the stream.

To input user data from the screen, you can use the **java.util.Scanner** class. First, you need to create an object of this class:

Scanner input = new Scanner (System.in);

After creating an object of the **java.util.Scanner** class, you can use the **nextXXX()** method to capture data. **XXX** represents any of the following data types: **Int, Long, Float, Double,** or **Boolean**.

For example, to capture an integer value from the user using **next() in Java, you would write the following code:**

int a = input.nextInt();

To get a String value, simply use **next()**.

**Read:** [Java Tools to Increase Productivity](https://www.developer.com/java/java-tools-productivity/)

### Java System Properties

Java provides the **Properties** class to store configuration values for your particular environment as **key/value** pairs. This class is part of the **java.util** package and it extends the **Hashtable** class.

It is important for a developer to know your system properties because they can affect the kind of notation that you use in your Java programs. A good example would be the _file separator_.

In Windows, the file separator is a backslash (“**\**“) , while in Linux it is a forward slash (“**/**“).

The **getProperties()** method of the **System** class allows programmers to check all the properties of your current environment. This method returns a **Properties** object.

As mentioned earlier, the **Properties** class uses the hashtable data structure. Therefore, you can use the hashtable methods to enumerate the key/value pairs.

The code example below, however, uses a simpler approach. It uses the **toString()** method to generate a comma separated list of key/value pairs:

import java.util.*;
import java.io.*;


class EnvProperties {
    public static void main(String\[\] args) throws Exception
    {
       Properties props = System.getProperties();
       System.out.println(props.toString());
              
   }
}

Final Thoughts on the Java Platform Environment
-----------------------------------------------

This programming tutorial discussed how developers can use the **System** class, **Properties** class, and set environment variables in the Java Platform Environment. Notice that none of the examples in this tutorial instantiated the **System** class; this is because the Java API defines **System** as a _final_ class. Therefore, you cannot create an instance of it.

**Read:** [Java Primitive Data Types](https://www.developer.com/java/java-primitive-data-types/)