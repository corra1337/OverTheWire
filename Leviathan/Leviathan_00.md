## 0 -> 1

### Writeup

I began by running "ls -la" to identify anything unusual in the home directory. There were several files like .bashrc, but the most interesting was a hidden directory called ".backup", and in it, was a file "bookmarks.html"

Opening up this document in vim, it was clear the file was too big to sort through manually, so I used grep to see if there were any bookmarks relating to leviathan1. There was and it revealed the flag.

```
leviathan0@leviathan:~$ ls -la
total 24
drwxr-xr-x 3 root root 4096 Jun 24 15:01 .
drwxr-xr-x 150 root root 4096 Jun 24 15:02 ..
-rw-r--r-- 1 root root 220 Feb 13 12:16 .bash_logout
-rw-r--r-- 1 root root 3651 Jun 24 14:58 .bashrc
-rw-r--r-- 1 root root 675 Feb 13 12:16 .profile
drwxr-x--- 2 leviathan1 leviathan1 4096 Jun 24 15:01 .backup
leviathan0@leviathan:~$ ls .backup
bookmarks.html
leviathan0@leviathan:~$ grep -i leviathan1 bookmarks.html
<DT><A HREF="http://leviathan.labs.overthewire.org/passwdrdu.html |
This will be fixed later. The password for leviathan1 is PiXaSWQqHq ADD DATE="1155348634" L
AST_CHARSET="ISO-8859-1"><D><rdf:RDF><password to leviathan</a>
```

<details>
<summary>Flag</summary>
PiXaSWQqHq
</details>