# MÁQUINA LOS 4O LADRONES

Lo primero que deberemos de hacer será escanear los puertos de la máquina para poder ver cuáles están abiertos:

```shell
nmap -p- -sS -sC -sV -vvv --min-rate 5000 -Pn 172.17.0.2
```

![NMAP](https://github.com/Isma-yo/photos/blob/main/Los40ladrones/foto.jpg)

Unicamente tenemos el puerto 80 abierto, así que vamos a ver que página web aparece:

![WEB](https://github.com/Isma-yo/photos/blob/main/Los40ladrones/foto2.jpg)

Vemos que unicamente es la página por defecto de Apache, además si investigamos en el código fuente tampoco encontramos nada útil.

Por lo que el siguiente paso será realizar fuzzing para ver si podemos encontrar alguna otra página:

```shell
gobuster dir -u http://172.17.0.2/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt
```

![GOBUSTER](https://github.com/Isma-yo/photos/blob/main/Los40ladrones/foto3.jpg)

Encontramos un archivo .txt, vamos a ver que contiene:

![TXT](https://github.com/Isma-yo/photos/blob/main/Los40ladrones/foto4.jpg)

Tenemos lo que posiblemente sea un nombre de usuario "toctoc" y 3 números que viendo el nombre de usuario podemos intuir que son 3 puertos con los que deberemos de hacer Port Knocking.

Así que vamos a investigar acerca de como podemos hacer Port Knocking. Yo he encontrado este repo en GitHub el cuál funciona a la perfección:

![REPO](https://github.com/Isma-yo/photos/blob/main/Los40ladrones/foto5.jpg)

Una vez descargado el .py seguiremos las intrucciones que nos proporciona la página y ejecutaremos el siguiente comando:

```shell
python3 knockit.py -b 172.17.0.2 7000 8000 9000
```

![KNOCK](https://github.com/Isma-yo/photos/blob/main/Los40ladrones/foto6.jpg)

Una vez acabado si ahora volvemos a hacer un escaneo de puertos con nmap podemos observar que se ha abierto el puerto 22 (si por algún casual llegas a este punto y no te muestra el puerto 22 prueba a ejecutar el comando anterior pero cambiando de orden los puertos):

![NMAP2](https://github.com/Isma-yo/photos/blob/main/Los40ladrones/foto7.jpg)

Ahora lo que podemos hacer ahora es intentar un ataque de fuerza bruta al puerto 22 con el usuario toctoc para ver si conseguimos obtener su contraseña:

```shell
hydra -l toctoc -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
```

![HYDRA](https://github.com/Isma-yo/photos/blob/main/Los40ladrones/foto8.jpg)

Después un de un rato largo (tarda un rato tranquilo), hemos encontrado la contraseña, así que vamos a acceder a la máquina:

![SSH](https://github.com/Isma-yo/photos/blob/main/Los40ladrones/foto9.jpg)

Si hacemos un sudo -l vemos que podemos ejecutar como cualquier usuario y sin contraseña /opt/bash, por lo que vamos a probar que pasa si lo ejecutamos con sudo:

```shell
sudo /opt/bash
```

![ROOT](https://github.com/Isma-yo/photos/blob/main/Los40ladrones/foto10.jpg)

Y ya somos root!








