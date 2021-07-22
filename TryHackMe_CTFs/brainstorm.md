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
	- open Windows 32-bit VM and transfer files across
	- open Immunity Debugger and run chatserver.exe
	- 
	
	- 
