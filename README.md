<h1>CTF</h1>



<h2>Description</h2>
"Archetype" Room on Hackthebox
<br />


<h2>Languages, Applications and Utilities Used</h2>

- <b>powershell</b>
- <b>smbclient</b>
- <b>nmap</b>
- <b>impacket</b>
- <b>winPEAS</b>

<h2>Environments Used </h2>

- <b>Hackthebox VM + Kali VM</b> 
  
<h2>What I learned</h2>

- <b>Use of xp_cmdshell</b>
- <b>Escalationpaths</b>

<h2>Write-Up:</h2>

SCAN (next time i need to use a screenshot instead of text haha):

└─$ nmap -A 10.129.115.148 <br>
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-08-26 13:53 EDT <br>
Nmap scan report for localhost (10.129.115.148) <br>
Host is up (0.039s latency). <br>
Not shown: 819 closed tcp ports (conn-refused), 177 filtered tcp ports (no-response) <br>
PORT     STATE SERVICE      VERSION <br>
135/tcp  open  msrpc        Microsoft Windows RPC <br>
139/tcp  open  netbios-ssn  Microsoft Windows netbios-ssn <br>
445/tcp  open  microsoft-ds Windows Server 2019 Standard 17763 microsoft-ds <br>
1433/tcp open  ms-sql-s     Microsoft SQL Server 2017 14.00.1000.00; RTM <br>
| ms-sql-info:  <br>
|   10.129.115.148:1433:  <br>
|     Version:  <br>
|       name: Microsoft SQL Server 2017 RTM <br>
|       number: 14.00.1000.00 <br>
|       Product: Microsoft SQL Server 2017 <br>
|       Service pack level: RTM <br>
|       Post-SP patches applied: false <br>
|_    TCP port: 1433 <br>
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback <br>
| Not valid before: 2024-08-26T16:20:44 <br> 
|_Not valid after:  2054-08-26T16:20:44 <br> 
|_ssl-date: 2024-08-26T17:53:59+00:00; 0s from scanner time. <br>
| ms-sql-ntlm-info: <br>
|   10.129.115.148:1433: <br>
|     Target_Name: ARCHETYPE <br> 
|     NetBIOS_Domain_Name: ARCHETYPE <br>
|     NetBIOS_Computer_Name: ARCHETYPE <br>
|     DNS_Domain_Name: Archetype <br>
|     DNS_Computer_Name: Archetype <br>
|_    Product_Version: 10.0.17763 <br>
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows <br>

Host script results: <br>
| smb-security-mode:  <br>
|   account_used: guest <br>
|   authentication_level: user <br>
|   challenge_response: supported <br>
|_  message_signing: disabled (dangerous, but default) <br>
| smb-os-discovery: <br>
|   OS: Windows Server 2019 Standard 17763 (Windows Server 2019 Standard 6.3) <br>
|   Computer name: Archetype <br>
|   NetBIOS computer name: ARCHETYPE\x00 <br>
|   Workgroup: WORKGROUP\x00 <br>
|_  System time: 2024-08-26T10:53:53-07:00 <br>
| smb2-security-mode: <br>
|   3:1:1:  <br>
|_    Message signing enabled but not required <br>
|_clock-skew: mean: 1h24m00s, deviation: 3h07m51s, median: 0s <br>
| smb2-time: <br>
|   date: 2024-08-26T17:53:51 <br>
|_  start_date: N/A <br>


The nmap scan shows SMB ports open and an MSSQL Server running on port 1433

Lets enumerate the SMB Shares
![image](https://github.com/user-attachments/assets/f3bafbbc-ba03-4de1-8c98-850fd52f8dbd)



this shows 4 shares, with the most interesting being /backups
we try to connect to the backup share

![image](https://github.com/user-attachments/assets/95a53e64-2cca-448f-a3d7-dfea61a37442)



lets check out the config file, there might be something interesting in there

![image](https://github.com/user-attachments/assets/d7db902f-099b-47d3-a4e6-642757bdb6b2)


we find a username and password for the sql service <br>
lets try to connect to it
we will be using mssqlclient.py from impacket

![image](https://github.com/user-attachments/assets/2bd5d44b-255b-4723-91e0-e1c61edab810)


using the help command, we can get a better idea of what we can do

![image](https://github.com/user-attachments/assets/b288844c-9c89-44b5-86bb-852506f74ec3)


we use enable_xp_cmdshell since its disabled by default. We luckily have the privileges to do so.

![image](https://github.com/user-attachments/assets/75774b6c-9ae4-4aff-97a1-54fd1772e282)


this allows us to execute commands on the machine like “whoami”

![image](https://github.com/user-attachments/assets/94e5920b-a1d9-4177-a8bd-a6b27a7a8be7)


we are in the system32 directory and cant write here

![image](https://github.com/user-attachments/assets/44e94120-6b52-4dea-9091-94854586d00d)


now we try to get a stable reverse shell with netcat <br>
we host a the file and download it into a writeable directory on the target machine via powershell and the xp_cmdshell functionality

![image](https://github.com/user-attachments/assets/d09b6935-30c6-4413-8f7c-839ff1885f23)


check the service to see if it was successful

![image](https://github.com/user-attachments/assets/7508d18c-ab9b-4f6a-8b28-4e24f0fd1854)


now we set up a listener and execute nc.exe to connect to our machine

![image](https://github.com/user-attachments/assets/24906c6d-1769-486b-9644-b5f34e28262a)

![image](https://github.com/user-attachments/assets/ba5ccc56-c467-4f8a-b67e-7dad1d7c29f8)



on the Desktop we find the user flag

![image](https://github.com/user-attachments/assets/eff0baa8-0b9a-4c88-8059-c08ab3097294)


now we try to elevate our privileges <br>
we download winpeas via powershell and run it


![image](https://github.com/user-attachments/assets/5956a554-5d92-442c-9e58-4fede73a1bf3)


![image](https://github.com/user-attachments/assets/cd09dd18-8f2a-4554-b1bb-a63eed9fb615)


cute. lets search possible escalations in the output <br>
while there is some potential paths for escalation one file at the end of the ouput look most interesting

![image](https://github.com/user-attachments/assets/6297c106-56c3-47b3-8bf6-1117dac84f2b)


since we are on a service account, its good to check for frequently accessed files and executed commands in stored files

![image](https://github.com/user-attachments/assets/c9d7fce9-067a-4fb8-9aa5-c69d5ad8afbf)


we find a potential cleartext password for the admin user <br>
lets try to connect via psexec

![image](https://github.com/user-attachments/assets/984f4b1b-2aa2-49b9-9d16-e3ff8414cfb5)



we are now ROOT on the machine and can read out the root flag located on the Desktop of the admin account

![image](https://github.com/user-attachments/assets/f51ed616-1900-492e-a19b-e3de65b9d11e)



