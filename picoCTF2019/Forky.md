# 🚀 Forky

- **📛 Challenge Name:** Forky
- **🎯 Difficulty:** Hard  
- **🔗 Challenge:** [View on PicoCTF](https://play.picoctf.org/practice/challenge/24?category=3&originalEvent=1&page=2)  
- **🐧 File Type:** Linux ELF Binary  


## Introduction
In this post, we will walk through solving the `Forky` challenge and use Ghidra and cutter to assist us in reverse-engineering (SRE) the challenge to uncover the flag.

## My Experience
New challenge called `Forky`. It reminds me of the game `Forky`. Let's take a look at the question.

<hr/>
In this program, identify the last integer value that is passed as parameter to the function doNothing().
<hr/>

Let's download it and use the `file` command in Linux to check what type of file this is. 
Let’s check the `file` command output below:

```console
remnux@remnux:/home/remnux/Downloads$ file vuln
/home/remnux/Downloads/vuln: ELF 32-bit LSB shared object, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=836c8d5ecaad6d64f4a358cf73d060d0c5050e87, not stripped

```


### Ghidra
```c

/* WARNING: Function: __x86.get_pc_thunk.bx replaced with injection: get_pc_thunk_bx */
/* WARNING: Unknown calling convention -- yet parameter storage is locked */

int main(void)

{
  int *nothingArg;
  
  nothingArg = (int *)mmap(0,4,3,0x21,0xffffffff,0);
  *nothingArg = 1000000000;
  fork();
  fork();
  fork();
  fork();
  *nothingArg = *nothingArg + 0x499602d2;
  doNothing(*nothingArg);
  return 0;
}
```

![Cutter main function Forky challenge](./image/Cutter-Forky.png)

```c
#include <stdio.h>

int main() {
    int a = 0xd4faf720;
    printf("%d\n", a);      
    return 0;
}

```


```console
remnux@remnux:/tmp$ gcc -o main ./main.c && ./main
-721750240
```


The flag is:
```
PICOCTF{-721750240}
```
