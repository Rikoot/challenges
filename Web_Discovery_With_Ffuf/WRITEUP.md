## Web Discovery With Ffuf Writeup
```
For this challenge, you are tasked with discovering five flags on the docker web site using `ffuf`. The flags are in the following format: `ffuf{<TEXT>}`.
```
I chose to split the five flags between the topics that are meant to be taught/tested by this challenge. To learn more about `ffuf`, head to the official [wiki](https://github.com/ffuf/ffuf/wiki). 
#### Directory Fuzzing
Using `ffuf`, we can brute force directories for a page that, once opened, contains a flag. 
```bash
ffuf -ic -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://localhost:8080/FUZZ -fc 500
```
#### Extension Fuzzing
Using `ffuf`, we can brute force file extensions to discover other pages present on the machine that give more information into the backend of this machine.  
```bash
ffuf -ic -w /usr/share/seclists/Discovery/Web-Content/web-extensions.txt:FUZZ -u http://localhost:8080/indexFUZZ -v -fc 500
```
#### V-Host Fuzzing
Using `ffuf`, we can brute force virtual hosts (v-hosts) to discover other websites that are hosted on this machine under a different hostname. 
```bash
ffuf -ic -v -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://localhost:8080/ -H 'Host: FUZZ' 
```
#### Parameter Fuzzing
##### GET
Using `ffuf`, we can brute force `GET` parameters to discover `PHP` functionality on the webpage. 
```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ  -u http://localhost:8080/index.php?FUZZ=key -fs 267 -v
```
##### POST
Using `ffuf`, we can brute force `POST` parameters to discover `PHP` functionality on the webpage. 
```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://localhost:8080/index.php -X POST -d 'FUZZ=key' -H 'Content-Type: application/x-www-form-urlencoded' -fs 267
```
