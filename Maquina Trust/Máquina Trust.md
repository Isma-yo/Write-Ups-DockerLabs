# MÁQUINA TRUST

Lo primero que deberemos de hacer será realizar un escaneo de puertos a la máquina, para ello usaremos el comando nmap:

```shell
nmap -p- -sS -sC -sV -vvv --min-rate 5000 -Pn 172.18.0.2
```

![NMAP](https://github.com/Isma-yo/photos/blob/main/Trust/foto.jpg)

Observamos que están abiertos tanto el puerto 80 como el 22, así que vamos a ver el contenido de la página web:

![WEB](https://github.com/Isma-yo/photos/blob/main/Trust/foto2.jpg)

Después de observar que no hay nada en el código fuente haremos fuzzinga para encontrar ficheros o directorios que no se puedan ver a simple vista:

```shell
gobuster dir -u http://172.18.0.2/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt
```
![FUZZ](https://github.com/Isma-yo/photos/blob/main/Trust/foto3.jpg)

Vemos que hemos encontrado un fichero llamado secret.php, así que vamos a ver su contenido:

![SECRET](https://github.com/Isma-yo/photos/blob/main/Trust/foto4.jpg)

No vemos nada extraño en la página web, sin embargo podemos observar que nos da el nombre de un usuario "Mario", por lo que vamos a intentar conseguir su contraseña a través de un ataque de fuerza bruta:

```shell
hydra -l mario -P /usr/share/wordlists/rockyou.txt ssh://172.18.0.2
```

![HYDRA](https://github.com/Isma-yo/photos/blob/main/Trust/foto5.jpg)

Ya hemos encontrado la contraseña del usuario, por lo que vamos a acceder por SSH a la máquina con las credenciales que hemos obtenido:

![SSH](https://github.com/Isma-yo/photos/blob/main/Trust/foto6.jpg)

Si ejecutamos el comando sudo -l vemos que podemos ejecutar el comando vim como cualquier usuario, así que vamos a irnos a la página de GTFOBins y vamos a ver como escalar privilegios usando vim:

![SUDO](https://github.com/Isma-yo/photos/blob/main/Trust/foto7.jpg)

https://gtfobins.github.io/gtfobins/vim/#sudo

```shell
sudo vim -c ':!/bin/sh'
```

![ROOT](https://github.com/Isma-yo/photos/blob/main/Trust/foto8.jpg)

Y ya somos root!







