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
every variable you declare in a function must be initialized before it can be used.
### parameters
Note that you **can’t assign a new value to any of a function’s parameter variables.** Behind the scenes, the **parameter variables are created as local val variables** that can’t be reused for other values.

### Functions with no return value

![[Kotlin-2023726-2.png]]

### Functions with single-expression bodies
If you have a function whose body consists of a single expression, you can
simplify the code by removing the curly braces and return statement from
the function declaration.

![[Kotlin-2023726-1 1.png]]


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
- If a subclass inherits from a superclass (or implements an interface) named `A`, you can use the code:    
	```
	super<A>.myFunction
	```
        to call the implementation of `myFunction` that’s defined in `A`.

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
	![[_assets/Kotlin-2023722-2.png]]

- You can return a value for a property by defining a custom getter using code like this:
	![[_assets/Kotlin-2023722-3.png]]
	However, interface properties don’t have backing fields. So properties **cannot have a setter to update the backing field.**

- when you declare that a class implement an interface, you don’t put parentheses after the interface name. This is because the parentheses are only needed in order to call the superclass constructor, and interfaces don’t have constructors.
	![[Kotlin-2023722-1 4.png]]
	![[Kotlin-2023722-1 5.png]]

## when, is and as
![[Kotlin-2023722-1 9.png]]

![[Kotlin-2023722-1 8.png]]

- USING **WHEN** AS AN EXPRESSION
	```
	var y = when (x) {  
	    0 -> true  
	    else -> false  
	}
	```

- The **is** operator usually performs a smart cast
	the compiler knows that the underlying object is a Wolf, so it’s safe to run any code that’s Wolf-specific. 
	![[Kotlin-2023722-1 6.png]]
	The is operator performs a smart cast whenever the compiler can guarantee that the variable can’t change between checking the object’s type and when it’s used.
	
	But there are some situations in which smart casting doesn’t happen. The is operator won’t smart cast a var property in a class, for example, because the compiler can’t guarantee that some other code won’t sneak in and update the property.
	![[Kotlin-2023722-1 10.png]]

- Use as to perform an explicit cast
	![[Kotlin-2023722-1 11.png]]

## data class
You define a data class by prefixing a normal class definition with the `**data**` keyword.
![[assets/Kotlin-2023722-2.png]]
Data classes automatically override their equals function in order to change the behavior of the == operator so that it checks for object equality based on the values of each object’s properties. 
- **Data objects are considered equal if their properties hold the same values.**
- toString returns the value of each property
	```
	val r1 = Recipe("Chicken Bhuna", false)  
	println(r1.toString())  
		 **Recipe(title=Chicken Bhuna, isVegetarian=false)**
	```
- The copy function lets you copy a data object, altering some of its properties. The original object remains intact.
	![[assets/Kotlin-2023722-4.png]]
- Data classes define componentN functions that let you destructure data objects
	- ![[Kotlin-2023722-1 12.png]]
- The === operator always lets you check whether two variables refer to the same underlying object.
- Data classes can’t be declared abstract or open, so you can’t use a data class as a superclass. 
- Data classes can implement interfaces, they can also inherit from other classes.
- If you plan on making a lot of Java calls to your Kotlin constructor or function, an alternative approach is to annotate each function or constructor that uses default parameter values with **`@JvmOverloads`**. This tells the compiler to automatically create overloaded versions that can more easily be called from Java.

	Here’s an example of how you use `@JvmOverloads` with a function:
	```
	@JvmOverloads fun myFun(str: String = ""){  
	    //Function code goes here  
	}
	```
	
	And here’s an example of how you use it with a class that has a primary constructor:
	```
	class Foo @JvmOverloads constructor(i: Int = 0){  
	    //Class code coes here
	    }
	```

## nullable and exception
You declare that a type is nullable by adding a question mark (?) to the end of the type. You can use a nullable type everywhere you can use a non-nullable type
![[Kotlin-2023722-2 2.png]]

- create an array of nullable types
	![[Kotlin-2023722-3 1.png]]

- access a nullable type’s functions and properties
	1. To access the underlying object’s functions and properties, you first have to establish that the variable’s value is not null. e.g. in if condition
	2. **?. is the safe call operator**. It lets you safely access a nullable type’s functions and properties. 
		```
		var w: Wolf? = Wolf()
		w?.eat()
		```
- You can use safe calls to assign values and assign values to safe calls:
	```
	var x = w?.hanger // x is null if w is null

	w?.hanger = 6 // does nothing if w is null
	```
 - Use let to run code if values are not null
	 The `**let**` keyword used in conjunction with the safe call operator `?.` tells the compiler that you want to perform some action when the value it’s operating on is not `null`. So the following code:
	```
	w?.let { 
	 println(it.hunger) 
	}
	```
		?.let allows you to run code for a value that’s not null.
	
	will only execute the code in its body if `w` is not `null`.

- The Elvis operator **?:** is a safe alternative to an `if` expression. It returns the value on its left if that is not null. Otherwise, it returns the value on its right.
	```
	w?.hunger ?: -1
	```
	is like saying “if w is not null and its hunger property is not null, return the value of the hunger property, otherwise return -1”. It does the same thing as the code:
	```
	if (w?.hunger != null) w.hunger else -1
	```
