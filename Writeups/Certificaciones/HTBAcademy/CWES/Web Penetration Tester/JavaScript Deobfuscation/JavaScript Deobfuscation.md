# JavaScript Deobfuscation

## Introduction
### Deobfuscation

**1. Repeat what you learned in this section, and you should find a secret flag, what is it?**

La flag se encuentra en el código fuente de la pagina web.

answer: **HTB{4lw4y5_r34d_7h3_50urc3}**

## Obfuscation

1. **Using what you learned in this section, try to deobfuscate 'secret.js' in order to get the content of the flag. What is the flag?**

Observando el código fuente de la página nos encontramos el archivo secret.js

<p align="center"> 
<img src="images/JavaScript Deobfuscation Practise secretsjs.png" width="600" alt="Resultado de Nmap">
</p>

En su interior aparecerá el codigo ofuscado, y para leer la flag q se encuentra en su interior habrá que desofuscarlo

```
eval(function (p, a, c, k, e, d) { e = function (c) { return c.toString(36) }; if (!''.replace(/^/, String)) { while (c--) { d[c.toString(a)] = k[c] || c.toString(a) } k = [function (e) { return d[e] }]; e = function () { return '\\w+' }; c = 1 }; while (c--) { if (k[c]) { p = p.replace(new RegExp('\\b' + e(c) + '\\b', 'g'), k[c]) } } return p }('g 4(){0 5="6{7!}";0 1=8 a();0 2="/9.c";1.d("e",2,f);1.b(3)}', 17, 17, 'var|xhr|url|null|generateSerial|flag|HTB|1_4m_7h3_53r14l_g3n3r470r|new|serial|XMLHttpRequest|send|php|open|POST|true|function'.split('|'), 0, {}))
```

