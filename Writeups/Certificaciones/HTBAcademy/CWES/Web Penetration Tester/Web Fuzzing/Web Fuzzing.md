# Web Fuzzing

## Directory and File Fuzzing

### Directory and File Fuzzing

1. **Within the "webfuzzing_hidden_path" path on the target system `(ie http://IP:PORT/webfuzzing_hidden_path/)`, fuzz for folders and then files to find the flag.**

```
wfuzz -c --hc 404 -w /usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-medium.txt http://154.57.164.82:30468/webfuzzing_hidden_path/FUZZ
```

![[Web Fuzzing Practise flag.png]]

```
ffuf -c -w /usr/share/dirb/wordlists/common.txt -u http://154.57.164.82:30468/webfuzzing_hidden_path/flag/FUZZ -e .php,.html,.txt,.bak,.js -fc 404
```

![[Web Fuzzing Practise flaghtml.png]]

answer: **HTB{w3b_f1l3_fuzz1ng_fl4g}**
### Recursive Fuzzing

1. **Recursively fuzz the "recursive_fuzz" path on the target system (ie `http://IP:PORT/recursive_fuzz/`) to find the flag.**

```
wfuzz -c --hc 404,403 -w /usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-medium.txt -u http://154.57.164.67:30319/recursive_fuzz/FUZZ -t 100
```

![[Web Fuzzing Practise level1.png]]

```
wfuzz -c --hc 404,403 -w /usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-medium.txt -u http://154.57.164.67:30319/recursive_fuzz/level1/FUZZ -t 100
```

> 000016015:   301        0 L      0 W        0 Ch        "level2" 

```
wfuzz -c --hc 404,403 -w /usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-medium.txt -u http://154.57.164.67:30319/recursive_fuzz/level1/level2/FUZZ - 100
```

> 000016861:   301        0 L      0 W        0 Ch        "level3"

```
wfuzz -c --hc 404,403 -w /usr/share/dirbuster/wordlists/directory-list-lowercase-2.3-medium.txt -u http://154.57.164.67:30319/recursive_fuzz/level1/level2/level3/FUZZ -t 100
```

> 000016217:   301        0 L      0 W        0 Ch        "threatcon_level2"

```
curl http://154.57.164.67:30319/recursive_fuzz/level1/level2/level3/threatcon_level2/  
```

![[Web Fuzzing Practise curl.png]]

answer: **HTB{d33p3r_d1rector1es_ar3_c00l}**
## Parameter and Value Fuzzing

### Parameter and Value Fuzzing

1. **What flag do you find when successfully fuzzing the GET parameter?**

```
ffuf -u "http://154.57.164.70:31093/get.php?x=FUZZ" -w /usr/share/seclists/Discovery/Web-Content/common.txt -fc 404 -v
```

![[Web Fuzzing Practise parametro.png]]

```
curl "http://154.57.164.70:31093/get.php?x=OA_HTML"  
```

![[Web Fuzzing Practise curlasdad.png]]

answer: **HTB{g3t_fuzz1ng_succ3ss}**

**Anotación**: Se observa que se utilizó la herramienta `ffuf` para fuzzear el parámetro GET `x` probando miles de palabras de un diccionario (`common.txt`) en la URL. Gracias a esto, se descubrió que el valor `0A_HTML` provocaba una respuesta exitosa (código 200). Finalmente, se realizó una petición con `curl` utilizando ese parámetro exacto (`x=0A_HTML`), lo que permitió descifrar y obtener con éxito la flag `HTB{g3t_fuzz1ng_succ3ss}`.

2. **What flag do you find when successfully fuzzing the POST parameter?**

```
ffuf -u http://154.57.164.70:31093/post.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "y=FUZZ" -w /usr/share/seclists/Discovery/Web-Content/common.txt -mc 200 -v
```

![[Web Fuzzing Practise sunwmc.png]]

```
curl -d "y=SUNWmc" http://154.57.164.70:31093/post.php
```

![[Web Fuzzing Practise fklag post.png]]

answer: **HTB{p0st_fuzz1ng_succ3ss}**

