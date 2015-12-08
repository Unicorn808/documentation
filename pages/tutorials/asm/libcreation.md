---
layout: page
title: Creating C/ASM libraries
permalink: /tutorials/asm/libcreation.html
---

If you are loking to create an ASM or C library, this is the place! Simply follow the simple steps below.

# Creation

Navigate into your CE C toolchain installation directory. Inside the *shared_libs* directory, you will find another directory called *template*. 

Create a copy of the *template* directory, and rename it to whatever your library does. For example, if it performs File I/O, call it something like *fileio*

Inside of your newly created directory, you will find the *MakeFile*. Open this in your editor, and change the following lines:

```
LIB = TEMPLTE
SOURCES = template.asm
```

**LIB** is the name of the library AppVar that will exist on your calculator. Try to make this somewhat descriptive.

**SOURCES** This countains the name of the files you wish to use for your library. If you rename the *template.asm* in your directory, you **must** also update it here.

Now, close the *MakeFile*, and open the *template.asm* file, or whatever you have renamed it to. It should look like this:

```
#include "../include/relocation.inc"

 .libraryName		"TEMPLTE"
 .libraryVersion	1
 
 .function "sample",_sample
 
 .beginDependencies
 .endDependencies

;== Start Library Code

_sample:
 ret

;== End Library Code

 .endLibrary
 ```
 
 Now, the very first thing you want to do is change this line:
 
 ```
 .libraryName		"TEMPLTE"
 ```
 
 **Make sure TEMPLTE is changed to the name of your AppVar that you set before in the *LIB* part of the Makefile!**
 
 The next line, ```.libraryVersion``` tells us which version this library is. If you add a function, it is nessasary to update the version. If you simply change a function, or add more features to it, it is not nessasary to update the version.
 
 **Note: Newer versions of libraries are *always* expected to be compatible with previous versions.**
 
 The lines containing ```.beginDependencies``` and ```.endDependencies``` is where you can use other libraries that you may be dependent on. More information can be found below, under the **Dependencies** heading.
 
 
# Programming

Great, now you are all set up! Let's start with how to program a library. First, note that libraries are a little different than writting straight assembly programs.

### Functions

The syntax for creating functions is as follows:

```
.function "{ function name }",{ source label }
```

To insert a new function into your library, just insert a new line right below the previous function. So if you had a library with 3 functions, it would look something like this:

```
#include "../include/relocation.inc"

 .libraryName		"TEMPLTE"
 .libraryVersion	1
 
 .function "sampleFunc0",_sample0
 .function "sampleFunc1",_sample1
 .function "sampleFunc2",_sample2
 
 .beginDependencies
 .endDependencies

;== Start Library Code

_sample0:
 ; does things
 ret

_sample1:
 ; does things
 ret

_sample2:
 ; does things
 ret

;== End Library Code

 .endLibrary
```

Note that *{ function name }* is the name of the function that you will use in your C/ASM code later, and the *{ source label }* is the location in your library code where the function is.

**Important note**: Because libraries are posistion independent, this means that any usage of a ```call```, ```jp```, or absolute location **must** be relocated. The following shows how:

```
;== Start Library Code

_sample0:
 .r jp _sample2

_sample1:
 .r call _sample0
 jr _sample0

_sample2:
 ld de,$000000
 .r2	ld (libdata),de
 .r  ld hl,libtext
 jp _PutS

libtext:
 .db "Test",0
libdata:
 .dl 100
 
;== End Library Code
```

Note the ```.r``` prefix on instructions. This tells the assembler to add the address to the relocation table, so that is can be posistion independent. Do this when you make calls or absolute jumps within your program. Note that relative jumps **do not** require this.
Its counterpart, ```.r2```, is used when the instruction uses ```ix```, or is { ```ld (imm24),de``` or ```ld (imm24),bc``` }. This is important, so be sure to watch out for it.

In C, functions recieve arguemnts from the stack in reverse order. Say you call from C a function such as this:

```sample(10, 20, 30);```

Then you can pull arguments out like this in assembly:

```
_sample:
 push ix
  ld ix,0
  add ix,sp
  ld hl,(ix+6)  ; hl = 30
  ld de,(ix+9)  ; de = 20 
  ld bc,(ix+12) ; bc = 10
 pop ix
 ret
```

Here's a helpful table: (Low to High)

Type         |  Size         | Memory
------------ | ------------- | ------------- 
char         | 3 bytes       | xx ?? ??
short        | 3 bytes       | xx xx ??
int          | 3 bytes       | xx xx xx
long         | 6 bytes       | xx xx xx xx ?? ??
float        | 6 bytes       | xx xx xx xx ?? ??
double       | 6 bytes       | xx xx xx xx ?? ??
pointer      | 3 bytes       | xx xx xx

There you are! Now you should be able to write your library in no time at all.

# Assembling

Once you are finished writting your library, you will want to open a command shell in your library's source directory. Simply type the command ```make```, which will assemble your library and produce some files.

One file is a **.8xv**; your library's AppVar which is sent to the calculator.

Another is a **.asm**, which is the linkable file that contains the jump table for your library.

Another is a **.lst**, which is simply the listing file for your assembly source. This is primarily used for debugging purposes.

The file **relocation_table** is created if your library uses absolute relocations. You can delete it if you want.

A **.h** file is also generated. Find out more about it below:


# Creating your header file

When you assemble your library, a header file is automatically generated for you. However, it assumes that all function's return type is void, and recieves no arguments. It is up to you to change these prototypes yourself, in order to match how your library functions actually work.

Here is the type definitions (or you can use the uint{x}_t definitions from *stdint.h*):

```
char    8 bits
short   16 bits
int     24 bits
long    32 bits
float   32 bits
double  32 bits
```

# Finishing Up

Great, now your header file should be complete. Now for one of the harder parts. From your base C toolchain installation, copy your header file to *.\include\ce*. 

Then copy the newly created **.asm** containing the jump table for your library. into *.\include\ce\asm*.

Now, whenever you want to use your libraries functions in your program, simply do:

```
#include <template.h>
```

Where *template.h* is the name of the header file you copied.

# Some important notes

In order to use your library, you must have LibLoad installed, along with your library's binary.