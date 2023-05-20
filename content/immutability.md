+++
title = "Immutability and why should you care"
date = 2023-05-23

[taxonomies]
tags = ["best practices"]
+++

Now, before we begin, I want to make it clear that I am not talking about immutability in the context of functional programming. Instead, I am talking about immutability in the context of programming in general.

## What is immutability?

Immutability is the property of an object that prevents it from being modified after it is created, it is in other words, read-only.

In most popular programming languages, the default behaviour is for objects to be mutable, but you can make them immutable often by using the `const` keyword.

Here is an example in C++:

```cpp
auto main() -> int {
    const int x = 5;
}
```

Here we have created an immutable integer `x` with the value of 5. If we try to modify `x`, the compiler will throw an error.

```cpp
auto main() -> int {
    const int x = 5;
    x = 10;
}
```

```bash
error: cannot assign to variable 'x' with const-qualified type 'const int'
         x = 10;
         ~ ^
note: variable 'x' declared const here
         const int x = 5;
         ~~~~~~~~~~^~~~~
```

Some of you may be thinking; "Well, I can just not use the `const` keyword and make everything mutable. That way, I won't ever run into this problem. JavaScript and Python for life!".

While yes, you can do that, but it is not a good idea.

And, before you ready your pitchforks, I am not saying that you should never use mutable variables, but rather, you should prefer immutability whenever possible.

Now, let's talk about why you should care about immutability and how it can help you write better code.

## Why should you care?

### 1. Initialization

From a [presentation](https://github.com/microsoft/MSRC-Security-Research/blob/master/presentations/2019_02_BlueHatIL/2019_01%20-%20BlueHatIL%20-%20Trends%2C%20challenge%2C%20and%20shifts%20in%20software%20vulnerability%20mitigation.pdf) by Matt Miller at BlueHat IL 2019, he mentioned that 70% of the vulnerabilities found in Microsoft products are memory safety issues.

According to the root cause analysis of these vulnerabilities, the 4th most common root cause is "unitialized use".

This is when a variable is used before it is initialized, which can lead to undefined behaviour.

Here is an example in C++:

```cpp
auto main() -> int {
    int x;
    std::cout << x;
}
```

```bash
2045585552
```

Here, we have created an integer `x` without initializing it. Then, when we try to print `x`, we get garbage values.

As `x` is uninitialized, its value is a random memory address. When we try to print `x`, we are printing whatever value is stored at that memory address.

While this example is trivial, it will lead to serious bugs in more complex programs, which can be hard to debug.

In this case, what if we had used `x` in a calculation, and assumed that it was 0? We would've gotten the wrong result.

Worst of all, this won't crash your program, so you won't know that there is a bug.

What if I tell you, this bug would've been caught at compile time if we had made `x` immutable?

```cpp
auto main() -> int {
    const int x;
    std::cout << x; 
}
```

```bash
error: default initialization of an object of const type 'const int'
         const int x;
                   ^
                     = 0
```

Here, the compiler will throw an error because we are trying to create an `const` integer `x` while attempting to perform a default initialization.

However, as `int` is a primitive type that is `const`, they do not have a default constructor, so the compiler does not know how to initialize `x` implicitly.

To fix this, we can follow the compiler's suggestion and initialize `x` explicitly to 0.

This is good as not only does immutability prevent us from modifying a variable, but it also prevents us from using a variable before it is initialized.

It is a nice bonus best practice that you get for free, just by preferring immutability by default.

### 2. State management

### 3. Scope

### 4. Performance

## Conclusion