**Anotación**: Se observa que se utilizó la herramienta `ffuf` para automatizar la búsqueda del valor correcto del parámetro POST `y` probando un diccionario (`common.txt`) en la dirección `post.php`, lo que permitió descubrir que el valor `SUNWmc` generaba una respuesta exitosa (`200 OK`); posteriormente, el comando `curl` se estructuró con la bandera `-d` para enviar ese dato exactamente en el **cuerpo (body)** de la petición HTTP bajo el formato clave-valor (`y=SUNWmc`) —y no en la URL—, ya que el servidor procesa exclusivamente peticiones de tipo POST para este script, logrando así revelar e imprimir la flag `HTB{p0st_fuzz1ng_succ3ss}`.
## Virtual Host and Subdomain Fuzzing

### Virtual Host and Subdomain Fuzzing

1. **Using GoBuster against the target system to fuzz for vhosts using the common.txt wordlist, which vhost starts with the prefix "web-"? Respond with the full vhost, eg web-123.inlanefreight.htb.

Un **vhost** (Virtual Host) opera a nivel del servidor web y se identifica mediante la cabecera `Host` en las peticiones HTTP, permitiendo que una única dirección IP o servidor aloje múltiples sitios web o dominios de forma simultánea. Esta tecnología optimiza los recursos de la infraestructura, aunque una mala configuración puede exponer aplicaciones internas o datos sensibles.

---

```
echo "154.57.164.69 inlanefreight.htb" | sudo tee -a /etc/hosts

cat /etc/hosts
```

![[Web Fuzzing Practise catasda.png]]

```
gobuster vhost -u http://inlanefreight.htb:31178 -w /usr/share/seclists/Discovery/Web-Content/common.txt --append-domain --exclude-status 400 -t 20
```

![[Web Fuzzing Practise web.png]]

**Anotación**: Para resolver la primera pregunta sobre vhosts, asociamos el dominio base `inlanefreight.htb` con la IP del servidor en el archivo `/etc/hosts` y ejecutamos la herramienta `gobuster vhost` utilizando el diccionario `common.txt` junto con la bandera `--append-domain`. Esto permitió manipular la cabecera `Host` de cada petición HTTP para enumerar servidores virtuales y dar con el vhost que empezaba por el prefijo "web-".

answer: **web-beans.inlanefreight.htb**

2. **Using GoBuster against inlanefreight.com to fuzz for subdomains using the subdomains-top1million-5000.txt wordlist, which subdomain starts with the prefix "su"? Respond with the full vhost, eg web.inlanefreight.com.**

el **DNS** (Domain Name System) opera a nivel de infraestructura de red utilizando registros específicos para traducir los nombres de dominio y sus subdominios en direcciones IP concretas. Este sistema se encarga de dirigir el tráfico de internet hacia el servidor correcto, ayudando a estructurar y organizar diferentes secciones o servicios de un sitio web.

---

```
gobuster dns --domain inlanefreight.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

![[Web Fuzzing Practise su.png]]

answer: **support.inlanefreight.com**

**Anotación**: Para la segunda pregunta sobre subdominios, empleamos el modo DNS de la herramienta con `gobuster dns`, especificando el dominio objetivo mediante `--domain inlanefreight.com` y utilizando la wordlist `subdomains-top1million-5000.txt`. A través de consultas de resolución DNS sobre las variaciones del diccionario, logramos mapear la estructura del dominio principal y hallar el subdominio que comenzaba con el prefijo "su".
## Filtering Fuzzing Output

### Filtering Fuzzing Output

1. **What flag do you find when successfully fuzzing the POST parameter?**

Este comando de `ffuf` realiza un ataque de fuzzing mediante el método POST contra la URL `http://154.57.164.76:32581/post.php`, enviando datos en formato de formulario (`-H "Content-Type: application/x-www-form-urlencoded"`) donde la palabra clave `FUZZ` dentro del parámetro `y=FUZZ` se sustituye progresivamente por cada elemento del diccionario especificado con `-w /usr/share/seclists/Discovery/Web-Content/common.txt`. Además, utiliza el filtro `-fs 36` para ocultar todas aquellas respuestas irrelevantes que devuelvan un tamaño de 36 bytes (como los errores comunes), permitiendo destacar únicamente los resultados válidos o diferentes —en este caso, encontrando con éxito que la entrada `SUNWmc` genera una respuesta con estado HTTP 200 y un tamaño de 26 bytes.

