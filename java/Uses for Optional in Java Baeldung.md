# Uses for Optional in Java | Baeldung
1\. Introduction[](#introduction)
---------------------------------

In this tutorial, we'll look at the purpose of the _Optional_ class in Java and some of the advantages of using it when building our applications.

2\. The Purpose of _Optional<T>_ in Java[](#the-purpose-of-optionallttgt-in-java)
---------------------------------------------------------------------------------

_Optional_ is a class that represents either presence or absence of something. Technically, an _Optional_ is a wrapper class for a generic type _T_, where the _Optional_ instance is empty if _T_ is _null_. Otherwise, it's full.

[As per the Java 11 documentation](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Optional.html), the**purpose of _Optional_ is to provide a return type that can represent the absence of value in scenarios where returning _null_ might cause unexpected errors,** like the infamous _NullPointerException_.

### 2.1. Useful Methods[](#1-useful-methods)

The _Optional_ class provides useful methods to help us work with that API. The important ones for this article are the _of(), orElse(), _and_ empty() _methods:

*   _of(T value)_ returns an instance of an _Optional_ with a value inside
*   _orElse(T other) _returns the value inside an _Optional_, otherwise returns _other_
*   Finally, _empty()_ returns an empty instance of _Optional_

We'll see these methods in action to build both the code that returns the _Optional_ and the code that uses it.

[![](_assets/fslogo-green.svg)
](https://freestar.com/?utm_campaign=branding&utm_medium=banner&utm_source=baeldung.com&utm_content=baeldung_leaderboard_mid_1)

AD

3\. Advantages of _Optional_[](#advantages-of-optional)
-------------------------------------------------------

We've looked at the purpose of _Optional_ and some of its methods. But, how can we benefit from that class in our programs? In this section, we'll see a few ways to use it that helps us to create clear and robust APIs.

### 3.1. _Optional_ vs. _null_[](#1-optional-vs-null)

Before the creation of the _Optional_ class, the way we had to represent the absence of value was by using _null_. The language doesn't obligate us to treat the _null_ case properly. In other words, **_null_ checks are sometimes necessary, but not mandatory**. Thus, creating methods that return _null_ proved to be a way to produce unexpected runtime errors like _NullPointerException_s.

On the other hand, an instance of **_Optional_ should always be treated properly at compile time to get the value inside of it**. This obligation to handle an _Optional_ at compile time results in fewer unexpected _NullPointerException_s.

Let's try an example simulating a database of _User_s:

```null
public class User {

    public User(String id, String name) {
        this.id = id;
        this.name = name;
    }

    private String id;

    private String name;

    public String getName() {
        return name;
    }

    public String getId() {
        return id;
    }
}
```

Let's also define the repository class that returns a _User_ if found. Otherwise, it returns _null_:

[![](_assets/fslogo-green.svg)
](https://freestar.com/?utm_campaign=branding&utm_medium=banner&utm_source=baeldung.com&utm_content=baeldung_leaderboard_mid_2)

```null
public class UserRepositoryWithNull {

    private final List<User> dbUsers = Arrays.asList(new User("1", "John"), new User("2", "Maria"), new User("3", "Daniel"));

    public User findById(String id) {

        for (User u : dbUsers) {
            if (u.getId().equals(id)) {
                return u;
            }
        }

        return null;
    }
}
```

Now, we'll write a unit test that shows how the code breaks with a _NullPointerException_ if we don't address _null_ using _null_ checks:

```null
@Test
public void givenNonExistentUserId_whenSearchForUser_andNoNullCheck_thenThrowException() {

    UserRepositoryWithNull userRepositoryWithNull = new UserRepositoryWithNull();
    String nonExistentUserId = "4";

    assertThrows(NullPointerException.class, () -> {
        System.out.println("User name: " + userRepositoryWithNull.findById(nonExistentUserId)
          .getName());
    });
}
```

**Java allows us to use the _getName()_ method from the object returned by _findById()_ without a _null_ check. In that case, we can only discover the problem at runtime.**

To avoid that, we can create another repository where if we find a _User,_ then we return a full _Optional_. Otherwise, we return an empty one. Let's see this in practice:

```null
public class UserRepositoryWithOptional {

    private final List<User> dbUsers = Arrays.asList(new User("1", "John"), new User("2", "Maria"), new User("3", "Daniel"));

    public Optional<User> findById(String id) {

        for (User u : dbUsers) {
            if (u.getId().equals(id)) {
                return Optional.of(u);
            }
        }

        return Optional.empty();
    }
}
```

Now, when we rewrite our unit test, we see how we must treat the _Optional_ first to get its value when we don't find any _User_:

```null
@Test
public void givenNonExistentUserId_whenSearchForUser_thenOptionalShouldBeTreatedProperly() {

    UserRepositoryWithOptional userRepositoryWithOptional = new UserRepositoryWithOptional();
    String nonExistentUserId = "4";

    String userName = userRepositoryWithOptional.findById(nonExistentUserId)
      .orElse(new User("0", "admin"))
      .getName();

    assertEquals("admin", userName);
}
```

In the case above, we didn't find any _User_, so we can return a default user using the _orElse()_ method. To get its value, it is mandatory to treat the _Optional_ properly at compile time. With that, we can mitigate unexpected errors at runtime.

[![](_assets/fslogo-green.svg)
](https://freestar.com/?utm_campaign=branding&utm_medium=banner&utm_source=baeldung.com&utm_content=baeldung_leaderboard_mid_3)

Two other options we can use instead of providing a default with the _orElse()_ method are to [throw an exception using _orElseThrow()_](https://www.baeldung.com/java-optional-throw-exception) or [use _orElseGet()_ to call a _Supplier_ function](https://www.baeldung.com/java-optional-or-else-vs-or-else-get).

### 3.2. Design Clear Intention APIs[](#2-design-clear-intention-apis)

As we discussed previously, _null_ was widely used to represent nothing. However, the meaning of _null_ is only clear to the person who created the API. Other developers that glance at that API may find different meanings for _null_.

Probably the name “Optional” is the main reason that _Optional_ is a useful tool when building our APIs. **_Optional_ in the return of a method provides a clear intention of what we should expect from that method: it returns something or nothing**. No documentation is needed to explain the return type of that method. The code explains itself.

Using the repository that returns _null_, we may find in the worst way that _null_ represents that a _User_ was not found in the database. Or maybe it could represent other things, like an error in connecting to the database, or the object not being initialized yet. It's hard to know.

On the other hand, using the repository that returns an _Optional_ instance makes it clear by looking just at the method signature that we may or may not find a user in the database.

**An important practice for designing clear APIs is to never return a _null_ _Optional_**. Methods should always return a valid instance of _Optional_, using the _static_ methods.

### 3.3. Declarative Programming[](#3-declarative-programming)

Another good reason to use the _Optional_ class is the ability to use a chain of fluent methods. It provides a “pseudo-stream” similar to the _stream()_ from collections, but with only one value. That means we can call methods like _map()_, and _filter() _on the value in it. That helps to create more [declarative programs, instead of imperative ones](https://www.baeldung.com/cs/imperative-vs-declarative-programming).

Let's say the requirement is to change the case of _User_‘s _name_ to uppercase if the name starts with the letter ‘M'.

[![](_assets/fslogo-green.svg)
](https://freestar.com/?utm_campaign=branding&utm_medium=banner&utm_source=baeldung.com&utm_content=baeldung_incontent_1)

First, let's look at the imperative way, using the repository that doesn't return an _Optional_:

```null
@Test
public void givenExistentUserId_whenFoundUserWithNameStartingWithMInRepositoryUsingNull_thenNameShouldBeUpperCased() {

    UserRepositoryWithNull userRepositoryWithNull = new UserRepositoryWithNull();

    User user = userRepositoryWithNull.findById("2");
    String upperCasedName = "";

    if (user != null) {
        if (user.getName().startsWith("M")) {
            upperCasedName = user.getName().toUpperCase();
        }
    }

    assertEquals("MARIA", upperCasedName);
}
```

Now, let's take a look at the declarative way, using the _Optional_ version of the repository:

```null
@Test
public void givenExistentUserId_whenFoundUserWithNameStartingWithMInRepositoryUsingOptional_thenNameShouldBeUpperCased() {

    UserRepositoryWithOptional userRepositoryWithOptional = new UserRepositoryWithOptional();

    String upperCasedName = userRepositoryWithOptional.findById("2")
      .filter(u -> u.getName().startsWith("M"))
      .map(u -> u.getName().toUpperCase())
      .orElse("");

    assertEquals("MARIA", upperCasedName);
}
```

The imperative way needs two nested _if_ statements to check if the object is not _null_ and filter the user name. If the _User_ is not found, the uppercase string remains empty.

In the declarative way, we use lambda expressions to filter the name and map the uppercase function to the _User_ found. If the _User_ is not found, we return an empty string using _orElse()_.

It's still a matter of preference which one we use. Both of them achieve the same result. The imperative way needs a bit more digging to understand what the code means. If we add more logic in the first or second _if_ statement, for example, it might create some confusion about what the intention of that code is. In this type of scenario, **declarative programming makes it clear what is the intent of the code**: to return the uppercased name, otherwise, return an empty string.

4\. Conclusion[](#conclusion)
-----------------------------

In this article, we looked at the purpose of the _Optional_ class and how to use it efficiently to design clear and robust APIs.

As always, the source code for the example is available [over on GitHub](https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-optional).

We’re finally running a Black Friday launch. All Courses are **30% off** until **the end of this week:**

### **[\>> GET ACCESS NOW](https://www.baeldung.com/all-courses)**