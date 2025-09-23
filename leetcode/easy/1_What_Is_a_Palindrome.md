# What Is a Palindrome?

**Definition:** A palindrome is a number or string that reads the same forwards and backwards.

Examples:
- **For numbers:** 121 is a palindrome, while 123 is not.
- **For strings:** "racecar" is a palindrome, but "hello" is not.

## Two Main Approaches to Check for a Palindrome

**Approach A:** Converting to a String

- **Concept:** Convert the integer into a string so you can easily compare it with its reverse.
    - `String Conversion:` Learn how to convert data types (e.g., using `str()` in Python, `toString()` in Java).
    - `String Reversal:` Understand how to reverse a string. In many languages, there are built-in functions or 
        slicing techniques (e.g., Python’s [::-1] slice).
    - `Comparison:` Learn how to compare two strings for equality.

```java
public boolean isPalindrome(int x) {
    if (x < 0) {
        return false;
    }
    var xStr = String.valueOf(x);

    // The contentEquals result is true if and only if this String represents 
    // the same sequence of char values as the specified sequence.
    return xStr.contentEquals(new StringBuilder(xStr).reverse());
}
```    

Let's break down the operations and analyze the `Big O` complexity:
- Negative Check: The condition `if (x < 0)` takes constant time, $O(1)$
- Conversion to String:
    - `String.valueOf(x)` converts the integer to a string.
    - Let $n$ be the number of digits in $x$; this operation is $O(n)$.
- Creating and Reversing with StringBuilder:
    - Constructing the `StringBuilder` from the string takes $O(n)$ because it copies each character.    
    - The `reverse()` method then reverses the string, which again processes each character once, making it $O(n)$.
- Comparing Strings: `contentEquals(...)` compares the original string to the reversed string, which in the worst-case scenario 
    requires checking all $n$ characters. This is $O(n)$.
    
---

- [Home](./../../README.md)
- [LeetCode](./../tutorials.md)