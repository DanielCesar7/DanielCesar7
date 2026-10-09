# What is Authentication
## Brute-Force Attacks

### Enumerating Users

1. **Enumerate a valid user on the web application. Provide the username as the answer.**

Me equivoco adrede para conseguir el siguiente mensaje **Unknown user** Para conseguir un usuario válido mediante fuerza bruta.

<p align="center"> 
<img src="images/What is Authentication Practise errpr.png" width="600" alt="Resultado de Nmap">
</p>

Este comando utiliza **`ffuf`** para realizar un ataque de fuerza bruta de nombres de usuario mediante peticiones HTTP POST, probando millones de combinaciones de una lista (_wordlist_) ubicada en SecLists. Sustituye la palabra clave `FUZZ` en los datos del formulario por cada elemento del diccionario y utiliza el parámetro `-fr` para ocultar automáticamente de la pantalla las respuestas repetitivas que contengan el texto _"Unknown user"_, facilitando así la identificación de cuentas válidas.

```
ffuf -w /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt -u http://154.57.164.71:31055/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "username=FUZZ&password=invalid" -fr "Unknown user"
```

<p align="center"> 
<img src="images/What is Authentication Practise usuario valido.png" width="600" alt="Resultado de Nmap">
</p>

El error cambia, ahora si acepta al usuario **cookster**.

<p align="center"> 
<img src="images/What is Authentication Practise inadli credfential.png" width="600" alt="Resultado de Nmap">
</p>

answer: **cookster**

### Brute-Forcing Passwords

1. **What is one prominent issue with passwords?**

Un problema importante con las contraseñas es su reutilización (**password reuse**). Cuando las personas reutilizan la misma contraseña en varios sitios web o servicios, una sola filtración de datos puede exponer esa contraseña y permitir que los atacantes accedan a muchas de sus cuentas. Esto aumenta enormemente el riesgo de robo de identidad y acceso no autorizado.

answer: **password reuse**

2. **What is the password of the user 'admin'?**

Requisitos de la contraseña del usuario admin

<p align="center"> 
<img src="images/What is Authentication Practise Requisitios de la contraseña.png" width="600" alt="Resultado de Nmap">
</p>

El comando toma el diccionario `rockyou.txt` y lo filtra paso a paso mediante tuberías para extraer únicamente aquellas líneas que contienen al menos una letra mayúscula (buscando el texto literal `:upper:`), una minúscula (`:lower:`), un número (`:digit:`) y una longitud mínima de 10 caracteres (`.{10}`), guardando finalmente el resultado depurado en un nuevo archivo llamado `custom_wordlist.txt`.

```
grep '[[:upper:]]' /usr/share/wordlists/rockyou.txt | grep '[[:lower:]]' | grep '[[:digit:]]' | grep -E '.{10}' > custom_wordlist.txt
```

Este comando de `ffuf` realiza un ataque de fuerza bruta contra el formulario de inicio de sesión de la URL especificada mediante peticiones `POST`, utilizando cada línea de tu diccionario (`custom_wordlist.txt`) como valor para la contraseña del usuario `admin` (reemplazando la etiqueta `FUZZ`), a la vez que descarta y oculta automáticamente todas las respuestas del servidor que contengan el mensaje de error "Invalid username" para facilitar la detección de credenciales válidas.

```
ffuf -w custom_wordlist.txt -u http://154.57.164.75:32246/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "username=admin&password=FUZZ" -fr "Invalid username"
```

<p align="center"> 
<img src="images/What is Authentication Practise ramirez12092.png" width="600" alt="Resultado de Nmap">
</p>

answer: **Ramirez120992**

### Brute-Forcing Password Reset Tokens

1. **On what do password recovery functionalities provided by web applications typically rely to allow users to recover their accounts?**

Las funciones de recuperación de contraseña proporcionadas por las aplicaciones web normalmente dependen de un token de restablecimiento único (**one-time reset token**)

