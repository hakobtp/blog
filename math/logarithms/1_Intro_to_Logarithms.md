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

---




- 🏠 [Home](./../../README.md)
- 📐 [Math Tutorials](./../tutorials.md)