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

>TODO add another example

## yoda conditions

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
if((10 < number) && (20 > number)) {
  //do something
}
```

For me it is a bit more cognitively challenging... but IMHO it is worth it.


## MISRA

MISRA and Defensive Programming has common techniques but are aiming for different things (? - zweryfikowac)
## C++

While usually there is no difference between variable == value and value == variable, and in principle there shouldn't be, in C++ there sometimes can be a difference in the general case if operator overloading is involved. For example, although == is expected to be symmetric, someone could write a pathological implementation that isn't.

Microsoft's _bstr_t string class suffers from an asymmetry problem in its operator== implementation

## Safe constants
## canaries values
## Clear data after usage
## All data is tainted until proven otherwise
you need to verify a data to use it
## reduce complexity and depth
##  tests?
testing can prove the presence of bugs, but not their absence
## true == true false == false, a nie !=
## NASA's 10 Rules for Developing Safety-Critical Code
The Power of 10: Rules for Developing Safety-Critical Code

The Power of 10 Rules were created in 2006 by Gerard J. Holzmann of the NASA/JPL Laboratory for Reliable Software.[1] The rules are intended to eliminate certain C coding practices which make code difficult to review or statically analyze. These rules are a complement to the MISRA C guidelines and have been incorporated into the greater set of JPL coding standards.[2] 

Holzmann is known for the development of the SPIN model checker (SPIN is short for Simple Promela Interpreter) in the 1980s at Bell Labs. This device can verify the correctness of concurrent software, since 1991 freely available. 

The ten rules are:[1]

    Avoid complex flow constructs, such as goto and recursion.
    All loops must have fixed bounds. This prevents runaway code.
    Avoid heap memory allocation.
    Restrict functions to a single printed page.
    Use a minimum of two runtime assertions per function.
    Restrict the scope of data to the smallest possible.
    Check the return value of all non-void functions, or cast to void to indicate the return value is useless.
    Use the preprocessor sparingly.
    Limit pointer use to a single dereference, and do not use function pointers.
    Compile with all possible warnings active; all warnings should then be addressed before release of the software.

NASA’s 10 rules for developing safety-critical code are:

    Restrict all code to very simple control flow constructs—do not use goto statements, setjmp or longjmp constructs, or direct or indirect recursion.
    Give all loops a fixed upper bound.
    Do not use dynamic memory allocation after initialization.
    No function should be longer than what can be printed on a single sheet of paper in a standard format with one line per statement and one line per declaration.
    The code's assertion density should average to minimally two assertions per function.
    Declare all data objects at the smallest possible level of scope.
    Each calling function must check the return value of nonvoid functions, and each called function must check the validity of all parameters provided by the caller.
    The use of the preprocessor must be limited to the inclusion of header files and simple macro definitions.
    Limit pointer use to a single dereference, and do not use function pointers.
    Compile with all possible warnings active; all warnings should then be addressed before the release of the software

    