answer: **one-time reset token**

2. **Which flag of seq pads numbers by prepending zeros to make them the same length?**

La bandera de seq que rellena los números anteponiendo ceros para que tengan la misma longitud es **-w**.

answer: **-w**

3. **How many possible values are there for a 6-digit OTP?**

A 6-digit OTP can range from **000000** to **999999**.

answer: **1000000**

4. **Takeover another user's account on the target system to obtain the flag.**

Si entramos con la ip del apartado **Brute-Forcing Passwords** e introducimos el usuario **admin** y la pass **Ramirez120992** se obtendrá la flag.

answer: **HTB{36da098385e641d54e1b2750721d816e}**

### Brute-Forcing 2FA Codes

1.  **Brute-force the admin user's 2FA code on the target system to obtain the flag.**

Authenticate to 154.57.164.73 , with user "<font color="#00b050">admin</font>" and password "<font color="#c00000">admin</font>"

Este comando utiliza la utilidad `seq` para generar una secuencia numérica desde el `0` hasta el `9999`, aplicando el parámetro `-w` para rellenar los números con ceros a la izquierda de modo que todos tengan una longitud uniforme de cuatro dígitos (formato de `0000` a `9999`), y redirige toda esta secuencia para guardarla en un archivo de texto llamado `tokens.txt`.

```
seq -w 0 9999 > tokens.txt
```

Se captura la sesión del 2fa.php

<p align="center"> 
<img src="images/Pasted image 20261003195118.png" width="600" alt="Resultado de Nmap">
</p>

Este comando de `ffuf` realiza un ataque de fuerza bruta contra un sistema de autenticación de doble factor en la URL indicada enviando peticiones `POST`, donde prueba cada token del archivo `tokens.txt` como valor del parámetro `otp` dentro de un cuerpo de formulario, manteniendo la sesión activa mediante la cookie `PHPSESSID`, y oculta automáticamente todas las respuestas del servidor que contengan el mensaje "Invalid 2FA Code" para facilitar la identificación del código válido.

```
ffuf -w ./tokens.txt -u http://154.57.164.77:30250/2fa.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -b "PHPSESSID=tp5es3c11u9bda47jbo3l0qrg0" -d "otp=FUZZ" -fr "Invalid 2FA Code"
```

Se reinicia la pagina, y se obtendrá la flag de este ejercicio

<p align="center"> 
<img src="images/What is Authentication Practise flagasdjnlas.png" width="600" alt="Resultado de Nmap">
</p>

answer: **HTB{9837b33a1ef678c380addf7ef8a517de}**

## Password Attacks

### Vulnerable Password Reset

1. **Which city is the admin user from?**

Se descarga el diccionario **world-cities.csv**

```
curl -L https://raw.githubusercontent.com/datasets/world-cities/master/data/world-cities.csv -o world-cities.csv
```

Utilizo el siguiente comando para obtener solo los nombres de las ciudades.

```
cat world-cities.csv | cut -d ',' -f1 > city_wordlist.txt
```

La imagen muestra una interfaz web de recuperación de contraseña que solicita responder una pregunta de seguridad sobre la ciudad de nacimiento del usuario `admin`, la cual devolvió un error rojo de "Incorrect response", mientras que a la derecha se observa la solicitud HTTP interceptada por Burp Suite enviando un valor de prueba en el parámetro `security_response`

<p align="center"> 
<img src="images/What is Authentication Practise password resetasdasd.png" width="600" alt="Resultado de Nmap">
</p>

Este comando de `ffuf` automatiza un ataque de fuerza bruta contra dicho formulario de pregunta de seguridad enviando peticiones `POST`, donde reemplaza la etiqueta `FUZZ` con cada una de las ciudades del diccionario `city_wordlist.txt`, mantiene la sesión activa con la cookie `PHPSESSID` correspondiente y oculta de forma automática las respuestas del servidor que contengan el mensaje de error "Incorrect response" para identificar la respuesta correcta.

