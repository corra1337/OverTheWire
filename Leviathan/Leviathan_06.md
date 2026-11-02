## 6 -> 7

### Writeup

I ran the binary, and saw it requested a 4 digit number. When I inputed a random number, it said "wrong". This led me to believe that like Leviathan 3->4, the binary was doing a password comparison within the binary. To look for the value my input was being compared against, I did a static analysis on the binary using radare2.

```
0x0804921a         3945f4          cmp dword [var_ch], eax
0x0804921d         752b            jne 0x804924a
```

The key lines are these two. Above it a function call atoi() on argv[1] stores it's value in eax, so the lines shown in readable language are:

Compare user input to dword  
jump if they aren't equal

The most important piece of information here is that the password is stored in dword. Looking in the radare output, we can see that early on, "0x1bd3" gets stored in dword. This is probably our password.

Converted the hex into decimal (0x1bd3 = 7123) and feeding that into the program gives us a shell belonging to user leviathan7, which can be used to view the flag stored in /etc/leviathan_pass/leviathan7.

```
leviathan6@leviathan:~$ ./leviathan6 7123
$ whoami
leviathan7
$ cat /etc/leviathan_pass/leviathan7
3zrlkaPTfH
$ []
```

<details>
<summary>Flag</summary>
3zrlkaPTfH
</details>