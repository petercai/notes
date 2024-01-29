

## SRP (Single responsibility Principle)
A module should have one, and only one, reason to change. -> A module should be responsible to one, and only one, actor.

## OCP (Open-close principle)
A software artifact should be open for extension but closed for modification, which suggests that you should design your software components in a way that allows you to add new functionality or behavior without modifying the existing code.


The OCP goal is accomplished by partitioning the system into components, and arranging those components into a dependency hierarchy that protects higher-level components from changes in lower-level components.
## LSP (Liskov Substitution Principle)
The principle is aimed at maintaining the integrity of a software system's design when inheritance is used: Objects of a superclass should be replaceable with objects of a subclass without affecting the correctness of the program.

To adhere to the LSP, a subclass must meet certain criteria:

1. **Behavior Preservation**: The subclass should maintain the behavior and invariants (preconditions and postconditions) of its superclass. This means that the methods of the subclass should work as expected without causing unexpected side effects or violating the contract established by the superclass.
    
2. **Method Contract Enrichment**: The subclass can strengthen the preconditions but should not weaken them. It can also weaken the postconditions but should not strengthen them. This allows the subclass to provide additional functionality while still adhering to the contract defined by the superclass.
    
3. **Consistent Return Types**: The return type of a method in the subclass must be compatible with the return type of the corresponding method in the superclass. This is important to ensure that client code using the superclass can seamlessly work with instances of the subclass.

## ISP (Interface Segregation Principle)
It emphasizes that a class should not be forced to implement interfaces it doesn't use, and that clients (classes that use the interface) should not be required to depend on methods they do not need.
The ISP encourages the creation of small, focused interfaces that are tailored to the specific needs of the classes that use them. This helps prevent the problem of "fat" interfaces that contain many methods, some of which might not be relevant to all implementing classes.

![[Clean Architecture-2023809-1.png]]
													||
													V
![[Clean Architecture-2023809-2.png]]
Segregated operations

***Depending on something that carries baggage that you don’t need can cause you troubles that you didn’t expect.***

The key ideas of the Interface Segregation Principle:

1. **Client-Specific Interfaces**: Each class that implements an interface should only be required to implement the methods that are directly relevant to its functionality. This ensures that classes are not burdened with unnecessary methods they won't use.
    
2. **Avoiding Interface Pollution**: When an interface becomes too large and includes methods that are not applicable to all classes, it can lead to confusion and code bloat. By keeping interfaces small and focused, you avoid the problem of including methods that have no relevance to certain classes.
    
3. **Dependency Minimization**: The ISP helps minimize the dependencies between classes. When classes depend on interfaces with only the methods they need, the risk of coupling unnecessary dependencies is reduced, leading to more modular and maintainable code.

## DIP (Dependency Inversion Principle)
## Common Reuse Principle