# MÁQUINA PARADISE

Lo primero que tendremos que hacer será realizar un escaneo de puertos a la máquina víctima usando nmap:

```shell
nmap -p- -sS -sC -sV -vvv --min-rate 5000 -Pn 172.17.0.2
```

![NMAP](https://github.com/Isma-yo/photos/blob/main/Paradise/foto.JPG)

Vemos que hay varios puertos abiertos, vamos a empezar mirando el contenido del puerto 80:

![WEB](https://github.com/Isma-yo/photos/blob/main/Paradise/foto2.JPG)

Observamos que hay 2 botones, si presionamos el que pone "Go to the paradise" nos lleva a una pagina donde unicamente hay fotos. Pero que pasa si miramos el codigo fuente?

![SEC](https://github.com/Isma-yo/photos/blob/main/Paradise/foto3.JPG)

Al final de la pagina hay algo escrito en base64, vamos a copiarlo y ver que dice:

```shell
echo 'ZXN0b2VzdW5zZWNyZXRvCg==' | base64 -d
```

![BAS](https://github.com/Isma-yo/photos/blob/main/Paradise/foto4.JPG)

Podria ser la contraseña de un usuario, sin embargo, lo primero que se debe hacer en estos casos es ver si hay un directorio escondido con este nombre:

![DIR](https://github.com/Isma-yo/photos/blob/main/Paradise/foto5.JPG)

Dentro del fichero nos encontramos con esto:

![HOLI](https://github.com/Isma-yo/photos/blob/main/Paradise/foto6.JPG)

Ya tenemos un usuario, ademas sabemos que su contraseña es insegura, asi que posiblemente sea vulnerble a un ataque de fuerza bruta. Para realizar el ataque usaremos la herramienta hydra:

```shell
hydra -l lucas -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
```

![PASS](https://github.com/Isma-yo/photos/blob/main/Paradise/foto7.JPG)

Tenemos credenciales, vamos a conectarnos por SSH:

![SSH](https://github.com/Isma-yo/photos/blob/main/Paradise/foto8.JPG)

Ya estamos dentro de la maquina, vamos a tratar la tty con el siguiente comando:

```shell
script /dev/null -c bash
```

Si ejecutamos el comando sudo -l, vemos que podemos ejecutar sed como el usuario andy. 

![SUDO](https://github.com/Isma-yo/photos/blob/main/Paradise/foto9.JPG)

Iremos a GTFOBins para ver como podemos realizar la escalada:

https://gtfobins.github.io/gtfobins/sed/#sudo

![ANDY](https://github.com/Isma-yo/photos/blob/main/Paradise/foto10.JPG)

Ya somos el usuario andy, si intentamos repetir la misma jugada veremos que nos pide contraseña, la cual no sabemos, pero podemos probar si hay algun binario que podamos ejecutar ya que tenia un permiso especial:

```shell
find / -perm -4000 2>/dev/null
```

![ESC](https://github.com/Isma-yo/photos/blob/main/Paradise/foto11.JPG)

El primero es bastante sospechoso, asi que vamos a ejecutarlo a ver que pasa:

![ROOT](https://github.com/Isma-yo/photos/blob/main/Paradise/foto12.JPG)

Perfecto, ya somos root!
