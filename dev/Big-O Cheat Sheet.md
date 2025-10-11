# Big-O Cheat Sheet
[Start of main content](#skip-link-target)

![](https://www.30secondsofcode.org/assets/blog_images/light-ring.jpg)

### [](#definition)Definition

Big-O notation, represents an algorithm's worst-case complexity. It uses algebraic terms to describe the complexity of an algorithm, allowing you to measure its efficiency and performance. Below you can find a chart that illustrates Big-O complexity:

![](https://www.30secondsofcode.org/assets/blog_images/big-o-complexity.png)

Simply put, `O(1)` stands for constant time complexity, which is the most efficient, while `O(n!)` stands for factorial time complexity, which is the least efficient. The `n` in the complexity represents the size of the input, so `O(n)` means that the algorithm's time complexity will grow linearly with the size of the input.

Apart from Big-O notation, there are other notations that are used to describe the complexity of an algorithm, such as `Ω` (Omega) and `Θ` (Theta). `Ω` describes the best-case complexity of an algorithm, while `Θ` describes the average-case complexity of an algorithm.

### [](#common-data-structure-operations)Common Data Structure operations

Different data structures have different time complexities for the same operations. For example, a linked list has `O(1)` time complexity for `insert` and `delete` operations, while an array has `O(n)` time complexity for the same operations. Below you can find average and worst time complexities for data structures used commonly in web development.

#### [](#average-time-complexity)Average time complexity

| Data Structure | Access | Search | Insertion | Deletion |
| --- | --- | --- | --- | --- |
| [**Array**](https://www.30secondsofcode.org/articles/s/js-native-data-structures) | Θ(1) | Θ(n) | Θ(n) | Θ(n) |
| [**Queue**](https://www.30secondsofcode.org/articles/s/js-data-structures-queue) | Θ(n) | Θ(n) | Θ(1) | Θ(1) |
| [**Stack**](https://www.30secondsofcode.org/articles/s/js-data-structures-stack) | Θ(n) | Θ(n) | Θ(1) | Θ(1) |
| [**Linked List**](https://www.30secondsofcode.org/articles/s/js-data-structures-linked-list) | Θ(n) | Θ(n) | Θ(1) | Θ(1) |
| [**Doubly Linked List**](https://www.30secondsofcode.org/articles/s/js-data-structures-doubly-linked-list) | Θ(n) | Θ(n) | Θ(1) | Θ(1) |
| **Skip List** | Θ(log n) | Θ(log n) | Θ(log n) | Θ(log n) |
| **Hash Table** | N/A | Θ(1) | Θ(1) | Θ(1) |
| [**Binary Search Tree**](https://www.30secondsofcode.org/articles/s/js-data-structures-binary-search-tree) | Θ(log n) | Θ(log n) | Θ(log n) | Θ(log n) |

#### [](#worst-time-complexity)Worst time complexity

### [](#array-sorting-algorithms)Array sorting algorithms

Similar to data structures, different array sorting algorithms have different time complexities. Below you can find the best, average and worst time complexities for the most common array sorting algorithms.

