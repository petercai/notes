# How To Use Loops in Java | DigitalOcean
_The author selected the [Free and Open Source Fund](https://www.brightfunds.org/funds/foss-nonprofits) to receive a donation as part of the [Write for DOnations](https://do.co/w4do-cta) program._

### Introduction

Writing repetitive tasks is common in programming. One option is to write the same or similar code as many times as needed to have a block of code executed repetitively. This approach is far from perfect because it will create a lot of duplicate code that is hard to read and maintain. A better option is to use [_loops_](https://en.wikipedia.org/wiki/Control_flow#Loops).

Loops are structures for controlling repetitive program flow. A typical loop has two parts. One part is a Boolean control condition. The other part is a code block that will be executed while the condition is true or until the condition is false. The Boolean condition is reevaluated with each run of the code block. The loop exit criteria is different for each type of loop, as you will learn in this tutorial.

There are two main types of loops: `while` and `for` loops. What type it is depends on the loop’s syntax and logic. The `while` loops depend on a Boolean condition. This condition could be general and while it is true, the code block is repeatedly executed. The `for` loops also repeatedly execute a code block, but the loop condition typically depends on the increment or decrement of an integer variable. When this variable reaches a certain target value, the loop will break.

In this tutorial, you will use `while` and `for` loops to create repetitive tasks and learn about the benefits and drawbacks of each one.

Prerequisites
-------------

To follow this tutorial, you will need:

*   An environment in which you can execute Java programs to follow along with the examples. To set this up on your local machine, you will need the following:
    
    *   Java (version 11 or above) installed on your machine, with the compiler provided by the Java Development Kit (JDK). For Ubuntu and Debian, follow the steps for [Option 1](https://www.digitalocean.com/community/tutorials/how-to-install-java-with-apt-on-ubuntu-22-04#option-1-installing-the-default-jre-jdk) in our tutorial [How To Install Java with Apt on Ubuntu 22.04](https://www.digitalocean.com/community/tutorials/how-to-install-java-with-apt-on-ubuntu-22-04). For other operating systems, including Mac and Windows, follow the [download options for Java installation](https://www.oracle.com/java/technologies/downloads/#java11).
    *   To compile and run the code examples, this tutorial uses [Java Shell](https://docs.oracle.com/javase/10/tools/jshell.htm#JSWOR-GUID-C337353B-074A-431C-993F-60C226163F00), also known as JShell, which is a [_Read-Evaluate-Print Loop_ (REPL)](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop) run from the command line. To get started with JShell, check out the [Introduction to JShell](https://docs.oracle.com/javase/10/jshell/introduction-jshell.htm#JSHEL-GUID-630F27C8-1195-4989-9F6B-2C51D46F52C8) guide.
*   Familiarity with Java and object-oriented programming, which you can find in our tutorial [How To Write Your First Program in Java](https://www.digitalocean.com/community/tutorials/how-to-write-your-first-program-in-java).
    
*   An understanding of Java data types, which is discussed in our tutorial [Understanding Data Types in Java](https://www.digitalocean.com/community/tutorials/understanding-data-types-in-java).
    

`while` Loops
-------------

The `while` loops monitor a generic Boolean conditional statement. For example, the conditional statement could verify whether two integer variables are equal, or whether two objects are the same, related, not `null`, or anything else that can be programmatically checked. Indeed, any Boolean statement could be used, which is why `while` loops are so powerful and universally used.

In this section, you will create your first programming loop in Java using the `while` keyword. You’ll use a single [`int`](https://www.digitalocean.com/community/tutorials/understanding-data-types-in-java#primitive-types) variable to control the loop. The `int` variable will be called `x` and will have an initial value of `3`. While, or as long as, `x` is bigger than `0`, the loop will continue executing a block of code. In this block of code, the value of `x` will be printed and, most importantly, postdecremented (using the `--` operator) with each execution. This [postdecrement](https://www.digitalocean.com/community/tutorials/how-to-use-operators-in-java#unary-operators) operation makes the current loop end after some executions. Without it, the loop will continue infinitely.

**Info:** To follow along with the example code in this tutorial, open the JShell tool on your local system by running the `jshell` command. Then you can copy, paste, or edit the examples by adding them after the `jshell>` prompt and pressing `ENTER`. To exit `jshell`, type `/exit`.

To test out this code, paste the following lines into `jshell`:

```
1.  int x = 3;
    
2.  while (x > 0) {
    
3.      System.out.println("x is " + x--);
    
4.  } 
```

On the first line, you define the `x` variable.

The loop begins on line 2 with the `while` keyword. The conditional statement `(x > 0)` controls the loop. It compares whether `x` is bigger than `0`. After that comes the opening bracket `{`, which denotes the start of the [block of code](https://www.digitalocean.com/community/tutorials/how-to-write-conditional-statements-in-java#differentiating-between-statements-and-blocks). This code block will be executed repeatedly while the condition is true (`x > 0`).

Inside the code block, the `System.out.println("x is " + x--);` statement on line 3 prints the current value of `x` using the `println()` method. (For more on the `System.out.println` statement, check out our tutorial [How To Write Your First Program in Java](https://www.digitalocean.com/community/tutorials/how-to-write-your-first-program-in-java#understanding-the-hello-world-program).) Inside the argument for `println()`, `x` is postdecremented by `1` with the code `x--`. With each execution, `x` decreases by `1`.

The block of code ends on the last line with the closing bracket `}`. This is also the end of the loop.

When you run this code in `jshell`, the following output will print:

```
Outputx ==> 3
x is 3
x is 2
x is 1

```

The first line confirms that `x` has received the value of `3`. On the following lines, the value of `x` is printed three times, decreasing by `1` each time. There are only three iterations where `x` equals `3`, `2`, and `1`. The loop breaks when `x` reaches `0` because the conditional statement `x > 0` is no longer satisfied. At this point, the loop exits and nothing else is printed.

The previous example is a common way to create a loop. The intention of the loop is clear, and the exit condition is straightforward: `x > 0`. In theory, you could make the condition more complex by adding additional variables and comparisons (such as `x > 0` and `y < 0`), but this is not considered a best practice from a [clean-code](https://www.oreilly.com/library/view/clean-code-a/9780136083238/) point of view. Complex conditions make the code hard to follow and understand while bringing little, if any, value.

`do-while` Loops
----------------

The `do-while` loop is less popular than `while` loops, but is still widely used. A `do-while` loop resembles a `while` loop, but they differ in one key aspect: the code block is executed first and then the loop condition is evaluated.

As a reminder, `while` loops first evaluate the condition and may never run the code block if the condition is not met. In `do-while` loops, however, there is always at least one execution of the code block before the loop control condition is evaluated. If the condition is true, the loop will continue additional iterations until it is no longer true. Because of this, `do-while` loops are useful in cases like asking a question and having the loop terminate only upon the correct answer.

In this section, you will create a `do-while` loop similar to the `while` loop in the previous section. The `int` variable will be called `x` and will have an initial value of `3`. This time, however, the condition is reversed (`x < 0`) and the code block will execute before the condition is assessed.

Open `jshell` and add the `do-while` loop:

```
1.  int x = 3;
    
2.  do {
    
3.      System.out.println("x is " + x--);
    
4.  } while (x < 0); 
```

The code has some similarities with the `while` loop from the previous section. First, there is the assignment of the variable `x`, which is again `3`.

On line 2, the loop starts with the keyword `do` and the opening bracket `{` for the code block. On line 3, the value of `x` is printed using the `println()` method. With the `println()` method, you also decrement the value of `x` using the [postdecrement operator](https://www.digitalocean.com/community/tutorials/how-to-use-operators-in-java#unary-operators) `--`.

On the last line is the closing bracket `}` for the code block, followed by the `while` keyword with the conditional statement `(x < 0)`. This is the part that differs from the previous `while` loop. Here, the condition is reversed: `x` should be smaller than `0`. In the previous example, the condition required `x` to be bigger than `0`. This change demonstrates that the code block will run once even though the condition is never satisfied.

When you run this code in `jshell`, the following output will print:

```
Outputx ==> 3
x is 3

```

The first line confirms that the value of `x` is `3`. The second line prints `x is 3`. This is the peculiarity of the `do-while` loop: even though the condition (`x < 0`) is not met, the code block is still executed at least once. This unconditional execution might not always be desired, which is why you need to be careful when using `do-while` loops. Because of this caveat, `do-while` loops are not used as often.

`for` Loops
-----------

An alternative loop structure is the `for` loop. It is also used to run a block of code repeatedly under a condition, but it has more options than a `while` loop. In the loop control condition, you can add the temporary variable, define the control condition, and change the value of the temporary variable. In this way, you bind together all the factors that control the loop, making it less likely to miss something or make a mistake. This allows you to keep your code clean and understandable. The `for` loop is more suitable when you want to write your code adhering to a more strict style.

Here is a typical `for` loop:

The loop starts with the `for` keyword on the first line. The conditional statement in the parenthesis has three parts:

*   `int x = 3` defines a variable (`x`). Defining a temporary variable here is convenient because they are only needed for the loop, so it makes sense to integrate them into the loop. In contrast, when you used a `while` loop, you had to declare the variable separately before the loop.
*   `x > 0` is the condition that has to be met for the block of code to be executed. In this case, `x` has to be bigger than `0`. This part is similar to the `while` loop control statement, which only contains the condition to be evaluated.
*   `x--` is the action taken during each successful iteration. In this case, a postdecrement operation decreases the value of `x` after each successful execution.

The `System.out.println("x is " + x);` statement on line 2 prints the value of `x` using the `println()` method. It receives the value of `x` unchanged. In contrast, you used the `println()` method to manipulate the value of `x` in the `while` loop. This is unnecessary here because the postdecrement operation is already taken care of in the third part of the conditional statement: `x--`.

Line 3 terminates the block of code and the whole loop structure with the closing bracket `}`.

When you run this code in `jshell`, the following output prints:

```
Outputx is 3
x is 2
x is 1

```

The output confirms that the block of code has been executed three times. The first time, the value of `x` is `3`. The second time, `x` is `2`, and the third time, `x` is `1`. After the third time, `x` becomes `0` and the loop exits without executing the block of code (which prints the value of `x`).

This `for` loop achieves the same result as the `while` loop. From a performance and resource usage point of view, there should be no difference, so it is mostly a matter of personal preference which loop to use. However, if the number of iterations is known, usually the `for` loop is preferred because it allows you to follow the code more closely.

`foreach` Loops
---------------

A `foreach` loop is similar to the other loops, but its benefit is that it is specifically designed to iterate over a group of values with as little code as possible. As long as you have a group of values, you can go through them without having to keep track of their number or follow the iteration progress. The `foreach` loop will ensure that if there are values in the group, they will be pulled one by one and presented to you.

Most programming languages have a convenient shortcut for the `foreach` loop. In Java, there is no dedicated `foreach` keyword, but instead, the `for` keyword is used. However, the `foreach` conditional statement differs from the common `for` loop, as you will discover next.

To work with `foreach` loops, you have to use a group of values, and an [array](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/arrays.html#:~:text=An%20array) is suitable for this purpose. An array is an object that contains values of the same type. In the following example, you will use an array containing `int` values.

Add the following code for a `foreach` loop to `jshell`:

```
1.  int[] numbers = {0, 1, 2};
    
2.  for (int x: numbers) {
    
3.      System.out.println(x);
    
4.  } 
```

On the first line, you create an `int` array holding three integers (`0`, `1`, and `2`).

The `foreach` loop starts on Line 2 with the `for` keyword. Then, you define a temporary variable `int x`, followed by a colon. The colon is used as a shortcut for the `foreach` operation. (The `foreach` loop statement is unique because this is the only place in Java where a colon is used.) After the colon is a collection of values, such as an array.

After the `foreach` loop statement is the code block that will be executed for each value of the `numbers` array. In this case, the `System.out.println(x);` statement will print the value of `x` using the `println()` method, which will refer to each value of the `numbers` array.

When you run the above code in `jshell`, the following output prints:

```
Outputnumbers ==> int[3] { 0, 1, 2 }
0
1
2

```

The first line confirms an array called `numbers` has been defined that contains the values `0`, `1`, and `2`. The following lines print the array’s values, each on a new line: `0`, `1`, and `2`.

While using the other loops is mostly a matter of personal choice, the `foreach` loop is the de-facto standard for iterating over a group of values. It is possible to create your own alternative of the `foreach` loop using a `for` or `while` loop, but it is not worth the effort and requires a deep understanding of the data structures.

Infinite Loops
--------------

Infinite loops are `for` or `while` loops that never terminate. They become infinite when the control condition is always true.

Here’s a common example of an infinite `while` loop:

The first line looks like a typical `while` loop, but instead of a condition such as `x > 0`, there is just the value `true`. In this case, when a control condition should be evaluated, `true` is always returned directly. In this case, there is no exit for the loop, so it will continue forever. The second line contains the `System.out.println("This will print forever.");` statement; it will print the text `This will print forever.` using the `println()` method.

When you run the above code in `jshell`, the following output will print until you terminate the code. Be ready to terminate it quickly with the keyboard combination `CTRL+C` or just close the terminal. Otherwise, it will continue endlessly:

```
OutputThis will print forever.
This will print forever.
...

```

Only the first two repetitions are shown but this output will continue until you press `CTRL+C` to terminate it manually.

With the example infinite loop, there is a clear intention to create such a loop because of the single `true` value placed instead of a Boolean statement. Such loops were popular in the past and may be present in legacy projects. You can still create such infinite loops, but there are better ways to create continuously running tasks, such as with [Timer](https://docs.oracle.com/javase/6/docs/api/java/util/Timer.html) tasks. `Timer` tasks are preferred because they are more sophisticated and offer more options.

Unfortunately, sometimes infinite loops are not intentional. You could make a mistake and cause an infinite loop. Consider the following code:

```
1.  int x = 3;
    
2.  while (x > 0) {
    
3.      System.out.println("x is " + x);
    
4.  } 
```

The above code looks similar to the very first example in this tutorial. On the first line, you define a variable `x` with the value `3`.

On line 2 comes the `while` keyword and the control condition, which states that the loop should continue while `x > 0` is true".

The `System.out.println("x is " + x);` statement on line 3 prints the value of `x` using the `println()` method. However, in contrast to the `while` loop example in the [first section](https://www.digitalocean.com/community/tutorials/how-to-use-loops-in-java) of this tutorial, where the loop ended after three runs, there is no postdecrement operation for `x` in the current example. Because of that, when you run this loop in `jshell`, you will get the following continuous output:

```
Outputx is 3
x is 3
...

```

These lines will repeat endlessly until you terminate the loop with `CTRL+C`. Such unintentional infinite loops are dangerous because they may cause a program to crash or to overload the machine running the program. That’s why you should be careful when creating loops and ensure there is a viable termination path. In the current example, you should ensure that `x` is decremented with each loop to meet the loop exit condition `x > 0`.

Conclusion
----------

In this tutorial, you used loop structures to create repetitive tasks. You learned when it is suitable to use `while` and `for` loops and wrote a few code examples with them. You also learned best practices for clean code and how to avoid common pitfalls related to loops.

For more on Java, check out our [How To Code in Java](https://www.digitalocean.com/community/tutorial_series/how-to-code-in-java) series.