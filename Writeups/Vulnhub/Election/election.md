## Información General

**- Dificultad:** Medio
**- Sistema operativo:** Linux
**- Vulnerabilidad explotada.** Serv-U FTP Server (Local Privilege Escalation)
**- Fecha de resolución:** 27/10/2025
**- Enlace:** https://www.vulnhub.com/entry/election-1,503

## Reconocimiento

TryHackme nos proporciona la ip de la máquina objetivo **192.168.0.100**

Voy a establecer en el fichero **/etc/hosts** la **ip de la mv objetivo**, la voy a llamar **election**

**ARP-SCAN**

```
sudo arp-scan -I eth0 --localnet --ignoredups
```

![[Pasted image 20251025144857.png]]
### Ping

Dependiendo del resultado podemos deducir si es una máquina linux o window, por ejemplo:

```
ping -c 1 192.168.0.100
```

Su ttl es 64. Por tanto, su sistema es Linux.
### Escaneo de puertos abiertos

#### Escaneo de puerto TCP

El comando que uso con nmap es:

```
sudo nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn 192.168.0.104
```

![[Pasted image 20251025145115.png]]

| Open port | Service | Version                                                      |
| --------- | ------- | ------------------------------------------------------------ |
| 22        | ssh     | OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0) |
| 80        | http    | Apache httpd 2.4.29 ((Ubuntu))                               |

#### Escaneo de puerto UDP

```
nmap -sU --top-ports 200 --min-rate=5000 -Pn 192.168.0.100
```

Todos los puertos están cerrados
## Exploración

### Fuzzing web

```
gobuster dir -u http://192.168.0.100/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x txt,py,php,sh
```

![[Pasted image 20251027011728.png]]

Luego, volvemos hacer un gobuster pero esta vez a la ruta election

```
gobuster dir -u http://192.168.0.104/election -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x txt,py,php,sh
```

![[Pasted image 20251027012235.png]]

Por último, lo volvemos hacer también con admin

```
gobuster dir -u http://192.168.0.104/election/admin/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x txt,py,php,sh
```

![[Pasted image 20251027012356.png]]

Visualizamos por tanto este contenido.

![[Pasted image 20251027012552.png]]

```
User: love
Pass: P@$$w0rd@123
```

Usamos estas credenciales por ssh

```
ssh 192.168.0.104@love
```

Conseguimos su flag 

![[Pasted image 20251027013942.png]]
## Explotación

### Escalada de Privilegios

#### Primer comando:

```
sudo -l
```

El usuario love no tiene permiso de root
#### Segundo comando:

```
find / -perm -4000 -ls 2>/dev/null | grep -v snap
```

![[Pasted image 20251027013252.png]]

Buscamos informacion como explotar este binario para subir de privilegio. En exploit db nos explica como hacerlo con un script en C. https://www.exploit-db.com/exploits/47009

![[Pasted image 20251027013657.png]]

Copia el script en /tmp, luego usamos los siguientes comandos:

```
gcc exploit.c -o exploit
./exploit
```

![[Pasted image 20251027013822.png]]

Así conseguimos ser root, conseguimos la flag

![[Pasted image 20251027013911.png]]
## Conclusión

La máquina **Election** en VulnHub es realmente interesante. Con una simple enumeración de los _paths_ de la IP usando gobuster se obtienen las credenciales necesarias para acceder por SSH, lo que la hace especialmente práctica para practicar reconocimento y enumeración. La escalada de privilegios, aunque sofisticada en su planteamiento, resulta bastante directa si conoces el binario **Serv-U**. En Exploit-DB existe un exploit en C que facilita la elevación de privilegios; aplicándolo correctamente se obtiene acceso root. En mi opinión, si sabes lo que haces desde el principio, puedes completar la máquina en un _speedrun_ — Tal vez, la escalada se salga un poco del nivel **ejptv2** pero aun así la recomiendo.