## 1 -> 2

### Writeup

I listed the home directory and it contained a binary called "check". Running it suggested that it expected a password. I suspected the password would be stored in the program, so I analysed the binary with radare2. In the main function, we can see that there are 4 strings stored, being "sex", "secret", "god", "love". Interestingly, only "sex" is stored to a marked address, being "s2".

```
0x0804091f3        c745e07365..    mov dword [s2], 0x786573
0x0804091fa        c745e07365..    mov dword [var_13h], 0x07265363 's 'secr'
0x080a49201        c745e07265..    mov dword [var_10h], 0x746572
0x080a49208        c745e07365..    mov dword [var_1ch], 0x646167 ; 'god'
0x080a4920f        c745e06fcf..    mov dword [var_18h], 0x65766f6c ; 'love'
```

Further down we also see that strcmp is called comparing the value at the address marked "s2" to an "s1" (s1 is where the user input is stored). This suggests that the password check is to see if the string is "sex" or not.

```
0x0868d49251       c8dafdfff      call  sym.imp.strcmp
                   int strcmp(const char *s1, const char *s2);
```

I reran the program and entered "sex", which then dropped me into a shell as leviathan2. From there I looked at the value stored in "/etc/leviathan_pass/leviathan2" to find the flag.

<details>
<summary>Flag</summary>
ERJ9jTYWXE
</details>