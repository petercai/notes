Use **var** for variables whose value you want to change, and **val** for ones whose value will stay the same.

We can rewrite this using an **`if` expression** like so:
```
println(if (x > y) "x is greater than y" else "x is not greater than y")
```

The code:
```
if (x > y) "x is greater than y" else "x is not greater than y"
```
is the `if` expression. It first checks the `if`’s condition: `x > y`. If this condition is _true_, the expression returns the `String` “x is greater than y”. Otherwise (`else`) the condition is _false_, and the expression returns the `String` “x is not greater than y” instead.

## function
every variable you declare in a function must be initialized before it can be used

## class
![[Pasted image 20230722125817.png]]

**you must initialize properties before you try to use them**.
- If you’re completely certain that you can’t assign an initial value to a property when you call the class constructor, you can prefix it with lateinit. This tells the compiler that you’re aware that the property hasn’t been initialized yet, and you’ll handle it later. If you wanted to mark the temperament property for late initialization, for example, you’d use:
```
lateinit var temperament: String
```
- You can only use `lateinit` with properties defined using `var`, and you can’t use it with any of the following types: `Byte`, `Short`, `Int`, `Long`, `Double`, `Float`, `Char` or `Boolean`. This is down to how these types are treated when the code runs in the JVM. This means that properties of any of these types must be initialized when the property is defined, or in an initializer block.

- you can use one or more **initializer blocks**. Initializer blocks are executed when the object is initialized, immediately after the constructor is called, and they’re prefixed with the init keyword.
![[Pasted image 20230722132247.png]]

- when you create a `val` property, the compiler secretly adds a getter for it. And when you create a `var` property, the compiler adds both a getter and a setter.
## Inheritance
![[Kotlin_class.png]]
The Animal() after the : calls the Animal’s constructor. This ensures that any Animal initialization code—such as assigning values to properties—gets to run. Calling the superclass constructor is mandatory: if the superclass has a primary constructor, then you must call it in the subclass header or your code won’t compile. And even if you haven’t explicitly added a constructor to your superclass, remember that the compiler automatically creates an empty one for you when the code gets compiled.

If the superclass constructor includes parameters, you must pass values for these parameters when you call the constructor.
![[1-Kotlin-2023722.png]]![[2-Kotlin-2023722.png]]

### Declare the superclass and its properties and functions as open
Before a class can be used as a superclass, you have to explicitly tell the compiler that this is allowed. You do this by prefixing the name of the class—and any properties or functions you want to override—with the keyword `**open**`. This tells the compiler that you’ve designed the class to be a superclass, and that you’re happy for the properties and functions you’ve declared as `open` to be overridden.
To use a class as a superclass, it must be declared as open. Everything you want to override must also be open.
![[Kotlin-2023722-1 1.png]]

## Override
An overridden function or property stays open until it’s declared final
![[Kotlin-2023722-1 2.png]]

### override properties
![[_assets/Kotlin-2023722-1.png]]
**if you define a property in the superclass using `val`, you _must_ override it in the subclass if you want to assign a different value to it**.
If a superclass property has been defined using var, you don’t need to override it in order to assign a new value to it, as var variables can be reused for other values.

- You can override a property’s getter and setter.
- You can override a val property in the superclass with a var property in the subclass.
- You can override a property’s type with one of the superclass version’s subtypes.

### override method:
- The function parameters in the subclass must match those in the superclass.
- The function return types must be compatible.

## Polymorphism
In essence, polymorphism allows objects of different types to be treated as if they belong to a common superclass or interface.
- Polymorphism is the ability to use any subtype object in place of its supertype. As different subclasses can have different implementations of the same function, it allows each object to respond to function calls in the way that’s most appropriate for each object.
- Polymorphism means “many forms”. It allows different subclasses to have different implementations of the same function.

## Abstract

- Abstract properties and functions don’t need to be marked as open.
- abstract functions **must** have no function bodies
- Abstract properties and functions define a common protocol so that you can use polymorphism.

## Interface
An interface lets you define common behavior **OUTSIDE** a superclass hierarch.
- Interfaces are used to define a protocol for common behavior so that you can benefit from polymorphism without having to rely on a strict inheritance structure.
- Interfaces are similar to abstract classes in that they can’t be instantiated, and they can define abstract or concrete functions and properties, but there’s one key difference: a class can implement multiple interfaces, but can only inherit from a single direct superclass. So using interfaces can provide the same benefits as using abstract classes, but with more flexibility.
-  **Interface functions** can be **abstract** or **concrete**
	![[Kotlin-2023722-1 3.png]]
	or 
	![[Kotlin-2023722-2.png]]

- You can return a value for a property by defining a custom getter using code like this:
	![[Kotlin-2023722-3.png]]
	However, interface properties don’t have backing fields. So properties **cannot have a setter to update the backing field.**

- when you declare that a class implement an interface, you don’t put parentheses after the interface name. This is because the parentheses are only needed in order to call the superclass constructor, and interfaces don’t have constructors.
	![[Kotlin-2023722-1 4.png]]