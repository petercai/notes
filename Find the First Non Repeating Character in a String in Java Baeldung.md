# Find the First Non Repeating Character in a String in Java | Baeldung
  

We’re finally running a Black Friday launch. All Courses are **30% off** until **the end of this week:**

### **[\>> GET ACCESS NOW](https://www.baeldung.com/all-courses)**

1\. Overview[](#overview)
-------------------------

In this tutorial, we're going to look at the different ways of finding the first non-repeating character in a String in Java.

We'll also try to analyze the running time complexities of the solutions.

2\. Problem Statement[](#problem-statement)
-------------------------------------------

**Given a String of characters as input, find the first non-repeating character in the string**. Here are a few examples:

Example 1: _Lullaby_

In this example, L is repeated three times. The first non-repeating character encounter occurs when we reach the character _u. _

Example 2: _Baeldung_

All characters in this example are non-repeating. As per the problem statement, we take the first one, B.

Example 3: _mahimahi_

There are no non-repeating characters in this example – all characters repeat once. So the output here is _null._

Finally, here are some additional points to keep in mind about the problem:

*   The input String can be of any length and **can contain a mix of upper case and lower case characters**
*   Our solution should take care of empty or null input
*   **The input String can have no non-repeating characters**, or in other words, there can be an input having all characters repeat at least once, in which case the output is _null_

With this understanding, let's try to approach the problem.

3\. Solutions[](#solutions)
---------------------------

### 3.1. Brute Force Solution[](#1-brute-force-solution)

First, we try to work out a brute-force solution to finding the first non-repeating character of a string. We start from the beginning of the string and take one character at a time, and compare the character against every other character of the string. If we find a match, that means that this character is repeated elsewhere in the string, so we move on to the next character. If there is no match for the character, we have found our solution, and we exit the program with the character.

The code looks like the following:

```null
public Character firstNonRepeatingCharBruteForceNaive(String inputString) {
    if (null == inputString || inputString.isEmpty()) {
        return null;
    }
    for (int outer = 0; outer < inputString.length(); outer++) {
        boolean repeat = false;
        for (int inner = 0; inner < inputString.length(); inner++) {
            if (inner != outer && inputString.charAt(outer) == inputString.charAt(inner)) {
                repeat = true;
                break;
            }
        }
        if (!repeat) {
            return inputString.charAt(outer);
        }
    }
    return null;
}
```

**The [time complexity](https://www.baeldung.com/java-algorithm-complexity) of the above solution is O(n²)** because of the two nested loops that we have. For every character that we visit, we're visiting all the characters of the input string.

A more compact solution of the same code is also presented below, which leverages the [_lastIndexOf _](https://www.baeldung.com/string/last-index-of)methods of the _String_ class. **When we find a character whose first index in the string is also the last index, that establishes that the character exists only at that index in the string and hence becomes the first non-repeating character.**

The time complexity of this is O(n) as well. It should be noted that the _lastIndexOf_ method runs in another O(n) time, in addition to the outer loop that is already running where we're taking a character at a time, thereby making this a O(n²) solution, similar to the previous one.

```null
public Character firstNonRepeatingCharBruteForce(String inputString) {
    if (null == inputString || inputString.isEmpty()) {
        return null;
    }
    for (Character c : inputString.toCharArray()) {
        int indexOfC = inputString.indexOf(c);
        if (indexOfC == inputString.lastIndexOf(c)) {
            return c;
        }
    }
    return null;
}
```

### 3.2. Optimized Solution[](#2-optimized-solution)

Let's see if we can do any better. The bottleneck of the approaches we discussed is that we're comparing each character against every other character that appears in the String, and we continue this unless we reach the end of the String or we find the answer. Instead, if we can remember the number of times each character appears, we won't need to compare every time and instead just look up the frequency of the character. We can use a _Map_ for this purpose, more specifically, a [_HashMap_](https://www.baeldung.com/java-hashmap).

The map would store the character as the key and its frequency in its value. As we visit every character, we have two choices:

1.  If the character already appears in the map, we append the current position to its value
2.  If the character does not appear in the map constructed so far, this is a new character, and we increment the number of times the character was seen

After we complete the computation for the entire String, we have a map that tells us the count of every character in the String. All that is left for us to do is now to iterate over the String one more time, and the first character whose value's size in the map is one is our answer.

This is what the code looks like:

```null
public Character firstNonRepeatingCharWithMap(String inputString) {
    if (null == inputString || inputString.isEmpty()) {
        return null;
    }
    Map<Character, Integer> frequency = new HashMap<>();
    for (int outer = 0; outer < inputString.length(); outer++) {
        char character = inputString.charAt(outer);
        frequency.put(character, frequency.getOrDefault(character, 0) + 1);
    }
    for (Character c : inputString.toCharArray()) {
        if (frequency.get(c) == 1) {
            return c;
        }
    }
    return null;
}
```

The above solution is much faster, **given that a lookup on a map is a constant time operation, or O(1)**. This means that the time to get the result will not increase with increasing input string size.

### 3.3. Additional Notes on the Optimized Solution[](#3-additional-notes-on-the-optimized-solution)

We should discuss some caveats on the optimized solution we discussed previously. The original question posits that the input can be of any length and can contain any character. This makes the choice of the _Map_ for the lookup purpose efficient.

**However, if we could restrict the input character set to only lower case characters/upper case characters/English alphabet characters etc., it would be a better design choice to use an array of a fixed size over a map**.

For example, if the input was restricted to only lowercase English characters, we could have used an array of size 26, where each index in the array refers to an alphabet, and the value could denote the frequency of the character in the string. Finally, the first character in the string whose value in the array is 1 is the answer. Here's the code for it:

```null
public Character firstNonRepeatingCharWithArray(String inputString) {
    if (null == inputString || inputString.isEmpty()) {
        return null;
    }
    int[] frequency = new int[26];
    for (int outer = 0; outer < inputString.length(); outer++) {
        char character = inputString.charAt(outer);
        frequency[character - 'a']++;
    }
    for (Character c : inputString.toCharArray()) {
        if (frequency[c - 'a'] == 1) {
            return c;
        }
    }
    return null;
}
```

**Note that the time complexity is still O(n), but we improved our space complexity to constant space.** This is because, no matter the length of the string, the length of the auxiliary space(the array) that we use to store the frequency will be constant.

4\. Conclusion[](#conclusion)
-----------------------------

In this article, we discussed the different approaches to finding the first non-repeating character in a string.

As usual, you can find all the code samples [over on GitHub](https://github.com/eugenp/tutorials/tree/master/core-java-modules/core-java-string-algorithms-3).

We’re finally running a Black Friday launch. All Courses are **30% off** until **the end of this week:**

### **[\>> GET ACCESS NOW](https://www.baeldung.com/all-courses)**