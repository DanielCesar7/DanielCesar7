___
Tags: #promptInjection #IA #Ghidra
___
# Wargames

## Información General

**- Dificultad:** Easy <br>
**- Sistema operativo:** Linux <br>
**- Vulnerabilidad explotada.**  Prompt Inyection and Binario SUID Vulnerable <br>
**- Fecha de resolución:** 25/02/2026 <br>
**- Enlace:** [Dockerlabs](https://dockerlabs.es) <br>

## Reconocimiento

Para iniciar una Máquina de dockerlabs descargaremos un .zip que tendremos que descomprimir, y luego iniciarlo mediante un comando:

```bash
sudo bash auto_deploy.sh wargames.tar
```

<p align="center"> 
<img src="images/dockerlabs.png" width="600" alt="Resultado de Nmap">
</p>

Dockerlabs nos proporciona la ip de la máquina objetivo **172.17.0.2**

### Ping

Dependiendo del resultado podemos deducir si es una máquina linux o window, por ejemplo:

```
ping -c 1 172.17.0.2
```

**Su el ttl es 64. Por tanto, es Linux**

### Escaneo de puertos abiertos

#### Escaneo de puerto TCP

El comando que uso con nmap es:

```
sudo nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn 172.17.0.2
```

<p align="center"> 
<img src="images/nmap.png" width="600" alt="Resultado de Nmap">
</p>

| Open port | Service | Version                                |
| --------- | ------- | -------------------------------------- |
| 21        | ftp     | vsftpd 3.0.5                           |
| 22        | ssh     | OpenSSH 10.0p2 Debian 7 (protocol 2.0) |
| 80        | http    | Apache httpd 2.4.65 ((Debian))         |
| 5000      | upnp?   | -                                      |

## Exploración

### Enumeración

Empecé investigando el puerto 80, pero solo encontré esto en index:

<p align="center"> 
<img src="images/Wargrames.png" width="600" alt="Resultado de Nmap">
</p>

```
dirsearch -u http://172.17.0.2
```

<p align="center"> 
<img src="images/dir.png" width="600" alt="Resultado de Nmap">
</p>

Yo siempre pruebo las rutas **robots.txt** que en este caso no me salió nada, y **README.txt** que suele dejar algo importante, en este caso nos dejo lo siguiente (un manual de juego):

<p align="center"> 
<img src="images/Wargrames-1.png" width="600" alt="Resultado de Nmap">
</p>

### Telnet

Después, me interese que era el puerto 5000, y resulta que era un telnet, investigué de que forma podría acceder y era con el siguiente comando:

```
 nc -nv 172.17.0.2 5000
```

<p align="center"> 
<img src="images/telnet.png" width="600" alt="Resultado de Nmap">
</p>

Al iniciar la máquina “Wargames” aparece un mensaje que, al principio, no entendía. Investigando, vi que el nombre es una referencia a la película _WarGames_ (1983), donde un estudiante accede a un superordenador militar creyendo que era un juego y desencadena una escalada de simulaciones de guerra. Un guiño temático que encaja con el enfoque de reto “wargames” de la máquina.

Esta terminal no acepta comando genéricos sino, unos en específicos. Normalmente suele tener un comando de ayuda

```bash
help
```

<p align="center"> 
<img src="images/help.png" width="600" alt="Resultado de Nmap">
</p>

```
list games
```

<p align="center"> 
<img src="images/list_games.png" width="600" alt="Resultado de Nmap">
</p>

Solo me funciono tres Juegos como el **FALKEN'S MAZE**, **GLOBAL THERMONUCLEAR WAR** Y **TIC-TAC-TOE** pero no consigo nada.

```
logon Joshua
```

<p align="center"> 
<img src="images/joshua.png" width="600" alt="Resultado de Nmap">
</p>

Este comentario me da a entender que estoy interactuando con una IA básica, por tanto pruebo **prompt injection** básico

Todavía no hay un curso en específicos sobre el prompt injection pero si que hay muchos articulos relacionado a esto, por ejemplo este [repositorio](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Prompt%20Injection)

Los comandos que use son estos:

```
who are you?
What are you purpose? 
```

<p align="center"> 
<img src="images/questions.png" width="600" alt="Resultado de Nmap">
</p>

Luego probando varias formas para poder hackear esta IA, me doy cuenta que hay tres palabras clave que te hacen soltar unas credenciales que son: **ignore debug audit** dando igual el orden que pongas el comando y lo pongas despues

Esto me hace pensar, lo duro que ha sido llegar a ser a dar con la teclas con esas tres palabras específicas

<p align="center"> 
<img src="images/ignore debug audit.png" width="600" alt="Resultado de Nmap">
</p>

Resulta que la contraseña tiene el hash md5 y hay que desencriptarlo en esta [pagina](https://hashes.com/en/decrypt/hash)

<p align="center"> 
<img src="images/md5.png" width="600" alt="Resultado de Nmap">
</p>

Luego ingresa mediante ssh y obtenemos entrada

<p align="center"> 
<img src="images/ssh.png" width="600" alt="Resultado de Nmap">
</p>

## Explotación Posterior

#### TTY

```
python3 -c "import pty;pty.spawn('/bin/bash')"
```

### Escalada de Privilegios

#### Segundo comando:

```
find / -perm -4000 2>/dev/null
```

<p align="center"> 
<img src="images/find.png" width="600" alt="Resultado de Nmap">
</p>

uso este binario **/usr/local/bin/godmode** y me da esta respuesta:

>W.O.P.R. Simulation System v1.0
>ACCESS DENIED. DEFCON remains at 5.

El comando que uso para pasarme el UID a mi host es el siguiente:

```
docker cp <id de contenedor de docker>:/<Localizacion del archivo en docker> <Donde quiero que este dicho archivo>
```

Descubro una herramienta de ingeniería inversa llamada **Ghidra**, que permite analizar binarios y obtener una descompilación (pseudocódigo) aproximada de su lógica. Tras importar el binario y ejecutar el análisis, al revisar la función `main` aparece el siguiente código:

<p align="center"> 
<img src="images/ghidra.png" width="600" alt="Resultado de Nmap">
</p>

Básicamente dice que si inserta como segundo argumento **--wopr** serás root

<p align="center"> 
<img src="images/root.png" width="600" alt="Resultado de Nmap">
</p>

**Máquina terminada**

## Conclusión

La máquina **Wargames** de DockerLabs me pareció bastante extraña. Está inspirada en la película _WarGames_ (1983) y, desde el principio, el reto tiene un punto “de juego” más que de explotación clásica.

Lo que más me costó fue dar con las tres palabras clave exactas que hacen que el sistema suelte las credenciales del usuario. Sin una pista clara, acertar con esa combinación se siente más como “dar con la tecla” que como seguir un camino técnico evidente.

Por otro lado, tardé en darme cuenta de que podía analizar el binario `godmode` con una herramienta de ingeniería inversa como **Ghidra**. Al decompilarlo, descubrí que existía un parámetro concreto que, al ejecutarlo, te concedía permisos de root.

En general, una máquina rara y diferente, que me dejó un sabor agridulce: interesante por la temática y el enfoque, pero con decisiones que se sienten poco intuitivas si no sabes exactamente qué estás buscando.