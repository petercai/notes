# Difference Between Map.ofEntries() and Map.of() | Baeldung
  

We’re finally running a Black Friday launch. All Courses are **30% off** until **the end of this week:**

### **[\>> GET ACCESS NOW](https://www.baeldung.com/all-courses)**

**1\. Introduction**[](#introduction)
-------------------------------------

Java 8 introduced the _Map.of()_ method, which makes it easier to create immutable maps. And Java 9 gained the _Map.ofEntries()_ method, which has slightly different functionality.

In this tutorial, we'll take a closer look at these two static factory methods for immutable maps and explain which one is suitable for which purpose.

**2\. _Map.of()_**[](#mapof)
----------------------------

The _Map.of()_ method **takes a specified number of key-value pairs as arguments** and returns an immutable map containing each key-value pair. The order of the pairs in the arguments corresponds to the order in which they are added to the map. If we try to add a key-value pair with a duplicate key, it'll throw an _IllegalArgumentException_. If we attempt to add a _null_ key or value, it will throw a _NullPointerException_.

Implemented as overloaded _static_ factory methods, the first one lets us create an empty map:

```null
static <K, V> Map<K, V> of() {
    return (Map<K,V>) ImmutableCollections.EMPTY_MAP;
}
```

Let's see the usage:

[![](https://a.pub.network/core/imgs/fslogo-green.svg)
](https://freestar.com/?utm_campaign=branding&utm_medium=banner&utm_source=baeldung.com&utm_content=baeldung_leaderboard_mid_1)

```null
Map<Long, String> map = Map.of();
```

There's also a method defined in the interface of _Map<K, V>_ that takes a single key and value:

```null
static <K, V> Map<K, V> of(K k1, V v1) {
    return new ImmutableCollections.Map1<>(k1, v1);
}
```

Let's call it:

```null
Map<Long, String> map = Map.of(1L, "value1");

```

Those factory **methods are overloaded nine more times, accepting up to ten keys and ten values**, as we can find in [OpenJDK 17](https://openjdk.org/projects/jdk/17/):

```null
static <K, V> Map<K, V> of(K k1, V v1, K k2, V v2, K k3, V v3, K k4, V v4, K k5, V v5, K k6, V v6, K k7, V v7, K k8, V v8, K k9, V v9, K k10, V v10) {
    return new ImmutableCollections.MapN<>(k1, v1, k2, v2, k3, v3, k4, v4, k5, v5, k6, v6, k7, v7, k8, v8, k9, v9, k10, v10);
}
```

Even though these methods are quite useful, it would be a mess to create a lot more of them. Also, we can't use the _Map.of()_ method to create a map from existing keys and values because this method only accepts undefined key-value pairs as arguments. This is where the _Map.ofEntries()_ method comes in.

**3\. _Map.ofEntries()_**[](#mapofentries)
------------------------------------------

The _Map.ofEntries()_ method **takes an unspecified number of _Map.Entry<K, V>_ objects as arguments** and also returns an immutable map. Again, the order of the pairs in the arguments is the same as the order in which they are added to the map. If we try to add a key-value pair with a duplicate key, it throws an _IllegalArgumentException_.

Let's look at the static factory method implementation according to [OpenJDK 17](https://openjdk.org/projects/jdk/17/):

```null
static <K, V> Map<K, V> ofEntries(Entry<? extends K, ? extends V>... entries) {
    if (entries.length == 0) { 
        var map = (Map<K,V>) ImmutableCollections.EMPTY_MAP;
        return map;
    } else if (entries.length == 1) {
        
        return new ImmutableCollections.Map1<>(entries[0].getKey(), entries[0].getValue());
    } else {
        Object[] kva = new Object[entries.length << 1];
        int a = 0;
        for (Entry<? extends K, ? extends V> entry : entries) {
            
            kva[a++] = entry.getKey();
            kva[a++] = entry.getValue();
        }
        return new ImmutableCollections.MapN<>(kva);
     }
}
```

The [variable arguments](https://www.baeldung.com/java-varargs) implementation allows us to pass a variable amount of entries.

For example, we can create an empty map:

```null
Map<Long, String> map = Map.ofEntries();
```

Or we can create and populate a map:

```null
Map<Long, String> longUserMap = Map.ofEntries(Map.entry(1L, "User A"), Map.entry(2L, "User B"));
```

A big advantage of the _Map.ofEntries()_ method is that we can also use it**to create a map from existing keys and values**. This isn't possible with the _Map.of()_ method because it only accepts undefined key-value pairs as arguments.

**4\. Conclusion**[](#conclusion)
---------------------------------

**The _Map.of()_ method is only suitable for maps with a maximum of 10 elements**. That's because it's implemented as 11 distinct overloaded methods that take from zero to ten name-value pairs as arguments. **The _Map.ofEntries()_ method, on the other hand, can be used for maps of any size** because it takes advantage of the feature [variable arguments](https://www.baeldung.com/java-varargs).

The complete examples are available [over on GitHub](https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-collections-maps-5).

We’re finally running a Black Friday launch. All Courses are **30% off** until **the end of this week:**

### **[\>> GET ACCESS NOW](https://www.baeldung.com/all-courses)**