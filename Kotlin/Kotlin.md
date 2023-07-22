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

you can use one or more **initializer blocks**. Initializer blocks are executed when the object is initialized, immediately after the constructor is called, and they’re prefixed with the init keyword.
![[Pasted image 20230722132247.png]]

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
### override properties
![[_assets/Kotlin-2023722-1.png]]
### override method: