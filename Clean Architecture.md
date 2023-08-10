




## LSP (iskov Substitution Principle)
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

## Common Reuse Principle