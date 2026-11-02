## 3 -> 4

### Writeup

I started by running the "./level3" program in the home directory. It prompted me for a password, so I thought a similar vulnerability to Leviathan 1->2 may be present. Using ltrace, I observed that the binary compared my input to "snfprintf", so I reran the program and typed that in as the password. This gave me a shell as leviathan4 which I used to read /etc/leviathan_pass/leviathan4, revealing the flag.

```
leviathan3@leviathan:$ ltrace ./level3
_libc_start_main["./level3"] <unfinished ...>
sprintf("hmnpfr", "kakaba")                        = -1
printf("%s\n")                                      = 20
fgets(Enter the password> meow
, 256, 0x7f7fa580)                          = 0xffffd26c
strcmp("meown", "snfprintf")                       = -1
printf("bzzzzzzzzzzip. WRONG\n")                   = 19
+++ exited (status 0) +++
leviathan3@leviathan:$ ./level3
Enter the password> snfprintf
You've got shell!
leviathan4
leviathan4
$ cat /etc/leviathan_pass/leviathan4
XiyBbRwAPt
$ []
```

<details>
<summary>Flag</summary>
XiyBbRwAPt
</details>