# Pickle Rick 

> This Rick and Morty themed challenge requires you to exploit a webserver to find 3 ingredients that will help Rick make his potion to transform himself back into a human from a pickle.

1. walk the website
- inspect homepage source code:
	- found username: **R1ckRul3s**
- check cookies
- find server version:
	- Apache/2.4.18 (Ubuntu) Server 80
- visit with Wappalyzer


2. find interesting directories 
- use gobuster:
 	- `gobuster dir -u http://10.10.13.41 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt`
	- found: **assets**, **server-status**


3. scan for vulnerabilities
- nmap aggressive scan:
	- `nmap -A -T4 10.10.13.41`
	- SSH is running: OpenSSH 7.2p2 Ubuntu 4ubuntu2.6
	- HTTP is running: Apache/2.4.18 (Ubuntu)

4. try SSH login
- `ssh R1ckRul3s@10.10.13.41`
	- requires public key authentication 


5. try SSH vulns:
- `searchsploit OpenSSH 7.2p2`
	- username enumeration vuln 
	- FAILED

6. look for interesting files 
- `gobuster dir -u http://10.10.13.41 -w /usr/share/wordlists/dirb/big.txt`
	- found "robots.txt" with "Wubbalubbadubdub"
- `gobuster dir -u http://10.10.13.41 -w /usr/share/wordlists/dirb/big.txt -x php`
	- found "login.php"

7. try logging in using "R1ckRul3s" and "Wubbalubbadubdub"
- **SUCCESS**

8. try executing commands in web app
- `ls` -> `cat Sup3rS3cretPickl3Ingred.txt` -> FAIL
- `ls` -> `cat clue.txt` -> FAIL

9. try to get reverse shell
- use [Payload all the Things](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md#bash-tcp)
	- try bash -> FAIL
	- try python -> FAIL
	- try perl -> SUCCESS 

10. try executing commands in reverse shell
- `ls` -> `cat Sup3rS3cretPickl3Ingred.txt` -> ingredient 1 = **mr. meeseek hair**
- `ls` -> `cat clue.txt` -> "look around filesystem"

11. explore file system 
- `cd /home/rick` -> `cat "second ingredients"` -> ingredient 2 = **1 jerry tear**
- `cd /root` -> FAIL
- check sudo permissions -> `sudo -l` -> ALL
- `sudo ls /root` -> `sudo cat /root/3rd.txt` -> ingredient 3 = **fleeb juice**
