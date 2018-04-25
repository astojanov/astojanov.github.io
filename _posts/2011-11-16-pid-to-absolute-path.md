---
layout:     post
title:      "Mac OS X: Resolve absolute path using process’ PID"
date:       2011-09-26 10:00:00 +0200
categories: blog
---

Mac OS X does not have a `/proc` file system. Therefore resolving the
absolute path and other process informations have to be obtained in a
different fashion. OS X has the `libproc` library, which can be used to
gather different process information. In order to find the absolute
path for a given PID, the following code can be used:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <errno.h>
#include <libproc.h>

int main (int argc, char* argv[])
{
	pid_t pid; int ret;
	char pathbuf[PROC_PIDPATHINFO_MAXSIZE];

	if ( argc > 1 ) {
		pid = (pid_t) atoi(argv[1]);
		ret = proc_pidpath (pid, pathbuf, sizeof(pathbuf));
		if ( ret <= 0 ) {
			fprintf(stderr, "PID %d: proc_pidpath ();\n", pid);
			fprintf(stderr,	"    %s\n", strerror(errno));
		} else {
			printf("proc %d: %s\n", pid, pathbuf);
		}
	}

	return 0;
}
```

I was surprised not to be able to easily find documentation for pid
information, which in my opinion is quite trivial to be implemented. I
had to go through the code of `lsof` to find the implementation for
reading `proc` txt section and finally learn about `proc_pidinfo`.
`proc_pidpath` is just a wrapper for `proc_pidinfo` to resolve the path.