```
ffuf -u http://154.57.164.76:32581/post.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "y=FUZZ" -w /usr/share/seclists/Discovery/Web-Content/common.txt -fs 36
```

![[Web Fuzzing Practise sunwsacasd.png]]

Este comando ejecuta una petición HTTP de tipo `POST` utilizando la herramienta `curl` hacia la URL especificada, enviando el parámetro `y` con el valor encontrado mediante el fuzzing (`SUNWmc`) directamente en el cuerpo de la solicitud. Al realizar esta consulta con el parámetro correcto, la aplicación del servidor procesa la entrada válida y devuelve la respuesta esperada que contiene la flag que estabas buscando.

```
curl -X POST -d "y=SUNWmc" http://154.57.164.76:32581/post.php
```

![[Web Fuzzing Practise flagasdiad.png]]

answer: **HTB{p0st_fuzz1ng_succ3ss}**

## Validating Findings

### Validating Findings

1. **Fuzz the target system using DirBuster-2007_directory-list-2.3-medium.txt, looking for a hidden directory. Once you have found the hidden directory, responsibly determine the validity of the vulnerability by analyzing the tar.gz file in the directory. Answer using the full Content-Length header, eg "Content-Length: 1337"**

```
gobuster dir -u http://154.57.164.61:30359/ -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -b 404
```

![[Web Fuzzing Practise gopbusteredasjkldklad.png]]

```
curl -s http://154.57.164.61:30359/ur-hiddenmember/ | grep "backup.tar.gz"
```

![[Web Fuzzing Practise targz.png]]

```
curl -I http://154.57.164.61:30359/ur-hiddenmember/backup.tar.gz
```

![[Web Fuzzing Practise 210.png]]

answer: **Content-Length: 210**
## Web APIs

### API Fuzzing

1. **What is the value returned by the endpoint that the api fuzzer has identified?**

```
ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt -u http://154.57.164.61:31018/FUZZ
```

![[Web Fuzzing Practiseczcmdcvt.png]]

```
curl http://154.57.164.61:31018/czcmdcvt
```

![[Web Fuzzing Practise fladagasdajk.png]]

answer: **h1dd3n_r357**

___

**Alternativa**

Esta herramienta es un **script automatizado de Python diseñado específicamente para realizar fuzzing en APIs web**. Su función principal es enviar peticiones HTTP de forma masiva contra una lista predefinida de rutas o endpoints para descubrir aquellos que están ocultos, no documentados o omitidos deliberadamente en la interfaz pública (como el panel de Swagger). De esta forma, te permite identificar superficies de ataque invisibles y encontrar rutas secretas o flags de manera rápida y estructurada.

```
git clone https://github.com/PandaSt0rm/webfuzz_api.git

cd webfuzz_api

python3 -m venv venv

source venv/bin/activate

pip3 install -r requirements.txt

python3 api_fuzzer.py http://154.57.164.61:31018 
```

![[Web Fuzzing Practise alternativadasdas.png]]
## Skills Assessment

### Skills Assessment

Se emplea `feroxbuster` para realizar una búsqueda masiva de directorios y archivos en la ruta raíz del servidor web objetivo, logrando descubrir rutas clave como `/admin/panel.php`.

```
feroxbuster -u http://154.57.164.78:30267/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -r -x php,html,txt,js -t 50 -k
```

![[Web Fuzzing Practise 58asdda.png]]

Al visitar la siguiente url `http://154.57.164.78:30267/admin/panel.php`

![[Web Fuzzing Practise accessid.png]]

Ante un mensaje de error que indicaba que el parámetro `accessID` no era correcto, se utilizó `ffuf` para realizar fuzzing de parámetros añadiendo un filtro por tamaño (`-fs 58`), lo que permitió identificar que el valor `getaccess` era válido.

```
ffuf -u http://154.57.164.78:30267/admin/panel.php?accessID=FUZZ -w /usr/share/dirb/wordlists/common.txt -fs 58
```

![[Web Fuzzing Practise getaccess.png]]

