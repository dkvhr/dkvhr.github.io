---
title: "Solution of the initial picoGym pwn challenges"
date: 2024-07-31 00:00:00 -0300
categories: [pwn, writeup]
tags: [pwn, writeup]
math: true
mermaid: true
# image: //
#   path: /commons/devices-mockup.png
#   width: 800
#   height: 500
#   alt: Responsive rendering of Chirpy theme on multiple devices.
---

This is a writeup of the 17 easy binary exploitation challenges from picoGym.
You can find these challenges at [picoGym](https://play.picoctf.org/practice)\
Let's get started ^^

# buffer overflow 0

We are given this C file:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <signal.h>

#define FLAGSIZE_MAX 64

char flag[FLAGSIZE_MAX];

void sigsegv_handler(int sig) {
  printf("%s\n", flag);
  fflush(stdout);
  exit(1);
}

void vuln(char *input){
  char buf2[16];
  strcpy(buf2, input);
}

int main(int argc, char **argv){

  FILE *f = fopen("flag.txt","r");
  if (f == NULL) {
    printf("%s %s", "Please create 'flag.txt' in this directory with your",
                    "own debugging flag.\n");
    exit(0);
  }

  fgets(flag,FLAGSIZE_MAX,f);
  signal(SIGSEGV, sigsegv_handler); // Set up signal handler

  gid_t gid = getegid();
  setresgid(gid, gid, gid);


  printf("Input: ");
  fflush(stdout);
  char buf1[100];
  gets(buf1);
  vuln(buf1);
  printf("The program will exit now\n");
  return 0;
}
```

By performing a checksec, we get the following result:
```
    Arch:     i386-32-little
    RELRO:    Full RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      PIE enabled
```

Before getting started, I created a debugging flag on a file named flag.txt. That way we can make the program run locally.

If we take a look at the code, the flag.txt file will be read from and written into `char flag[FLAGSIZE]`. The only way of displaying the contents of the flag is by triggering the signal function. That means that if we trigger a segmentation fault, for example, then we will have the flag contents displayed.

Luckily, the program uses the `gets` function. It will write into the variable `char buf1[100]` anything we want without considering its size. If we take a look at the `vuln` function, we will see that it copies from the input argument (which will be the buf1) into a buffer with size 16.

So, if we overwrite the return address of the vuln function with some nonsense, that will definitely trigger a segmentation fault.

You can just put a lot of `A` characters to make sure that the return address gets overwritten with it. I put `AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA`. By looking at the program on the gdb we can see this:

![bof0](/assets/img/picogym-pwn-easy/bof0/bof0.png)

The program would crash as we have 'AAAA' as the return address and that is not valid. Running the program _outside_ gdb will make it go through the `sigsegv_handler` function, ultimately giving us the flag

flag: `picoCTF{ov3rfl0ws_ar3nt_that_bad_c5ca6248}`