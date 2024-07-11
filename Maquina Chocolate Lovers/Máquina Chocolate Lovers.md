# MÁQUINA CHOCOLATE LOVERS

Lo primero que tendremos que hacer será realizar un escaneo de puertos a la máquina víctima usando nmap:

```shell
nmap -p- -sS -sC -sV -vvv --min-rate 5000 -Pn 172.17.0.2
```

![NMAP](https://github.com/Isma-yo/photos/blob/main/Chocolate%20Lovers/foto.jpg)

Únicamente se encuentra abierto el puerto 80, así que vamos a ver su contenido:

![WEB](https://github.com/Isma-yo/photos/blob/main/Chocolate%20Lovers/foto2.jpg)

Es la página por defecto de Apache, sin embargo, si miramos el código fuente podemos ver que hace especial hincapié em un directorio en concreto. Si hacemos fuzzing vemos que no hay nada más interesante, así que vamos a acceder a ese directorio:

![WEB2](https://github.com/Isma-yo/photos/blob/main/Chocolate%20Lovers/foto3.jpg)

Si nos damos cuenta hay una página donde podemos iniciar sesión llamada admin.php, vamos a acceder:

![AD](https://github.com/Isma-yo/photos/blob/main/Chocolate%20Lovers/foto4.jpg)

Antes de hacer fuerza bruta podemos probar combinaciones cómunes como admin:admin, admin:admin123...

![LOG](https://github.com/Isma-yo/photos/blob/main/Chocolate%20Lovers/foto5.jpg)

La combinación correcta era admin:admin, así que ya estamos dentro. Si vamos a la sección de plugin vemos que podemos instalar uno llamado My image, lo instalaremos ya que vamos a usarlo para escalar privilegios:

![PL](https://github.com/Isma-yo/photos/blob/main/Chocolate%20Lovers/foto6.jpg)

Una vez instalado vemos que podemos subir archivos, así que nos subiremos una reverse shell:

![REV](https://github.com/Isma-yo/photos/blob/main/Chocolate%20Lovers/foto7.jpg)

Al subirlo deberemos de acceder a la siguiente página para poder ejecutarlo:

http://172.17.0.2/nibbleblog/content/private/plugins/my_image/

![PHP](https://github.com/Isma-yo/photos/blob/main/Chocolate%20Lovers/foto8.jpg)

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

Si hacemos un sudp -l observamos que podemos ejecutar php como el usuario chocolate. Para ello iremos a la página de GTFOBins y buscaremos como hacerlo:

https://gtfobins.github.io/gtfobins/php/#sudo

```shell
sudo -u chocolate php -r "system('/bin/bash');"
```

![CHO](https://github.com/Isma-yo/photos/blob/main/Chocolate%20Lovers/foto9.jpg)

Ya somos el usuario chocolate, si hacemos un ps -ef vemos que hay un script que no para de ejecutarse, podemos intentar convertirlo en una reverse shell para escalar privilegios:

![PS](https://github.com/Isma-yo/photos/blob/main/Chocolate%20Lovers/foto10.jpg)

Vamos a subirnos desde nuestra máquina víctima la reverse shell que hemos utilizado antes, borraremos la otra y le cambiaremos el nombre para que se llame igual que el script que habia antes. Una vezhecho todo esto nos pondremos a la escucha y esperaremos a que nos llegue la conexión:

![WG](https://github.com/Isma-yo/photos/blob/main/Chocolate%20Lovers/foto11.jpg)

![ROOT](https://github.com/Isma-yo/photos/blob/main/Chocolate%20Lovers/foto12.jpg)

Y ya somsos root!















