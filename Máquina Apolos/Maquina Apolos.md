# MÁQUINA APOLOS

Lo primero que tendremos que hacer será realizar un escaneo de puertos a la máquina víctima usando nmap:

```shell
nmap -p- -sS -sC -sV -vvv --min-rate 5000 -Pn 172.17.0.2
```

![NMAP](https://github.com/Isma-yo/photos/blob/main/Apolos/foto.JPG)

Podemos observar que unicamente esta abierto el puerto 80, asi que vamos a ver que contiene la pagina web:

![WEB](https://github.com/Isma-yo/photos/blob/main/Apolos/foto2.JPG)

No se ve nada a simple vista ni el codigo fuente, asi que vamos a hacer fuzzing a ver que directorios encontramos:

```shell
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt
```

![GOB](https://github.com/Isma-yo/photos/blob/main/Apolos/foto3.JPG)

Vemos que existe una pagina en la que podemos registrarnos y crear un usuario, vamos a crear un usuario prueba:

![PR](https://github.com/Isma-yo/photos/blob/main/Apolos/foto4.JPG)

Nos logueamos con el usuario prueba:

![LOG](https://github.com/Isma-yo/photos/blob/main/Apolos/foto5.JPG)

Hay un apartado donde podemos añadir productos al carrito, probaremos a interceptar la peticion con burpsuite para ver si podemos ver algo con un SQLmap:

![BURP](https://github.com/Isma-yo/photos/blob/main/Apolos/foto6.JPG)

Guardamos el resultado en un fichero que en mi caso llame pruebaTienda:

![BURP2](https://github.com/Isma-yo/photos/blob/main/Apolos/foto7.JPG)

Y una vez guardado ejecutamos el siguiente comando:

```shell
sqlmap -r pruebaTienda --dbs --batch --dump
```

![SQL](https://github.com/Isma-yo/photos/blob/main/Apolos/foto8.JPG)

Perfecto, tenemos la contraseña del usuario admin, vamos a guardarla en un fichero para poder crackearla con John:

```shell
john --wordlist=/usr/share/wordlists/rockyou.txt hash
```

![PASS](https://github.com/Isma-yo/photos/blob/main/Apolos/foto9.JPG)

Estamos dentro, se puede ver que abajo aparece un boton para ir a la seccion de administracion:

![ADMIN](https://github.com/Isma-yo/photos/blob/main/Apolos/foto10.JPG)

Vamos al apartado de configuracion y vemos que podemos subir archivos:

![CONF](https://github.com/Isma-yo/photos/blob/main/Apolos/foto11.JPG)

Si intentamos subir una reverse shell con extension .php vemos que no nos deja:

![FILE](https://github.com/Isma-yo/photos/blob/main/Apolos/foto12.JPG)

Investigamos que otras extensiones pueden ser compatibles y encontramos esta pagina que habala de bypassear subida de ficheros https://vulp3cula.gitbook.io/hackers-grimoire/exploitation/web-application/file-upload-bypass:

![EXT](https://github.com/Isma-yo/photos/blob/main/Apolos/foto13.JPG)

Probamos a subir una reverse shell con extension .phtml y vemos que nos deja:

![ALM](https://github.com/Isma-yo/photos/blob/main/Apolos/foto14.JPG)

Cuando hemos hecho fuzzing hemos encontrado un directorio uploads seguramente es el lugar donde se haya subido el archivo:

![YA](https://github.com/Isma-yo/photos/blob/main/Apolos/foto15.JPG)

Una vez alli nos ponemos en escucha por el puerto que hemos especificado en la reverse shell y esperamos a recibir la conexion

![IN](https://github.com/Isma-yo/photos/blob/main/Apolos/foto16.JPG)

Ya estamos dentro de la máquina, por lo que primero que debermeos de hacer será realizar un tratamiento de la tty:

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

Podemos ver que existe otro usuario llamado luisillo_o, tras un rato dando vueltas no encontramos nada por lo que optaremos por la opcion de realizar un ataque de fuerza bruta para obtener la contraseña. Para ello usaremos el script que se puede descargar en el siguiente link:

https://github.com/Maalfer/Sudo_BruteForce/blob/main/Linux-Su-Force.sh

Nos lo pasaremos desde nuestra maquina atacante junto con el diccionario a usar (en mi caso el rockyou) y ejecutaremos el script como nos lo indica:

```shell
./Linux-Su-Force.sh luisillo_o rockyou.txt
```

Tras un rato esperando se encuentra la contraseña (esta por la linea 6000 del rockyou asi que es normal que tarde).

Ya somos el usuario luisillo_o, si ejecutamos el comando id veremos que pertenece al grupo shadow, lo que nos hace pensar que la escalada va por ahi, y efectivamente.

Buscamos como podemos crackear contraseñas si pertenecemos al grupo shadow y yo he utilizado los pasos de esta pagina:

https://erev0s.com/blog/cracking-etcshadow-john/

![SHA](https://github.com/Isma-yo/photos/blob/main/Apolos/foto17.JPG)

Ejecutamos el siguiente comando y la salida la guardamos en un nuevo fichero:

```shell
unshadow passwd shadow > unshadow.txt
```

E intentamos crackear con john la contraseña:

```shell
john --format=crypt --wordlist=/usr/share/wordlists/rockyou.txt unshadow.txt
```

![R6](https://github.com/Isma-yo/photos/blob/main/Apolos/foto18.JPG)

Nos convertimos en root con su root y probamos la contraseña:

![ROOT](https://github.com/Isma-yo/photos/blob/main/Apolos/foto19.JPG)

Y ya somos root!
