---
title: "Funny behavior of bool in C"
date: 2023-12-16
categories:
  - blog
  - software
tags:
  - C
  - asm
  - gcc
  - clang
  - TCC
  - Tiny C Compiler
  - MSVC
  - bool
  - compilers
  - Hamming
  
toc: true
toc_label: "Table of Contents"
toc_icon: "cog"
---

# Why should you be careful with *bool* in C ?

The case discussed here is architecture/compiler specific, so it might work a bit different in your current environment. Still I strongly encourage to play with it a bit.

## Problem
*Bool* has been added to C99, over 22 years ago. The rules are quite simple:
0 is a `false`, 1 is `true` and to use it we should include `stdbool.h` header. So far, so good, so simple.

Consider following code:

```c
#include "stdbool.h"
#include "stdio.h"
#include "string.h"

int main() {

    bool var = false;

    if (true == var) {
        printf("true! \n");
    }

    if (false == var) {
        printf("false! \n");
    }
}
```
Nothing tricky, as we can predict:
`false!` will be printed out. However what would happen, if for some reason the bool memory space would be altered? This is not an unlikely even and can be caused by a hardware glitch or programming error. So we will use ```memset``` to alter out bool variable.

Let's simulate it with following code:

```c
#include "stdbool.h"
#include "stdio.h"
#include "string.h"

int main() {

    bool var = false;
    memset(&var, 0x2, sizeof(var));

    if (true == var) {
        printf("true! \n");
    }

    if (false == var) {
        printf("false! \n");
    }
}
```

What will be printed out this time? 
Well... it depends... every compiler might handle this differently and you might be surprised by the outcome.

Let's check few compilers. All of them are set with, -O0 not to optimize our code:

## gcc 12.2 x86-64:

The output is:

`true!`

`false!`

We can access both if statements.. and this should not happen!

Before we take a look at the assembly consider that after preprocessing our function will look like this:
```c
int main() {

    _Bool var = 0;
    memset(&var, 0x2, sizeof(var));

    if (1 == var) {
        printf("true! \n");
    }

    if (0 == var) {
        printf("false! \n");
    }
}
```

```true``` and ```false``` will be replaced by ```0``` and ```1```

So let's take a look at our assembly (I added comments to explain whats going on).

### Assembly
```c
if (true == var)
```
Will look like this:
```asm
movzx   eax, BYTE PTR [rbp-1] 
test    al, al 
je      .L2
```
Line bye line it means:
1. Put var in `eax` register
2. `al` is the least significant byte of `eax`, `test` is a bitwise AND, so what we do here is:
   
    `if (al & al == 0)` if this is true the `zero flag` will be set to 1.
3. If `zero flag` is set we jump to section .L2 (printf inside if in this case)
   
   So if our bool is set to `true` (or we force the memory there to be anything but 0) it will result in printing `true!`

Now lets examine the assembly of:
```c
if (false == var)
```

```asm
movzx   eax, BYTE PTR [rbp-1]
xor     eax, 1
test    al, al
je      .L3
```

And the meaning of this is:

1. Put var in `eax` register
2. We perform *XOR* on `eax` register. So if `eax` is holding the value od 1 it will be set to 0, in any other case its value will be different than 0.
3. As before we test `al` against `al` and if it is 0, we set the  `zero flag`.
4. If `zero flag` is set we jump to section .L3 (printf inside if in this case)
   
So if our bool is set to anything but `true` we will reach insides of second `if` statement.
And this is why any memory value other than 0 and 1 can cause us some trouble.
Imagine debugging such scenario!

The good news is that other compilers also can bring us the tears of pure joy!
## clang 15.0.0 x86-64

so for *clang* out output is: `false!`

however if I modify the program like this:
```c
memset(&var, 0x3, sizeof(var));
```

output is: `true!`


and for:
```c
memset(&var, 0x4, sizeof(var));
```

output is: `false!`

You can see the pattern, every odd number will give us `true!`
and every even number will give us: `false!`.

Still far from expected...

But there is a compiler that works quite good!

## Tiny C Compiler 0.9.27 x86-64

Finally some good news!
If we set anything but 0 or 1, we will not enter any if statement.
We can say that this behaves as expected. 


## Solution

However quite often we are forced to work with certain type of compiler like ```gcc``` or ```clang```.

If we we might want to use out own definition of bool.

Let's rewrite our program and make few changes.

1. We change type of our variable ferom ```bool``` to ```uint8_t```
2. We will redefine what ```true``` and ```false``` are

```c
#include "stdio.h"
#include "string.h"
#include "stdint.h"

#define MY_FALSE 0xA5U
#define MY_TRUE 0x5AU

int main() {

    uint8_t var = false;
    memset(&var, 0x3, sizeof(var));

    if (true == var) {
        printf("true! \n");
    }

    if (false == var) {
        printf("false! \n");
    }
}
```

With implementation like this all mentioned **compilers behaves correctly**.

But.. you might wanter why I used ```0xA5U``` and ```0x5AU```.

It is a good practise to have constant value with high ```Hamming distance``` for safety and security reasons. I'll probably write about it later in separate post.

The ```U``` only emphasize the fact that out value is unsigned, so we will avoid implicit cast, since ```0xA5``` is a signed value assigned to and unsigned variable ```uint8_t```. 

## The end

And I think that all I wanted to share. If you encounter another funny behavior please let me know :) 