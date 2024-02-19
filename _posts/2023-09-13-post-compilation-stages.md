---
title: "Compilation stages"
date: 2023-09-13
categories:
  - blog
  - software
tags:
  - C
  - C++
  - define
  - preprocessor
  - compilation
  - assembly
  - linking
toc: true
toc_label: "Table of Contents"
toc_icon: "cog"
---
![UC](https://github.com/porzechowski/blog/blob/master/assets/images/under_construction.webp?raw=true)
# How do we go from source code to binary file?

There are few stages that need to happen to turn our C/C++ source code to the running program.

# Preprocessing

First thing that happen is a stage called ***preprocessing*** which is performed by a... preprocessor, which resolves ***preprocessor directives*** (they begin with a `#`).
This happens line after line in each source file.
***preprocessor directives*** can be divided in the following way:

- Directives like `#includes`, `#defines` or `#line` or are replaced with a corresponding content.

- Preprocessor also decides witch part of code to process based in directives like: `#ifdef`, `#ifndef`, `#if`, `#endif`, `#else` and `#elif`

- `#error` - aborts compilation process when it's reached,

- `#pragma` - provides additional information for the compiler. This directives are platform and compiler specific. If directive is not supported by a certain compiler it is ignored and no error is generated.

Also during this stage comments are removed from a code.

The output of this stage is typically called a `.i` (for C), `.ii` (for C++) for each source file.

# Compilation

During this stage each `.i/.ii` file is parsed into assembly code - `.s` file.
Assembler code generated in this step is processor specific. During this stage syntax analysis is carried out, so errors and warnings will emerge.

# Assembly

Now from the assembly code from `.s` file, a file with a machine code is created - `.o` object file.

# Linking

At this point we have many object files `.o` with machine code inside - still not a proper executable. This is why we need a linker, program that produces executable by linking object files together. If we use any external library it will also be linked at this point. 

If all above steps go well, as a result we will obtain an executable file.
Depending on target platform our executable file will have different format and extension (like: `.exe`, `.elf`).

Full list can be found here: [List of executable files](https://en.wikipedia.org/wiki/Comparison_of_executable_file_formats)

<script src="https://utteranc.es/client.js"
        repo="[ENTER REPO HERE]"
        issue-term="pathname"
        theme="github-light"
        crossorigin="anonymous"
        async>
</script>