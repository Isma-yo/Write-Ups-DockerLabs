# MÁQUINA BORAZUWARAH

Lo primeroq que tendremos que hacer será realizar un escaneo de puertos a la máquina víctima usando nmap:

```shell
nmap -p- -sS -sC -sV -vvv --min-rate 5000 -Pn 172.17.0.2
```

![NMAP](https://github.com/Isma-yo/photos/blob/main/Borazuwarah/foto.jpg)

Vemos que tanto el puerto 22 como el 80 están abiertos, así que vamos a ver el contenido de la página web:

![WEB](https://github.com/Isma-yo/photos/blob/main/Borazuwarah/foto2.jpg)

Solo hay una imagen, por lo que no nos queda otra que descargarla y guardarla.

Es posible que cuente con información útil así que usaremos el comando exiftool para ver si hay algo:

```shell
exiftool imagen.jpeg
```

![FOTO](https://github.com/Isma-yo/photos/blob/main/Borazuwarah/foto3.jpg)

Hemos encontrado un usuario, ahora hay que encontrar su contraseña, para ello usaremos fuerza bruta:

```shell
hydra -l borazuwarah -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
```

![PASS](https://github.com/Isma-yo/photos/blob/main/Borazuwarah/foto4.jpg)

Accedemos a la máquina a través de SSH:

![PASS](https://github.com/Isma-yo/photos/blob/main/Borazuwarah/foto5.jpg)

Si hacemos un sudo -l vemos que puede ejecutar cualquier comando como cualquier usuario, así que vamos a convertirnos en root con sudo su:

![ROOT](https://github.com/Isma-yo/photos/blob/main/Borazuwarah/foto6.jpg)

Y ya somos root!