- The !! operator deliberately throws a NullPointerException
	![[Kotlin-2023722-5.png]]
	If w and hunger are is not null, as asserted, the value of the hunger property is assigned to x. But if w or hunger is null, a NullPointerException will get thrown

- `throw` has a return type of `Nothing
	- This is a special type that has no values, so a variable of type Nothing? can only hold a null value. The following code, for example, creates a variable named x of type Nothing? that can only be null:
	```
	var x = null
	```
	- You can also use Nothing to denote code locations that can never be reached.
		You can, say, use it as the return type of a function that never returns:
		```
		fun fail(): Nothing {  
		    throw BadException()  
		}
		```
		The compiler knows that the code stops execution after `fail()` is called.
	- In Java, it has to declared when a method throws an exception, but doesn't need in Kotlin
		Kotlin doesn’t differentiate between checked and unchecked exceptions.
- Use a safe cast (`as?`) to avoid getting a `ClassCastException`
	**as?** lets you perform a safe explicit cast. If the cast fails, it returns null.
	```
	val wolf = r as? Wolf
	```

## Collections

1. array
	![[Kotlin-2023722-6.png]]

2. List, Set and Map
	1. List
	    - Immutable list
    		```
    		val shopping: List<String>
    		shopping = listOf("Tea", Eggs", "Milk")
    		```
    		a List is immutable—you can’t update any of the references it stores.
    		Lists and other collections can hold references to any type of object: Strings, Ints, Ducks, Pizzas and so on.
		
		- Mutable list
	        ```kotlin
            val mShopping = mutableListOf("Tea", "Eggs")
            ```

	

## lambdas

A lambda’s type is a function type. A lambda’s type takes the form:

```
(parameters) -> return_type
```


### define a lambda
A lambda—or lambda expression—is a block of code that you can pass around just like an object. 
- The lambda starts and ends with curly braces {}. All lambdas are defined within curly braces, so they can’t be omitted.

- Lambdas can have **single parameters, multiple parameters, or none at all**. The parameter definition is followed by ->. -> is used to separate any parameters from the body. 

- The -> is followed by the lambda body. This is the code that you want to be executed when the lambda runs. The body can have multiple lines, and the **last evaluated expression** in the body is used as the lambda’s return value.

- If the lambda has no parameters, you can omit the ->. 
	![[_assets/Kotlin-2023726-1.png]]

- You can assign a lambda to a variable
	-  no parameter with return 
		![[Kotlin-2023726-1 2.png]]
	 - one parameter with return
		 ![[Kotlin-2023726-1 3.png]]
	 - You can replace a single parameter with ***it***
		```
		val addFive: (Int) -> Int = { x: Int -> x + 5 }
		```
		 As the lambda has a single parameter, x, and the compiler can infer that x is an Int, we can omit the x parameter from the lambda, and replace it with it
		```
		val addFive: (Int) -> Int = { it + 5 }
		```
	- Use Unit to say a lambda has no return value
		```
		val myLambda: () -> Unit = { println("Hi!") }
		```


### execute a lambda
You invoke a lambda by calling its **invoke()** function, passing in the values for
any parameters.
```
val addInts = { x: Int, y: Int -> x + y }
val result = addInts.invoke(6, 7)
```

### Higher-order function
A function that uses a lambda as a parameter or return value is known as a
higher-order function. E.g. convert a temperature from Centigrade to Fahrenheit, or convert a weight from kilograms to pounds, depending on the formula that we pass to it in the lambda argument.
declare a function
![[assets/Kotlin-2023727-1.png]]
implement the function
![[assets/Kotlin-2023727-2.png]]
run the function
![[assets/Kotlin-2023727-3.png]]
- If **the final parameter** of a function you want to call is a lambda, you can move the lambda argument outside the function call’s parentheses
	
	![[_assets/Kotlin-2023727-1.png]]

- If you have a function that has just one parameter, and that parameter is a lambda, you can omit the parentheses entirely when you call the function.
	e.g.  for convertFive() function
	```
	fun convertFive(converter: (Int) -> Double) : Double {
		val result = converter(5)
		println("5 is converted to $result")
		return result
	}
	```
	You can call it like this:
	![[Kotlin-2023727-1 1.png]]

- A function can return a lambda
	The exact lambda that’s returned by the below function depends on the value of the String that’s passed to it.
	![[Kotlin-2023727-1.png]]
	The following code, for example, invokes getConversionLambda’s return value to get the value of 2.5 kilograms in pounds, and assigns it to a variable named pounds:
	![[Kotlin-2023727-2.png]]
	And the following example above uses getConversionLambda to get a lambda that converts a temperature from Centigrade to Fahrenheit, and then passes it to the convert function.

- A *type alias* lets you provide an alternative name for an existing type, which you can then use in your code. You define a type alias using the **typealias** keyword.
	
	![[Kotlin-2023727-3.png]]
	![[Kotlin-2023727-5.png]]
	You can use **typealias** to provide an alternative name for any type, not just function types. e.g.
	```
	typealias DuckArray = Array<Duck>
	```
