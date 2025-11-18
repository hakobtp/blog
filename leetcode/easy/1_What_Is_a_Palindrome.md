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

**Overall Complexity**
- Time Complexity: Since the major operations (conversion, reversal, and comparison) are each $O(n)$, 
    the overall time complexity is $O(n)$, where $$n$$ is the number of digits in the number $x$.
- Space Complexity: The space required is also $O(n)$ because you store: 
    - The string representation of $x$ (with $n$ characters).
    - The reversed string (also $n$ characters).

Even though $n$ is generally small (e.g., at most 10 or 11 for typical integer sizes), 
the theoretical complexity is linear with respect to the number of digits.

---

**Approach B:** Reversing the Number Mathematically
- Concept: Reverse the digits of the number using arithmetic operations, and then compare the reversed number to the original.
    - Techniques to Learn:
        - Modulo and Division: Use the modulo operator to extract the last digit (e.g., x % 10) and 
            division to remove the last digit (e.g., x // 10 in Python).
        - Building the Reversed Number: Learn how to construct the reversed number by iteratively adding digits 
            (e.g., multiplying the current reversed number by 10 and then adding the extracted digit).
        - Edge Case Handling: Understand why negative numbers or numbers ending with zero (other than 0 itself) might need special consideration.

```java
public boolean isPalindrome(int x) {
    if (x < 0) {
        return false;
    }
    int original = x;
    int reverse = 0;
    while (x != 0) {
        reverse = reverse * 10 + x % 10;
        x = x / 10;
    }
    return original == reverse;
}
```
- Original Value Storage: The variable original holds the initial value of $x$, so you can compare it with the reversed number later.
- Reversal Logic: The loop extracts each digit from $x$ (using `x % 10`) and builds the reversed number. Each iteration removes 
    the last digit by dividing $x$ by 10.
- Final Comparison: Once the loop completes, original is compared to reverse to check if the number is a palindrome.    

**Complexity**
- Time Complexity: $O(logx)$ because the number of iterations depends on the number of digits in $x$.
- Space Complexity: $O(1)$ since only a few extra variables are used regardless of the input size.

## Edge Cases

- **Negative Numbers:** Typically, negative numbers are not considered palindromes because of the minus sign.
- **Numbers with Trailing Zeros:** For example, 10 is not a palindrome even though 01 (if you reversed it) might be considered the same as 10 when ignoring leading zeros.
- **Zero:** Zero is usually considered a palindrome.

---

- [Home](./../../README.md)
- [LeetCode](./../tutorials.md)