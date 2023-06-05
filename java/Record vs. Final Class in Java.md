# Record vs. Final Class in Java
  

1\. Overview[](#overview)
-------------------------

In this tutorial, we'll learn the differences between a record class and a final class in Java.

2\. What Is a Record?[](#what-is-a-record)
------------------------------------------

**[Record](https://www.baeldung.com/java-record-keyword) classes are special classes that act as transparent carriers for immutable data**. They are immutable classes (all fields are _final_) and are implicitly _final_ classes, which means they can't be extended.

There are some restrictions that we need to keep in mind when writing a record class:

*   We cannot add the _extends_ clause to the declaration since every record class implicitly extends the abstract class [Record](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Record.html), and Java doesn't allow multiple inheritance
*   We cannot declare instance variables or instance initializers in a record class
*   We cannot declare [_native_](https://www.baeldung.com/java-native) methods in a record class

A record class declaration includes a name, type parameters (if the record is generic), a header containing the record's components, and a body:

```null
public record Citizen (String name, String address) {}
```

The Java compiler then automatically generates the _private_, _final_ fields, getters, a public constructor, and the _equals_, _hashCode_, and _toString_ methods.



Moreover, we can add instance methods as well as static fields and methods to the record body:

```null
public record USCitizen(String firstName, String lastName, String address) {
    static int countryCode;

    
    static {
        countryCode = 1;
    }

    public static int getCountryCode() {
        return countryCode;
    }

    public String getFullName() {
        return firstName + " " + lastName;
    }
}
```

The above record contains a static field, a static method, and an instance method.

3\. What Is a Final Class?[](#what-is-a-final-class)
----------------------------------------------------

**[Final](https://www.baeldung.com/java-final) classes cannot be extended**. They are declared with the _final_ keyword:

```null
final class Rectangle {
    private double length;
    private double width;

    
}
```

Here, we cannot create another class that extends the _Rectangle_ class since it's a final class.

4\. What Are the Differences?[](#what-are-the-differences)
----------------------------------------------------------

Records are final classes themselves. **However, record classes have more constraints compared to regular final classes**. On the other hand, records are more convenient to use in certain situations since they can be declared in a single line, and the compiler automatically generates everything else. We can use records if we need a simple data carrier that is immutable and won't inherit from other classes.



5\. Summary[](#summary)
-----------------------

In this short article, we learned the differences between records and final classes in Java.

As always, the code snippets used in this article are available [over on GitHub](https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-14).