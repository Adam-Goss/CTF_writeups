# Blaster

> A blast from the past! Exploitation without Metasploit.

<br>

Process:
- 1) scanning 
  - port scan: `nmap -T4 <ip>`
    - total open ports = **2**
  - aggressive scan: `nmap -T4 -A -p ... <ip>`
    - title of web page = **IIS Windows Server**
- 2) enumerate website:
  - `gobuster dir -u <host> -w <wordlist>`
  - hidden directory = **/retro**
  - visit website:
    - possible username = **wade**
    - found possible password = **parzival**
- 3) log in to RDP using credentials found 
  - found user flag = **THM{HACK_PLAYER_ONE}**
  - look to see what user was doing on server:
    - found user was researching CVE == **

