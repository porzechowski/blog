---
title: "Be careful with bool in C"
date: 2022-12-16
categories:
  - blog
  - software
tags:
  - C
  - asm
  - gcc
  - clang
  - msvc
  - bool
toc: true
toc_label: "Table of Contents"
toc_icon: "cog"
---

# Why you should be careful with *bool* in C ?

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
`var!` will be printed out. However what would happen, if for some reason the bool memory space would be altered? This is not an unlikely even and can be caused by a bit flip or programming error.
We can simulate it with following code:

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

What will be printed out this time? This is an undefined behavior and depends on the compiler used.

Let's check few compilers, all of them are set not to optimize our code:

## gcc 12.2 x86-64:

The output is:

`true!`

`false!`

We can access both if statements with a single bool variable!
So let's take a look at our assembly (I added comments to explain whats going on).

Assembly of:
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

Still far from perfect.

However the MSVC compilers seems to handle this correctly.

# Solution

One thing that comes to mind is to replace *bool* with variable like *uint8_t* and declare *true* and *false* on our own.

So our code would look like this:
```C
#include "stdio.h"
#include "string.h"
#include "stdint.h"

const uint8_t false_value = 0U;
const uint8_t true_value = 0U;

int main() {

    uint8_t var = false_value;
    memset(&var, 0x1, sizeof(var));

    if (true_value == var) {
        printf("true! \n");
    }

    if (false_value == var) {
        printf("false! \n");
    }
}
```