```
ffuf -w city_wordlist.txt -u http://154.57.164.67:30660/security_question.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -b "PHPSESSID=k488dji139mre87nahdnlhndvg" -d "security_response=FUZZ" -fr "Incorrect response"
```

<p align="center"> 
<img src="images/What is Authentication Practise machester.png" width="600" alt="Resultado de Nmap">
</p>

answer: **Machester**

2. **Reset the admin user's password on the target system to obtain the flag.**

Se introduce la ciudad.

<p align="center"> 
<img src="images/What is Authentication Practise machester sadikasd.png" width="600" alt="Resultado de Nmap">
</p>

Una vez acertado nos permite cambiar la pass, y nos permite el cambio.

<p align="center"> 
<img src="images/What is Authentication Practise passowrd reseta dwsajids.png" width="600" alt="Resultado de Nmap">
</p>

Ingresamos las credenciales y se obtiene la flag del ejercicio.

<p align="center"> 
<img src="images/What is Authentication Practise flag-1.png" width="600" alt="Resultado de Nmap">
</p>

answer: **HTB{d4740b1801d9880ff70de227a54309f0}**

## Authentication Bypasses

### Authentication Bypass via Direct Access

1. **Apply what you learned in this section to bypass authentication to obtain the flag.**

Se **Activa "Intercept"** en Burp Suite.

Se Navega al endpoint `/admin.php` desde el navegador.

**Interceptar la respuesta** (no solo la petición):
    - Clic derecho sobre la petición → **"Do intercept" → "Response to this request"**.

<p align="center"> 
<img src="images/What is Authentication Practise responseadsad.png" width="600" alt="Resultado de Nmap">
</p>

**Forward** (reenviar) la petición para capturar la respuesta del servidor.

<p align="center"> 
<img src="images/What is Authentication Practise fowradsads.png" width="600" alt="Resultado de Nmap">
</p>

Con la **respuesta interceptada**, editarla:
    - Cambiar el código de estado de **`302 Found`** a **`200 OK`**.

<p align="center"> 
<img src="images/What is Authentication Practise 200.png" width="600" alt="Resultado de Nmap">
</p>

**Forward** la respuesta modificada.

<p align="center"> 
<img src="images/What is Authentication Practise htbflagasdd,ad.png" width="600" alt="Resultado de Nmap">
</p>

answer: **HTB{913ab2d84b8db21854c696dee1f1db68}**

### Authentication Bypass via Parameter Modification

1. **Apply what you learned in this section to bypass authentication to obtain the flag.** 

Authenticate to with user "<font color="#00b050">htb-stdnt</font>" and password "<font color="#c00000">AcademyStudent!</font>"

Esta imagen muestra una solicitud HTTP capturada en Burp Suite dirigida a `/admin.php` mediante el método `GET` utilizando un parámetro `user_id=183` y una cookie `PHPSESSID`, la cual responde con el mensaje "Could not load admin data"; esto ilustra exactamente de dónde se extrajeron la estructura de la URL, el parámetro variable, la cookie de sesión y el filtro que dieron forma al comando de `ffuf` empleado previamente.

<p align="center"> 
<img src="images/What is Authentication Practise could no load admin data.png" width="600" alt="Resultado de Nmap">
</p>

Para generar una secuencia numérica desde el `1` hasta el `9999` **sin ceros a la izquierda** (es decir, que imprima `372` en lugar de `0372`), simplemente debes omitir el parámetro `-w` del comando `seq`.

```
seq 1 9999 > user_ids.txt
```

Este comando de `ffuf` realiza una prueba de enumeración o fuerza bruta mediante peticiones `GET` en la URL especificada, reemplazando la etiqueta `FUZZ` con cada uno de los valores del archivo `tokens.txt` en el parámetro `user_id`, manteniendo la sesión activa mediante la cookie `PHPSESSID`, y ocultando automáticamente aquellas respuestas del servidor que contengan el mensaje "Could not load admin data" para detectar registros de usuarios válidos o existentes.

