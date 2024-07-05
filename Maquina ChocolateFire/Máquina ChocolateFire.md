# MÁQUINA CHOCOLATE FIRE

Lo primero que deberemos de hacer será realizar un escaneo de puertos con nmap:

```shell
nmap -p- --open -sS -sC -sV -vvv --min-rate 5000 -Pn 172.17.0.2
```

![NMAP](https://github.com/Isma-yo/photos/blob/main/ChocolateFire/foto.png)

Podemos comprobar que hay más puertos abiertos de lo normal, sin embargo, nosotros deberemos de fijarnos unicamente en el 22 y en el 9090, ya que es el que contiene OpenFire:

![OP](https://github.com/Isma-yo/photos/blob/main/ChocolateFire/foto2.png)

En esta página podemos observar la versión de OpenFire que es la 4.7.4, así que vamos a buscar si hay alguna vulnerabilidad de esta versión, que si la hay:

https://github.com/miko550/CVE-2023-32315

Deberemos de seguir los pasos que nos dan y una vez lo tengamos ejecutamos el comando y vemos que nos ha creado un usuario con contraseña:

```shell
python3 CVE-2023-32315.py -t http://172.17.0.2:9090
```

![CVE](https://github.com/Isma-yo/photos/blob/main/ChocolateFire/foto3.png)

Una vez dentro deberemos de irnos al apartado de plugins para instalar el plugin que nos ha proporcionado la página anterior (el archivo .jar):

![OP2](https://github.com/Isma-yo/photos/blob/main/ChocolateFire/foto4.png)

Ahora podemos ver que nos aparece otra pestaña con el nombre de “Management Tool”:

![OP3](https://github.com/Isma-yo/photos/blob/main/ChocolateFire/foto5.png)

Nos pide una contraseña la cuál es 123.

Una vez accedemos podemos ejecutar comandos de forma remota:

![OP4](https://github.com/Isma-yo/photos/blob/main/ChocolateFire/foto6.png)

Observamos que existe un usuario llamado “chocolatitochingon”:

![USR](https://github.com/Isma-yo/photos/blob/main/ChocolateFire/foto7.png)

Vamos a intentar obtener su contraseña usando fuerza bruta (recordemos que el puerto 22 estaba abierto):

```shell
hydra -l chocolatitochingon -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
```

![HYDRA](https://github.com/Isma-yo/photos/blob/main/ChocolateFire/foto8.png)

Accedemos:

![IN](https://github.com/Isma-yo/photos/blob/main/ChocolateFire/foto9.png)

Vemos que podemos ejecutar el binario dpkg con el usuario pinguinacio, así que vamos a buscar en GTFOBins si podemos escalar privilegios:

https://gtfobins.github.io/gtfobins/dpkg/

![DPKG](https://github.com/Isma-yo/photos/blob/main/ChocolateFire/foto10.png)

```shell
sudo -u pinguinacio /usr/bin/dpkg -l
```

Una vez hemos escalado privilegios como dice GTFOBins, vemos que somos el usuario “pinguinacio”:

![PINGUI](https://github.com/Isma-yo/photos/blob/main/ChocolateFire/foto11.png)

Vemos que binarios podemos ejecutar como el usuario root y vemos que existe un script el cual podemos ejecutar, este script pide introducir un parámetro por lo que podemos probar a insertar código y ver si funciona:

![SCRIPT](https://github.com/Isma-yo/photos/blob/main/ChocolateFire/foto12.png)

```shell
a[$(/bin/bash>&2)]
```

![ROOT](https://github.com/Isma-yo/photos/blob/main/ChocolateFire/foto13.png)

Y ole, ya somos root.

# FORMA ALTERNATIVA:

Sin embargo, existe una forma mucho más fácil de conseguir convertirnos el usuario root y que podíamos haber ejecutado hace mucho antes. Si desde la página web donde hacíamos la ejecución de comandos le damos el permiso especial a /bin/bash podremos convertirnos en root sin tener que pasar por otros usuarios:

![ROOT2](https://github.com/Isma-yo/photos/blob/main/ChocolateFire/foto14.png)

