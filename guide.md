---
layout: default
title: GameFolk's Guide to GBDK
---

# GameFolk's Guide to GBDK

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

### Beware C99 incompatibilities, use C89 if possible

When using lcc, you should write strict C89 (also known as ANSI C). While lcc
will accept many C99 constructs and GNU extensions, SDCC will not compile them
properly. This could cause parsing errors, link errors, or even segmentation
faults. Some limitations of C89 are as follows:

  * No inline variable declarations. Variables should be declared at the top of
    the function in which they reside.
  * No C++ style (`for (int i = 0;)`) for loops. Declare the index variable
    before the loop.
  * No C++ style `//` comments. Delimit all comments with `/*` and `*/`.

Consider using another C compiler such as gcc or clang to compile your code
before compiling it with lcc. I recommend using clang with the `-std=c89` and
`-pedantic` (or `-pedantic-errors`) flags to help catch bugs that stem from
compiling C99 with lcc.

*Note:* your compiler may have trouble compiling GBDK headers such as `gb/gb.h`
and complain about implicit declarations of GBDK library functions. Add the
following lines after the initial includes in that file to silence those errors.

```c
#ifndef __LCC__
#define NONBANKED ;
#endif
```

### Static data should be declared `const`

Due to inefficiencies in how SDCC implements loads from ROM, static data such as
tiles and music should __always__ be declared `const`. If not, the data gets
copied into RAM *and* takes up about six times the ROM space.

### Do not use external linkage

Avoid using the `extern` keyword. SDCC does not implement external linkage
correctly, and thus will compile incorrect code and fail silently.

### Test on an accurate emulator

You should use as accurate an emulator as possible when testing your GBDK
program. I recommend [BGB]. Although it is a native Windows program, it runs
nicely under WINE. Many other documents recommend no$gmb, but this emulator does
not accurately reflect how GameBoy hardware will run your program.

[GBDK]: http://gbdk.sourceforge.net
[BGB]: http://bgb.bircd.org
[AUR package]: http://aur.archlinux.org/packages/gbdk

### Converting images

Two programs that convert images files to GBDK-ready C code are [MegaMan_X's GameBoy ToolKit](http://www.yvan256.net/projects/gameboy/#gbtk) (GBTK) and [PCX2GB] (http://www.yvan256.net/projects/gameboy/#pcx2gb). GBTK allows one to choose between several scaling and dithering algorithms and to visually manipulate the resulting image, but does not optimize for empty space/ repeated tiles. PCX2GB is relatively opaque but produces much smaller files. One solution to these limitations is to use both programs:

* Open your image with GBTK and manipulate it until you are satisfied. 
* Export the result as a BMP file.
* Convert the BMP to a PCX with GIMP:
    * Image -> Mode -> Indexed
    * Generate optimum palette -> Convert
    * File -> Export As -> ZSoft PCX image
* Run PCX2GB on your DOS platform of choice:
    * `pcx2gb o d myimage.pcx myimage.c myimage.map`

This will produce two files, one containing the image tile data ( `char tiledata[]` ) and the other the tile map ( `char tilemap[]` ). You can combine these arrays into one C source file (don't forget to replace the `char`s with `const UBYTE`s!) and then display the image like this:

    disable_interrupts();
    DISPLAY_OFF;
    LCDC_REG = 0x01;
    BGP_REG = 0xE4;
    set_bkg_data(0, 255, tiles);
    set_bkg_tiles(0, 0, 20, 18, map);
    DISPLAY_ON;
    enable_interrupts();
