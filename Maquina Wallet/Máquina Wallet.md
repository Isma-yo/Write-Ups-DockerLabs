# MÁQUINA WALLET

Lo primero que tendremos que hacer será realizar un escaneo de puertos a la máquina víctima usando nmap:

```shell
nmap -p- -sS -sC -sV -vvv --min-rate 5000 -Pn 172.17.0.2
```

![NMAP](https://github.com/Isma-yo/photos/blob/main/Wallet/foto.jpg)

Únicamente está accesible el puerto 80, así que vamos a ver su contenido:

![WEB](https://github.com/Isma-yo/photos/blob/main/Wallet/foto2.jpg)

Vemos una página la cuál no esconde nada, si hacemos click en el botón de Get a quote, podemos ver que nos redirije a un dominio, vamos a añadirlo a nuestro fichero /etc/hosts:

```shell

sudo nano /etc/hosts

172.17.0.2    panel.wallet.dl
```

![HOSTS](https://github.com/Isma-yo/photos/blob/main/Wallet/foto3.jpg)

Si ahora refrescamos la página nos encontraremos con una pantalla la cuál nos obliga a crear un usuario si queremos continuar, así que lo haremos:

![CREATE](https://github.com/Isma-yo/photos/blob/main/Wallet/foto4.jpg)

Hecho esto nos registramos y vemos que estamos dentro. Si nos vamos al apartado de Acerca de vemos que tenemos una versión de Wallos, podemos buscar si existe alguna vulnerabilidad:

![VER](https://github.com/Isma-yo/photos/blob/main/Wallet/foto5.jpg)

https://sploitus.com/exploit?id=EDB-ID:51924

Seguimos los pasos de la página y vemos que conseguimos guardar la reverse shell. Una vez guardada nos vamos al lugar donde se almacenan y vemos que ahí esta, así que la ejecutamos mientras estamos en escucha y estamos dentro:

![IN](https://github.com/Isma-yo/photos/blob/main/Wallet/foto6.jpg)

Ahora tendremos que tratar la tty (terminal) con los siguientes comandos:

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

Si ejecutamos el comando sudo -l vemos que podemos utilizar awk como el usuario pylon, vamos a buscar como escalar privilegios en GTFOBins:

https://gtfobins.github.io/gtfobins/awk/#sudo

```shell
sudo -u pylon awk 'BEGIN {system("/bin/bash")}'
```

![PY](https://github.com/Isma-yo/photos/blob/main/Wallet/foto7.jpg)

Ya somos Pylon, tras buscar por los distintos directorios encontramos algo interesante en su directorio home, sin emvbargo no sabemos la contraseña:

![ZIP](https://github.com/Isma-yo/photos/blob/main/Wallet/foto8.jpg)

Si intentamos pasarlo a nuestra máquina para sacar la contraseña vemos que no hay ninguna manera de hacerlo, así que una manera es codificarlo y una vez codificado copiar el contenido en nuestra máquina atacante donde lo descodificaremos:

![BAS](https://github.com/Isma-yo/photos/blob/main/Wallet/foto9.jpg)

![PASS](https://github.com/Isma-yo/photos/blob/main/Wallet/foto10.jpg)

Ya sabemos la contraseña, veamos que hay dentro:

![PING](https://github.com/Isma-yo/photos/blob/main/Wallet/foto11.jpg)

Repetiremos lo mismo que con el usuario Pylon, hacemos un sudo -l y comprobamos que podemos ejecutar sed como cualquier usuario y sin contraseña, vamos a escalar privilegios:

https://gtfobins.github.io/gtfobins/sed/#sudo

```shell
sudo sed -n '1e exec sh 1>&0' /etc/hosts
```

![ROOT](https://github.com/Isma-yo/photos/blob/main/Wallet/foto12.jpg)

Y ya somos root!