```
ffuf -w tokens.txt -X GET -b 'PHPSESSID=kk7qmk84a44appm9mjhp9ogb16' -u 'http://154.57.164.82:30535/admin.php?user_id=FUZZ' -fr 'Could not load admin data'
```

<p align="center"> 
<img src="images/What is Authentication Practise 372.png" width="600" alt="Resultado de Nmap">
</p>

Se cambia en burpsuite la id **183** por **372** Se obtiene la flag

<p align="center"> 
<img src="images/What is Authentication Practise htb flagasdakasd.png" width="600" alt="Resultado de Nmap">
</p>

answer: **HTB{63593317426484ea6d270c2159335780}**

## Session Attacks

### Attacking Session Tokens

1. **A session token can be brute-forced if it lacks sufficient what?**

La entropía (**entropy**) en este contexto se refiere al nivel de aleatoriedad e imprevisibilidad del token generado. Si el token de sesión es demasiado corto o contiene datos predecibles (estáticos), la cantidad de valores posibles es lo suficientemente pequeña como para que un atacante adivine sistemáticamente un token válido, lo que lleva a la toma de control de la sesión.

answer: **entropy**

2. **Obtain administrative access on the target to obtain the flag.**

Authenticate to with user "<font color="#00b050">htb-stdnt</font>" and password "<font color="#ff0000">AcademyStudent!</font>"

La imagen muestra el resultado la cookie `session` , que usaremos a continuación.

<p align="center"> 
<img src="images/What is Authentication Practiseburpsuitealdkasd.png" width="600" alt="Resultado de Nmap">
</p>

El comando `echo "757365723d6874622d7374646e743b726f6c653d75736572" | xxd -r -p` toma una cadena de texto en formato hexadecimal y utiliza la herramienta `xxd` con los parámetros `-r` (revertir) y `-p` (modo plano/plain) para traducirla y mostrarla en texto legible por humanos, revelando que el contenido de la sesión codificada era "user=stdnt;role=user".

```
echo "757365723d6874622d7374646e743b726f6c653d75736572" | xxd -r -p
```

<p align="center"> 
<img src="images/What is Authentication Practise student htb.png" width="600" alt="Resultado de Nmap">
</p>

El comando `echo -n "user=stdnt;role=admin" | xxd -p` toma la nueva cadena de texto modificada (asegurándose con `-n` de que no se añada un salto de línea al final) y utiliza `xxd` sin el modo revertir para convertirla y representarla en formato hexadecimal puro, generando el nuevo código `757365723d6874622d7374646e743b726f6c653d61646d696e` que ahora incluye el rol de administrador para reemplazarlo en la cookie de la aplicación.

```
echo -n "user=stdnt;role=admin" | xxd -p
```

<p align="center"> 
<img src="images/What is Authentication Practise adminroleasdas.png" width="600" alt="Resultado de Nmap">
</p>

## Skill Assessment

### Skill Assessment

1. **Combine the attacks you have learned in this module to obtain the flag.**

La imagen muestra una solicitud HTTP de tipo POST interceptada (en un proxy como Burp Suite o una herramienta similar) dirigida a la ruta `/login.php`, donde se envían las credenciales de prueba `username=dsa&password=dsa123` junto con la cookie de sesión `PHPSESSID=acnp18aj74pgd8jvah3mhspsui`, obteniendo como respuesta en el navegador integrado un mensaje de error que advierte sobre un usuario o contraseña desconocidos (`Unknown username or password`).

<p align="center"> 
<img src="images/What is Authentication Practise unkweusernameorpasswrodas.png" width="600" alt="Resultado de Nmap">
</p>

