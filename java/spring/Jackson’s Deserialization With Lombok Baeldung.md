# Jackson’s Deserialization With Lombok | Baeldung
**1\. Overview**[](#overview)
-----------------------------

More often than not, when working with [Project Lombok](https://projectlombok.org/), we'll want to combine our data-related classes with a JSON framework like [Jackson](https://www.baeldung.com/jackson). This is particularly true given that JSON is widespread in most modern APIs and data services.

In this quick tutorial, **we'll take a look at how we can configure our Lombok [builder classes](https://www.baeldung.com/lombok-builder) to work seamlessly with Jackson**.

**2\. Dependencies**[](#dependencies)
-------------------------------------

All we'll need to get started with is the _[org.projectlombok](https://search.maven.org/classic/#search%7Cga%7C1%7Cg%3A%22org.projectlombok%22)_ added to our _pom.xml_:

```null
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.24</version>
</dependency>
```

And of course, we'll need the [_jackson-databind_](https://search.maven.org/search?q=g:com.fasterxml.jackson.core) dependency as well:

```null
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.14.1</version>
</dependency>
```

**3\. A Simple Fruit Domain**[](#a-simple-fruit-domain)
-------------------------------------------------------

Let's go ahead and define a Lombok-enabled class with an id and a name to represent a fruit:

```null
@Data
@Builder
@Jacksonized
public class Fruit {
    private String name;
    private int id;
}
```

Let's walk through the key annotations of our POJO:

*   First things first, we start by adding the _@Data_ annotation to our class – this generates all the boilerplate normally associated with a simple POJO, such as getters and setters
*   Then, we add the [_@Builder_](http://baeldung.com/lombok-builder) annotation –  a helpful mechanism for object creation using the [Builder pattern](https://www.baeldung.com/creational-design-patterns#builder)
*   Finally and most importantly, we add the _@Jacksonized_ annotation

To expand briefly, the _@Jacksonized_ annotation is an add-on annotation for _@Builder_. **Using this annotation lets us automatically configure the generated builder class to work with Jackson's deserialization**.

It is important to note that this annotation only works when there is also a _@Builder_ or a [_@SuperBuilder_](https://www.baeldung.com/lombok-builder-inheritance) annotation present.

Finally, we should mention that although _@Jacksonized_ was introduced in Lombok v1.18.14. **It is still considered an [experimental feature](https://projectlombok.org/features/experimental/).**

4\. Deserializing and Serializing[](#deserializing-and-serializing)
-------------------------------------------------------------------

With our domain model now defined, let's go ahead and write a unit test to deserialize a fruit using Jackson:

```null
@Test
public void withFruitJSON_thenDeserializeSucessfully() throws IOException {
    String json = "{\"name\":\"Apple\",\"id\":101}";
        
    Fruit fruit = newObjectMapper().readValue(json, Fruit.class);
    assertEquals(new Fruit("Apple", 101), fruit);
}
```

The simple _readValue()_ API of the _ObjectMapper_ is good enough. We can use it to deserialize a JSON fruit string into a _Fruit_ Java object.

Likewise, we can use the _writeValue()_ API to serialize a _Fruit_ object as JSON output:

```null
@Test
void withFruitObject_thenSerializeSucessfully() throws IOException {
    Fruit fruit = Fruit.builder()
      .id(101)
      .name("Apple")
      .build();

    String json = newObjectMapper().writeValueAsString(fruit);
    assertEquals("{\"name\":\"Apple\",\"id\":101}", json);
}
```

The test shows how we can build a _Fruit_ using the Lombok builder API and that the serialized Java object matches the expected JSON string.

**5\. Working With a Customized Builder**[](#working-with-a-customized-builder)
-------------------------------------------------------------------------------

Sometimes we might need to work with a customized builder implementation rather than the one Lombok generates for us. For example, **when the names of our bean's properties are different from those of the fields in the JSON string**.

Let's imagine we want to deserialize the following JSON string:

```null
{
    "id": 5,
    "name": "Bob"
}
```

But the properties on our POJO do not match:

```null
@Data
@Builder(builderClassName = "EmployeeBuilder")
@JsonDeserialize(builder = Employee.EmployeeBuilder.class)
@AllArgsConstructor
public class Employee {

    private int identity;
    private String firstName;

}
```

**In this scenario, we can use the _@JsonDeserialize_ annotation with the _@JsonPOJOBuilder_ annotation**, which we can insert on the generated builder class to override Jackson's defaults:

```null
@JsonPOJOBuilder(buildMethodName = "createEmployee", withPrefix = "construct")
public static class EmployeeBuilder {

    private int idValue;
    private String nameValue;

    public EmployeeBuilder constructId(int id) {
        idValue = id;
        return this;
    }
            
    public EmployeeBuilder constructName(String name) {
        nameValue = name;
        return this;
    }

    public Employee createEmployee() {
        return new Employee(idValue, nameValue);
    }
}
```

Then we can go ahead and write a test as before:

```null
@Test
public void withEmployeeJSON_thenDeserializeSucessfully() throws IOException {
    String json = "{\"id\":5,\"name\":\"Bob\"}";
    Employee employee = newObjectMapper().readValue(json, Employee.class);

    assertEquals(5, employee.getIdentity());
    assertEquals("Bob", employee.getFirstName());
}
```

The result shows that a new _Employee_ data object has been successfully re-created from a JSON source despite a mismatch in properties' names.

**6\. Conclusion**[](#conclusion)
---------------------------------

In this short article, we've seen two simple approaches to configure our Lombok [builder classes](https://www.baeldung.com/lombok-builder) to work seamlessly with Jackson.

Without the _@Jacksonized_ annotation_,_ we'd have to specifically customize our builder class(es). However, using _@Jacksonized_ lets us use the Lombok-generated builder class.

As always, the full source code of the article is available [over on GitHub](https://github.com/eugenp/tutorials/tree/master/lombok-modules/lombok-2).

We’re finally running a Black Friday launch. All Courses are **30% off** until **the end of this week:**

### **[\>> GET ACCESS NOW](https://www.baeldung.com/all-courses)**