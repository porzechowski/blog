---
title: "Defense programming in embedded systems"
date: 2024-02-13
categories:
  - blog
  - software
  - security
  - safety
tags:
  - C
  - C++
  - defense programming
  - defense coding
  - embedded
  - software
  - security
  - safety
  - yoda conditions
toc: true
toc_label: "Table of Contents"
toc_icon: "cog"
---

# Defense programming is it what?

The topic for today is Defensive Programming (DP).

DP is a set of techniques and guidelines that aims to reduce the number of problems you might encounter with our software.
Bad things happen. The causes are different. We might do a typo, someone might pass a NULL pointer when he shouldn't, there might be a glitch in hardware or error in documentation.

Defensive programming aims to prevent and mitigate faults and errors that will arise sooner or later.
As everything it comes with a cost, you will need to invest time and complexity of your code might increase.

Defensive programming is a wide set, so I will probably miss few things
Now let me describe some techniques that are relevant when talking about defensive programming for embedded systems.

## Don't trust external code

This is old and most of you know this one as check against `NULL` pointer. However we should expand this rule and say that anything handled to us by some external entity cannot be trusted and shall be verified before being used.

The simplest example is good old
```C 
if(ptr != NULL)
```

However if we take something more complex like a frame, the same rule still applies.

Consider following frame

```C
[Header | Content | CRC]
```

We mustn't touch `Header` or `Content` before we verify that CRC is correct. Sounds simple but saves a lot of trouble when we find out that CRC is incorrect in the middle of processing data from the frame.

## Clear data after usage

This one is about good manners. 

**CLEAN AFTER YOURSELF!**

If you are using some data, and you no longer need it, the good thing to do is to clear it from the buffer!

Thanks to that, they won't be reused by mistake or stolen.

## Yoda conditions

Agree you may not, but...

This is one of the simplest and most common technique to prevent nasty bugs.

Consider following code:

```C
if(ptr == NULL) {
  //do something
}
```
With a stupid programmer typo it is super easy to NULL a pointer with a small typo

```C
if(ptr = NULL) {
  //do something
}
```

And here where Yoda comes to help. We will reverse items in `if` bracket, so when we make a typo now:

```C
if(NULL = ptr) {
  //do something
}
```

The compiler will come to the rescue, program will not compile and we will see similar message:

`error: lvalue required as left operand of assignment`

It also works fine with other constants like numbers or `const` variables. So if by mistake we do something like this:

```C
const bool isGood = true;
if(isGood = false) {
  //do something
}
```

We shall see compiler message:

`error: assignment of read-only variable`

However, it get's a bit messy when dealing with numerical intervals. If we need our number to be between 10 and 20, we would normally write it like this:

```C
const int16_t number = 10;
if((number > 10) && (number < 20)) {
  //do something
}
```

But with defensive approach we would write it this way:

```C
const int16_t number = 10;
if((10 <= number) && (20 >= number)) {
  //do something
}
```

For me it is a bit more cognitively challenging... but worth it.




## Safe constants

This technique is commonly use when safety and security are demanded. 

I described it [here](https://porzechowski.github.io/blog/blog/software/security/post-software-countermeasures/#hamming-distance)

## Canaries values

It sucked to be a canary in a mine some time ago.
If there was something bad in the air you would be first one to know.

Software engineers borrowed this beautiful term to name a technique that allows us to detect altering buffers.

The idea is very simple.
We place known values around a buffer. Let's say out canary value is 0xAA. All we do is to place this value around our buffer in memory:

```C
[0xAA, 0xAA, 0xAA | BUFFER | 0xAA, 0xAA, 0xAA]
```

Then we usually check the integrity of these values in some intervals. So when we detect following situation:


```C
[0xAA, 0xAA, 0xAA | BUFFER | 0x00, 0xAA, 0xAA]

```

`0x00` tells us that there was buffer overflow or the buffer was overwritten.

## true == true false == false, a nie !=

When in doubt
```C
if(true == boolVar)
or
if(false != boolVar)
```

Always use 
```C
if(true == boolVar)
```

Why? Cause compilers are [unpredictable](https://porzechowski.github.io/blog/blog/software/security/post-software-countermeasures/#hamming-distance)

And instead of 0 or 1, variable memory can be equal to 2.

## NASA's 10 Rules for Developing Safety-Critical Code

A 10 good additional techniques comes from NASA:

The ten rules are:

    1. Avoid complex flow constructs, such as goto and recursion.

    2. All loops must have fixed bounds. This prevents runaway code.

    3. Avoid heap memory allocation.

    4. Restrict functions to a single printed page.

    5. Use a minimum of two runtime assertions per function.

    6. Restrict the scope of data to the smallest possible.

    7. Check the return value of all non-void functions, or cast to void to indicate the return value is useless.

    8.Use the preprocessor sparingly.

    9. Limit pointer use to a single dereference, and do not use function pointers.
    
    10. Compile with all possible warnings active; all warnings should then be addressed before release of the software.

1. Input Validation: Checking user inputs or external data to ensure they conform to expected formats and ranges, preventing buffer overflows or unexpected behavior.

2. Error Handling: Implementing error-checking code to handle situations where functions or operations fail. This may involve returning error codes, using `errno`, or throwing exceptions.

3. Memory Management: Properly allocating and deallocating memory to avoid memory leaks, buffer overflows, and undefined behavior.

4. Boundary Checking: Ensuring that arrays and data structures are accessed within their valid bounds to prevent memory corruption.

5. Assertions: Using `assert` statements to validate assumptions and detect unexpected conditions during debugging.

6. Comments and Documentation: Providing clear comments and documentation to explain code behavior, assumptions, and constraints.

7. Defensive Coding Patterns: Applying well-established coding patterns and practices that enhance reliability, such as avoiding global variables, using const correctness, and avoiding magic numbers.