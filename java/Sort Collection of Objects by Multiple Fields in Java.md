# Sort Collection of Objects by Multiple Fields in Java | Baeldung
  

We’re finally running a Black Friday launch. All Courses are **30% off** until **next Friday:**

### **[\>> GET ACCESS NOW](https://www.baeldung.com/all-courses)**

1\. Overview[](#overview)
-------------------------

When programming, we often need to sort collections of objects. The sorting logic can sometimes become difficult to implement if we want to sort the objects on multiple fields. In this tutorial, we'll discuss several different approaches to the problem, along with their pros and cons.

2\. Example _Person_ Class[](#example-person-class)
---------------------------------------------------

Let's define a _Person_ class with two fields, _name_ and _age_. We'll be comparing _Person_ objects first based on _name_ and then on _age_ throughout our examples:

```null
public class Person {
    @Nonnull private String name;
    private int age;

    
    
}
```

Here, we've added a _@Nonnull_ annotation to keep the examples simple. But in production code, we may need to handle the comparison of nullable fields.

3\. Use _Comparator.compare()_[](#use-comparatorcompare)
--------------------------------------------------------

Java provides the _[Comparator](https://www.baeldung.com/java-comparator-comparable)_ interface for comparing two objects of the same type. We can implement its _compare(T o1, T o2)_ method with customized logic to perform the desired comparison.

### 3.1. Check Different Fields One by One[](#1-check-different-fields-one-by-one)

Let's compare the fields one after another:

```null
public class CheckFieldsOneByOne implements Comparator<Person> {
    @Override
    public int compare(Person o1, Person o2) {
        int nameCompare = o1.getName().compareTo(o2.getName());
        if(nameCompare != 0) {
            return nameCompare;
        }
        return Integer.compare(o1.getAge(), o2.getAge());
    }
}
```

Here, we use the _String_ class's _compareTo()_ method and the _Integer_ class's _compare()_ method to compare the _name_ and _age_ fields one after the other.

This requires a lot of typing, and sometimes handling of many special cases. Therefore, it's hard to maintain and scale when we have more fields to compare. **Generally, it's not recommended to use this method in production code.**

### 3.2. Use Guava’s _ComparisonChain_[](#2-use-guavas-comparisonchain)

First, let's add the Google Guava library [dependency](https://search.maven.org/artifact/com.google.guava/guava-bom/31.1-jre/pom) to our _pom.xml_:

```null
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.1-jre</version>
</dependency>
```

We can simplify the logic by using the _ComparisonChain_ class from this library:

```null
public class ComparisonChainExample implements Comparator<Person> {
    @Override
    public int compare(Person o1, Person o2) {
        return ComparisonChain.start()
          .compare(o1.getName(), o2.getName())
          .compare(o1.getAge(), o2.getAge())
          .result();
    }
}
```

Here, we use the _compare(int left, int right)_ and _compare(Comparable<?> left, Comparable<?> right)_ methods in _ComparisonChain_ to compare _name_ and _age_, respectively.

**This approach hides the comparison details and only exposes what we care about — the fields we'd like to compare and the order in which they should be compared.** Also, we should note that we don't need any additional logic for _null_ handling as the library methods take care of it. Therefore, it becomes easier to maintain and scale.

### 3.3. Sorting With Apache Commons' _CompareToBuilder_[](#3-sorting-with-apache-commons-comparetobuilder)

First, let's add the [dependency](https://search.maven.org/artifact/org.apache.commons/commons-lang3/3.12.0/jar) for Apache Commons to the _pom.xml_:

```null
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```

Similar to the previous example, we can use _CompareToBuilder_ from Apache Commons to reduce the boilerplate code needed:

```null
public class CompareToBuilderExample implements Comparator<Person> {
    @Override
    public int compare(Person o1, Person o2) {
        return new CompareToBuilder()
          .append(o1.getName(), o2.getName())
          .append(o1.getAge(), o2.getAge())
          .build();
    }
}
```

This approach is very similar to Guava's _ComparisonChain_ — **it also hides the comparison details and is easily maintainable and scalable_._**

4\. Use _Comparator.comparing()_ and Lambda Expression[](#use-comparatorcomparing-and-lambda-expression)
--------------------------------------------------------------------------------------------------------

Since Java 8, there are several _static_ methods added to the _Comparator_ interface that can take lambda expressions to create a _Comparator_ object. We can use its [_comparing()_ method](https://www.baeldung.com/java-8-comparator-comparing) to construct the _Comparator_ we need:

```null
public static Comparator<Person> createPersonLambdaComparator() {
    return Comparator.comparing(Person::getName)
      .thenComparing(Person::getAge);
}
```

**This approach is much more concise and readable as it directly takes the getters of the _Person_ class**.

It also keeps the maintainability and scalability characteristics of the approaches we saw earlier. Additionally, **the getters here are lazily evaluated**, compared to the immediate evaluation in the previous approaches. As a result, its performance is better and more suitable for latency-sensitive systems that require a lot of large data comparisons.

Moreover, this approach only uses core Java classes and doesn't require any third-party libraries as dependencies. Overall, **this is the most recommended approach**.

5\. Examine the Comparison Results[](#examine-the-comparison-results)
---------------------------------------------------------------------

Let's test the four comparators we saw and inspect their behaviors. All these comparators can be invoked in the same way and should produce the same result:

```null
@Test
public void testComparePersonsFirstNameThenAge() {
    Person person1 = new Person("John", 21);
    Person person2 = new Person("Tom", 20);
    
    Person person3 = new Person("John", 22);

    List<Comparator<Person>> comparators =
      Arrays.asList(new CheckFieldsOneByOne(),
        new ComparisonChainExample(),
        new CompareToBuilderExample(),
        createPersonLambdaComparator());
    
    for(Comparator<Person> comparator : comparators) {
        Assertions.assertIterableEquals(
          Arrays.asList(person1, person2, person3)
            .stream()
            .sorted(comparator)
            .collect(Collectors.toList()),
          Arrays.asList(person1, person3, person2));
    }
}
```

Here, _person1_ has the same name (“John”) as _person3,_ but is younger (21 < 22), while _person3′_s name (“John”) is lexicographically less than _person2_‘s name (“Tom”). So, the final ordering is _person1_, _person3_, _person2_.

Also, we should **note that if we don't have the _@Nonnull_ annotation on the class variable _name_, we'd need to add extra logic to handle the null case in all the approaches except for Apache Commons' _CompareToBuilder_ (which has native null handling built in)**.

6\. Conclusion[](#conclusion)
-----------------------------

In this article, we learned different approaches for comparing on multiple fields when sorting collections of objects.

As always, the source code for the examples is available [over on GitHub](https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-4).

We’re finally running a Black Friday launch. All Courses are **30% off** until **next Friday:**

### **[\>> GET ACCESS NOW](https://www.baeldung.com/all-courses)**