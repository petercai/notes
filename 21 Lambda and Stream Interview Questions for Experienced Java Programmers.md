# 21 Lambda and Stream Interview Questions for Experienced Java Programmers
My favorite Java interview questions from Lambda expression and Stream API for experienced Java developers
----------------------------------------------------------------------------------------------------------

[

![](https://miro.medium.com/v2/resize:fit:700/0*z3JiNMjkpR8eUUJu.png)


](https://medium.com/@somasharma_81597/membership)

Hello friends, If you are preparing for Java developer interviews then you may have come across my earlier articles like [**_25 Advanced Java questions_**,](https://medium.com/javarevisited/200-coursera-plus-discount-and-best-new-year-deals-for-developers-in-2023-eb2b682575?source=post_page-----87b243276134----0----------------------------) [_25 Spring Framework questions_](https://medium.com/javarevisited/25-spring-framework-interview-questions-for-1-to-3-years-experienced-java-programmers-567f268ed897), [_20 SQL queries from Interviews_](https://medium.com/javarevisited/20-sql-queries-for-programming-interviews-a7b5a7ea8144?source=user_profile---------0----------------------------), [_50 Microservices questions_](https://medium.com/javarevisited/50-microservices-interview-questions-for-java-programmers-70a4a68c4349?source=user_profile---------3----------------------------),  [_60 Tree Data Structure Questions_](https://medium.com/javarevisited/top-60-tree-data-structure-coding-interview-questions-every-programmer-should-solve-89c4dbda7c5a?source=user_profile---------2----------------------------), [15 System Design Questions](https://medium.com/javarevisited/7-system-design-problems-to-crack-software-engineering-interviews-in-2023-13a518467c3e?source=user_profile---------14----------------------------), and [_35 Core Java Questions_](https://medium.com/javarevisited/top-10-java-interview-questions-for-3-to-4-years-experienced-programmers-c4bf6d8b5e7b).

More than 10K people have read those articles and I am really thankful to all of them. I like to write about Java interview questions because I believe in regular preparation and I learn a lot when I write about Java interviews or prepare for them.

Most of my Java knowledge is the **result of my research for finding answers to Java questions** asked to me on interviews and that’s the reason I am always in search of new and challenging Java questions.

Prior to Java 8, Java Interviews were used to be easy and mostly focused around [_Collection_](https://medium.com/javarevisited/50-java-collections-interview-questions-for-beginners-and-experienced-programmers-4d2c224cc5ab)_s_ and [_Multi-threading_](https://javarevisited.blogspot.com/2014/07/top-50-java-multithreading-interview-questions-answers.html)  but from Java 8 onward, it has become more and more tough with increased focus on Lambda and Stream API.

These 2 are tricky topics and if you have not worked or used them extensively you will most likely to struggle on interviews and that’s why I am sharing these frequently asked Java interview questions from [**Lambda Expression and Stream API**](https://medium.com/javarevisited/8-best-lambdas-stream-and-functional-programming-courses-for-java-developers-3d1836a97a1d) which can help you to learn essential concepts and prepare better.

And, if you are not a Medium member then I highly recommend you to join Medium and read great stories from great authors from real field. You can **join Medium** [**here**](https://medium.com/@somasharma_81597/membership)

So, what are we waiting for, here is the list of popular Java 8 questions related to Stream and Lambda Expression

1.  **What is a lambda expression and how is it used in Java 8? (**[**answer**](https://javarevisited.blogspot.com/2014/02/10-example-of-lambda-expressions-in-java8.html#axzz6ieZZarMY)**)**  
    hint — its a shortcut to pass code to a method, at the moment you can only pass code or lambda expression to a method which accept a functional interface.
2.  **What is difference between Lambda expression and Anonymous class? Are they same? (**[**answer**](https://javarevisited.blogspot.com/2015/01/how-to-use-lambda-expression-in-place-anonymous-class-java8.html#axzz7rK6hQd58)**)**  
    hint — anonymous class can implement non-functional interface as well. It’s also slower than lambda.
3.  **What is functional interface in Java? Can you declare your own functional interface? (**[**answer**](https://javarevisited.blogspot.com/2018/01/what-is-functional-interface-in-java-8.html)**)**  
    hint — any method with just one abstract method like Runnable, Callable or Predicate, Consumer etc
4.  **Is @Functional annotation mandatory for declaring functional interface in Java? (**[**answer**](https://javarevisited.blogspot.com/2018/01/what-is-functional-interface-in-java-8.html)**)**  
    hint — no, you can have functional interface without that annotation like Runnable, Comparator, and Callable.
5.  **Can you use an interface with one abstract method from your old JAR as functional interface without recompiling them with Java 8 compiler?** (answer)  
    hint — yes, you can, no need to recompile code with Java8 compiler
6.  **Can you explain the difference between map and flatMap in Java 8. (**[**answer**](https://javarevisited.blogspot.com/2016/03/difference-between-map-and-flatmap-in-java8.html)**)**  
    you can use map to convert one object to another object while flatmap can also flatten apart from conversion. For example, you can use flatmap to convert a List of List of integer to List of String.
7.  **Which method is used to filter objects in a stream of data? (**[**answer**](https://www.java67.com/2016/08/java-8-stream-filter-method-example.html)**)**  
    hint — you can use filter method to fitler data in stream, it accepts a Predicate which can be used to pass the filtering criterion.
8.  **What is difference between Stream and IntStream class? (answer)**  
    hint — IntStream is a specialized class for int primitive, it will prevent auto-boxing and provide improved performance.
9.  **How can you use the reduce method to perform a reduction operation on a stream of data? (**[**answer**](https://www.java67.com/2016/09/map-reduce-example-java8.html)**)**  
    You can see this example to learn about how to use map and reduce.
10.  **Can you name 5 different types of Functional interface in Java? (**[**answer**](https://www.java67.com/2022/12/top-5-functional-interface-every-java.html)**)**  
    hint —  
    Supplier  
    Predicate  
    Consumer  
    Runnable  
    Function
11.  How can you use the collect method to collect the elements of a stream into a collection? (answer)
12.  What is difference between Supplier and Consumer interface in Java? (answer)
13.  How to perform a parallel computation on a stream of data? (answer)
14.  What is difference between Predicate and Function interface in Java? (answer)
15.  How can you use the Stream API to perform a sorted operation on a stream of data? ([answer](https://www.java67.com/2018/10/java-8-stream-and-functional-programming-interview-questions-answers.html))
16.  **How do you debug Stream in Java? Which method of Stream API can be helpful? (**[**answer**](https://www.java67.com/2016/09/java-8-streampeek-example.html)**)**  
    hint — you can use peek() method to debug stream and see data during different stage of stream pipeline.
17.  How to perform a distinct operation on a stream of data? (answer)
18.  How to perform a grouping and aggregation operation on a stream of data? (answer)
19.  How can you use the Stream API to perform a matching operation on a stream of data? (answer)
20.  **Can you declare more than one method in a functional interface in Java? (answer)**  
    hint — Yes, you can add as many static and default methods as you want as interface can have those but only one abstract method.
21.  **If @Functional annotation is not mandatory then why do you use with functional interface? what benefit it provides? (answer)**  
    hint — it helps during compile time by throwing error if your interface is not really a functional interface for example it contains two abstract method. This is similar to @Override annotation.

That’s all about the **21 Stream and Lambda Expression interview questions.** It’s important to have a good understanding of the Stream API and how to use it to perform operations on a stream of data. You should also be familiar with the different intermediate and terminal operations that are available in the Stream API and how to use them effectively.

Additionally, you should be familiar with lambda expressions, which are used to pass behavior as a method argument. And, If you need more Java 8 questions you can also see this [_50 Java 8 questions_](https://javarevisited.blogspot.com/2021/05/java-8-stream-lambda-expression-d.html) and [15 Java Stream questions](https://www.java67.com/2018/10/java-8-stream-and-functional-programming-interview-questions-answers.html) for more preparation.

And, if you are not a Medium member then I highly recommend you to join Medium and read great stories from great authors from real field. You can **join Medium** [**here**](https://medium.com/@somasharma_81597/membership)