# MÁQUINA r00tless

Lo primero que tendremos que hacer será realizar un escaneo de puertos a la máquina víctima usando nmap:

```shell
nmap -p- -sS -sC -sV -vvv --min-rate 5000 -Pn 172.18.0.2
```

![NMAP](https://github.com/Isma-yo/photos/blob/main/r00tless/foto.JPG)

Vemos que hay unos cuantos puertos abiertos, vamos a mirar primero el contenido del puerto 80:

![WEB](https://github.com/Isma-yo/photos/blob/main/r00tless/foto2.JPG)

Nos encontramos con una pagina la cual nos permite subir un fichero, anets de hacerlo miraremos si hay algoen el codigo fuente que no es el caso, asi que haremos fuzzing para ver si hay algun fichero que pueda interesarnos:

```shell
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt
```

![FUZZ](https://github.com/Isma-yo/photos/blob/main/r00tless/foto3.JPG)

Hay un readme.txt que puede contener informacion interesante:

![READ](https://github.com/Isma-yo/photos/blob/main/r00tless/foto4.JPG)

Basicamente nos esta dando a entender que los archivos que subamos en la pagina principal se subiran al directorio .ssh de la maquina victima.

No sabemos que usuarios puede haber en la maquina, sin embargo, cuando hemos hecho el NMAP hemos visto que tenia Samba instalado, por lo que podemos probar a ejecutar el comando enum4linux para ver que informacion nos da:

```shell
enum4linux 172.18.0.2
```

![USERS](https://github.com/Isma-yo/photos/blob/main/r00tless/foto5.JPG)

Tenemos 4 posibles usuarios, asi que sabiendo que los archivos que subamos a la maquina iran al directorio .ssh podemos crear un par de claves con el comando ssh-keygen y subir la clave publica a la maquina victima:

```shell
ssh-keygen -t rsa -b 4096
```

![KEYS](https://github.com/Isma-yo/photos/blob/main/r00tless/foto6.JPG)

Subimos el archivo authorized_keys a la maquina:

![UPLOAD](https://github.com/Isma-yo/photos/blob/main/r00tless/foto7.JPG)

Se ha subido, genial, ahora tendremos que probar con los usuarios que hemos obtenido anteriormente a ver con cual podemos acceder via SSH (para conectarse hay que utilizar la clave privada):

![IN](https://github.com/Isma-yo/photos/blob/main/r00tless/foto8.JPG)

Finalmente el usuario correcto era "passsamba", asi que ya estamos dentro.

Si investigamos que hay dentro del directorio en el que estamos veremos que hay un fichero "note.txt" que al leerlo nos da una posible contraseña para un usuario (es lo primero que hay que pensar en los CTF):

![NOTE](https://github.com/Isma-yo/photos/blob/main/r00tless/foto9.JPG)

Tras probar con los usuarios que nos quedaban vemos que conseguimos convertirnos en el usuario "sambauser":

![SAMBA](https://github.com/Isma-yo/photos/blob/main/r00tless/foto10.JPG)

Despues de un buen rato pensando como poder avanzar ya que no podemos hacer mucho encontre un directorio sospechoso con nombre "srv" en la raiz de la maquina:

![SRV](https://github.com/Isma-yo/photos/blob/main/r00tless/foto11.JPG)

Dentro del directorio "read_only_share" encontramos un secret.zip que seguramente contendra alguna contraseña, asi que nos lo pasamos a nuestra maquina atacante levantado un servidor en Python por el puerto 8080:

![PY](https://github.com/Isma-yo/photos/blob/main/r00tless/foto12.JPG)

Ya lo tenemos en nuestra maquina, asi que vamos a crackearlo con zip2john y john:

![JOHN](https://github.com/Isma-yo/photos/blob/main/r00tless/foto13.JPG)

Tenemos la contraseña del usuario root-false perfecto, vamos a convertirnos en el:

![FALSE](https://github.com/Isma-yo/photos/blob/main/r00tless/foto14.JPG)

Somos el usuario root-false, si buscamos que hay en su directorio home observamos lo siguiente:

![MESSAGE](https://github.com/Isma-yo/photos/blob/main/r00tless/foto15.JPG)

Parece que hay un nuevo usuario mario con la contraseña que vemos en el mensaje, habra que ver donde podemos utilizarla.

Tras no saber donde mas mirar encontre que dentro del directorio /etc/apache2 hay otra pagina web a la que podemos acceder utilizando otra IP:

![APACHE](https://github.com/Isma-yo/photos/blob/main/r00tless/foto16.JPG)

Si hacemos un curl a la IP para ver el contenido de la web observamos lo siguiente:

![LOGIN](https://github.com/Isma-yo/photos/blob/main/r00tless/foto17.JPG)

Hay un apartado de login, donde seguramente podremos utilizar el usuario mario y la contraseña que habia dentro del message.txt, para ellos ejeuctaremos el siguiente comando:

```shell
curl -vvv -d 'username=mario&password=pinguinodemarioelmejor' http://10.10.11.5
```

![MARIO](https://github.com/Isma-yo/photos/blob/main/r00tless/foto18.JPG)

En el apartado de location hay una nueva ruta donde puede haber algo, miraremos el contenido de esa ruta:

```shell
curl -vvv http://10.10.11.5/super_secure_page/admin.php/
```

![ULTRA](https://github.com/Isma-yo/photos/blob/main/r00tless/foto19.JPG)

Encontramos un fichero algo sospechoso, veremos que contiene:

```shell
curl -vvv http://10.10.11.5/ultramegatextosecret.txt
```

![FICH](https://github.com/Isma-yo/photos/blob/main/r00tless/foto20.JPG)

A primera vista es un texto normal y corriente, sin embargo, salta a la vista que hay unas palabras separadas por _ como si de una contraseña se tratara, ademas vemos que esta escrito por less, uno de los usuarios con los que contaba la maquina, por lo que...:

![LESS](https://github.com/Isma-yo/photos/blob/main/r00tless/foto21.JPG)

Eso es, era su contraseña, ya somos el usuario less, ejecutaremos el comando sudo -l para ver si podemos ejecutar algo con el usuario root sin utilizar contraseña:

![SUDO](https://github.com/Isma-yo/photos/blob/main/r00tless/foto22.JPG)

Vemos que podemos ejecutar el comando chown para cambiar el propietario de los archivos, buscaremos como explotarlo en GTFOBins:

https://gtfobins.github.io/gtfobins/chown/#sudo

Lo haremos con el fichero /etc/passwd, donde una vez que seamos propietarios del fichero podremos editarlo para quitarle la x al usuario root y de este modo no tenga contraseña:

![ALMOST](https://github.com/Isma-yo/photos/blob/main/r00tless/foto23.JPG)

![99](https://github.com/Isma-yo/photos/blob/main/r00tless/foto24.JPG)

Perfecto, vamos a probar a convertirnos en root:

![ROOT](https://github.com/Isma-yo/photos/blob/main/r00tless/foto25.JPG)

Y ya somos root!

