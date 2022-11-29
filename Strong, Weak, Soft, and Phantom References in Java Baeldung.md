# Strong, Weak, Soft, and Phantom References in Java | Baeldung
  

We’re finally running a Black Friday launch. All Courses are **30% off** until **the end of this week:**

### **[\>> GET ACCESS NOW](https://www.baeldung.com/all-courses)**

1\. Introduction[](#introduction)
---------------------------------

When we program in Java, we often use hard references, usually without even thinking about it — and for a good reason, because they're the best option for most circumstances. However, sometimes we need more control over the time when objects are available for clearing by the garbage collector.

In this article, we'll explore the differences between hard and various non-hard reference types and when we can use them.

2\. Hard References[](#hard)
----------------------------

A hard (or strong) reference is the default type of reference, and most of the time, we may not even think about when and how referenced objects are garbage collected. **The object can't be garbage collected if it's reachable through any strong reference.** Let's say we create an _ArrayList_ object and assign it to the _list_ variable:

```null
List<String> list = new ArrayList<>;
```

The garbage collector can't collect this list because we hold a strong reference to it in the _list_ variable. But if we then nullify the variable:

```null
list = null;
```

Now and only now, the _ArrayList_ object can be collected because nothing holds a reference to it.

[![](https://a.pub.network/core/imgs/fslogo-green.svg)
](https://freestar.com/?utm_campaign=branding&utm_medium=banner&utm_source=baeldung.com&utm_content=baeldung_leaderboard_mid_1)

AD

3\. Beyond Hard References[](#non-hard)
---------------------------------------

There's a good reason why hard references are the default. They let the garbage collector work as intended, so we don't have to worry about managing memory allocation. Despite that, there are situations in which we want to collect objects and deallocate memory even if we still hold a reference to these objects.

4\. Soft References[](#soft)
----------------------------

A [soft reference](https://www.baeldung.com/java-soft-references) tells the garbage collector that a referenced object can be collected at the collector's discretion. The object can stay in the memory for some time until the collector decides that he needs to collect it. That'll happen, especially when JVM is at risk of running out of memory. **All soft references to objects reachable only by soft reference should be cleared out before the _OutOfMemoryError_ exception is thrown.**

We can use a soft reference easily by wrapping it around our object:

```null
SoftReference<List<String>> listReference = new SoftReference<List<String>>(new ArrayList<String>());
```

If we want to retrieve the referent, we can use the _get_ method. Because the object could be already cleared, we need to check for it:

```null
List<String> list = listReference.get();
if (list == null) {
    
}
```

### 4.1. Use Case[](#1-use-case)

Soft references can be used **to make our code more resilient to errors connected to insufficient memory**. For example, we could create a memory-sensitive cache that automatically evicts objects when memory is scarce. We wouldn't need to manage the memory manually, as the garbage collector would do it for us.

[![](https://a.pub.network/core/imgs/fslogo-green.svg)
](https://freestar.com/?utm_campaign=branding&utm_medium=banner&utm_source=baeldung.com&utm_content=baeldung_leaderboard_mid_2)

5\. Weak References[](#weak)
----------------------------

**Objects referenced only by [weak references](https://www.baeldung.com/java-weak-reference) aren't prevented from being collected.** From the perspective of garbage collection, they could not exist at all. If a weakly referenced object should be protected from being cleared, it should also be referenced by some hard reference.

### 5.1. Use Case[](#1-use-case-1)

Weak references are most often used **to create canonicalizing mappings**. These are mappings that map only objects that can be reached. A great example is _WeakHashMap_, which works like normal _HashMap_, but its keys are weakly referenced, and they are automatically removed when the referent is cleared.

Using _WeakHashMap_, we can create a short-living cache that clears objects that are no longer used by other parts of the code. If we used a normal _HashMap,_ the mere existence of the key in the map would prohibit it from being cleared by the garbage collector.

6\. Phantom References[](#phantom)
----------------------------------

Similarly to weak references, [phantom references](https://www.baeldung.com/java-phantom-reference) don't prohibit the garbage collector from enqueueing objects for being cleared. The difference is **phantom references must be manually polled from the reference queue before they can be finalized**. That means we can decide what we want to do before they are cleared.

### 6.1. Use Case[](#1-use-case-2)

Phantom references are great **if we need to implement some finalization logic**, and they're considerably more reliable and flexible than the [_finalize_](https://www.baeldung.com/java-finalize) method. Let's write a simple method that will go through the reference queue and perform cleanup for all references:

[![](https://a.pub.network/core/imgs/fslogo-green.svg)
](https://freestar.com/?utm_campaign=branding&utm_medium=banner&utm_source=baeldung.com&utm_content=baeldung_leaderboard_mid_3)

```null
private static void clearReferences(ReferenceQueue queue) {
    while (true) {
        Reference reference = queue.poll();
        if (reference == null) {
            break; 
        }
        cleanup(reference);
    }
}
```

Phantom references don't allow us to retrieve their referent using the _get_ method. Because of that, it's a common practice to extend the _PhantomReference_ class with our own that contains information important for the cleanup logic.

Other important use cases for phantom references are **debugging and memory leak detection**. Even if we don't need to perform any finalizing operations, we can use phantom references to observe what objects are being deallocated and when.

7\. Conclusion[](#summary)
--------------------------

In this article, we explored hard and different types of non-hard references and their use cases. We learned that soft references could be used for memory protection, weak references for canonicalizing mappings, and phantom references for fine-grained finalization.

We’re finally running a Black Friday launch. All Courses are **30% off** until **the end of this week:**

### **[\>> GET ACCESS NOW](https://www.baeldung.com/all-courses)**