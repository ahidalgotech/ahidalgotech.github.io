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

