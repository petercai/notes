# Lambda Expressions and Functional Interfaces: Tips and Best Practices 
**1\. Overview**[](#overview)
-----------------------------

Now that Java 8 has reached wide usage, patterns and best practices have begun to emerge for some of its headlining features. In this tutorial, we'll take a closer look at functional interfaces and lambda expressions.

**2\. Prefer Standard Functional Interfaces**[](#standard)
----------------------------------------------------------

Functional interfaces, which are gathered in the **[java.util.function](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/package-summary.html)** package, satisfy most developers' needs in providing target types for lambda expressions and method references. Each of these interfaces is general and abstract, making them easy to adapt to almost any lambda expression. Developers should explore this package before creating new functional interfaces.

Let's consider an interface_ Foo_:

```null
@FunctionalInterface
public interface Foo {
    String method(String string);
}
```

In addition, we have a method _add() _in some class _UseFoo_, which takes this interface as a parameter:

```null
public String add(String string, Foo foo) {
    return foo.method(string);
}
```

To execute it, we would write:

[![](_assets/fslogo-green.svg)
](https://freestar.com/?utm_campaign=branding&utm_medium=banner&utm_source=baeldung.com&utm_content=baeldung_leaderboard_mid_1)

AD

```null
Foo foo = parameter -> parameter + " from lambda";
String result = useFoo.add("Message ", foo);
```

If we look closer, we'll see that _Foo_ is nothing more than a function that accepts one argument and produces a result. Java 8 already provides such an interface in _[Function<T,R>](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/Function.html)_ from the [java.util.function](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/function/package-summary.html) package.

Now we can remove interface _Foo_ completely and change our code to:

```null
public String add(String string, Function<String, String> fn) {
    return fn.apply(string);
}
```

To execute this, we can write:

```null
Function<String, String> fn = 
  parameter -> parameter + " from lambda";
String result = useFoo.add("Message ", fn);
```

**3\. Use the _@FunctionalInterface_ Annotation**[](#functionalInterface)
-------------------------------------------------------------------------

Now let's annotate our functional interfaces with _[@FunctionalInterface](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/FunctionalInterface.html)._ At first, this annotation seems to be useless. Even without it, our interface will be treated as functional as long as it has just one abstract method.

However, let's imagine a big project with several interfaces; it's hard to control everything manually. An interface, which was designed to be functional, could accidentally be changed by adding another abstract method/methods, rendering it unusable as a functional interface.

[![](_assets/fslogo-green.svg)
](https://freestar.com/?utm_campaign=branding&utm_medium=banner&utm_source=baeldung.com&utm_content=baeldung_leaderboard_mid_2)

AD

By using the _@FunctionalInterface_ annotation, the compiler will trigger an error in response to any attempt to break the predefined structure of a functional interface. It is also a very handy tool to make our application architecture easier to understand for other developers.

So we can use this:

```null
@FunctionalInterface
public interface Foo {
    String method();
}
```

Instead of just:

```null
public interface Foo {
    String method();
}
```

**4\. Don't Overuse Default Methods in Functional Interfaces**[](#methods)
--------------------------------------------------------------------------

We can easily add default methods to the functional interface. This is acceptable to the functional interface contract as long as there is only one abstract method declaration:

```null
@FunctionalInterface
public interface Foo {
    String method(String string);
    default void defaultMethod() {}
}
```

Functional interfaces can be extended by other functional interfaces if their abstract methods have the same signature:

```null
@FunctionalInterface
public interface FooExtended extends Baz, Bar {}
	
@FunctionalInterface
public interface Baz {	
    String method(String string);	
    default String defaultBaz() {}		
}
	
@FunctionalInterface
public interface Bar {	
    String method(String string);	
    default String defaultBar() {}	
}
```

Just as with regular interfaces, **extending different functional interfaces with the same default method can be problematic**.

For example, let's add the _defaultCommon()_ method to the _Bar_ and _Baz_ interfaces:

```null
@FunctionalInterface
public interface Baz {
    String method(String string);
    default String defaultBaz() {}
    default String defaultCommon(){}
}

@FunctionalInterface
public interface Bar {
    String method(String string);
    default String defaultBar() {}
    default String defaultCommon() {}
}
```

In this case, we'll get a compile-time error:

```null
interface FooExtended inherits unrelated defaults for defaultCommon() from types Baz and Bar...
```

To fix this, the _defaultCommon()_ method should be overridden in the _FooExtended_ interface. We can provide a custom implementation of this method; however, **we can also reuse the implementation from the parent interface**:

```null
@FunctionalInterface
public interface FooExtended extends Baz, Bar {
    @Override
    default String defaultCommon() {
        return Bar.super.defaultCommon();
    }
}
```

It's important to note that we have to be careful. **Adding too many default methods to the interface is not a very good architectural decision.** This should be considered a compromise, only to be used when required for upgrading existing interfaces without breaking backward compatibility.

**5\. Instantiate Functional Interfaces With Lambda Expressions**[](#instantiate)
---------------------------------------------------------------------------------

The compiler will allow us to use an inner class to instantiate a functional interface; however, this can lead to very verbose code. We should prefer to use lambda expressions:

```null
Foo foo = parameter -> parameter + " from Foo";
```

Over an inner class:

```null
Foo fooByIC = new Foo() {
    @Override
    public String method(String string) {
        return string + " from Foo";
    }
};

```

**The lambda expression approach can be used for any suitable interface from old libraries.** It is usable for interfaces like _Runnable_, _Comparator_, and so on; **h****owever, this** **doesn't mean that we should review our whole older code base and change everything.**

[![](_assets/fslogo-green.svg)
](https://freestar.com/?utm_campaign=branding&utm_medium=banner&utm_source=baeldung.com&utm_content=baeldung_incontent_1)

AD

**6\. Avoid Overloading Methods With Functional Interfaces as Parameters**[](#overloading)
------------------------------------------------------------------------------------------

We should use methods with different names to avoid collisions:

```null
public interface Processor {
    String process(Callable<String> c) throws Exception;
    String process(Supplier<String> s);
}

public class ProcessorImpl implements Processor {
    @Override
    public String process(Callable<String> c) throws Exception {
        
    }

    @Override
    public String process(Supplier<String> s) {
        
    }
}
```

At first glance, this seems reasonable, but any attempt to execute either of the _ProcessorImpl_‘s methods:

```null
String result = processor.process(() -> "abc");
```

Ends with an error with the following message:

```null
reference to process is ambiguous
both method process(java.util.concurrent.Callable<java.lang.String>) 
in com.baeldung.java8.lambda.tips.ProcessorImpl 
and method process(java.util.function.Supplier<java.lang.String>) 
in com.baeldung.java8.lambda.tips.ProcessorImpl match
```

To solve this problem, we have two options. **The first option is to use methods with different names:**

```null
String processWithCallable(Callable<String> c) throws Exception;

String processWithSupplier(Supplier<String> s);
```

**The second option is to perform casting manually,** which is not preferred:

```null
String result = processor.process((Supplier<String>) () -> "abc");
```

**7\. Don’t Treat Lambda Expressions as Inner Classes**[](#inner)
-----------------------------------------------------------------

Despite our previous example, where we essentially substituted inner class by a lambda expression, the two concepts are different in an important way: scope.

When we use an inner class, it creates a new scope. We can hide local variables from the enclosing scope by instantiating new local variables with the same names. We can also use the keyword _**this**_ inside our inner class as a reference to its instance.

Lambda expressions, however, work with enclosing scope. We can’t hide variables from the enclosing scope inside the lambda’s body. In this case, the keyword _**this**_ is a reference to an enclosing instance.

[![](_assets/fslogo-green.svg)
](https://freestar.com/?utm_campaign=branding&utm_medium=banner&utm_source=baeldung.com&utm_content=baeldung_incontent_2)

AD

For example, in the class _UseFoo,_ we have an instance variable _value:_

```null
private String value = "Enclosing scope value";
```

Then in some method of this class, place the following code and execute this method:

```null
public String scopeExperiment() {
    Foo fooIC = new Foo() {
        String value = "Inner class value";

        @Override
        public String method(String string) {
            return this.value;
        }
    };
    String resultIC = fooIC.method("");

    Foo fooLambda = parameter -> {
        String value = "Lambda value";
        return this.value;
    };
    String resultLambda = fooLambda.method("");

    return "Results: resultIC = " + resultIC + 
      ", resultLambda = " + resultLambda;
}
```

If we execute the _scopeExperiment()_ method, we'll get the following result: _Results: resultIC = Inner class value, resultLambda = Enclosing scope value_

As we can see, by calling _this.value_ in IC, we can access a local variable from its instance. In the case of the lambda, _this.value_ call gives us access to the variable _value,_ which is defined in the _UseFoo_ class, but not to the variable _value_ defined inside the lambda's body.

**8\. Keep Lambda Expressions Short and Self-explanatory**[](#short)
--------------------------------------------------------------------

If possible, we should use one line constructions instead of a large block of code. Remember, **lambdas should be an** **expression, not a narrative.** Despite its concise syntax, **lambdas should specifically express the functionality they provide.**

This is mainly stylistic advice, as performance will not change drastically. In general, however, it is much easier to understand and to work with such code.

This can be achieved in many ways; let's have a closer look.

### **8.1. Avoid Blocks of Code in Lambda's Body**[](#1-avoid-blocks-of-code-in-lambdas-body)

In an ideal situation, lambdas should be written in one line of code. With this approach, the lambda is a self-explanatory construction, which declares what action should be executed with what data (in the case of lambdas with parameters).

[![](_assets/fslogo-green.svg)
](https://freestar.com/?utm_campaign=branding&utm_medium=banner&utm_source=baeldung.com&utm_content=baeldung_incontent_3)

If we have a large block of code, the lambda's functionality is not immediately clear.

With this in mind, do the following:

```null
Foo foo = parameter -> buildString(parameter);
```

```null
private String buildString(String parameter) {
    String result = "Something " + parameter;
    
    return result;
}
```

Instead of:

```null
Foo foo = parameter -> { String result = "Something " + parameter; 
    
    return result; 
};
```

**It is important to note, we shouldn't use this “one-line lambda” rule as dogma**. If we have two or three lines in lambda's definition, it may not be valuable to extract that code into another method.

### **8.2. Avoid Specifying Parameter Types**[](#2-avoid-specifying-parameter-types)

A compiler, in most cases, is able to resolve the type of lambda parameters with the help of **[type inference](https://docs.oracle.com/javase/tutorial/java/generics/genTypeInference.html)**. Consequently, adding a type to the parameters is optional and can be omitted.

We can do this:

```null
(a, b) -> a.toLowerCase() + b.toLowerCase();
```

Instead of this:

```null
(String a, String b) -> a.toLowerCase() + b.toLowerCase();
```

### **8.3. Avoid Parentheses Around a Single Parameter**[](#3-avoid-parentheses-around-a-single-parameter)

Lambda syntax only requires parentheses around more than one parameter, or when there is no parameter at all. That's why it's safe to make our code a little bit shorter, and to exclude parentheses when there is only one parameter.

[![](_assets/fslogo-green.svg)
](https://freestar.com/?utm_campaign=branding&utm_medium=banner&utm_source=baeldung.com&utm_content=baeldung_incontent_4)

AD

So we can do this:

```null
a -> a.toLowerCase();
```

Instead of this:

```null
(a) -> a.toLowerCase();
```

### **8.4. Avoid Return Statement and Braces**[](#4-avoid-return-statement-and-braces)

**Braces** and _**return**_ statements are optional in one-line lambda bodies. This means that they can be omitted for clarity and conciseness.

We can do this:

```null
a -> a.toLowerCase();
```

Instead of this:

```null
a -> {return a.toLowerCase()};
```

### **8.5. Use Method References**[](#5-use-method-references)

Very often, even in our previous examples, lambda expressions just call methods which are already implemented elsewhere. In this situation, it is very useful to use another Java 8 feature, **[method references](https://docs.oracle.com/javase/tutorial/java/javaOO/methodreferences.html)**.

The lambda expression would be:

```null
a -> a.toLowerCase();
```

We could substitute it with:

[![](_assets/fslogo-green.svg)
](https://freestar.com/?utm_campaign=branding&utm_medium=banner&utm_source=baeldung.com&utm_content=baeldung_incontent_5)

AD

```null
String::toLowerCase;
```

This is not always shorter, but it makes the code more readable.

**9\. Use “Effectively Final” Variables**[](#variables)
-------------------------------------------------------

Accessing a non-final variable inside lambda expressions will cause a compile-time error, **b****ut that doesn’t mean that we should mark every target variable as _final._**

According to the “**[effectively final](https://docs.oracle.com/javase/tutorial/java/javaOO/localclasses.html)**” concept, a compiler treats every variable as _final_ as long as it is assigned only once.

It's safe to use such variables inside lambdas because the compiler will control their state and trigger a compile-time error immediately after any attempt to change them.

For example, the following code will not compile:

```null
public void method() {
    String localVariable = "Local";
    Foo foo = parameter -> {
        String localVariable = parameter;
        return localVariable;
    };
}
```

The compiler will inform us that:

```null
Variable 'localVariable' is already defined in the scope.
```

This approach should simplify the process of making lambda execution thread-safe.

**10\. Protect Object Variables From Mutation**[](#mutation)
------------------------------------------------------------

One of the main purposes of lambdas is use in parallel computing, which means that they're really helpful when it comes to thread-safety.

[![](_assets/fslogo-green.svg)
](https://freestar.com/?utm_campaign=branding&utm_medium=banner&utm_source=baeldung.com&utm_content=baeldung_incontent_6)

AD

The “effectively final” paradigm helps a lot here, but not in every case. Lambdas can't change a value of an object from enclosing scope. But in the case of mutable object variables, a state could be changed inside lambda expressions.

Consider the following code:

```null
int[] total = new int[1];
Runnable r = () -> total[0]++;
r.run();
```

This code is legal, as _total_ variable remains “effectively final,” but will the object it references have the same state after execution of the lambda? No!

Keep this example as a reminder to avoid code that can cause unexpected mutations.

**11\. Conclusion**[](#conclusion)
----------------------------------

In this article, we explored some of the best practices and pitfalls in Java 8's lambda expressions and functional interfaces. Despite the utility and power of these new features, they are just tools. Every developer should pay attention while using them.

The complete **source code** for the example is available in [this GitHub project](https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-lambdas). This is a Maven and Eclipse project, so it can be imported and used as is.