# MÁQUINA LITTLEPIVOTING

Lo primero que deberemos hacer será realizar un escaneo de puertos a la máquina a la que tenemos acceso:

```shell
nmap -p- --open -sS -sC -sV -vvv --min-rate 5000 -Pn 172.17.0.2
```

![NMAP](https://github.com/Isma-yo/photos/blob/main/LittlePivoting/foto.png)

Si ponemos el la IP en el navegador vemos que se nos abre una página por defecto de Apache, y en el código fuente no observamos nada extraño por lo que realizaremos fuzzing usando gobuster:

```shell
gobuster dir -u http://10.10.10.2/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt
```

![BUST](https://github.com/Isma-yo/photos/blob/main/LittlePivoting/foto2.png)

Si accedemos al directorio shop vemos que pone que hay un error con el método GET y la palabra archivo, por lo que vamos a intentar hacer un LFI a ver si podemos leer el contenido del archivo /etc/passwd:

![PASSWD](https://github.com/Isma-yo/photos/blob/main/LittlePivoting/foto3.png)

Vemos que hay un usuario con nombre manchi, por lo que vamos a intentar obtener su contraseña a través de un ataque de fuerza bruta:

![HYDRA](https://github.com/Isma-yo/photos/blob/main/LittlePivoting/foto4.png)

Tras investigar vemos que el usuario no esta en la lista de sudoers y que tampoco tiene nigun SUID importante por lo que pasamos a realizar un ataque de fuerza bruta para encontrar la contraseña del usuario seller:

![WGET](https://github.com/Isma-yo/photos/blob/main/LittlePivoting/foto5.png)

![SCRIPT](https://github.com/Isma-yo/photos/blob/main/LittlePivoting/foto6.png)

Una vez somos el usuario seller vemos que puede ejecutar php con sudo, por lo tanto nos vamos a la página de GTFOBins y miramos que podemos hacer, lo probamos y ya somos root:

![SELLER](https://github.com/Isma-yo/photos/blob/main/LittlePivoting/foto7.png)

Ahora ya podemos ver a la otra máquina pero desde la máquina a la que hemos obtenido acceso, y nosotros queremos tenerla desde nuestra máquina kali, por lo tanto ahora deberemos de usar chisel para poder crear un tunel:

![CHIS](https://github.com/Isma-yo/photos/blob/main/LittlePivoting/foto8.png)

![CHIS2](https://github.com/Isma-yo/photos/blob/main/LittlePivoting/foto9.png)

Ya tenemos el tunel creado, ahora ya podemos hacer un nmap desde nuestra máquina kali a la máquina objetivo que es la 20.20.20.3, para ello usaremos el comando proxychains:

![CHAIN](https://github.com/Isma-yo/photos/blob/main/LittlePivoting/foto10.png)

Vemos que están abiertos el puerto 80 y el 22, así que vamos a ver el contenido del puerto 80, pero para poder visualizar el contenido deberemos de configurar el proxy:

![PROXY](https://github.com/Isma-yo/photos/blob/main/LittlePivoting/foto11.png)

Y ya si ponemos la IP de la máquina podemos acceder:

![WEB](https://github.com/Isma-yo/photos/blob/main/LittlePivoting/foto12.png)

Vemos que no hay nada importante por lo que vamos a hacer fuzzing, aunque hay que cambiar un poco el comando de gobuster para poder hacer el fuzzing de manera correcta:

```shell
gobuster dir -u http://10.10.10.2/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt --proxy socks5://127.0.0.1:1080
```

![FUZZ](https://github.com/Isma-yo/photos/blob/main/LittlePivoting/foto13.png)

Al entrar al secret.php vemos que nos da un nombre de usuario:

![SEC](https://github.com/Isma-yo/photos/blob/main/LittlePivoting/foto14.png)

Con el usuario mario podemos realizar un ataque de fuerza bruta para ver si sacamos su contraseña. Al igual que con gobuster el comnado cambia un poco:

```shell
proxychains hydra -l mario -P /usr/share/wordlists/rockyou.txt ssh://20.20.20.3
```

![CHAIN2](https://github.com/Isma-yo/photos/blob/main/LittlePivoting/foto15.png)

Vemos que hemos encontrado su contraseña asi que podemos acceder a través de ssh:

```shell
proxychains ssh mario@20.20.20.3
```

![SSH](https://github.com/Isma-yo/photos/blob/main/LittlePivoting/foto16.png)

Si hacemos un sudo -l vemos que podemos ejecutar vim como sudo así que si volvemos a la página de GTFOBins vemos que si hay un exploit:

![EP](https://github.com/Isma-yo/photos/blob/main/LittlePivoting/foto17.png)

Ahora deberemos de descargar chisel en la máquina 30.30.30.2, para ello deberemos de compartir desde la máquina 20.20.20.2 ya que es la cuál a la que si tiene acceso.

![PY](https://github.com/Isma-yo/photos/blob/main/LittlePivoting/foto18.png)

Desde la máquina 20.20.20.2 ejecutamos el comando socat:

![SOCAT](https://github.com/Isma-yo/photos/blob/main/LittlePivoting/foto19.png)

Y desde la máquina 30.30.30.2 ejecutamos chisel:

![CHISEL](https://github.com/Isma-yo/photos/blob/main/LittlePivoting/foto20.png)

Creamos otro proxy para la nueva máquina:

![PROXY2](https://github.com/Isma-yo/photos/blob/main/LittlePivoting/foto21.png)

Y vemos que si ponemos la IP en el navegador conseguimos acceder:

![UF](https://github.com/Isma-yo/photos/blob/main/LittlePivoting/foto22.png)

Procedemos a listar los directorios con gobuster:

```shell
gobuster dir -u http://10.10.10.2/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt --proxy socks5://127.0.0.1:1111
```

![GB](https://github.com/Isma-yo/photos/blob/main/LittlePivoting/foto23.png)

Como hemos visto que podemos subir archivos vamos a subir uno que nos genere una reverse shell a la máquina 30.30.30.2:

![REV](https://github.com/Isma-yo/photos/blob/main/LittlePivoting/foto24.png)

Desde la máquina 10.10.10.2 ejecutamos socat para crear un túnel:

![TUN](https://github.com/Isma-yo/photos/blob/main/LittlePivoting/foto25.png)

Y desde la 20.20.20.2 ejecutamos socat:

![SOCAT2](https://github.com/Isma-yo/photos/blob/main/LittlePivoting/foto26.png)

Teniendo esto listo nos ponemos a la escucha por el puerto que hemos asignado en el túnel con la máquina 10.10.10.2 y ejecutamos la reverse shell:

![WWWDATA](https://github.com/Isma-yo/photos/blob/main/LittlePivoting/foto27.png)

Tras obtener acceso y tratar la tty, vemos que podemos hacer una escalada de privilegios usando env, así que vamos a la página de GTFOBins buscamos como hacerlo y ya somos root:

![ROOT](https://github.com/Isma-yo/photos/blob/main/LittlePivoting/foto28.png)




