# Web Request

## HTTP Fundamentals

### HyperText Transfer Protocol (HTTP)

1. **To get the flag, start the above exercise, then use cURL to download the file returned by '/download.php' in the server shown above.**

```
curl 154.57.164.67:30623/download.php
```

<p align="center"> 
<img src="images/WebRequestPrácticahtbflagsda.png" width="600" alt="Resultado de Nmap">
</p>

answer: **HTB{64$!c_cURL_u$3r}**

### HTTP Requests and Responses

1. **What is the HTTP method used while intercepting the request? (case-sensitive)** 

```
curl 154.57.164.67:30623 -v
```

<p align="center"> 
<img src="images/WebRequestPrácticaget123.png" width="600" alt="Resultado de Nmap">
</p>

Es **GET** porque así lo indica la primera línea de la petición enviada por el cliente: `> GET / HTTP/1.1`, donde el símbolo `>` marca que esa línea salió de curl hacia el servidor, y la estructura de esa línea siempre es "MÉTODO RUTA VERSIÓN_HTTP". Curl usó **GET** porque es su método por defecto cuando no se especifica otro con la opción `-X`

answer: **GET**

2. **Send a GET request to the above server, and read the response headers to find the version of Apache running on the server, then submit it as the answer. (answer format: X.Y.ZZ)**

```
curl 154.57.164.67:30623 -v
```

<p align="center"> 
<img src="images/WebRequesPrácticacurlsdf.png" width="600" alt="Resultado de Nmap">
</p>

answer: **2.4.41**

### HTTP Headers

1. **The server above loads the flag after the page is loaded. Use the Network tab in the browser devtools to see what requests are made by the page, and find the request to the flag.**

Se visita la pagina web `http://154.57.164.82:32307/` en el código fuente, en el apartado **Network**, recargando la pagina aparece lo siguiente:

<p align="center"> 
<img src="images/WebRequesPrácticared1213fwsddsa.png" width="600" alt="Resultado de Nmap">
</p>

Visitando la siguiente ruta `http://154.57.164.82:32307/flag_327a6c4304ad5938eaf0efb6cc3e53dc.txt` se obtiene la siguiente flag:

answer: **HTB{p493_r3qu3$t$_m0n!t0r}**
## HTTP Methods

### GET

1. **The exercise above seems to be broken, as it returns incorrect results. Use the browser devtools to see what is the request it is sending when we search, and use cURL to search for 'flag' and obtain the flag.**

_Authenticate to 154.57.164.82 , with user "admin" and password "admin"_

```
curl http://admin:admin@154.57.164.82:31025/search.php?search=flag
```

answer: **flag: HTB{curl_g3773r}**

### POST

1. **Obtain a session cookie through a valid login, and then use the cookie with cURL to search for the flag through a JSON POST request to '/search.php'**

_Authenticate to 154.57.164.82 , with user "admin" and password "admin"_

Este comando le dice a `curl` que haga una petición **POST** al archivo `search.php` del servidor en esa IP y puerto, enviando como cuerpo de la petición un JSON `{"search":"flag"}` (indicado con `-d` y con la cabecera `-H 'content-type: application/json'` para que el servidor sepa que debe interpretarlo como JSON), y adjuntando la cookie de sesión `PHPSESSID=omf15fhnfj5eubefmq1l2lu839` mediante `-b`, que es como decirle al servidor "yo ya estoy logueado, aquí está mi identificador de sesión" así que al enviarla el servidor te reconoció como usuario válido y ejecutó la búsqueda `"flag"`, devolviendo el resultado que contenía la flag

```
curl -X POST -d '{"search":"flag"}' -b 'PHPSESSID=omf15fhnfj5eubefmq1l2lu839' -H 'content-type: application/json' http://154.57.164.82:32240/search.php
```

<p align="center"> 
<img src="images/WebRequestPrácticacurlsdfdas.png" width="600" alt="Resultado de Nmap">
</p>

answer: **HTB{p0$t_r3p34t3r}**

### CRUD API

1. **First, try to update any city's name to be 'flag'. Then, delete any city. Once done, search for a city named 'flag' to get the flag.**

Paso 1 — Actualizar (PUT):

Esto le dice a la API "toma la ciudad que se llama `london` y cámbiale el nombre a `flag`". El `-X PUT` es el método HTTP usado normalmente para _actualizar_ un recurso existente, y el `-d` manda el JSON con el nuevo valor.

```
curl -X PUT http://154.57.164.82:30374/api.php/city/london -d '{"city_name":"flag"}' -H 'Content-Type: application/json'
```

Paso 2 — Comprobar que funcionó (GET):

Aquí simplemente consulta la ciudad que ahora se llama `flag`, y el resultado confirma que sí existe: `{"city_name": "flag", "country_name": "(UK)"}`. El `jq` solo sirve para formatear el JSON más bonito y legible.

```
curl -s http://154.57.164.82:30374/api.php/city/flag | jq
```

<p align="center"> 
<img src="images/WebRequestPrácticaflagcityaKDJAS.png" width="600" alt="Resultado de Nmap">
</p>

Paso 3 — Borrar otra ciudad (DELETE):

Esto borra la ciudad llamada `Baltimore`. Este paso parece "de relleno" en el reto (probablemente para demostrar que el DELETE funciona), pero no afecta directamente a la ciudad `flag`.

```
curl -X DELETE http://154.57.164.82:30374/api.php/city/Baltimore
````

Y aquí está la sorpresa: ahora el campo `country_name` ya no dice `(UK)`, sino que contiene la flag completa:

```
curl -s http://154.57.164.82:30374/api.php/city/flag | jq
```

<p align="center"> 
<img src="images/WebRequestPrácticaflafafg.png" width="600" alt="Resultado de Nmap">
</p>

En conclusión: Este último ejercicio sirve para practicar los verbos **HTTP** de una **API REST** (**PUT** para actualizar, **DELETE** para borrar, **GET** para consultar), y al completar la secuencia exacta que pedía el reto, el servidor revela la **flag** como recompensa.

answer: **HTB{crud_4p!_m4n!pul4t0r}**