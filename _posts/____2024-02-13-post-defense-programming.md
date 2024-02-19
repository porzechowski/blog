---
title: "Defense programming"
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
  - software
  - security
  - safety
  - yoda conditions
toc: true
toc_label: "Table of Contents"
toc_icon: "cog"
---

# Defense programming is it what?

The topic for today is defensive programming.

It is a set of techniques and guidelines that aims to reduce the number of bugs an errors in our code.

The defense programming has many faces. We might focus on security, safety

## yoda
not all the code goes through full testing at all times: there is always an edge case.

do agree 42 < $foo is harder to read than $foo > 42

if (bCondition = NULL)  // typo here
{
 // code never executes
}

if (NULL = bCondition) //  error -> compiler complains
{
 // ...
}



## MISRA

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

    