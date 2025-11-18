# Radicals and Rational Exponents

When you square a number (multiply it by itself) and then take its square root, you get the original number back.

$$
4^2 = 16 \quad\Longrightarrow\quad \sqrt{16} = 4
$$

The square root is like the **opposite action** of squaring — just like subtraction is the opposite of addition. To “undo” squaring, we take the square root.

- $$\sqrt{}$$ called the **radical**.
- $$a$$ called the **radicand** (the number under the radical).
- $$\sqrt{a}$$ called a **radical expression**.


If $$a$$ is a positive number, the square root of $$a$$ is a number that, when multiplied by itself, equals $$a$$.

$$
\sqrt{a} \times \sqrt{a} = a
$$        

It can be **positive** or **negative**, because: $$4 \times 4 = 16$$ and $$(-4) \times (-4) = 16$$.

However, in most cases, we use the principal square root, which means the nonnegative root. This is what you see on a calculator.

> **Principal Square Root**
>
> The **principal square root** of $$a$$ is the nonnegative number that, when multiplied by itself, equals $$a$$. It is written as a
> **radical expression**, with a symbol called a **radical** over the term called the **radicand**: $$\sqrt{a}$$.

---

**The Product Rule**

If $$a$$ and $$b$$ are nonnegative, the square root of the product $$ab$$ is equal to the product of the square roots of $$a$$ and $$b$$.
$$
\sqrt{ab}=\sqrt{a} \cdot \sqrt{b}
$$

---

**The Quotient Rule**

The square root of the quotient $$\frac{a}{b}$$ is equal to the quotient of the square roots of $$a$$ and $$b$$, where $$b\neq 0$$.

$$
\sqrt{\frac{a}{b}} = \frac{\sqrt{a}}{\sqrt{b}}
$$

---

**Adding and Subtracting Square Roots**

We can add or subtract radical expressions only when they have the same radicand and when they have the same radical
type such as square roots. For example, the sum of $$\sqrt{2}$$ and $$3\sqrt{2}$$ is $$4\sqrt{2}$$. However, it is often possible 
to simplify radical expressions, and that may change the radicand.

**Example:**

$$
\sqrt{2} + \sqrt{18} = \sqrt{2} + \sqrt{9 \times 2} = \sqrt{2} + \sqrt{9} \times \sqrt{2} = \sqrt{2} + 3 \times \sqrt{2} = 4\sqrt{2}
$$


## How to Remove Square Roots from Denominators

When we simplify a fraction in math, we try to avoid having a square root (radical) in the denominator. 
The process of removing the radical is called rationalizing the denominator.

We use the fact that multiplying any expression by $$1$$ does not change its value. 
To remove the radical, we multiply both the numerator (top) and denominator (bottom) by a special form of $$1$$ that makes the denominator a whole number or at least removes the radical.

### Denominator with a Single Radical Term

If the denominator has only one term with a radical, multiply both numerator and denominator by that radical.

Example:

$$
\frac{5}{\sqrt{3}} \times \frac{\sqrt{3}}{\sqrt{3}} = \frac{5\sqrt{3}}{3}
$$

### Denominator with Two Terms (Rational + Radical)

If the denominator has two terms—a rational number and a radical—multiply by the conjugate of the denominator. 
The conjugate is made by changing the sign between the two terms.

Example:

$$
\frac{4}{2 + \sqrt{5}} \times \frac{2-\sqrt{5}}{2-\sqrt{5}} = \frac{4(2-\sqrt{5})}{(2+\sqrt{5})(2-\sqrt{5})}
$$

When you multiply $$(2+\sqrt{5})(2-\sqrt{5})$$ , you are using the difference of squares formula:

$$
(a+b)(a-b)=a^2-b^2
$$

Here: 
- $$a = 2$$.
- $$b=\sqrt{5}$$.

So:

$$
(2+\sqrt{5})(2-\sqrt{5}) = 2^2 - (\sqrt{5})^2 = 4-5=-1
$$

The denominator becomes $$−1$$, so:

$$
\frac{4(2-\sqrt{5})}{-1} = -8 + 4\sqrt{5}
$$


## Understanding nth Roots

When we talk about roots in math, we often think of square roots. But there are also **cube roots**, **4th roots**, **5th roots**, 
and so on.

Roots are the **inverse** (opposite) of powers. Just like the square root is the opposite of squaring a number, the cube root is the opposite of cubing a number, and the nth root is the opposite of raising a number to the power of $$n$$.

These functions are useful when we want to find **the number that, when raised to a certain power, gives a specific result**.

Example: Cube Roots
- If we know that: $$a^3=8$$.
- we ask: What number, when raised to the 3rd power, equals $$8$$?
- Since: $$2^3=8$$ we say $$2$$ is the cube root of $$8$$.


The **nth root** of a number $$a$$ is a number that, when raised to the power of $$n$$, equals $$a$$.

Example: $$(-3)^5=-243$$. So $$-3$$ is the **5th root** of $$−243$$.

### The Principal nth Root

If $$a$$ is a real number with at least one nth root, then the **principal nth root** of $$a$$ is the number with 
the same sign as $$a$$ that, when raised to the nth power, equals $$a$$.

We write the principal nth root as: 

$$\sqrt[\leftroot{-2} \uproot{2} n]{a}$$

where:
- $$n$$ is a **positive integer** ($$ \geq 2$$)
- $$n$$ is called the **index** of the radical
- $$a$$ is called the **radicand**

Example: $$\sqrt[\leftroot{-2} \uproot{2} 4]{81}=3$$ because $$3^4=81$$

---

- [Home](./../../../README.md)
- [Math Tutorials](./../../tutorials.md)
- [Scientific Notation](./3_Scientific_Notation.md)
