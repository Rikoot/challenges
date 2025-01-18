```
For this challenge, you are tasked with discovering five flags on the docker web site using `ffuf`. The flags are in the following format: `ffuf{<TEXT>}`.
```
#### Directory Fuzzing
Find `alumni`:
```
ffuf -ic -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://localhost:8080/FUZZ   
```
#### Extension Fuzzing
Find `.php`:
```
ffuf -ic -w /usr/share/seclists/Discovery/Web-Content/web-extensions.txt:FUZZ -u http://localhost:8080/indexFUZZ -v -fc 500
```
#### V-Host Fuzzing
Find `student`:
```
ffuf -ic -v -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://localhost:8080/ -H 'Host: FUZZ' 
```
#### Parameter Fuzzing
##### GET
Find `department`:
```
ffuf -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ  -u http://localhost:8080/index.php?FUZZ=key -fs 267 -v
```
##### POST
Find `dev`:
```
ffuf -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://localhost:8080/index.php -X POST -d 'FUZZ=key' -H 'Content-Type: application/x-www-form-urlencoded' -fs 267
```