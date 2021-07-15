# Skynet

> A vulnerable Terminator themed Linux machine.

<br>

### Process:

1. discovery and scanning:
	- find if machine is up: `sudo nmap -sn 10.10.60.122` = yes
	- port scan machine:
		- `sudo nmap -T4 10.10.60.122`
		- `sudo nmap -T4 -A -p 22,80,110,139,143,445 10.10.60.122`
2. enumeration:
	- check out the website on port 80
	- enumerate SMB:
		- `smbmap -H 10.10.60.122` - found "anonymous" share with read permissions
		- `smbclient \\\\10.10.160.122\\anonymous` - found "attention.txt" and "log1.txt"
			- "log1.txt" contains a list of possible passwords
	- try bruteforcing pop3 login with passwords in "log1.txt":
		- find miles username: `sudo nmap -p 445 --script=smb-enum-users 10.10.160.122` = "milesdyson"
	- try to find email login on web page:
		- `gobuster dir -u http://10.10.60.122 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt`
		- found: "/squirrelmail"
	- use Burp intruder to try and login using "log1.txt" as password payload list
		- miles password for emails = **cyborg007haloterminator**
3. exploitation:
	- login to miles email account at "http://10.10.160.122/squirrelmail"
	- found new SMB password in emails = *)s{A&2Z=F^n_E.B\`*
	- use SMB password to login to miles SMB share:
		- `smbclient \\\\10.10.60.122\\milesdyson -U "milesdyson"` -> [new SMB password]
	- look through SMB share
		- found "important.txt" file that contains a CMS URL "/45kra24zxs28v3yd"
	- go to new CMS URL found 
	- name of hidden directory = **45kra24zxs28v3yd**
	- name of vulnerability when you include a remote file for malicious purposes = **remote file inclusion**
	- ... 
