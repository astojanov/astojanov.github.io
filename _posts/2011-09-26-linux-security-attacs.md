---
layout:     post
title:      "Prepare Linux for security attacks"
date:       2011-09-26 10:00:00 +0200
categories: blog
---

In order to be able to perform simple buffer overflow attacks, Linux should be
stripped from all security mechanisms enabled by default. In this case it is the
**ASLR** and **ExecShield**. The first one is address space randomization to randomize
the starting address of heap and stack. In order a buffer overflow to be
successful, guessing the address is essential. The ExecShield is a protection
mechanism that disallows executing any code that is stored in the stack.

ASLR:

```
sudo echo 0 > /proc/sys/kernel/randomize_va_space
```

or, in oder to have it permanently on every reboot, just add the following lines in `/etc/sysctl.conf`

```
kernel.exec-shield = 1
```

In some linux distributions, we should be cereful to avoid the stack smashing. GCC uses
protection such that it emit extra code to check for buffer overflows. Taken directly
from the GCC documentation, having the option `-fstack-protector` enabled, the compiled
code will have the stack smashing protection by:

```
This is done by adding a guard variable to functions with vulnerable objects.
This includes functions that call alloca, and functions with buffers larger
than 8 bytes.The guards are initialized when a function is entered and then
checked when the function exits.  If a guard check fails, an error message
is printed and the program exits.
```

To make sure that this is disabled, we compile the target programs (just for testing) with:

```
-fno-stack-protector
```

<br />