# MÁQUINA SJD

Lo primero que tendremos que hacer será realizar un escaneo de puertos a la máquina víctima usando nmap:

```shell
nmap -p- -sS -sC -sV -vvv --min-rate 5000 -Pn 172.17.0.2
```

![NMAP](https://github.com/Isma-yo/photos/blob/main/Sjd/foto.JPG)

Podemos osbervar que tanto el puerto 80 como el 22 se encuentran abiertos, miraremos primero el contenido de la pagina web:

![WEB](https://github.com/Isma-yo/photos/blob/main/Sjd/foto2.JPG)

Mas alla de un posible usuario por un correo de contacto el cual se encuentra en la parte inferior de la pagina no observamos nada extraño, además en el código fuente tampoco vemos nada.

Realizaremos fuzzing con la herramienta de gobuster a ver si encontramos algo oculto:

```shell
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt
```

![GO](https://github.com/Isma-yo/photos/blob/main/Sjd/foto3.JPG)

Encontramos un fichero sospechoso con el nombre "pass.txt", revisaremos su contenido:

![PASS](https://github.com/Isma-yo/photos/blob/main/Sjd/foto4.JPG)

Al parecer tenemos 3 posibles usuarios con sus contraseñas, las cuales parecen estar cifradas. Para descifrarlas nos iremos a la pagina de CyberChef, donde nos identificara y resolvera el tipo de cifrado:

```
https://gchq.github.io/CyberChef/
```

![CASI](https://github.com/Isma-yo/photos/blob/main/Sjd/foto5.JPG)

Tras poner la supuesta contraseña del usuario root podemos ver que nos lo descifra al valor 1971. Probaremos a conectarnos por SSH a la maquina utilizando dichas credenciales:

![ROOT](https://github.com/Isma-yo/photos/blob/main/Sjd/foto6.JPG)

Ha funcionado, ya somos root!
