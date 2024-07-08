# MÁQUINA UPLOAD

Lo primero que deberemos de hacer será reailzar un escaneo de puertos a la máquina víctima, para ello usaremos Nmap:

```shell
nmap -p- -sS -sC -sV -vvv --min-rate 5000 -Pn 172.17.0.2
```

![NMAP](https://github.com/Isma-yo/photos/blob/main/Upload/foto.jpg)

Observamos que únicamente está el puerto 80 abierto, por lo que vamos a ver el contenido web:

![WEB](https://github.com/Isma-yo/photos/blob/main/Upload/foto2.jpg)

Tenemos la opción de subir un archivo cosa que nos da la posibilidad de subirnos una reverse shell a la máquina para luego ejecutarla.

Sin embargo, antes estaría bien hacer fuzzing para ver que más ficheros o directorios hay en la máquina:

```shell
gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt
```

![FUZZ](https://github.com/Isma-yo/photos/blob/main/Upload/foto3.jpg)

Observamos que hay un directorio uploads que podemos intuir que será el lugar donde se almacenarán los distintos arhcivos que subamos dese la página principal.

Así que ahora si que vamos a subirnos una reverse shell:

![REV](https://github.com/Isma-yo/photos/blob/main/Upload/foto4.jpg)

Vemos que se ha subido correctamente, así que vamos a ver si está en el uploads:

![UPLOADS](https://github.com/Isma-yo/photos/blob/main/Upload/foto5.jpg)

Ahí esta, por lo que vamos a ponernos a la escucha por el puerto que hayamos configurado en la reverse shell para obtener la conexión:

![CONEX](https://github.com/Isma-yo/photos/blob/main/Upload/foto6.jpg)

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

Después de tratar la tty si ejecutamos el comando sudo -l vemos que podemos ejecutar env como usuario root sin contraseña:

![ENV](https://github.com/Isma-yo/photos/blob/main/Upload/foto7.jpg)

Para ello iremos a la página de GTFOBins y buscaremos como escalar privilegios con el comando env:

https://gtfobins.github.io/gtfobins/env/#sudo

![ROOT](https://github.com/Isma-yo/photos/blob/main/Upload/foto8.jpg)






