+++
title = "Immutability and why should you care"
date = 2023-05-20

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

### Initialization

From a [presentation](https://github.com/microsoft/MSRC-Security-Research/blob/master/presentations/2019_02_BlueHatIL/2019_01%20-%20BlueHatIL%20-%20Trends%2C%20challenge%2C%20and%20shifts%20in%20software%20vulnerability%20mitigation.pdf) by Matt Miller at BlueHat IL 2019, he mentioned that 70% of the vulnerabilities found in Microsoft products are memory safety issues.

According to the root cause analysis of these vulnerabilities, the 4th most common root cause is "uninitialized use".

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

As `x` is uninitialized, when we try to print `x`, we are printing whatever value is stored at that memory address.

While this example is trivial, it will lead to serious bugs in more complex programs, which can be hard to debug.

If you're using pointers, you can get a segfault if you try to dereference an uninitialized pointer, as it is pointing to a random memory address that you do not own.

In this case, what if we had used `x` in a calculation, and assumed that it was 0? We would've gotten the wrong result.

Worst of all, this won't crash your program, so you may not notice the bug until it's too late.

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

### Clarity and correctness

Immutability makes it clear to the reader that the variable will not be modified after it is created.

If you've ever worked on a large codebase, you will know that it is hard to keep track of all the variables and their values.

You step through the code with a debugger, go through the call stack, and try to figure out what is going on with one variable, only to realize "Wait, this variable is not being modified at all". So then you go back to the call stack and try to figure out what is going on with another variable.

For example, let's say we have a function that takes in a vector of integers and returns the sum of all the integers in the vector.

```cpp
auto sum(std::vector<int>& numbers) -> int {
    int result = 0;
    for (auto number : numbers) {
        result += number;
    }
    return result;
}

auto main() -> int {
    std::vector<int> numbers = {1, 2, 3, 4, 5};
    int total = sum(numbers);
    // more code here
}
```

We declare `numbers` as a vector of integers from 1 to 5, and result as the sum of all the integers in `numbers`.

Let's say there's more code after this, we cannot be confident whether that `numbers` and `total` have been modified down the path.

So instead, we would have to go through the code and track down how they are being used.

But what if we had made `numbers` and `total` immutable?

```cpp
auto sum(const std::vector<int>& numbers) -> int {
    int result = 0;
    for (const auto number : numbers) {
        result += number;
    }
    return result;
}

auto main() -> int {
    const std::vector<int> numbers = {1, 2, 3, 4, 5};
    const int total = sum(numbers);
    // more code here
}
```

Now to me, this is very clear that `numbers` and `total` could be read in this scope, but they will not be modified.

Also, notice the change we made to the `sum` function, we made `numbers` parameter immutable as well.

That is a contract that we are making with the caller of the function, that we will not modify `numbers` in the function, and this gives the caller the confidence that the object they passed in won't change, without having to go through the function to check.

In the `sum` function, we also make `number` immutable when we iterate through `numbers`, as we do not need to modify `number` in the loop.

But now that we have done all this, it becomes clear that in this entire snippet, the only thing that can have its state changed is `result`.

Not is this code clearer, but it is also correct and safer, as we have made it impossible to modify `numbers` and `total` by accident. I'm sure your reviewer will appreciate as well.

## Conclusion

Immutability is a simple but powerful way you can use to write code that is safer, clearer and correct. If you use them, more than likely, you will run into fewer bugs and surprises.

These are just some of the benefits of immutability, and there are more. But in my opinion, these are the most important ones.

If you wish to start using immutability in your code, here are some tips:

- Always make your variables immutable by default, and only make them mutable when you need to.
- If you need to make a variable mutable, try to limit its scope as much as possible. [The root of all evil is shared mutable state](http://henrikeichenhardt.blogspot.com/2013/06/why-shared-mutable-state-is-root-of-all.html).
- If you need to perform a complex initialization, you may use a lambda function, and `const` initialize the variable with the result of the lambda function.

E.g.

```cpp
auto main() -> int {
    const std::string name = []() {
        std::string name;
        std::cin >> name;
        return name;
    }();
}
```

- Try the Rust programming language, as it is designed with immutability by default when declaring variables. It is very powerful and allows to easily create immutable variables with expressions.

E.g.

```rs
// Rust uses immutable variables by default
// To declare a mutable variable, you need to use the `mut` keyword
fn main() {
    let vector = vec![1, 2, 3, 4, 5];
    let total = vector.iter().sum();
}
```

I hope that this article has convinced you to prefer immutability whenever possible, if you have any questions or feedback, feel free to reach out to me in the **Contact me** section of my [about page](/about).
