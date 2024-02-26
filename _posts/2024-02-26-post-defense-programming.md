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

DP is a set of techniques and guidelines that aims to reduce the number of problems you might encounter when developing software.
Bad things happen and the causes are different. We might do a typo, someone might pass a NULL pointer when he shouldn't, there might be a glitch in hardware or error in documentation.

Defensive programming aims to prevent and mitigate faults and errors that will arise sooner or later.
As everything it comes with a cost. You will need to invest time and complexity of your code might increase.

Defensive programming is a wide set of techniques, so I will probably miss few.
Now let me describe some techniques that are relevant when talking about defensive programming for embedded systems.

## Don't trust external code

Fist one is old and most of you know are probably bored to death with hearing about it. The famous check against `NULL` pointer. However I want to expand this rule and say that anything handled to us by some external entity cannot be trusted and shall be verified before being used.

The simplest example is good old
```C 
if(ptr != NULL)
```

But this rule also applies to the boundary checking. Ensure that your data structures are accessed within valid bounds. So if you have and array and and index it's worth checking that index is within array size.

Also, if we take more complex input, like a frame, the same rule still applies.

Consider following frame

```C
[Header | Content | CRC]
```

We mustn't touch `Header` or `Content` before we verify that CRC is correct. Sounds simple but saves a lot of trouble when we find out that CRC is incorrect in the middle of processing data from the frame.

## Clear data after usage

This one is about good manners. 

**CLEAN AFTER YOURSELF!**

If you are using some data, and you no longer need it, the good thing to do is to clear it from the buffer.

Thanks to that, the data won't be reused by a mistake or leaked.

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

I described it [here](https://porzechowski.github.io/blog/blog/software/security/post-software-countermeasures/#hamming-distance), so I won't repeat myself here.

## Canaries values

It sucked to be a canary in a mine some time ago.
If there was something bad in the air you would be first one to know.

Software engineers borrowed this beautiful term to name a technique that allows us to detect altered buffers.

The idea is very simple.
We place known values around a buffer. Let's say our canary value is 0xAA. All we do is to place this value around our buffer in memory:

```C
[0xAA, 0xAA, 0xAA | BUFFER | 0xAA, 0xAA, 0xAA]
```

Then we check the integrity of these values in some intervals.
For example it could be another thread that evey 100ms checks if our canaries values didn't change.
So if the thread detects following situation:

```C
[0xAA, 0xAA, 0xAA | BUFFER | 0x00, 0xAA, 0xAA]

```

`0x00` tells us that there was buffer overflow or the buffer was overwritten and we can react accordingly.

## Be precise

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


## Function should fail as default

For this one take a look at the following code:

```C
bool ValidateFrame(uint8_t command, uint32_t time) {
    if((0U <= command) && (4U >= command)) {
        return false;
    }

    if((time - previousTime) > 100U){
        return false;
    }

    return true;
}
```

Function 'ValidateFrame' is trying to validate command and time - we don't care why.
If any of the parameters is in the wrong boundary, function will return `false`.
But if by stupid typo accident we removed or modify one if statement, the function might return 'true' instead of 'false'... I know this sounds far fetched and unlikely... but it happens.

I could also argue that single return statement improves readability and makes debugging easier.

## const is a default state

One of the things I like about Rust is that variables are immutable by default.
In C and C++ we have the opposite, so it's up to us to take care of that.
Generally it is always a good idea to make variable 'const' and "un-const" it if needed.

# NASA's 10 Rules for Developing Safety-Critical Code

This one is lightly connected with a topic.
NASA introduced few rules that helps us keep out code easy to analyze and maintain. You can think of this as a microMISRA :)

The ten NASA rules are:

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

Why do I mention this rules? 

I strongly believe that best and super-uber effective technique is to keep your code simple as fuck. Don't try to impress others with how smart you are. If code is simple and readable you have a great chance to spot a bug or shady behavior before the run-time.
And when bad things happen, simple code is easier to debug and analyze.
Keep in mind that embedded in not a web development. Sometimes to update a software you need to drive for 6 hours or fly for 12. Trust me I've been there, so after a few trips you will put more effort to spot problems early.

# Additional remarks

I know that for every rule in this post, we could give a few good examples why the opposite is better. Software development is not a science, there are no universal rules that applies to every scenario.

I'm also fully aware that good tests would probably catch most of the problems, but multilayer approach is better than single layer one, so IMO it's better to have tests **and** some defensive coding practices.

Also, if we care about safety and/or security I find this guidelines to be a good starting point when discussing project details.

Let me know if you agree or disagree.
I really want to know your opinions on the subject.