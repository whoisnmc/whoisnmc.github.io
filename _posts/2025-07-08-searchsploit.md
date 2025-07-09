---
title: SearchSploit - Tools
categories: [ Linux ]
tags: [Tools]
---

<img src="/assets/img/Tools/searchsploit/searchsploit.png" alt="Captura de pantalla de la herramienta SearchSploit">

## SearchSploit: Encuentra exploits en segundos desde tu terminal.

La ciberseguridad ofensiva requiere rapidez, eficiencia y buenas herramientas. Una de las más útiles para encontrar vulnerabilidades conocidas es SearchSploit, una utilidad incluida en Kali Linux que permite consultar la base de datos de Exploit-DB sin conexión.

## ¿Qué es SearchSploit?
SearchSploit es una herramienta de línea de comandos que permite buscar exploits locales, remotos, scripts de prueba de concepto, y otras referencias de seguridad en una enorme base de datos mantenida por Offensive Security.

- Viene preinstalada en Kali Linux.
- No necesita conexión a internet.
- Es ideal para entornos restringidos o pruebas rápidas.

## ¿Cómo se usa?

Buscar vulnerabilidades por palabra clave:

```bash
searchsploit apache 2.4
```



Buscar exploits para un software específico:

```bash
searchsploit vsftpd 2.3.4
```



Incluir resultados que contengan varios términos:

```bash
searchsploit windows kernel privilege escalation
```



Mostrar ruta exacta al archivo del exploit:

```bash
searchsploit -p OpenSMTPD 6.6.2p1
```



Copiar el exploit a tu carpeta actual:

```bash
searchsploit -m 50082
```



- Los exploits se almacenan localmente en <mark>/usr/share/exploitdb.</mark>

## Actualizar la base de datos

Es importante mantener SearchSploit al día con los nuevos exploits que se publican. Puedes hacerlo con:

```bash
searchsploit -u
```



## Importante
SearchSploit está diseñado para uso ético y educativo. Solo debes usar exploits en sistemas que te pertenecen o para los cuales tienes autorización explícita.

## Más información

Página oficial de Exploit-DB: https://www.exploit-db.com/

Repositorio en GitHub: https://github.com/offensive-security/exploitdb
