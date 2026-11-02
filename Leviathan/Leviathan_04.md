## 4 -> 5

### Writeup

Started by running "ls -la" and found a hidden directory called .trash that contained a binary. Running it printed out several binary numbers. I made a script to translate these numbers and it revealed the flag.

```
leviathan4@leviathan: $ ls -la
total 24
drwxr-xr-x  1 leviathan4 leviathan4  4096 Jun 24 15:01 .
drwxr-xr-x 150 root      root       4096 Jun 24 15:02 ..
-rw-r--r--  1 root       root       220 Feb 13 12:16 .bash_logout
drwxr-xr-x  1 root       root       4096 Jun 24 15:01 .profile
-rw-r--r--  1 root       root       3651 Jun 24 12:01 .bashrc
-rwsr-xr-x  1 leviathan5 leviathan5 1804 Jun 24 15:01 .trash
leviathan4@leviathan: $ ls -la
leviathan4@leviathan:$ ./.trash
leviathan4@leviathan:$ ls -la
```

```python
# convert.py
import sys

for x in (sys.argv[1:]):
    arr = []
    for i in str(x):
        if i == '0':
            arr.append('')
        if i == '1':
            arr.append('1')
    print(''.join(arr))
```

<details>
<summary>Flag</summary>
Bub9gZ3BGU
</details>