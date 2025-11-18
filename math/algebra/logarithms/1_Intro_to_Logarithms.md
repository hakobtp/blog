# Intro to Logarithms

```info
Author      Ter-Petrosyan Hakob
```
---

You will learn what logarithms are, and evaluate some basic logarithms.
This will prepare you for future work with logarithm expressions and functions.

## What is a logarithm?

Logarithms are another way of thinking about exponents. 
For example, we know that $$2$$ raised to the $$4^{th}$$ power equals $$16$$. 
expressed by the **exponential** equation $$2^4=16$$.

Now, suppose someone asked us, `"raised to which power equals 16?"` The answer would be $$4$$. 
This is expressed by the **logarithmic** equation $$\log_{2}(16)=4$$, read as `"log base two of sixteen is four"`.

$$
2^4 =16 \Leftrightarrow \log_{2}(16)=4
$$

Both equations describe the same relationship between the numbers 2, 4 and 16, where 2 is the **base** and 4 is the **exponent**. 
The difference is that while the exponential form isolates the power, 16, the logarithmic form isolates the exponent, 4.

Here are more examples of equivalent logarithmic and exponential equations.

|Logarithmic form|Exponential form|
|:---------------|:---------------|
|$$\log_{2}(8)=3$$|$$2^3=8$$|
|$$\log_{3}(81)=4$$|$$3^4=81$$|
|$$\log_{5}(25)=2$$|$$5^2=25$$|

## Definition of a logarithm

Generalizing the examples above leads us to the formal definition of a logarithm.

$$
 \log_{\textcolor{red}{b}}(\textcolor{orange}{a}) = \textcolor{blue}{c} \Leftrightarrow \textcolor{red}{b}^{\textcolor{blue}{c}} = \textcolor{orange}{a}
$$

Both equations describe the same relationship between $$\textcolor{orange}{a}$$, $$\textcolor{red}{b}$$, and $$\textcolor{blue}{c}$$:

- $$\textcolor{red}{b}$$ is the base.
- $$\textcolor{blue}{c}$$ is the exponent.
- $$\textcolor{orange}{a}$$ is called the argument

> 📝 **NOTE:** When rewriting an exponential equation in log form or a log equation in exponential form,
> it is helpful to remember that the base of the logarithm is the same as the base of the exponent.

## Evaluating logarithms

Great! Now that we understand the relationship between exponents and logarithms, let's see if we can evaluate logarithms. 

For example, let's evaluate $$\log_4(64)$$

Let's start by setting that expression equal to $$x$$.

$$
\log_4(64) = x
$$

Writing this as an exponential equation gives us the following

$$
4^x=64
$$        

4 to what power is 64, Well, $$4^3=64$$ and so $$ \log_4(64) = 3$$

As you become more practiced, you may find yourself condensing a few of these steps and evaluating 
$$ \log_4(64)$$ just by asking, **4 to what power is 64?**


## Restrictions on the variables

 $$\log_{\textcolor{red}{b}}(\textcolor{orange}{a})$$ is defined when the base $$\textcolor{red}{b}$$ 
 is positive—and not equal to 1—and the argument $$\textcolor{orange}{a}$$ is positive. 
 These restrictions are a result of the connection between logarithms and exponents.

|Restriction|Reasoning|
|:----------:|:-------|
|$$\textcolor{red}{b} > 0$$ |In an exponential function, the base $$\textcolor{red}{b}$$ is always defined to be positive.|
|$$\textcolor{orange}{a} > 0$$ |$$\log_{\textcolor{red}{b}}(\textcolor{orange}{a}) = \textcolor{blue}{c}$$ means that $$\textcolor{red}{b}^{\textcolor{blue}{c}} = \textcolor{orange}{a}$$. Because a positive number raised to any power is positive, meaning $$\textcolor{red}{b}^{\textcolor{blue}{c}} > 0$$, it follows that $$\textcolor{orange}{a} > 0$$.|
|$$\textcolor{red}{b} \neq 1$$|Suppose, for a moment, that  $$\textcolor{red}{b}$$ could be 1. Now consider the equation $$\log_1(3)=x$$. The equivalent exponential form would be $$1^x=3$$. But this can never be true since 1 to any power is always 1. So, it follows that $$\textcolor{red}{b} \neq 1$$|

