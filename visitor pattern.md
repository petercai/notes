# visitor pattern
The Visitor Design Pattern is a behavioral design pattern that allows you to separate an algorithm from an object structure on which it operates. The pattern allows adding new operations to existing object structures without changing the structures themselves.

In the Visitor design pattern, you create a Visitor interface with a set of methods that corresponds to the operations you want to perform on the elements of an object structure. The elements of the object structure then accept visitors and apply the visitor's operations on themselves.

The Visitor design pattern can be useful in cases where you have a complex object structure, and you want to perform operations on it without changing its structure. This way, you can add new operations to the structure without changing its code, making the code more maintainable and flexible.