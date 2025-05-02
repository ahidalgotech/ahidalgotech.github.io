---
layout: single
title: Find me - The Hackers Labs Write up
date: 2025-05-01
classes: wide
header:
  teaser: /assets/images/imagenfindme.jpg
  teaser_home_page: true
  icon: /assets/images/thlicon.png
categories:
  - TheHackersLabs
  - Easy
tags:
  - Easy
  - The Hackers Labs
 
---
# Find Me - Write Up - The Hackers Labs
---------------

<img src="/assets/images/findme.png" alt="Texto alternativo" width="400" />

Como siempre empezamos realizando un escaneo en este caso con **arp-scan**
para ver los dispositivos que tenemos corriendo por nuestra red local, esto lo haremos con el comando:

```bash
arp-scan -I eth0 --localnet --ignoredups` > y esto nos reporta lo siguiente:
````

En este caso nuestra máquina victima es la **192.168.1.146**

Ahora vamos a realizar un *ping* con `ping -c 2 192.168.1.146` Para ver que la máquina está bien desplegada y no nos arroja ningun tipo de error.

Y vemos que todo está bien.

<img src="/assets/images/Thehackerslabs-Findme/1.png" alt="Texto alternativo" width="400" />

**TTL 64 = Nos indica que estamos ante una máquina **Linux**

Ahora vamos a proceder al escaneo para comprobar que puertos están abiertos en dicha máquina con *Nmap*

```
nmap -p- --open --min-rate 5000 -sS -Pn -n -vvv 192.168.1.146 Y nos arroja lo siguiente:

```
<img src="/assets/images/Thehackerslabs-Findme/2.png" alt="Texto alternativo" width="400" />

*Encontramos abiertos los puertos 21,22,80 y 8080*

Ahora vamos a realizar un escaneo nuevamente con *nmap* para descubir que *servicios y versiones* corren para los puertos encontrados.

```
nmap -p21,22,80,8080 -sCV 192.168.1.146

```

<img src="/assets/images/Thehackerslabs-Findme/3.png" alt="Texto alternativo" width="400" />

El escaneo nos reporta lo siguiente, podemos ver muchas cosas pero la que a nosotros nos interesa es que por el puerto *21 FTP* está habilitado el usuario *anonymous* para poder entrar sin tener que proporcionarle contraseña.
Asique antes de nada vamos a echar un ojo por ahí a ver que nos encontramos.

```
ftp 192.168.1.146 # Ip de la máquina víctima.

```

