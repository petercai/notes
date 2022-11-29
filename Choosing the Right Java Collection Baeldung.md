# Choosing the Right Java Collection | Baeldung
  

We’re finally running a Black Friday launch. All Courses are **30% off** until **the end of this week:**

### **[\>> GET ACCESS NOW](https://www.baeldung.com/all-courses)**

1\. Introduction[](#introduction)
---------------------------------

In this tutorial, we're going to discuss how to choose the proper collection interface and class in the Java library. We skip legacy collections, such as [_Vector_](https://www.baeldung.com/java-arraylist-vs-vector "Vector link")_, [Stack](https://www.baeldung.com/java-stack "Stack link"), and [Hashtable](https://www.baeldung.com/java-hash-table "Hashtable link")_ in our discussion as we need to avoid using them in favor of the new collections. Concurrent collections deserve a separate topic, so we don't discuss them either.

2\. Collection Interfaces in the Java Library[](#collection-interfaces-in-the-java-library)
-------------------------------------------------------------------------------------------

It's very useful to know the organization of the collection interfaces and classes in the Java library before trying to use them efficiently. The _[Collection](https://docs.oracle.com/javase/8/docs/api/java/util/Collection.html) _interface is the root of all the collection interfaces. [_List_](https://docs.oracle.com/javase/8/docs/api/java/util/List.html), _[Set](https://docs.oracle.com/javase/8/docs/api/java/util/Set.html),_ and [_Queue_](https://docs.oracle.com/javase/8/docs/api/java/util/Queue.html) interfaces extend the _Collection._

Maps in the Java library are not treated as regular collections, so the [_Map_](https://docs.oracle.com/javase/8/docs/api/java/util/Map.html) interface doesn't extend _Collection._ Here's the diagram for interface relationships in the Java library:

[![](https://www.baeldung.com/wp-content/uploads/2022/11/1-1.png)
](https://www.baeldung.com/wp-content/uploads/2022/11/1-1.png)

Any concrete collection implementation (collection class) is derived from one of the collection interfaces. The semantics of collection classes are defined by their interfaces, as concrete collections provide specific implementations for operations that their parent interfaces define. Consequently, we need to choose the proper collection interface before selecting the suitable collection class.

3\. Choose the Right Collection Interface[](#choose-the-right-collection-interface)
-----------------------------------------------------------------------------------

Choosing the right collection interface is somewhat straightforward. Indeed, the diagram below shows a logical interface selection flow:

[![](https://a.pub.network/core/imgs/fslogo-green.svg)
](https://freestar.com/?utm_campaign=branding&utm_medium=banner&utm_source=baeldung.com&utm_content=baeldung_leaderboard_mid_1)

[![](https://www.baeldung.com/wp-content/uploads/2022/11/Interface-Selection-Diagram-1.png)
](https://www.baeldung.com/wp-content/uploads/2022/11/Interface-Selection-Diagram-1.png)

To summarize, we use lists when the insertion order of elements matters and there are duplicate elements. Sets are used when elements are treated as a set of objects, there are no duplicates, and the insertion order doesn't matter.

Queues are used when _LIFO_, _FIFO_, or removal by priority semantics is required, and finally, maps are used when the association of keys and values is needed.

4\. Choose the Right Collection Implementation[](#choose-the-right-collection-implementation)
---------------------------------------------------------------------------------------------

Below we can find the comparison tables of collection classes separated by the interfaces they implement. The comparisons are made based on common operations and their performance. Specifically, the performance of operations is estimated using [_Big-O_](https://www.baeldung.com/java-algorithm-complexity) notation. A more practical guide to operations' duration in Java collections can be found in the [benchmark of collection operations](https://www.baeldung.com/java-collections-complexity).

### 4.1. Lists[](#1-lists)

Let's start with a list comparison table. Common operations for lists are adding and removing elements, accessing an element by index, traversal of the elements, and finding an element:

| Lists Comparison Table | Add/remove element in the beginning | Add/remove element in the middle | Add/remove element in the end | Get i-th element (random access) | Find element | Traversal order |
| _[ArrayList](https://www.baeldung.com/java-arraylist)_ | _O(n)_ | _O(n)_ | _O(1)_ | _O(1)_ | _O(n)_, _O(log(n))_ if sorted | as inserted |
| [_LinkedList_](https://www.baeldung.com/java-linkedlist) | _O(1)_ | _O(1)_ | _O(1)_ | _O(n)_ | _O(n)_ | as inserted |

As we can see, _ArrayList_ is good at adding and removing elements in the end, as well as having random access to elements. Conversely, it's bad at adding and removing elements at arbitrary positions. Meanwhile, _LinkedList_ is good at adding and removing elements at any position. However, it doesn't support true _O(1)_ random access. So, regarding lists, the default choice is _ArrayList_ until we need fast element addition and removal at any position.

[![](https://a.pub.network/core/imgs/fslogo-green.svg)
](https://freestar.com/?utm_campaign=branding&utm_medium=banner&utm_source=baeldung.com&utm_content=baeldung_leaderboard_mid_2)

AD

### 4.2. Sets[](#2-sets)

For sets, we're interested in adding and removing elements, traversal of elements, and finding an element:

| Sets Comparison Table | Add element | Remove element |  Find element | Traversal order |
| [_HashSet_](https://www.baeldung.com/java-hashset) | amortized _O(1)_ | amortized _O(1)_ | _O(1)_ | random, scattered by the hash function |
| [_LinkedHashSet_](https://www.baeldung.com/java-linkedhashset) | amortized _O(1)_ | amortized _O(1)_ | _O(1)_ | as inserted |
| [_TreeSet_](https://www.baeldung.com/java-tree-set) | _O(log(n))_ | _O(log(n))_ | _O(log(n))_ | sorted, according to elements comparison criterion |
| [_EnumSet_](https://www.baeldung.com/java-enumset) | _O(1)_ | _O(1)_ | _O(1)_ | according to the definition order of the enum values |

As we can see, the default choice is the _HashSet_ collection, as it's very fast for all the operations it supports. Furthermore, if also the insertion order of elements matters, we go with _LinkedHashSet_. Basically, it's an extension of _HashSet_, which keeps track of elements' insertion order by using a linked list structure internally.

If the elements need to be sorted and the sorted order needs to be preserved while adding and removing elements, then we go with _TreeSet_.

If the elements of the set are just enumeration values of a single enum type, then the wisest choice is _EnumSet_.

### 4.3. Queues[](#3-queues)

Queues can be divided into two groups:

[![](https://a.pub.network/core/imgs/fslogo-green.svg)
](https://freestar.com/?utm_campaign=branding&utm_medium=banner&utm_source=baeldung.com&utm_content=baeldung_leaderboard_mid_3)

AD

1.  _LinkedList_, [_ArrayDeque_](https://www.baeldung.com/java-array-deque) – [_Queue_](https://www.baeldung.com/java-queue) interface implementations can act as the stack, queue, and dequeue data structures. Generally, _ArrayDeque_ is faster than _LinkedList_. Hence it's the default choice
2.  _[PriorityQueue](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/PriorityQueue.html) – Queue _interface implementation backed by the binary heap data structure. Used for fast (_O(1)_) element retrieval, which has the highest priority. Addition and removal work in _O(log(n))_ time

### 4.4. Maps[](#4-maps)

Similarly to sets, we consider the operations of adding and removing elements, traversal of elements, and finding an element for maps:

| Maps Comparison Table | Add element | Remove element |  Find element | Traversal order |
| [_HashMap_](https://www.baeldung.com/java-hashmap) | amortized _O(1)_ | amortized _O(1)_ | _O(1)_ | random, scattered by the hash function |
| [_LinkedHashMap_](https://www.baeldung.com/java-linked-hashmap) | amortized _O(1)_ | amortized _O(1)_ | _O(1)_ | as inserted |
| [_TreeMap_](https://www.baeldung.com/java-treemap) | _O(log(n))_ | _O(log(n))_ | _O(log(n))_ | sorted, according to elements comparison criterion |
| [_EnumMap_](https://www.baeldung.com/java-enum-map) | _O(1)_ | _O(1)_ | _O(1)_ | according to the definition order of the enum values |

The selection logic for maps is similar to the selection logic for sets: we use _HashMap_ by default, _LinkedHashMap_ if additionally, insertion order is important, _TreeMap_ for sorting, and _EnumMap_ when keys belong to values of a specific enum type.

Lastly, there are two implementations of the _Map_ interface, which have very specific applications: [_IdentityHashMap_](https://www.baeldung.com/java-identityhashmap), and [_WeakHashMap_](https://www.baeldung.com/java-weakhashmap).

5\. Concrete Collection Selection Diagram[](#concrete-collection-selection-diagram)
-----------------------------------------------------------------------------------

We can extend the diagram for choosing the proper collection interface for selecting concrete collection implementations:

[![](https://www.baeldung.com/wp-content/uploads/2022/11/Concrete-Collection-Selection-Diagram.png)
](https://www.baeldung.com/wp-content/uploads/2022/11/Concrete-Collection-Selection-Diagram.png)

6\. Conclusion[](#conclusion)
-----------------------------

In this article, we went through collection interfaces and collection classes in the Java library. Moreover, we proposed methods for selecting the correct interface and implementation.

We’re finally running a Black Friday launch. All Courses are **30% off** until **the end of this week:**

### **[\>> GET ACCESS NOW](https://www.baeldung.com/all-courses)**