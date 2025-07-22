# Limits and the Euler Number


Mathematics is full of concepts that might seem abstract at first but are crucial for 
understanding the natural world and various scientific principles. Two such concepts are the limit (often
denoted as _lim_) and the Euler number (denoted as _e_). Let’s explore what these mean in simple
terms and how they relate to each other.

## What is a Limit?

A limit is a fundamental idea in mathematics that describes the value that a function or
sequence _approaches_ as the input gets closer to a certain point. Think of a limit as a destination
that a function or a series of numbers is trying to reach, even if it never actually gets there.

Imagine you’re filling a cup with water very slowly. The closer you get to the top, the closer you
are to the cup being full. If you keep adding water drop by drop, you’ll approach a limit—the
cup being full. But as long as you keep adding tiny drops without overflowing, you’re always
getting closer to that limit without quite reaching it.

In mathematical terms, if we look at the function $$f(x)=\frac{1}{x}$$.
- As `x` becomes a very large number (like 1000, 10000, etc.), the value of $$f(x)=\frac{1}{x}$$ becomes smaller and smaller.
- The limit of $$f(x)$$ as `x` approaches infinity is `0`. This is written as:

 $$
 
 \lim_{x \to \infty} \frac{1}{x} = 0
 
 $$

 This notation means that as `x` gets larger and larger, the value of $\frac{1}{x}$ gets closer and closer to `0`.

## Introducing the Euler Number ($e$)

The Euler number, denoted as $e$, is a special number in mathematics, approximately equal to $2.718$. 
It is named after the Swiss mathematician Leonhard Euler and is one of the most important numbers in 
mathematics because it arises naturally in many different contexts, especially those involving growth, decay, and continuous processes.

The number $e$ can be understood through the concept of continuously compounding interest. Here’s a simplified explanation:

1. **Compounded Interest:** Imagine you have `1$` in a bank account with a `100%` interest rate per year.
    - If the interest is compounded once a year, you’d have `2$` at the end of the year $$1+1=2$$. 
    - If the interest is compounded every six months, you’d have more than `2.25$` at the end of the year.
        - After 6 months:  $1 + \frac{1}{2} = 1.5$.
        - After 1 year:  $1.5 + \frac{1.5}{2} = 2.25$.
    - If it is compounded monthly, weekly, daily, or every second, the amount keeps increasing, but not by much.        

2. **Continuous Compounding:** Now, think about compounding the interest an infinite
number of times within the same year. The formula that represents the amount you
would have after one year, as the compounding becomes infinite, is:

$$
\lim_{x \to \infty} \left( 1+\frac{1}{n} \right)^n= 0
$$

---

- [Home](./../../../README.md)
- [Math Tutorials](./../../tutorials.md)