Para ello usaremos esta página [unpacker](https://matthewfl.com/unPacker.html)

<p align="center"> 
<img src="images/JavaScript Deobfuscation Practise paker flag.png" width="600" alt="Resultado de Nmap">
</p>

answer: **HTB{1_4m_7h3_53r14l_g3n3r470r!}**
## Deobfuscation Examples

### HTTP Requests

1. **Try applying what you learned in this section by sending a 'POST' request to '/serial.php'. What is the response you get?**

Este comando utiliza la herramienta `curl` para enviar una petición HTTP de tipo **`POST`** (`-X POST`) directamente a la ruta `/serial.php` del servidor objetivo, simulando la acción que el script de JavaScript realizaba en segundo plano en la página web. La opción **`-s`** (_silent_) le indica a `curl` que se ejecute en modo silencioso, ocultando la barra de carga y los mensajes informativos para mostrar en la terminal únicamente la respuesta limpia que devuelva el servidor como resultado de esa petición.

```
curl -s http://154.57.164.70:30190/serial.php -X POST
```

<p align="center"> 
<img src="images/JavaScript Deobfuscation Practise caosaxdmjodalsd.png" width="600" alt="Resultado de Nmap">
</p>

answer: **N2gxNV8xNV9hX3MzY3IzN19tMzU1NGcz**

### Decoding

1. **Using what you learned in this section, determine the type of encoding used in the string you got at previous exercise, and decode it. To get the flag, you can send a 'POST' request to 'serial.php', and set the data as "serial=YOUR_DECODED_OUTPUT".** 

En primer lugar lo que se consiguió del ejercicio anterior está en base64, el siguiente comando lo descodifica.

```
echo "N2gxNV8xNV9hX3MzY3IzN19tMzU1NGcz" | base64 -d
```

<p align="center"> 
<img src="images/JavaScript Deobfuscation Practise base64.png" width="600" alt="Resultado de Nmap">
</p>

Luego, realizando el siguiente comando se consigue la flag del ejercicio.

```
curl -s http://154.57.164.70:30190/serial.php -X POST -d "serial=N2gxNV8xNV9hX3MzY3IzN19tMzU1NGcz"
```

<p align="center"> 
<img src="images/JavaScript Deobfuscation Practise htb flag.png" width="600" alt="Resultado de Nmap">
</p>

answer: **HTB{ju57_4n07h3r_r4nd0m_53r14l}**

## Skills Assesment 

### Skills Assesment

1. **Try to study the HTML code of the webpage, and identify used JavaScript code within it. What is the name of the JavaScript file being used?**

Observando el código fuente de la página se encuentra el nombre del archivo.

<p align="center"> 
<img src="images/JavaScript Deobfuscation Practiseapimin.png" width="600" alt="Resultado de Nmap">
</p>

answer: **api.min.js**

2. **Once you find the JavaScript code, try to run it to see if it does any interesting functions. Did you get something in return?**

A la hora de clikear en el archivo **api.min.js** nos aparecerá lo siguiente:

```
eval(function (p, a, c, k, e, d) { e = function (c) { return c.toString(36) }; if (!''.replace(/^/, String)) { while (c--) { d[c.toString(a)] = k[c] || c.toString(a) } k = [function (e) { return d[e] }]; e = function () { return '\\w+' }; c = 1 }; while (c--) { if (k[c]) { p = p.replace(new RegExp('\\b' + e(c) + '\\b', 'g'), k[c]) } } return p }('t 5(){6 7=\'1{n\'+\'8\'+\'9\'+\'a\'+\'b\'+\'c!\'+\'}\',0=d e(),2=\'/4\'+\'.g\';0[\'f\'](\'i\',2,!![]),0[\'k\'](l)}m[\'o\'](\'1{j\'+\'p\'+\'q\'+\'r\'+\'s\'+\'h\'+\'3}\');', 30, 30, 'xhr|HTB|_0x437f8b|k3y|keys|apiKeys|var|flag|3v3r_|run_0|bfu5c|473d_|c0d3|new|XMLHttpRequest|open|php|n_15_|POST||send|null|console||log|4v45c|r1p7_|3num3|r4710|function'.split('|'), 0, {}))
```

Se usará esta página [UnPacker](https://matthewfl.com/unPacker.html) deobfuscation del codigo anterior y se consigue la flag.

<p align="center"> 
<img src="images/JavaScript Deobfuscation Practise htb flag corcheteee.png" width="600" alt="Resultado de Nmap">
</p>

answer: **HTB{j4v45cr1p7_3num3r4710n_15_k3y}**

3. **As you may have noticed, the JavaScript code is obfuscated. Try applying the skills you learned in this module to deobfuscate the code, and retrieve the 'flag' variable.**

>Como se habrá notado, el código JavaScript está ofuscado. Intente aplicar las habilidades que aprendió en este módulo para desofuscar el código y recuperar la variable ‘flag’.

Ya había desofuscado el código con UnPacker, lo que significa que la primera bandera que vi fue la respuesta. Los consejos también mencionaron que era necesario unir las respuestas.

<p align="center"> 
<img src="images/JavaScript Deobfuscation Practise htb asdkljdas.png" width="600" alt="Resultado de Nmap">
</p>

En lugar de HTB{n’+’3v3r_’+’run_0'+’bfu5c’+’473d_’+’c0d3!’+’}. Eliminé los ‘+’ y los formateé correctamente para mostrar HTB{n3v3r_run_0bfu5c473d_c0d3!}

answer: **HTB{n3v3r_run_0bfu5c473d_c0d3!}**

4. **Try to Analyze the deobfuscated JavaScript code, and understand its main functionality. Once you do, try to replicate what it's doing to get a secret key. What is the key?**

En la actividad anterior justo como estábamos consiguiendo la flag de la actividad, también se encontró un nuevo endpoint **keys.php**

<p align="center"> 
<img src="images/JavaScript Deobfuscation Practise keysphp.png" width="600" alt="Resultado de Nmap">
</p>

Usamos el método **POST** (indicado con `-X POST`) porque el propio código JavaScript de la aplicación está programado explícitamente para comunicarse con el servidor a través de esa vía mediante instrucciones como `xhr.open('POST', ...)`. Mientras que una petición `GET` estándar solo sirve para solicitar ver o descargar una página, `POST` le indica al servidor que se está enviando una solicitud de procesamiento de datos o interacción; por lo tanto, replicar exactamente este método es indispensable para hablar con el _endpoint_ de la misma forma en que lo hace la página web y lograr que el servidor responda correctamente en lugar de rechazar la conexión.

```
curl -s http://154.57.164.75:32024/api.min.js -X POST
```

<p align="center"> 
<img src="images/JavaScript Deobfuscation Practise hexadecimal.png" width="600" alt="Resultado de Nmap">
</p>

answer: **4150495f70336e5f37333537316e365f31355f66756e**

5. **Once you have the secret key, try to decide it's encoding method, and decode it. Then send a 'POST' request to the same previous page with the decoded key as "key=DECODED_KEY". What is the flag you got?**

Se utilizó este comando para **decodificar una cadena en formato hexadecimal** y transformarla en texto legible (`API_p3n_73571n6_15_fun`). La herramienta `xxd` con los parámetros `-p -r` (_revert plain hex dump_) toma los códigos hexadecimales y los traduce de vuelta a caracteres de texto normal que los humanos podemos leer.

```
echo "4150495f70336e5f37333537316e365f31355f66756e" | xxd -p -r
```

<p align="center"> 
<img src="images/JavaScript Deobfuscation Practise api.png" width="600" alt="Resultado de Nmap">
</p>

Se empleó este comando para enviar una petición HTTP **POST** de forma silenciosa (`-s`) al _endpoint_ `/keys.php` del servidor objetivo, adjuntando la clave recién decodificada dentro del parámetro `key`. Al recibir este dato correcto, el servidor lo validó y respondió entregando la **bandera final** del ejercicio (`HTB{r34dy_70_h4ck_my_w4y_1n_2_HTB}`).

```
curl -s http://154.57.164.75:32024/keys.php -X POST -d "key=API_p3n_73571n6_15_fun" 
```

<p align="center"> 
<img src="images/JavaScript Deobfuscation Practise htb asdjklasdkasd.png" width="600" alt="Resultado de Nmap">
</p>

answer: **HTB{r34dy_70_h4ck_my_w4y_1n_2_HTB}**