El comando utiliza la herramienta `ffuf` para realizar un ataque de fuerza bruta automatizada enviando peticiones POST masivas a `http://154.57.164.78:32327/login.php`; para ello, carga el diccionario de nombres de usuario (`xato-net-10-million-usernames.txt`) iterando la etiqueta `FUZZ` en el parámetro `username`, mantiene fija la contraseña (`password=dsa123`), incluye la cookie de sesión activa de la imagen y utiliza el filtro `-fr 'Unknown username or password'` para descartar todas las respuestas genéricas y mostrar únicamente aquellas que indiquen un resultado válido.

```
ffuf -w /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt -X POST -H 'Content-Type: application/x-www-form-urlencoded' -b 'PHPSESSID=acnp18aj74pgd8jvah3mhspsui' -d 'username=FUZZ&password=1234' -u 'http://154.57.164.78:32327/login.php' -fr 'Unknown username or password'
```

<p align="center"> 
<img src="images/What is Authentication Practise gladys-1.png" width="600" alt="Resultado de Nmap">
</p>

La imagen muestra una interfaz de registro de la aplicación web que rechaza un intento de contraseña por no cumplir con una política de seguridad estricta, la cual exige exactamente 12 caracteres, al menos un dígito, una letra minúscula, una letra mayúscula y ningún carácter especial.

<p align="center"> 
<img src="images/What is Authentication Practise necesida passs.png" width="600" alt="Resultado de Nmap">
</p>

El comando utiliza `grep` con expresiones regulares perl-compatibles (`-P`) para filtrar el diccionario masivo `rockyou.txt` según esas reglas específicas de la política (dígitos, minúsculas, mayúsculas, exclusión de caracteres especiales y longitud exacta de 12), guardando el resultado filtrado en un nuevo archivo llamado `rockyou_modificado.txt`.

```
grep  -P '^(?=.*[0-9])(?=.*[a-z])(?=.*[A-Z])[A-Za-z0-9]{12}$' /usr/share/wordlists/rockyou.txt > rockyou_modificado.txt
```

La imagen muestra la interfaz web de inicio de sesión de la aplicación, en la cual se ha introducido un intento de acceso fallido que genera una alerta visual de error en rojo con el texto genérico de credenciales incorrectas (`Invalid credentials.`).

<p align="center"> 
<img src="images/What is Authentication Practise invalido.png" width="600" alt="Resultado de Nmap">
</p>

El comando utiliza la herramienta `ffuf` para llevar a cabo un ataque de fuerza bruta automatizado contra la contraseña del usuario `gladys` en el servidor remoto (`http://154.57.164.78:32327/login.php`), empleando para ello el diccionario personalizado `rockyou_modificado.txt` mediante la etiqueta `FUZZ` en el campo del formulario, enviando la cookie de sesión activa e incorporando un filtro de respuesta para descartar las líneas que coincidan con el error predeterminado.

```
ffuf -w rockyou_modificado.txt -X POST -H 'Content-Type: application/x-www-form-urlencoded' -b 'PHPSESSID=acnp18aj74pgd8jvah3mhspsui' -d 'username=gladys&password=FUZZ' -u 'http://154.57.164.78:32327/login.php' -fr 'Invalid credentials'
```

<p align="center"> 
<img src="images/What is Authentication Practise pass.png" width="600" alt="Resultado de Nmap">
</p>

Por tanto, las credenciales que obtenemos son **gladys**:**dWinaldasD13**. Después, se nos pedirá el método 2fa

<p align="center"> 
<img src="images/What is Authentication Practise otp.png" width="600" alt="Resultado de Nmap">
</p>

Luego, con burpsuite cambiamos el endpoint **/2fa.php** por **/profile.php**

<p align="center"> 
<img src="images/What is Authentication Practise flag skill assesment htb.png" width="600" alt="Resultado de Nmap">
</p>

answer: **HTB{d86115e037388d0fa29280b737fd9171}**