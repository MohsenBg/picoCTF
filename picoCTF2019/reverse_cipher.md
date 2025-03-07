# reverse_cipher

## Introduction
In this post, we will walk through solving the `reverse_cipher` challenge and use Ghidra to assist us in reverse-engineering (SRE) the challenge to uncover the flag.

## My Experience
Let's first check what question says and see if we can find any good information
<hr/>
We have recovered a binary and a text file. Can you reverse the flag.
<hr/>

Let's download two files and inspect their contents.
We have two files: one called rev and the other rev_this. The question mentions that one is a binary file and the other is a text file. To determine their types, I used the file command in Linux. You can also use other tools or other programs like `hexdump`, or `binwalk` to analyze the files more easily.

```bash
/home/remnux/Downloads/rev: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=523d51973c11197605c76f84d4afb0fe9e59338c, not stripped
```
The output shows we have an ELF file, so we can use REMnux as our environment because it provides useful tools and programs like Ghidra.

Let's open Ghidra and attempt to reverse-engineer (RSE) it. First, we’ll locate the main function and inspect its contents (I've cleaned up some Ghidra code and added comments to make the analysis easier to follow).

```C
void main(void)
{
  size_t _result_read_flag;
  char flag [23];
  char local_41;
  int res_flag_read;
  FILE *rev_file;
  FILE *flag_file;
  uint j;
  int i;
  char tmp_char;
  
  flag_file = fopen("flag.txt","r");
  rev_file = fopen("rev_this","a");
  if (flag_file == (FILE *)0x0) {
    puts("No flag found, please make sure this is run on the server");
  }
  if (rev_file == (FILE *)0x0) {
    puts("please run this on the server");
  }
  _result_read_flag = fread(flag,0x18,1,flag_file);
  res_flag_read = (int)_result_read_flag;
  if ((int)_result_read_flag < 1) {
                    /* WARNING: Subroutine does not return */
    exit(0);
  }
  for (i = 0; i < 8; i = i + 1) {
    tmp_char = flag[i];
    fputc((int)tmp_char,rev_file);
  }
  for (j = 8; (int)j < 23; j = j + 1) {
    if ((j & 1) == 0) {
      tmp_char = flag[(int)j] + 5;
    }
    else {
      tmp_char = flag[(int)j] + -2;
    }
    fputc((int)tmp_char,rev_file);
  }
  tmp_char = local_41;
  fputc((int)local_41,rev_file);
  fclose(rev_file);
  fclose(flag_file);
  return;
}
```
As you can see in the main function provided by Ghidra, the binary attempts to open the flag.txt file and checks if it exists. If the file is not found, it outputs the following error:
```c
puts("No flag found, please make sure this is run on the server");
```

Next, it tries to open or create the rev_file, where it will later append text. If something goes wrong during this process, it outputs the following error:
```c
puts("please run this on the server");
```

The program first attempts to read the flag.txt file and store its content in a char array of length 23. I renamed that variable to flag in Ghidra.

Next, it retrieves the first 8 characters of the flag and appends them to the rev_file. If we check the first 8 characters in the rev_this file provided, they are `picoCTF{`.

Then, a loop is used to process characters 8 to 23 in the flag.

```c
for (j = 8; (int)j < 23; j = j + 1) {
  if ((j & 1) == 0) {
    tmp_char = flag[(int)j] + 5;
  }
  else {
    tmp_char = flag[(int)j] + -2;
  }
    fputc((int)tmp_char,rev_file);
  }
```
In summary, if the index of the character in the array is even, it adds 5; if the index is odd, it subtracts 2, then writes the modified character to rev_this.

After the loop, the program closes the files and ends. So, let’s reverse-engineer (RSE) this.

We have the content of rev_this provided in the question:

```text
picoCTF{w1{1wq84fb<1>49}
```

As we seen earlier, the first 8 characters remain unchanged `picoCTF{`, so we only need to reverse-engineer characters 8 to 23. Let's write Python code to do that

```python
def gen_flag():
    rev_flag = list("picoCTF{w1{1wq84fb<1>49}")
    for i in range(8, 23):
        if (i & 1) == 0:
            rev_flag[i] = chr(ord(rev_flag[i]) - 5)  
        else:
            rev_flag[i] = chr(ord(rev_flag[i]) + 2) 
    return "".join(rev_flag) 

print(gen_flag())
```

My Python code is simple: it checks if the character’s index is odd, subtracts 5; if even, it adds 2. This reverses the logic we saw in the Ghidra code.

Let’s run it and get the flag.


The flag is:
```
picoCTF{r3v3rs36ad73964}
```