Por tanto, se visita la siguiente url `http://154.57.164.78:30267/admin/panel.php?accessID=getaccess`

![[Web Fuzzing Practise fuzzingashrftgbadas.png]]

Tras visitar el panel con el parámetro descubierto, el servidor indicó que se debía buscar en un VHost; por ello, se lanzó un fuzzing de cabeceras de tipo `Host` utilizando `ffuf`, revelando el subdominio oculto `hidden`.

```
ffuf -w /usr/share/dirb/wordlists/common.txt:FUZZ -u http://fuzzing_fun.htb:30267/ -H 'Host: FUZZ.fuzzing_fun.htb' -fc 403
```

![[Web Fuzzing Practise hidden.png]]

Se ejecutó una petición `curl` contra la dirección base enviando la cabecera correspondiente al VHost descubierto, obteniendo como respuesta una pista que indicaba la necesidad de buscar en la ruta `/godeep`

```
curl -s http://fuzzing_fun.htb:30267/ -H "Host: hidden.fuzzing_fun.htb"
```

![[Web Fuzzing Practise godeep.png]]

Al realizar la consulta a la ruta `/godeep/` con la cabecera del VHost, el servidor devolvió el mensaje "Keep going...", confirmando que se avanzaba por el camino correcto pero que aún restaban subdirectorios por explorar.

```
curl -s http://fuzzing_fun.htb:30267/godeep/ -H "Host: hidden.fuzzing_fun.htb"
```

![[Web Fuzzing Practise keep going.png]]

Se aplicó un nuevo fuzzing de directorios dentro de la ruta `/godeep/` manteniendo la cabecera del VHost, lo que permitió descubrir el siguiente subdirectorio válido llamado `stoneedge`.

```
ffuf -u http://fuzzing_fun.htb:30267/godeep/FUZZ -w /usr/share/dirb/wordlists/common.txt -H "Host: hidden.fuzzing_fun.htb"
```

![[Web Fuzzing Practise stoneedge.png]]

Se comprobó el nuevo directorio mediante una petición `curl` con la cabecera de host virtual, lo cual arrojó el mensaje "Almost there...", indicando que el objetivo estaba cada vez más cerca.

```
curl -s http://fuzzing_fun.htb:30267/godeep/stoneedge/ -H "Host: hidden.fuzzing_fun.htb"
```

![[Web Fuzzing Practise almost there.png]]

Se repitió el proceso de fuzzing utilizando `ffuf` dentro de la carpeta `stoneedge`, descubriendo un nuevo subdirectorio interno denominado `bbclone`.

```
ffuf -u http://fuzzing_fun.htb:30267/godeep/stoneedge/FUZZ -w /usr/share/dirb/wordlists/common.txt -H "Host: hidden.fuzzing_fun.htb"
```

![[Web Fuzzing Practise bbclone.png]]

Se hizo una nueva verificación con `curl` sobre la ruta `bbclone` bajo el mismo VHost, obteniendo la respuesta "Just a bit more..." como indicio de que se estaba a un paso de la meta.

```
curl -s http://fuzzing_fun.htb:30267/godeep/stoneedge/bbclone/ -H "Host: hidden.fuzzing_fun.htb"
```

![[Web Fuzzing Practise just a bit more.png]]

Se ejecutó un último escaneo de fuzzing dentro de la ruta `bbclone` para hallar la subcarpeta final, encontrando el directorio `typo3`.

```
ffuf -u http://fuzzing_fun.htb:30267/godeep/stoneedge/bbclone/FUZZ -w /usr/share/dirb/wordlists/common.txt -H "Host: hidden.fuzzing_fun.htb"
```

![[Web Fuzzing Practise typo3.png]]

Finalmente, se realizó la petición `curl` al endpoint definitivo descubierto (`typo3/`) con su respectiva cabecera de host, logrando recuperar la flag buscada

```
curl -s http://fuzzing_fun.htb:30267/godeep/stoneedge/bbclone/typo3 -H "Host: hidden.fuzzing_fun.htb"
```

![[Web Fuzzing Practise falgasdahjkd.png]]

answer: **HTB{w3b_fuzz1ng_sk1lls}**