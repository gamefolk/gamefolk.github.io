---
layout: default
title: Gamefolk's Guide to GBDK
---

# Gamefolk's Guide to GBDK

## Preface

[GBDK] is a cross-platform compiler from C to Z80 assembly for the Nintendo
GameBoy. This tutorial assumes that you are comfortable with the command line
and the C programming language.

## Installation
Before beginning, please ensure that you have installed the toolkit
properly from the link above. There is also an [AUR package] available for Arch
Linux users.

## Gotchas

One of the main issues with GBDK is its toolchain. The compiler, lcc, doesn't
often emit errors during compilation, but the backed, SDCC, will fail miserably
when presented with the bad code. This can be very frustrating, as your program
will compile correctly yet fail. Here are a few issues that you might run into
when using GBDK:

### C99 Incompatibility

When using lcc, you should write strict C89 (also known as ANSI C). While lcc
will accept many C99 constructs and GNU extensions, SDCC will not compile them
properly. This could cause parsing errors, link errors, or even segmentation
faults. Some limitations of C89 are as follows:

  * No inline variable declarations. Variables should be declared at the top of
    the function in which they reside.
  * No C++ style (`for (int i = 0;)`) for loops. Declare the index variable
    before the loop.
  * No C++ style `//` comments. Delimit all comments with `/*` and `*/`.

I recommend using another C compiler such as gcc or clang to compile your code
before compiling it with lcc. Consider using you are using another compiler to
check your code before sending it lcc, I recommend using the `-std=c89` and
`-pedantic` (or `-pedantic-errors`) to help catch bugs that stem from compiling
C99 with lcc.

*Note:* your compiler may have trouble compiling GBDK headers such as `gb/gb.h`
and complain about implicit declarations of GBDK library functions. Add the
following lines after the initial includes in that file to silence those errors.

```c
#ifndef __LCC__
#define NONBANKED ;
#endif
```

### External Linkage

Avoid using the `extern` keyword. SDCC does not implement external linkage
correctly, and thus will compile incorrect code and fail silently.

## Testing

You should use as accurate an emulator as possible when testing your GBDK
program. I recommend [BGB]. Although it is a native Windows program, it runs
nicely under WINE. Many other documents recommend no$gmb, but this emulator does
not accurately reflect how GameBoy hardware will run your program.

[GBDK]: http://gbdk.sourceforge.net
[BGB]: http://bgb.bircd.org
[AUR package]: http://aur.archlinux.org/packages/gbdk
