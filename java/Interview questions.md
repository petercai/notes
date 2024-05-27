   

# **Object-Oriented Programming (OOP)** interview questions and answers for Java developers:

## 1. **What is a Class in Object-Oriented Programming?**
    
    - A class is a blueprint used to create objects. It defines the properties (attributes) and behaviors (methods) that objects of that class will have.
## 2. **What is an Object in OOP?**
    
    - An object is an instance of a class. It represents a specific state of the class, with its own data and behavior.
## 3. **What is Abstraction in Java?**
    
    - Abstraction is an OOP technique that hides complexities from clients. It allows you to define a high-level interface without specifying the implementation details.
## 4. **What is Inheritance in Java?**
    
    - Inheritance is an object-oriented technique that allows you to reuse code and functionalities from existing classes. It establishes a parent-child relationship between classes, where a child class inherits properties and methods from its parent class.
## 5. **What is Encapsulation (or Data Hiding) in Java?**
    
    - Encapsulation is an OOP concept that involves hiding data (attributes) within a class so that it can be changed later without impacting other parts of the program. It is achieved using access modifiers like `private`.
## 6. **What is Polymorphism in Java?**
    
    - Polymorphism allows objects of different classes to be treated as objects of a common superclass. It enables dynamic method dispatch and method overriding.
## 7. **What is the difference between `==` and `.equals()` in Java?**
    
    - The `==` operator compares object references (memory addresses), while `.equals()` compares the actual content of objects (based on the overridden `equals()` method).
## 8. **What is the `final` keyword in Java?**
    
    - The `final` keyword can be applied to classes, methods, and variables. A `final` class cannot be subclassed, a `final` method cannot be overridden, and a `final` variable cannot be reassigned after initialization.
## 9. **What are abstract classes and interfaces?**
    
    - Abstract classes are partially implemented classes that cannot be instantiated directly. Interfaces define a contract for classes to implement specific methods. A class can implement multiple interfaces but inherit from only one abstract class.
## 10. **What is the difference between `ArrayList` and `LinkedList`?**
    
    - `ArrayList` is backed by an array, provides fast random access, but slower insertion/deletion. `LinkedList` uses a doubly linked list, provides fast insertion/deletion, but slower random access.

 ## What is encapsulation, and how to achieve that?
 
Every object consists of data members and member functions. The value of data members denotes the state of an object, and the member functions define the behavior of an object. Behavior is normally controlled by the value of data members. So, if we want an object to behave as designed by the creator, then the data members should not be exposed directly to the end users. This hiding of internal structure or data members from external world is termed as encapsulation.

 
We can achieve encapsulation by declaring the data members as private and member functions as public. By this way, the users of the system will able to access only the member function and not the data members.

## To achieve complete encapsulation, can we declare the class with a private access specifier?

 

One may find luxury to declare a class as private to achieve the encapsulation automatically. But, it is not permitted. The reason is obvious. If you declare a class with private, then it is not accessible to the outside world, not even to JVM, which accesses the class to load and execute the main method.
## What is the difference between composition and aggregation?

Both these are types of association in Java. Both these concepts use the other object as a part of it. But, there is one difference. Once we use composition, if we destroy the main object, the object that is associated with it also gets destroyed or not meaningful. In case of aggregation, even if we delete the main object, the other object still exists in the system.

One of the simple examples of composition is about a car and engine. A car is composed of an engine, but if we destroy the car, the engine on its own cannot perform. On the other hand, if we take an example of a school and students, where the school will have students associated with it. In this case, even if we destroy the school object, students can still exist and perform their role in the system.