## Special logarithms

While the base of a logarithm can have many different values, there are two bases that are used more often than others. 
Specifically, most calculators have buttons for only these two types of logarithms. Let's check them out.

### The common logarithm

The common logarithm is a logarithm whose base is 10 (`"base-10 logarithm"`). 
When writing these logarithms mathematically, we omit the base. It is understood to be 10.

$$
\log_{10}(x) = \log(x)
$$

### The natural logarithm

The natural logarithm is a logarithm whose base is the number $$e$$.
$$e$$ is a mathematical constant. It is an irrational number that is approximately equal to $$2.718$$.
It appears in many contexts that involve limits, which you will likely learn about as you study calculus. 
For now, just treat $$e$$ as you would any other number.

Instead of writing the base as $$e$$ , we indicate the logarithm with $$\ln$$.

$$
\log_{e}(x) = \ln(x)
$$

This table summarizes what we need to know about these two special logarithms:

|Name               |Base   |Regular notation   |Special notation   |
|:------------------|:-----:|:-----------------:|:-----------------:|
|Common logarithm   |10     |$$\log_{10}(x)$$   |$$\log(x)$$        |
|Natural logarithm  |$$e$$  |$$\log_{e}(x)$$    |$$\ln(x)$$         |

While the notation is different, the idea behind evaluating the logarithm is exactly the same!

## Why are we studying logarithms?

As you just learned, logarithms reverse exponents. For this reason, they are very helpful for solving exponential equations.

For example the result for $$2^x=5$$ can be given as a logarithm, $$x=\log_{2}(5)$$. 
You will learn how to evaluate this logarithmic expression over the following lessons.

Logarithmic expressions and functions also turn out to be very interesting by themselves, 
and are actually very common in the world around us. For example, many physical phenomena are measured with logarithmic scales.

## Evaluating logarithms

**Example 1:**  
- To solve $$\log_{8}(2)$$ we want to find the exponent `x` such that: $$8^x=2$$. 
- To solve this, let’s express `8` as a power of `2`: $$8=2^3$$. 
- Substituting this into the equation, we get: $$(2^3)^x=2$$. 
- Simplifying the left side, we get: $$2^{3x}=2^1$$.
- Since the bases are the same, we can set the exponents equal to each other: $$3x=1$$.
- Now, solve for `x`: $$x=\frac{1}{3}$$.
- So: $$\log_{8}(2)=\frac{1}{3}$$.

This means that `2` is `8` raised to the power of $$x=\frac{1}{3}$$, which is the cube root of `8`.

**Example 2:**
- To solve $$\log_{8}(\frac{1}{2})$$, we want to find the exponent `x` such that: $$8^x=\frac{1}{2}$$.
- We know that $$8=2^3$$. So we can rewrite the equation using this: $$(2^3)^x=\frac{1}{2}$$.
- Using the power of a power rule $$(a^m)^n=a^{mn}$$ , we can simplify: $$2^{3x}=\frac{1}{2}$$.
- The fraction $$\frac{1}{2}$$ can be expressed as a negative exponent: $$\frac{1}{2}=2^{-1}$$.
- So, the equation becomes: $$2^{3x}=2^{-1}$$.
- Since the bases are the same, we can set the exponents equal to each other: $$3x=-1$$.
- Divide both sides by `3` to solve for `x`: $$x=-\frac{1}{3}$$.
- Conclusion: $$\log_{8}(\frac{1}{2})=-\frac{1}{3}$$.

This means that to get $$\frac{1}{2}$$ from `8`, you need to raise `8` to the power of $$-\frac{1}{3}$$.

---

- [Home](./../../../README.md)
- [Math Tutorials](./../../tutorials.md)
- [Exponential VS Logarithms](./2_Relationship_between_exponential_and_logarithms.md)